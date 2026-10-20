# ADR-009: Kein eigenes Passwortsystem, externer OIDC-Anbieter

**Status:** vorgeschlagen für Stufe 2, Anbieter offen (Q4)

**Entscheidung:** Anmeldung für Redaktion und Beitragende über einen OIDC-Anbieter mit EU-Datenhaltung. Redaktion und Admin: Passkey oder zweiter Faktor Pflicht, geprüft im Backend über die Claims `amr`/`acr`. Beitragende: Magic Link oder Passkey. Ablauf: Authorization Code mit PKCE, `state` und `nonce`. Rollen (`contributor`, `moderator`, `editor`, `admin`) verwaltet das Backend selbst.

**Begründung:** Passwortspeicherung, Wiederherstellung und MFA sind sicherheitskritisch und kein Produktkern. Stufe 1 braucht keine Anmeldung: Die Redaktion arbeitet über GitHub (ADR-012).

**Kandidaten:** Zitadel (Cloud EU oder selbst gehostet), Supabase Auth (EU-Region), Ory. Keycloak ist für 1–2 Personen zu schwer.

**Folgen:**
- [F] Apple 4.8: Wer einen Drittanbieter- oder Social-Login für das Hauptkonto anbietet, muss zusätzlich eine gleichwertige, datensparsame Login-Option anbieten; eine Ausnahme gilt für Apps mit ausschließlich eigenem Kontosystem ([Guidelines](https://developer.apple.com/app-store/review/guidelines/)). Sign in with Apple erfüllt das, ist aber nicht namentlich vorgeschrieben. Mit nur Magic Link und Passkey über den eigenen OIDC-Mandanten entfällt die Pflicht [A, vor Stufe 2 prüfen].
- [F] Apple 5.1.1(v): Wer Konten anbietet, muss die Löschung in der App ermöglichen.
