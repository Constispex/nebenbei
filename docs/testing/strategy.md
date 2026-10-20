# Teststrategie, CI/CD, Betrieb und Qualitäts-Gates

Stand 20.10.2026. Legende: **[F]** belegt, **[A]** Annahme, **[E]** Empfehlung. Stufen und Module siehe [plan.md](../plan.md). Grundsatz: Werkzeuge und Gates wachsen mit der Stufe; was in Stufe 1 nichts absichert, kommt erst vor dem öffentlichen Start.

## 1. Testpyramide und Testgrenzen

| Ebene | Werkzeug | Inhalt | Ab Stufe |
|---|---|---|---|
| App-Kern (Unit) | Jest | `proximity`, `queue`, `walkSession` aus ADR-011 mit Golden-Fixtures (Positionsfolgen → erwartete Ereignisse); keine Expo-Mocks nötig, weil der Kern rein ist | 1 |
| App-UI | Jest + React Native Testing Library | Onboarding, Berechtigungsfluss, Liste „Zuletzt gehört“, Einstellungen | 1 |
| Content-Build (Unit) | `go test`, Tabellen-Tests | Schema- und Prüfregeln mit Positiv- und Negativbeispielen (fehlende Quelle, NC-Lizenz, `javascript:`-Link, Markup, Radius außerhalb), Cache-Schlüssel, Paketformat gegen JSON Schema | 1 |
| Messpunkt (Integration) | testcontainers-go mit Postgres | Speicherung nur der erlaubten Felder, Plausibilitätsgrenzen, kein IP-Feld; Log-Test: eine Anfrage erzeugt keine Logzeile mit IP, User-Agent oder Request-Body | 1 |
| Vertrag | JSON Schema in `packages/schema/` (Regionspaket, `config.json`); OpenAPI für den Messpunkt | Go-Build und App validieren gegen dieselben Beispiel-Dateien | 1 |
| E2E | Maestro [E]; [F] `setLocation` mockt den Standort ([Doku](https://docs.maestro.dev/api-reference/commands/setlocation)) | 3–5 Kernpfade: Onboarding → Region laden → Spaziergang mit simulierter Annäherung → Fakt hören → Meldelink | 1 (vor Beta-Builds) |
| Echtgeräte | Checkliste, iPhone + Pixel + Samsung | Hintergrund-Audio, gesperrter Bildschirm, Unterbrechungen, Akku über 30 min | 1 (pro Release) |
| Backend-Integration | testcontainers-go mit `postgis/postgis` | sqlc-Abfragen, Migration „up“, Geo-Abfragen mit Golden-Fixtures, Veröffentlichungsablauf (alte Fassung sichtbar bis Audio fertig), nur veröffentlichte Fassungen über die API | 2 |
| API | OpenAPI-Validierung im Strict-Server, `oasdiff` gegen `main` | Statuscodes, RFC 9457, keine Breaking Changes | 2 |
| Sicherheit erweitert | gosec, CodeQL oder Semgrep, ZAP-Baseline gegen Staging | | 2 |

**Testgrenzen:** Interfaces nur für externe Dienste (TTS, Objektspeicher, OIDC, Uhr). Datenbankzugriffe werden nicht gemockt, sondern gegen echtes Postgres/PostGIS getestet. Hintergrund-Audio und -Standort lassen sich nicht verlässlich automatisieren; dafür gibt es die Echtgeräte-Checkliste.

**Golden-Fixtures (`testdata/geo/`):** Orte plus Positionsfolgen plus erwartete Fakt-IDs. Dieselben Dateien prüfen in Stufe 1 die App-Nähe-Logik und das Paketformat, in Stufe 2 zusätzlich die Kachelabfrage in Go. Fälle: Punkt genau auf dem Radius; GPS-Sprung durch schlechte Genauigkeit; zwei überlappende Radien; kurzes Streifen ohne Verweildauer; Rückkehr innerhalb 24 h; Wechsel auf Radfahrgeschwindigkeit; ab Stufe 2 Punkt auf Kachelgrenze und Auslösebereich über zwei Kacheln.

## 2. Lokale Entwicklung
- **Docker Compose:** Stufe 1: `db` (Postgres für den Messpunkt), `minio` (Objektspeicher für den Content-Build). Stufe 2: `postgis/postgis` in gepinnter Version wie in Produktion, `mailpit` für Magic Links. `.env.example` ohne Secrets.
- **Migrationen:** goose, nur SQL, keine Down-Migrationen ([ADR-004](../architecture/decisions/ADR-004-pgx-sqlc.md)).
- **Seed:** kleine Beispielregion mit 20 Fakten unter `content/example/`, damit App und Build ohne echte Inhalte laufen.
- **Reproduzierbarkeit:** Lockfiles committet; Go- und Node-Versionen festgeschrieben (`go.mod` `toolchain`, `.nvmrc`); Container-Images mit Digest.

## 3. CI-Pipeline (GitHub Actions)

**Immer (ab Stufe 1):**
- Alle Actions auf Commit-SHA gepinnt; [F] GitHub kann das per Policy erzwingen ([Changelog](https://github.blog/changelog/2025-08-15-github-actions-policy-now-supports-blocking-and-sha-pinning-actions/)). Workflow-weit `permissions: contents: read`; keine Secrets in `pull_request_target`.
- Renovate oder Dependabot inklusive `github-actions`, etwa 7 Tage Wartezeit.

**Pull Request (Ziel < 10 min):**
1. Format, Lint, Typen: gofmt, golangci-lint, `tsc --noEmit`, eslint.
2. Tests: Go-Unit und -Integration, Jest.
3. Vertrag: Beispiele gegen JSON Schema; generierter Code aktuell (`git diff --exit-code`).
4. Sicherheit: govulncheck (blockiert bei jedem Fund, weil es nur erreichbare Schwachstellen meldet), `npm audit --omit=dev --audit-level=high`, gitleaks. Ausnahmen nur mit Begründung und Ablaufdatum in einer Datei im Repo.
5. Plattform-Dateien: erzeugtes `AndroidManifest.xml` und `Info.plist` enthalten keine Hintergrund-Standortberechtigung.
6. Content: Build im Prüfmodus über alle Inhalte (ohne TTS-Aufrufe).

**main:** Content-Build mit TTS und Upload in eine Staging-Umgebung des Objektspeichers; Freigabe für Produktion über eine geschützte Environment.
**Mobile:** EAS Build und Submit für TestFlight und geschlossenen Play-Test, ausgelöst per Tag; Expo-Token nur in der Environment `mobile-release` mit Freigabe.
**Ab Stufe 2 zusätzlich:** Container-Image (distroless, ohne root), SBOM, Image-Scan, Signatur; Staging-Deploy mit Smoke-Test; `oasdiff`; Migration „up vom letzten Release“; gosec und CodeQL/Semgrep; nightly Maestro, ZAP-Baseline, Restore-Test.

## 4. Betrieb

**Stufe 1** (kein Inhalts-Backend):
- Auslieferung vom eigenen Hetzner-Server mit Coolify ([decisions.md](../decisions.md)), keine Verzeichnis-Listung, Bandbreite im Blick. Ein CDN kommt erst, wenn die Last es verlangt.
- Messpunkt: ein kleiner Go-Dienst mit Postgres auf einer kleinen Instanz. Für einen Endpunkt wäre auch SQLite denkbar; Postgres, sqlc und goose sind gewählt, weil sie in Stufe 2 ohnehin kommen und der Messpunkt so zum ersten Baustein des Backends wird; tägliches Backup (Dump) mit 30 Tagen Aufbewahrung; Uptime-Check. Fällt er aus, gehen Messdaten verloren, aber die App funktioniert weiter.
- Crash-Berichte: Sentry SaaS mit EU-Datenspeicherung ([F] Frankfurt, auch im kostenlosen Plan, Region später nicht änderbar, [Doku](https://docs.sentry.io/organization/data-storage-location/)) oder GlitchTip selbst gehostet; `sendDefaultPii=false`, kein Session-Tracking, keine HTTP-Breadcrumbs, IP-Speicherung im Projekt abgeschaltet und serverseitiges Data Scrubbing aktiv.

**Stufe 2:**
- **Hosting [E]:** Ausgangspunkt ist der eigene Hetzner-Server mit Coolify ([decisions.md](../decisions.md)) und selbst betriebenem Postgres mit PostGIS (Backups mit pgBackRest oder WAL-G). Vor Stufe 2 neu bewerten; Alternative mit weniger Betriebsaufwand: Scaleway (Managed PostgreSQL, Container mit dauerhaft einer Instanz, Object Storage) als geringster Betriebsaufwand bei einem EU-Anbieter; PostGIS-Verfügbarkeit vor Vertragsschluss mit `SELECT * FROM pg_available_extensions` prüfen ([Tutorial](https://www.scaleway.com/en/docs/tutorials/postgis-managed-databases)). Render Frankfurt unterstützt PostGIS ([F], [Doku](https://render.com/docs/postgresql-extensions)), ist aber ein US-Anbieter.
- **Observability:** JSON-Logs mit Feld-Allowlist und Redaktions-Test; Metriken (Anfragen, Fehler, Latenz pro Endpunkt, DB-Pool, Job-Rückstand).
- **Alarme (Startwerte):** 5xx-Rate > 1 % über 5 min; p95-Latenz > 500 ms über 10 min; TTS-Job seit > 30 min fehlgeschlagen nach 5 Versuchen mit exponentiellem Backoff; Backup-Fehler; Kosten > 80 % des Monatsbudgets.
- **Rollback und Migrationen:** Expand/Contract; Rollback = vorheriges Image; nie `down` in Produktion; die App erhält über `/v1/config` eine Mindestversion.
- **Backups:** PITR ≥ 7 Tage, täglicher Dump in getrennten Speicher; monatlicher automatisierter Restore-Test. Ziele: RPO ≤ 15 min, RTO ≤ 4 h [A].
- **EAS-Token:** Robot-Token nur mit Build- und Submit-Rechten, ohne Update-Rechte.
- **Over-the-Air-Updates:** nur mit Code-Signing und Vier-Augen-Freigabe (ADR-006); [F] Native Änderungen und Berechtigungen brauchen einen Store-Build ([FAQ](https://docs.expo.dev/eas-update/faq/)).

## 5. Messaufbauten

| Messung | Aufbau |
|---|---|
| Auslösegenauigkeit (WP-01) | Referenzzeitpunkt „Eintritt in den Radius“ aus einer unabhängigen Spur: zweites Gerät oder GPS-Logger mit 1-s-Aufzeichnung, alternativ markierte Punkte im Gelände, an denen die testende Person eine Taste drückt; Abstand und Verzögerung je Auslösung nachträglich berechnet |
| Akku (WP-01) | Gerät auf 100 % geladen, Helligkeit minimal, Display aus, keine anderen Apps im Vordergrund, Mobilfunk an, WLAN aus, Außentemperatur notiert; 60 min Spaziergang; Ablesung des Akkustands zu Beginn und Ende, dazu Xcode Energy Log bzw. Battery Historian; je Gerät drei Durchgänge, Median zählt |
| Geo-Abfragen (Stufe 2) | 50.000 synthetische Orte in Deutschland; 1.000 zufällige Kachelabfragen nach Aufwärmen; p95 aus dem Integrationstest auf dem CI-Runner |
| Kartenleistung (falls Karte) | Mittelklasse-Android (z. B. Samsung A-Serie), 500 Punkte, 30 s Schwenken; Bildrate aus dem Android-GPU-Profiling |
| Barrierefreiheit | VoiceOver und TalkBack: Kernpfad ohne Sicht bedienbar; größte Schriftgröße: keine abgeschnittenen Texte (Screenshot-Prüfung) |

## 6. Definition of Done

**Stufe 1** – ein Arbeitspaket ist fertig, wenn:
- CI grün ist (Lint, Typen, Tests, govulncheck ohne Fund, npm audit ohne High/Critical, gitleaks).
- Neue Logik im App-Kern und im Content-Build Tests mit Golden-Fixtures bzw. Negativbeispielen hat.
- Keine Positions-, Kachel- oder Kennungsdaten in Logs, Crash-Berichten oder Netzwerkanfragen landen (bei Standort- oder Netzwerkänderungen: Mitschnitt).
- Bei Standort oder Audio: Echtgeräte-Checkliste abgehakt.
- Ein Review durch eine zweite Person stattfand oder, bei nur einer Person, ein dokumentiertes Self-Review mit Checkliste.

**Stufe 2 zusätzlich:** Coverage-Gate (Domänenpakete ≥ 80 %, App-Kern ≥ 80 %, sonst keine Regression > 2 Prozentpunkte); OpenAPI aktuell ohne unbeabsichtigten Breaking Change; Migration nach Expand/Contract; neue Felder mit Personenbezug im Verarbeitungsverzeichnis.

**Gates vor dem öffentlichen Start:** Restore geprobt und RPO/RTO erreicht; Alarme aktiv und einmal ausgelöst getestet; höchstens 1 Absturz pro 100 Spaziergänge in der Beta (Crash-Berichte im Verhältnis zur Zahl der Spaziergang-Zusammenfassungen; kein Session-Tracking nötig); Auftragsverarbeitungsverträge unterschrieben; rechtliche Freigaben (WP-38).
