# Entscheidungen

Einzige Quelle für Grundsatzentscheidungen von Constantin. Andere Dokumente verlinken hierher und wiederholen die Entscheidung nicht. Architekturentscheidungen mit Begründung stehen in den [ADRs](architecture/decisions/).

## Beantwortet am 20.10.2026

| Nr. | Frage | Antwort | Folge |
|---|---|---|---|
| Q1 | Sprachen im Team | Go + TypeScript | ADR-003 bleibt |
| Q2 | Pilotregion | Nienburg/Weser mit Umland | Feldtest-Bereich (1–2 km²) wird in WP-06 festgelegt |
| Q3 | Hosting | Eigener Server bei Hetzner (Anbieter noch zu bestätigen), Coolify | Stufe 1 ohne CDN; AV-Vertrag in der Hetzner-Konsole abschließen; Stufe 2 neu bewerten |
| Q4 | Auth-Anbieter | Erst vor Stufe 2 | wird in WP-32 entschieden |
| Q5 | Wer schreibt Inhalte | Claude recherchiert und schreibt; Constantin prüft jeden Fakt per Pull Request | H-Content misst Constantins Prüfzeit pro Fakt |
| Q6 | Testgeräte | Samsung S23 Ultra, Xiaomi 13T; kein iPhone | Spikes und Feldtest nur Android; iOS in WP-08 vor der Beta |
| Q7 | Geschäftsmodell | Nicht kommerziell | H-Revenue entfällt; NC-Lizenzen erlaubt, aber markiert |
| Q8 | Entscheidungswerte | Lockerer als vorgeschlagen (etwa ein Viertel niedriger) | Werte in [stages-and-hypotheses.md](product/stages-and-hypotheses.md#entscheidungsregel) |
| Q9 | Karte in der Beta | Ja | Kacheln als PMTiles im Regionspaket; WP-17 in Phase C |
| Q10 | Redaktion mit Git | Erledigt durch Q5 | – |
| – | Touren im Feldtest | Nein, nur freies Gehen | H-Differenz nicht verfolgt |
| – | Eigene Aufnahme | Constantin spricht 10 Fakten ein | A6 wird im Feldtest geprüft |

## Ergänzungen
- **Inhalte unter NC-Lizenz** werden im Build markiert, damit man sie jederzeit herausfiltern kann. Werbung oder Spenden könnten später als kommerziell gelten. Förderprogramme für Kultur- und Heimatprojekte sind eine mögliche Finanzierung.
- **Zielgruppen** werden getrennt ausgewertet: Einheimische, zu Besuch, Wandern/Rad.
- **Umsetzung** beginnt erst nach ausdrücklicher Freigabe des Plans durch Constantin.
