# Nebenbei

Nebenbei ist eine App für Android und iOS, die beim Spazierengehen ortsbezogene Fakten als Audio abspielt. Das Projekt ist nicht kommerziell. Projektsprache ist Deutsch; Code, Bezeichner und Commit-Messages auf Englisch.

## Stand

Das Repository enthält bisher nur die Planung unter `docs/`. **Umsetzung erst nach ausdrücklicher Freigabe durch Constantin.** Bis dahin nur Planungsdokumente ändern. Ein Prototyp oder Experiment wird als Arbeitspaket beschrieben und erst nach Freigabe gebaut.

## Wo was steht

Einstieg ist `docs/plan.md`. Jede Angabe hat genau eine Quelle; andere Dateien verlinken dorthin und wiederholen sie nicht.

| Thema | Quelle |
|---|---|
| Grundsatzentscheidungen | `docs/decisions.md` |
| Hypothesen, MVP-Umfang, Go/No-Go-Werte | `docs/product/stages-and-hypotheses.md` |
| Systembild, Repo-Struktur, Abhängigkeitsregeln, Konventionen | `docs/architecture/overview.md` |
| Architekturentscheidungen mit Begründung | `docs/architecture/decisions/` |
| Parameter der Nähe-Logik | `docs/architecture/decisions/ADR-011-app-core.md` |
| Arbeitspakete mit Abnahmekriterien | `docs/roadmap/implementation-plan.md` |
| Offene Fragen, Annahmen, Prüfpunkte | `docs/roadmap/open-questions.md` |

## Arbeitsregeln

- **Standort bleibt auf dem Gerät.** Kein Code darf Position, Spur oder Route an einen Server senden. Inhalte, Audio und Kartenkacheln kommen vorab im Regionspaket (ADR-007).
- **Keine Secrets** in Code, Doku, Logs oder Commits. Logs ohne Koordinaten, IPs, E-Mails oder Tokens.
- **Nicht ohne Freigabe:** Abhängigkeits-Updates, DB-Migrationen, Infrastruktur- oder Secret-Änderungen, destruktive Befehle, Änderungen an externen Diensten (Server, Stores, Hetzner, Coolify).
- **Arbeiten per Pull Request** von einem eigenen Branch; Constantin prüft und merged. Fakten-Inhalte kommen ebenfalls per PR, jeder mit belegbarer Quelle und Lizenz.
- **Ein Arbeitspaket pro PR.** Abnahmekriterien des Pakets sind die Definition of Done; wie geprüft wird, steht in `docs/testing/strategy.md`.
- **Doku mitziehen:** Ändert sich eine Entscheidung oder ein Wert, nur die Quelldatei ändern und Verweise prüfen. Neue Grundsatzentscheidungen in `docs/decisions.md`, neue Architekturentscheidungen als ADR.
- **Kennzeichnung in der Doku:** [F] belegt mit Quelle, [A] Annahme, [E] Empfehlung.
