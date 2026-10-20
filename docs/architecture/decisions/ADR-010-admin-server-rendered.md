# ADR-010: Redaktionswerkzeug serverseitig gerendert im Monolithen

**Status:** zurückgestellt bis Stufe 2 (Stufe 1 nutzt Git, ADR-012)

**Auslöser:** Community-Beiträge brauchen Moderation, oder die Redaktion arbeitet nicht mit Git, oder mehr als etwa 1.000 Fakten.

**Entscheidung (für Stufe 2):** Schlichtes Web-Werkzeug im Go-Monolithen, serverseitig gerendert (`html/template`, wenig JavaScript, z. B. htmx), auf eigener Subdomain. Anmeldung über OIDC (ADR-009). Sitzungen:
- Session-ID wird nach dem Login neu vergeben (Schutz vor Session-Fixation).
- Cookie mit `__Host-`-Präfix, `Secure`, `HttpOnly`, `SameSite=Strict`; Sitzungen serverseitig in einer Tabelle `staff_session` mit kurzer Laufzeit.
- CSRF-Token für alle schreibenden Formulare; Content-Security-Policy ohne Inline-Skripte.
- Kein CORS (gleiche Origin).
- Alle angezeigten Fremdtexte (Meldungen, Beiträge) werden escaped; Links nur mit `https:`.

**Begründung:** Keine zweite Frontend-Anwendung und kein zweiter Build; nutzt dieselben Services wie die API.

**Verworfen:** eigene React-Admin-App; fertige Headless-CMS (Geo, Revisionen und Lizenzen passen nicht ohne Umwege).
