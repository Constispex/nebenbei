# ADR-001: Modularer Monolith in einem Prozess

**Status:** vorgeschlagen (in Stufe 1 nur der Messpunkt, voll ab Stufe 2)

**Kontext:** 1–2 Entwickler, kleines Datenvolumen, wenige Hintergrundaufgaben (TTS, Löschfristen).

**Entscheidung:** Ein Go-Modul, ein Binary, ein Prozess. HTTP-Server und Job-Verarbeitung (Goroutine) laufen zusammen; kein Rollen-Flag. Betrieb mit genau einer dauerhaft laufenden Instanz (kein Herunterskalieren auf null), weil sonst keine Jobs laufen. Fachmodule unter `backend/internal/<modul>`.

**Begründung:** Geringster Betriebsaufwand; Transaktionen über Module hinweg sind einfach; Module lassen sich später an ihren Grenzen herauslösen. Eine eigene Worker-Rolle bringt bei einigen hundert TTS-Jobs keinen Nutzen.

**Verworfen:**
- Microservices: verteilte Transaktionen, mehr Deploys und Observability ohne Skalierungsbedarf.
- Serverless mit Skalierung auf null: Jobs und periodische Aufgaben laufen nicht zuverlässig.
- Backend-as-a-Service (Supabase als Ganzes): Lizenz-, Moderations- und TTS-Logik bräuchte trotzdem eigenen Code. Als gehostetes Postgres bleibt Supabase eine Option.

**Risiken:** Modulgrenzen verwässern. Gegenmittel: Tabellen-Hoheit pro Modul, Review, ab mehr als drei Fachmodulen depguard.
