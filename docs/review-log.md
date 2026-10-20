# Entstehung und Review-Protokoll

## Phase 0: Bestandsaufnahme
Das Repository war nicht lesbar (kein Zugriff der Claude-GitHub-App). Annahme: leeres Repository. Keine bestehenden Konventionen, Skills oder `CLAUDE.md` im Repo prüfbar.

## Phase 1: Untersuchungen (parallel, nur lesend und recherchierend)

| Agent | Thema | Wichtigste Beiträge | Übernommen? |
|---|---|---|---|
| A | Architektur | Ein Repo ohne Monorepo-Tooling; Modulform; Abhängigkeitsregeln; OpenAPI-first; `net/http` statt Gin; pgx + sqlc; Jobs in Postgres; Expo | ja; Worker-Rolle später durch ARC-3 ersetzt |
| B | Sicherheit, Datenschutz | „Standort bleibt auf dem Gerät“; TTS nur beim Veröffentlichen; kein eigener Passwort-Stack; Hintergrundstandort vermeiden; 11 Rechtsfragen | ja |
| C | Daten, Geo, Quellen | Datenmodell mit Revisionen und Lizenztabelle; PostGIS-Abfragen; Lizenzfolgen ODbL/CC BY-SA; Komoot nur GPX-Datei; Quellenvergleich | ja; Wikidata und GPX aus dem MVP in Stufe 2 verschoben (kein Bezug zur Kernhypothese) |
| D | Mobile, Audio, UX | Expo + expo-location + expo-audio; Spaziergang-Modus ohne Hintergrundberechtigung; vorproduziertes TTS; Audio-Soll-Verhalten; sechs Spikes | ja; Karte auf „nach der Beta“ verschoben |
| E | Tests, CI, Betrieb | Testpyramide; Maestro; testcontainers; goose; CI-Härtung; Hosting-Optionen; Definition of Done | ja, aber gestaffelt nach Stufe (ARC-7) |

Einschränkung aller Untersuchungen: Viele offizielle Seiten waren aus der Umgebung nicht direkt abrufbar; Aussagen stützen sich dann auf Suchergebnis-Auszüge und sind markiert.

## Phase 2: Synthese
Zusammenführung der fünf Berichte zu Version 1; Konflikte selbst entschieden (Kartenkacheln: kein tile.openstreetmap.org; Supabase als Ganzes verworfen, aber als gehostetes Postgres möglich; Wikidata und GPX nicht im MVP).

## Phase 3: Review-Schleifen

### Loop 1: Entwurf
Version 1 mit Stufe 1 als Beta auf vollem Backend (Go, PostGIS, Redaktionswerkzeug, OIDC, Jobs, Kachel-API).

### Loop 2: Unabhängige Kritik (vier Reviewer, parallel)

| Review | Befunde | Kernaussage |
|---|---|---|
| Sicherheit (SEC) | 12 (1 kritisch, 4 hoch, 7 mittel) | „Standort bleibt auf dem Gerät“ hielt nicht: Audio-Abruf beim Auslösen und Kachel-Nachladen verraten die Position; versteckte Kennungen in SDKs; OTA-Updates als Angriffspfad |
| Architektur (ARC) | 12 (4 hoch, 6 mittel, 2 niedrig) | Kachel-Versionierung unvollständig; Veröffentlichung und Audio widersprüchlich; Worker-Rolle unnötig; App-Architektur fehlt; Werkzeugumfang zu groß |
| Produkt/MVP (PRD) | 10 (2 kritisch, 3 hoch, 5 mittel) | Testet Technik vor Nachfrage; Inhaltskosten als größtes Risiko nicht geführt; Backend für die Beta unnötig; Wiederkehr nicht messbar |
| Verifikation (VER) | 15 (2 hoch, 10 mittel, 3 niedrig) | Zwei Faktenaussagen falsch (TTS-Höchstpreis, Apple 4.8); viele Abnahmekriterien ohne Messmethode oder widersprüchlich |

13 stichprobenartig geprüfte Fakten wurden bestätigt, darunter Android-Geofence-Limit, expo-audio-3-Minuten-Grenze, RNTP-v5-Preis, Go-1.22-Routing, Apple 5.1.1(v), Overpass-Limits, Komoot ohne API, `ST_TileEnvelope` in EPSG:3857, Kachelgröße z13, Sentry-EU-Region.

### Loop 3: Überarbeitung (Version 2)

**Übernommen, mit größter Wirkung:**
- Neue **Stufe 0 Feldtest** mit dem Spike-Build vor jedem Plattformbau (PRD-1, PRD-10).
- **Inhalte in Git mit Build-Skript** statt Backend und Redaktionswerkzeug in Stufe 1 (PRD-3, ADR-012); Backend, PostGIS, OIDC, Jobs und Redaktionswerkzeug nach Stufe 2.
- **Regionspaket mit Audio vorab laden**; keine inhaltsbezogenen Anfragen während des Spaziergangs; Netzwerkmitschnitt als Abnahme (SEC-1, SEC-2, ARC-1).
- Hypothesen H-Content, H-Segment, H-Differenz, H-Revenue; Entscheidungsregel mit Go/Umsteuern/Stopp vor den Tests (PRD-2, PRD-4 bis PRD-7, VER-1).
- Wiederkehr ohne Kennung über lokal gezählte Klassen (PRD-4, SEC-3, SEC-10).
- Kein `expo-updates` in Stufe 0/1; Crash-SDK ohne Session-Tracking und Breadcrumbs; Konten mit zweitem Faktor (SEC-4, SEC-5).
- Veröffentlichungsablauf mit `pending_revision_id` und globaler `content_version`; versionierte, unveränderliche URLs (ARC-1, ARC-2, ARC-9).
- Ein Prozess statt Worker-Rolle (ARC-3); ADR-011 App-Kern (ARC-4); nur Punktorte in Stufe 1 (ARC-5); weniger Module (ARC-6); Werkzeuge gestaffelt (ARC-7); Testgrenzen und Golden-Fixtures (ARC-8); nur SQL-Migrationen ohne Down (ARC-10); BlobStore, AAC, Cache-Header (ARC-11).
- Injektionswege geschlossen: nur `https:`-Links, TTS als Klartext ohne SSML (SEC-7); Bucket ohne Listung, harte Kostenbremse, Löschung archivierter Audios (SEC-8); Rate-Limit am CDN (SEC-9); Löschkonzept um Dritte und Beta-Teilnehmende ergänzt (SEC-10); Datenarten-Matrix für Store-Angaben (SEC-11); Pflicht-Pause ab Radfahrgeschwindigkeit, Auslösebereiche nicht an Straßenquerungen (SEC-12); Sitzungs- und OIDC-Härtung für Stufe 2 (SEC-6).
- Faktenkorrekturen: TTS-Preise mit Studio-Stimmen, Apple 4.8 korrekt wiedergegeben (VER-4, VER-5).
- Messbarkeit: Messaufbauten für Auslösung, Akku, Geo-Abfragen, Karte, Barrierefreiheit; Parameter der Nähe-Logik; Rate-Limit-Startwert; Alarmschwellen; geschlossener statt interner Play-Test; Referenzrouten; Schwachstellen-Gate präzisiert; IP-Regeln widerspruchsfrei; Zeitgranularität einheitlich „Tag“ (VER-2, VER-3, VER-6 bis VER-15).

**Abgelehnt oder abgewandelt, mit Begründung:**
- *PRD-1, Wizard-of-Oz oder fremde Audioguide-Plattform:* nicht als Hauptweg. Der Spike-Build entsteht ohnehin und prüft zusätzlich die automatische Auslösung bei gesperrtem Bildschirm, die eine Playlist nicht abbilden kann.
- *PRD-9, Quellen-Seite streichen:* abgelehnt. Die Seite ist für Open-Source-Lizenzen und Attribution ohnehin nötig und klein.
- *PRD-9, manuelles Abspielen streichen:* abgewandelt. Statt „In der Nähe“ gibt es „Zuletzt gehört“, damit die Produktanforderung „manuell abspielen“ erfüllt bleibt, ohne das Leitbild „Handy in der Tasche“ zu verwässern.
- *SEC-3, Beta-Token pro Person:* abgewandelt zu einem gemeinsamen Beta-Schlüssel im Build. Ein Token pro Person wäre eine Kennung; der gemeinsame Schlüssel reicht gegen zufälligen Missbrauch in einer geschlossenen Beta.
- *ARC-7, gitleaks und npm audit verschieben:* abgelehnt. Beide sind billig und schützen schon in Stufe 1.

### Loop 4: Abschlussprüfung (unabhängig)
Ein eigener Prüfer hat Version 2 gegen die feste Checkliste geprüft: alle 10 Punkte erfüllt, teils mit Einschränkungen. Von 16 kritischen und hohen Befunden waren 11 vollständig und 5 teilweise umgesetzt; dazu 15 kleinere Widersprüche (falsche WP-Verweise, ein nicht messbares Gate ohne Session-Tracking, Radius-Abweichung, unvollständige Quellen im Regionspaket, Einzelabruf `/v1/facts` gegen ADR-007, fehlende Einwilligung in der Muss-Liste, Diagramm). Ein toter Link wurde nicht gefunden.

**Zweite und letzte Überarbeitungsrunde:** alle 15 Widersprüche korrigiert; Teilbefunde ergänzt (Sentry ohne IP-Speicherung, Robot-Token ohne Update-Rechte, Log-Test für den Messpunkt in Stufe 1, grobe Vorladung in Blöcken und kein Einzelabruf für Stufe 2).

**Bewusst offen geblieben (als offene Entscheidung, nicht weiter geschleift):**
- Die genaue Vorladestrategie für Stufe 2 wird vor deren Einführung neu bewertet (ADR-007).
- Ein getrennter Touristen-Arm in der Beta ist optional und hängt an H-Segment (Q2, WP-05).
- Für den Messpunkt in Stufe 1 ist der Stack (Postgres, sqlc, goose, OpenAPI) größer als nötig; das ist begründet in Kauf genommen, weil er der erste Baustein des Stufe-2-Backends ist.

## Klärungsrunde mit Constantin (20.10.2026)
Alle offenen Entscheidungen Q1–Q10 und die Thesen wurden einzeln abgefragt. Ergebnis und Folgen stehen in [decisions.md](decisions.md). Größte Änderungen am Plan: nicht kommerziell (NC-Lizenzen erlaubt, H-Revenue entfällt), Inhalte von Claude recherchiert und von Constantin vollständig geprüft, Android zuerst mit neuem WP-08 für iOS, Karte in der Beta mit Offline-Kacheln (WP-17 nach Phase C), drei Zielgruppen mit etwa 45 Beta-Teilnehmenden, lockerere Entscheidungswerte, eigener Hetzner-Server mit Coolify statt CDN in Stufe 1.
