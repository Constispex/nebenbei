# Architektur-Überblick

Quelle für Systembild, Repository-Struktur, Abhängigkeitsregeln und Konventionen. Begründungen stehen in den [ADRs](decisions/).

## Stufe 1

```
Git-Repo: content/<region>/*.yaml
        │  (Redaktion per Pull Request; Review = Freigabe)
        ▼
Build-Skript (Go-CLI, in CI)
  prüft Schema, Quelle, Lizenz ─ erzeugt TTS (Cache per Hash) ─ baut Regionspaket
        │
        ▼
Eigener Server (Hetzner, Coolify) ──► App
  config.json                     lädt config, dann Regionspaket
  regions/<id>/<version>/...      (Fakten, Audio, Kartenkacheln als PMTiles)
                                  Nähe-Logik, Player, Karte, alles lokal
                                  │
Messpunkt (Go, ein Endpunkt) ◄────┘ Spaziergang-Zusammenfassung (Klassen)
```

## Stufe 2 (Ziel)

```
App ──► CDN ──► Go-Monolith (ein Prozess: API, Redaktion, Jobs) ──► PostgreSQL + PostGIS
                     │                                              (Inhalte, Jobs)
                     └──► TTS-Anbieter, Objektspeicher
```
Die App ändert sich beim Umstieg kaum: Das Regionspaket-Format wird zur Kachel-Antwort, `config.json` zum Endpunkt `/v1/config`.

## Repository-Struktur

```
nebenbei/
├─ spikes/                      # Wegwerf-Code aus Phase A (WP-01/02), nicht Teil des Produkts
├─ apps/mobile/                 # Expo-App
│  └─ src/{core,adapters,ui}    # core = reines TypeScript (ADR-011)
├─ content/                     # Stufe 1: Inhalte als YAML, Schema, Audio-Cache-Index
├─ tools/contentbuild/          # Go-CLI: prüfen, TTS, Paket bauen, hochladen
├─ backend/                     # Messpunkt (Stufe 1), Monolith (Stufe 2)
│  ├─ cmd/server/
│  ├─ internal/{platform,feedback}   # Stufe 2: content, audio, admin (UI), transport/http; später accounts, contributions, imports
│  └─ db/{migrations,queries}/
├─ api/openapi.yaml             # Vertrag: Messpunkt, später alle Endpunkte
├─ packages/schema/             # gemeinsame Formate (Regionspaket, Fixtures)
├─ testdata/geo/                # Golden-Fixtures für Go und Jest
├─ infra/                       # compose.yaml, Dockerfile
├─ docs/
└─ .github/workflows/           # pfadgefiltert
```

## Abhängigkeitsregeln (Backend, ab Stufe 2 voll wirksam)
1. Richtung: `transport`/`admin` → `<modul>.Service` → Store/Clients → `platform`. Nie umgekehrt.
2. Module sprechen andere Module nur über deren Service an. Zyklen sind verboten.
3. Jede Tabelle gehört einem Modul.
4. Geschäftslogik kennt kein HTTP; Domänenfehler werden erst im Transport zu RFC-9457-Problem-Details.
5. Interfaces nur für externe Dienste (TTS, Objektspeicher, OIDC, Uhr); Datenbankzugriff wird gegen echtes PostGIS getestet, nicht gemockt.
6. Durchsetzung zunächst über Go-`internal/` und Review; depguard, sobald es mehr als drei Fachmodule gibt.

Fachmodule in Stufe 2: `content` (Orte, Fakten, Revisionen, Quellen, Lizenzen, Geo-Abfragen, Veröffentlichung), `audio`, `feedback`, später `accounts`, `contributions`, `imports`. Dazu die Adapter `admin` (Redaktionsoberfläche) und `transport/http` sowie `platform`.

## Konventionen
- Fehler: `application/problem+json` (RFC 9457)
- Konfiguration: Umgebungsvariablen, typisiert, Abbruch beim Start bei Fehlern
- Logging: `log/slog` als JSON; Feld-Allowlist; keine Koordinaten, Kachel-IDs, IPs, E-Mails oder Tokens
- Versionierung: `config.json` bzw. `/v1/config` liefert `min_app_version`, `content_version`, `schema_version`; Inhalte liegen unter versionierten, unveränderlichen URLs (`Cache-Control: immutable`); die App vergleicht lokal, der Server wertet keine Versions-Header aus
- TypeScript `strict`; API-Typen generiert
