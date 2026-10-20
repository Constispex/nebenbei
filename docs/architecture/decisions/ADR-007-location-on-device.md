# ADR-007: Standort bleibt auf dem Gerät; Regionspaket, später Kacheln

**Status:** vorgeschlagen

**Kontext:** Standortdaten sind das größte Datenschutzrisiko. OS-Geofences taugen nicht für dichte Faktfelder: [F] Android erlaubt höchstens 100 Geofences und empfiehlt mindestens 100–150 m Radius bei Minuten Latenz ([Android](https://developer.android.com/develop/sensors-and-location/location/geofencing)); für iOS werden 20 Regionen genannt [A]. Das Review hat gezeigt, dass auch grobe Abrufe Spuren erzeugen: Wer Audio erst beim Auslösen lädt, verrät Position und Uhrzeit auf etwa 50 m genau; wer Kacheln beim Gehen nachlädt, verrät die Kachelgrenzen und den Startbereich (Wohnort).

**Entscheidung:**
- **Stufe 1:** Die App lädt das ganze Regionspaket (Metadaten, alle Audiodateien und die Kartenkacheln der Region als PMTiles-Datei) vor dem ersten Spaziergang, bevorzugt im WLAN, und danach nur bei neuer `content_version`. Während eines Spaziergangs gibt es keine inhaltsbezogenen Netzwerkzugriffe. Damit sieht der Server nur „Region X wurde geladen“.
- **Stufe 2 (mehrere Regionen):** Kacheln auf Zoom 13 (in Deutschland etwa 3 × 3 km) unter versionierten URLs (`?v=content_version`, `immutable`). Die App lädt Kacheln **mitsamt Audio und allen Details** in groben Blöcken (z. B. alle Zoom-13-Kacheln innerhalb einer Zoom-11-Kachel, etwa 12 × 12 km) beim Start und lädt beim Gehen mit Vorlauf den nächsten Block, damit Ladezeitpunkte nicht mit Positionen zusammenfallen. Einen Abruf einzelner Fakten beim Auslösen gibt es nicht. Vor Einführung bewerten, ob die Store-Angaben dann „ungefährer Standort“ erfordern (Sicherheitsdokument, Bedrohung 1 und Datenarten-Matrix in WP-04).
- Keine Speicherung von Nutzerstandorten, Spuren oder Routen auf dem Server; GPX-Dateien werden auf dem Gerät gelesen.
- Auslieferung in Stufe 1 vom eigenen Server (Hetzner, Coolify), Zugriffslogs abgeschaltet oder mit gekürzter IP und kurzer Frist; ein späteres CDN nur mit EU-Standort und vertraglich gleicher Regel.
- Die Karte lädt nie Kacheln aus dem Netz; sie nutzt nur die Kacheln im Regionspaket.

**Begründung:** Senkt das Datenschutzrisiko, macht die Nutzung offline-fähig, kein Regionen-Limit, CDN entlastet den Server.

**Verworfen:** Server-Umkreisabfrage mit exakten Koordinaten; OS-Geofencing; Audio-Abruf beim Auslösen.

**Risiken:** Paketgröße. Bei 300 Fakten zu je etwa 75 s und 32 kbit/s AAC etwa 90 MB, dazu die Kartenkacheln für Nienburg und Umland (geschätzt 20–60 MB) [A, eigene Rechnung]; die Größe wird vor dem Download angezeigt. Wird sie zu groß, Region in Teilgebiete teilen.

**Abnahme:** Netzwerkmitschnitt eines Spaziergangs zeigt keine inhaltsbezogenen Anfragen.
