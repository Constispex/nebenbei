# ADR-002: OpenAPI-first mit Codegenerierung

**Status:** vorgeschlagen

**Entscheidung:** `api/openapi.yaml` (OpenAPI 3.1, Fallback 3.0, falls die Generatoren es erfordern) ist die Quelle der Wahrheit. Go-Server-Schnittstelle mit `oapi-codegen` (Strict-Server, Ziel `net/http`), App-Client mit `openapi-typescript` + `openapi-fetch`. CI prüft, ob generierter Code aktuell ist, und erkennt Breaking Changes mit `oasdiff`.

**Umfang:** In Stufe 1 beschreibt die Spec nur den Messpunkt; das Regionspaket und `config.json` werden als JSON Schema in `packages/schema/` beschrieben und in Go und TypeScript gegen dieselben Beispiele geprüft. Der Breaking-Change-Check mit `oasdiff` kommt mit der ersten Store-Version, weil es vorher keine alten App-Versionen im Feld gibt.

**Begründung:** Eine Quelle, typsichere Clients auf beiden Seiten, keine Drift zwischen App und Backend. [F] oapi-codegen unterstützt std-http, Gin, chi und Echo sowie einen Strict-Server-Modus ([GitHub](https://github.com/oapi-codegen/oapi-codegen)).

**Verworfen:** Code-first (Spec läuft der Implementierung hinterher), GraphQL (Overhead für eine kleine Lese-API), tRPC (setzt TS-Backend voraus).

**Risiken:** [F] Unvollständige 3.1-Unterstützung, Unions werden zu `any`. Schema bewusst einfach halten. Alte App-Versionen leben lange: `/v1`, nur additive Änderungen, Mindestversion.
