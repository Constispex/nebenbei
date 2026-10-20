# Architekturentscheidungen (ADR)

Format: Kontext, Entscheidung, Begründung, verworfene Alternativen, Folgen und Risiken. Status „vorgeschlagen“ heißt: wird mit der Freigabe des Plans angenommen. Neue Entscheidungen bekommen eine neue Nummer; überholte ADRs werden als „ersetzt durch“ markiert, nicht gelöscht.

| Nr. | Titel | Gilt ab | Status |
|---|---|---|---|
| 001 | Modularer Monolith in einem Prozess | Stufe 1 (Messpunkt), voll Stufe 2 | vorgeschlagen |
| 002 | OpenAPI-first mit Codegenerierung | Stufe 1 | vorgeschlagen |
| 003 | Go mit `net/http` statt Gin | Stufe 1 | vorgeschlagen, hängt an Q1 |
| 004 | pgx, sqlc und reine SQL-Migrationen mit goose | Stufe 1 | vorgeschlagen |
| 005 | Hintergrundjobs in Postgres | Stufe 2 | vorgeschlagen |
| 006 | Expo statt bare React Native | Stufe 0 | vorgeschlagen |
| 007 | Standort bleibt auf dem Gerät; Regionspaket, später Kacheln | Stufe 1 | vorgeschlagen |
| 008 | TTS vorproduziert, Audio als unveränderliche Dateien | Stufe 1 | vorgeschlagen, Anbieter nach S4 |
| 009 | Kein eigenes Passwortsystem, externer OIDC-Anbieter | Stufe 2 | vorgeschlagen, Anbieter offen |
| 010 | Redaktionswerkzeug serverseitig gerendert im Monolithen | Stufe 2 | zurückgestellt |
| 011 | App-Kern als reines TypeScript, Zustand außerhalb von React | Stufe 1 | vorgeschlagen |
| 012 | Inhalte in Git mit Build-Skript für Stufe 1 | Stufe 1 | vorgeschlagen |
