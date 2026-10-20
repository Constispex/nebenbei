# ADR-005: Hintergrundjobs in Postgres

**Status:** vorgeschlagen, gilt ab Stufe 2 (Stufe 1 hat keine Jobs; TTS läuft im Build-Skript)

**Entscheidung:** Job-Queue in Postgres im selben Prozess (ADR-001). Bevorzugt River (pgx-v5-Treiber), sofern der Prüfpunkt „vor Stufe 2“ in open-questions.md (spätestens in WP-30) Version, Reife und Lizenz bestätigt; sonst eine Tabelle `job` mit `SELECT … FOR UPDATE SKIP LOCKED`. Einsatz: TTS-Erzeugung bei Veröffentlichung, Löschfristen, später Importe.

**Begründung:** [F] Jobs werden in derselben Transaktion eingereiht wie die fachliche Änderung und erst mit dem Commit sichtbar ([pkg.go.dev](https://pkg.go.dev/github.com/riverqueue/river)). Keine zusätzliche Infrastruktur.

**Verworfen:** RabbitMQ, Kafka, NATS (Betriebsaufwand ohne Bedarf).

**Risiken:** River vor dem Einsatz an der offiziellen Quelle prüfen [A].
