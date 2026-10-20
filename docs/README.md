# Nebenbei – Planungsunterlagen

Stand: 20.10.2026 · Version 2.1 nach Klärungsrunde · Status: **Entwurf zur Freigabe**, noch keine Implementierung.

Nebenbei ist eine App für Android und iOS, die beim Spazierengehen ortsbezogenes Wissen als Audio abspielt: Geschichte, Natur, regionale Kultur und Wissenschaft.

## Bestandsaufnahme (Phase 0)

Das Repository `github.com/Constispex/nebenbei` enthielt am 20.10.2026 nur einen Initial-Commit mit einer `.gitignore` (Visual-Studio-Vorlage). Es gab keinen Code, keine Konventionen und keine CI. Die Planung ist unter dieser Annahme entstanden und bleibt gültig.

## Dokumente

| Datei | Inhalt |
|---|---|
| [plan.md](plan.md) | **Einstieg:** Zusammenfassung, Wegweiser „wo was steht“, Antworten auf die zehn Leitfragen, Risiken |
| [decisions.md](decisions.md) | Grundsatzentscheidungen von Constantin (einzige Quelle) |
| [product/stages-and-hypotheses.md](product/stages-and-hypotheses.md) | Hypothesen, MVP-Umfang je Stufe, Messung, Entscheidungsregel |
| [architecture/overview.md](architecture/overview.md) | Systembild, Repository-Struktur, Abhängigkeitsregeln, Konventionen |
| [architecture/decisions/](architecture/decisions/) | Architekturentscheidungen (ADR) |
| [security/threat-model-and-privacy.md](security/threat-model-and-privacy.md) | Bedrohungsmodell, Sicherheits- und Datenschutzanforderungen, offene Rechtsfragen |
| [data/data-model-and-sources.md](data/data-model-and-sources.md) | Datenmodell, Geo-Abfragen, Quellen, Lizenzen, Attribution |
| [mobile/mobile-and-audio.md](mobile/mobile-and-audio.md) | Mobile-Architektur, Standort, Audio, UX, Spikes |
| [testing/strategy.md](testing/strategy.md) | Teststrategie, CI, Betrieb, Qualitäts-Gates, Definition of Done |
| [roadmap/implementation-plan.md](roadmap/implementation-plan.md) | Arbeitspakete mit Abhängigkeiten und Abnahmekriterien |
| [roadmap/open-questions.md](roadmap/open-questions.md) | Offene Entscheidungen, Annahmen, Prüfpunkte |
| [review-log.md](review-log.md) | Wie der Plan entstanden ist: Untersuchungen, Reviews, Änderungen |

## Kennzeichnung

- **[F]** belegte Tatsache mit Quelle
- **[A]** Annahme oder nicht abschließend verifiziert
- **[E]** Empfehlung

Viele offizielle Seiten (docs.expo.dev, support.google.com, go.dev, docs.sqlc.dev) waren aus der Rechercheumgebung nicht direkt abrufbar. Die Aussagen dazu stützen sich auf Suchergebnis-Auszüge dieser Seiten. Wo eine Entscheidung daran hängt, steht ein Prüfpunkt in [open-questions.md](roadmap/open-questions.md).

Keine dieser Unterlagen ersetzt eine rechtliche Prüfung.
