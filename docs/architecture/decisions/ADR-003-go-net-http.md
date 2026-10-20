# ADR-003: Go mit `net/http` statt Gin

**Status:** vorgeschlagen

**Entscheidung:** Backend in Go. Router ist `http.ServeMux` der Standardbibliothek; chi als Ausweg, falls Middleware-Gruppen fehlen.

**Begründung:** [F] Seit Go 1.22 kann `ServeMux` Methoden und Pfadparameter ([go.dev](https://go.dev/blog/routing-enhancements)). Da Routing und Validierung aus OpenAPI generiert werden, fällt Gins Hauptvorteil weg. Keine Bindung an `gin.Context`, Standard-Middleware passt direkt. Der Vorteil ist moderat; [F] Gin wird aktiv gepflegt (v1.12.0, Feb. 2026, [Releases](https://github.com/gin-gonic/gin/releases)) und wäre keine Fehlentscheidung.

**Verworfen:**
- Gin/Echo: eigener Context-Typ ohne Zusatznutzen bei Codegenerierung.
- TypeScript-Backend (Fastify/NestJS): Ein-Sprachen-Vorteil ist real, Typen teilen wir aber schon über OpenAPI. Go bietet ein statisches Binary, wenig Speicher, starke Nebenläufigkeit für Jobs und pgx/sqlc. **Ausnahme:** Beherrscht das Team faktisch nur TypeScript, ist Fastify + Kysely die bessere Wahl (offene Frage Q1).

**Risiken:** etwas mehr Middleware von Hand (gering).
