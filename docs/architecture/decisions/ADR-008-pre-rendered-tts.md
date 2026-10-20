# ADR-008: TTS vorproduziert, Audio als unveränderliche Dateien

**Status:** vorgeschlagen, Anbieter nach Spike S4

**Entscheidung:**
- Audio wird einmal pro Text und Stimme erzeugt (Stufe 1 im Build-Skript, Stufe 2 als Job bei Veröffentlichung) und im Objektspeicher abgelegt. Die App spielt nur Dateien ab; System-TTS (`expo-speech`) höchstens als Vordergrund-Fallback.
- Format AAC (`.m4a`), mono, etwa 32–48 kbit/s; Opus wird auf iOS nicht in allen Wegen unterstützt [A].
- Schlüssel `audio/<sha256(tts_text + stimme + engine_version)>.m4a`, unveränderlich, `Cache-Control: public, max-age=31536000, immutable`. Die URL entsteht aus `CDN_BASE_URL + key`.
- Schnittstelle `BlobStore { Put(key, reader, contentType); Delete(key) }` mit S3-Implementierung (lokal MinIO) und Datei-Implementierung für Tests.
- Bucket ohne Listung; nur veröffentlichte Audios liegen im öffentlichen Bucket. Vorschauen für Entwürfe entstehen lokal oder in einem privaten Bucket.
- Audios archivierter Fakten werden beim nächsten Build bzw. per Job gelöscht und aus dem CDN entfernt.
- TTS-Text wird als **Klartext** übergeben, nicht als SSML; Aussprachehilfen nur über eine Positivliste (z. B. Lexikon des Anbieters).

**Begründung:**
- [F] Ein Apple-Forenbericht zeigt, dass System-TTS bei gesperrtem Bildschirm pausiert ([Forum](https://developer.apple.com/forums/thread/849627)).
- Einheitliche Qualität, derselbe Abspielweg wie für spätere Sprecheraufnahmen, offline-fähig.
- Kein TTS-Aufruf pro Nutzer: kein Kostenrisiko durch anonyme Nutzung.
- [F] Google Cloud TTS kostet je 1 Mio. Zeichen 4 USD (Standard/WaveNet), 16 USD (Neural2), 30 USD (Chirp 3 HD); Studio-Stimmen deutlich mehr ([Preise](https://cloud.google.com/text-to-speech/pricing)). 10.000 Fakten zu 600 Zeichen kosten mit Chirp 3 HD etwa 180 USD vor Freikontingent [A, eigene Rechnung], jede Textänderung kostet erneut.

**Verworfen:** System-TTS als Hauptweg; TTS auf Abruf; SSML aus Redaktionstext (Injektionsweg).

**Risiken:** Nutzungsrechte am erzeugten Audio für kommerzielle Verbreitung je Anbieter prüfen (Rechtsfrage 4).
