# ADR-004: pgx, sqlc und reine SQL-Migrationen mit goose

**Status:** vorgeschlagen

**Entscheidung:**
- pgx v5 mit pgxpool, sqlc für typisierte Abfragen.
- Migrationen ausschließlich als SQL-Dateien im goose-Format, ins Binary eingebettet; sqlc liest dasselbe Verzeichnis. Go-Migrationen sind für Schemaänderungen verboten (sqlc sähe sie nicht), höchstens für reine Datenkorrekturen.
- Keine Down-Migrationen: Rollback heißt, das vorherige Image auszuliefern; Migrationen folgen Expand/Contract. Getestet wird „up auf leerer Datenbank“ und ab dem ersten Release „up vom letzten Release-Stand“.
- Geometrien werden in SQL umgewandelt (`ST_AsGeoJSON`, `ST_MakePoint`), nicht als Go-Typen gemappt.
- Getrennte DB-Rollen für Migration und Laufzeit.

**Begründung:** PostGIS-Abfragen sind reines SQL; ein ORM stört dort. sqlc prüft Abfragen zur Build-Zeit gegen das Schema. Down-Migrationen würden gepflegt, aber nie benutzt.

**Verworfen:** GORM/ent (PostGIS umständlich, versteckte Abfragen); Atlas (zusätzliches Werkzeug mit Lizenzfragen ohne Vorteil bei reinen SQL-Migrationen); golang-migrate (gleichwertig, Wahl ist Geschmackssache).

**Risiken:** Dynamische Filter sind in sqlc mühsam: `sqlc.narg`-Muster oder einzelne handgeschriebene pgx-Abfragen im selben Store.
