# Nebenbei – Zentraler Plan

Stand: 20.10.2026 · Version 2.1 (nach Klärungsrunde) · Entwurf zur Freigabe

Kurzer Einstieg. Details stehen in den verlinkten Fachdateien.

## 1. Zusammenfassung

**Produktidee:** Wer zu Fuß unterwegs ist, startet einen „Spaziergang“, steckt das Handy weg und hört kurze, gut erzählte Fakten zu Orten in der Nähe: Geschichte, Natur, regionale Kultur, Wissenschaft.

**Wichtigste Erkenntnis aus der Planung:** Das größte Risiko ist nicht die Technik, sondern ob Menschen das wiederholt wollen und ob sich gute Inhalte bezahlbar herstellen lassen. Der Plan testet das deshalb zuerst und mit den billigsten Mitteln, und baut die Plattform erst danach.

**Technische Leitidee:** **Der genaue Standort verlässt das Gerät nicht.** Die App lädt die Inhalte einer ganzen Region vorab (Texte und Audio) und entscheidet lokal, wann sie einen Fakt abspielt. Der Server sieht weder Position noch Spur noch Route. Audio wird einmal vorab erzeugt und als Datei ausgeliefert.

**Zielarchitektur (Stufe 2):** Expo-App (React Native + TypeScript), Go-Backend als modularer Monolith in einem Prozess, PostgreSQL + PostGIS, OpenAPI als Vertrag, Jobs in Postgres, Audio in einem EU-Objektspeicher hinter einem CDN, Betrieb auf dem eigenen Hetzner-Server mit Coolify, bei Bedarf ergänzt um ein CDN oder Managed Postgres.

**Weg dorthin in drei Stufen:**

| Stufe | Frage | Technik |
|---|---|---|
| **0 Feldtest** | Wollen Menschen das beim Gehen, und in welcher Form (Stimme, Länge, Dichte)? | Spike-App (Android) mit 30–60 fest eingebauten Fakten, 10–15 Personen in Nienburg |
| **1 Beta** | Kommen sie wieder? Was kostet die Prüfung eines Fakts? Für welche Gruppe lohnt es sich? | App + statisches Regionspaket aus Git-versionierten Inhalten + kleiner Messpunkt |
| **2 Öffentlicher Start** | Trägt es mit Community, mehreren Regionen, im Store? | Go-Backend mit PostGIS, Redaktionswerkzeug, Konten, Beiträge, Moderation |

Jede Stufe endet mit einer vorab festgelegten Go/No-Go-Entscheidung.

### Bewertung des ursprünglichen Technikvorschlags

| Vorschlag | Empfehlung | Warum |
|---|---|---|
| React Native + TS | **bleibt**, konkret mit Expo | [ADR-006](architecture/decisions/ADR-006-expo.md) |
| Go + Gin | **Go bleibt, Router `net/http`** | Mit OpenAPI-Codegenerierung fällt Gins Hauptnutzen weg; Gin wäre keine Fehlentscheidung ([ADR-003](architecture/decisions/ADR-003-go-net-http.md)) |
| PostgreSQL + PostGIS | **bleibt**, ab Stufe 2 | Stufe 1 kommt ohne Datenbank für Inhalte aus |
| pgx/pgxpool + sqlc | **bleibt** | [ADR-004](architecture/decisions/ADR-004-pgx-sqlc.md) |
| Docker Compose | **bleibt** für lokale Entwicklung | |
| GitHub Actions | **bleibt**, Härtung gestaffelt | [strategy.md](testing/strategy.md) |
| Modularer Monolith | **bleibt**, ein Prozess | [ADR-001](architecture/decisions/ADR-001-modular-monolith.md) |
| – | **neu:** Inhalte in Git für Stufe 1 ([ADR-012](architecture/decisions/ADR-012-content-in-git.md)), Regionspaket und Standort auf dem Gerät ([ADR-007](architecture/decisions/ADR-007-location-on-device.md)), vorproduziertes TTS ([ADR-008](architecture/decisions/ADR-008-pre-rendered-tts.md)), App-Kern ohne React ([ADR-011](architecture/decisions/ADR-011-app-core.md)) | |

### Wettbewerb und Positionierung [A, Recherche in WP-05]
Naheliegende Alternativen, jeweils auf Aktualität zu prüfen: Audioguide-Plattformen mit festen Touren (z. B. izi.TRAVEL, VoiceMap, städtische Hörtouren), standortbasierte Audio-Apps aus den USA (z. B. Autio), Outdoor-Apps mit Highlights (Komoot, Outdooractive), Stadtrallyes (Actionbound), und fürs Ohr vor allem Podcasts und Musik. Vermutete Abgrenzung: **freies Gehen statt fester Tour**, automatisch, ohne aufs Handy zu schauen. Touren werden im Feldtest bewusst nicht getestet ([decisions.md](decisions.md)).

## 2. Wo was steht

Jede Angabe hat genau eine Quelle. Andere Dokumente verlinken dorthin, statt sie zu wiederholen.

| Thema | Quelle |
|---|---|
| Grundsatzentscheidungen von Constantin | [decisions.md](decisions.md) |
| Hypothesen, MVP-Umfang je Stufe, Messung, Go/No-Go-Werte | [product/stages-and-hypotheses.md](product/stages-and-hypotheses.md) |
| Systembild, Repository-Struktur, Abhängigkeitsregeln, Konventionen | [architecture/overview.md](architecture/overview.md) |
| Begründete Architekturentscheidungen | [architecture/decisions/](architecture/decisions/) |
| Parameter der Nähe-Logik | [ADR-011](architecture/decisions/ADR-011-app-core.md) |
| Standort, Audio, UX, Spikes | [mobile/mobile-and-audio.md](mobile/mobile-and-audio.md) |
| Datenmodell, Quellen, Lizenzen | [data/data-model-and-sources.md](data/data-model-and-sources.md) |
| Bedrohungsmodell, Datenschutz, Rechtsfragen | [security/threat-model-and-privacy.md](security/threat-model-and-privacy.md) |
| Tests, CI, Betrieb, Definition of Done | [testing/strategy.md](testing/strategy.md) |
| Arbeitspakete | [roadmap/implementation-plan.md](roadmap/implementation-plan.md) |
| Noch offene Fragen, Annahmen, Prüfpunkte | [roadmap/open-questions.md](roadmap/open-questions.md) |

## 3. Kurz zu den Stufen

- **Stufe 0:** Feldtest mit der Android-Spike-App in Nienburg. Details und Entscheidungsregel: [stages-and-hypotheses.md](product/stages-and-hypotheses.md).
- **Stufe 1:** Beta mit statischem Regionspaket vom eigenen Server. Systembild: [overview.md](architecture/overview.md).
- **Stufe 2:** öffentlicher Start mit Go-Backend und PostGIS, erst nach positiver Beta.

## 4. Antworten auf die Leitfragen

**1. Kleinster sinnvoller MVP:** Stufe 0 als Feldtest, dann Stufe 1 als statische Beta ([stages-and-hypotheses.md](product/stages-and-hypotheses.md)).

**2. Muss, optional, nicht:** [stages-and-hypotheses.md](product/stages-and-hypotheses.md).

**3. Architektur und Modulgrenzen:** [overview.md](architecture/overview.md) und die ADRs.

**4. Externe Quellen → Wissensdatenbank:** In Stufe 1 gibt es keine automatischen Importe; die Redaktion erfasst Quellen von Hand im selben Modell (Quelle, Lizenz, Abrufzeit, externe Revision). Ab Stufe 2: Abruf-Job → `candidate` mit Rohdaten und Lizenz → Dublettenabgleich → redaktionelle Prüfung → Revision mit Quellenverknüpfung → TTS → CDN. Die App fragt nie externe Dienste direkt an. Details: [data-model-and-sources.md](data/data-model-and-sources.md).

**5. Quellen, Lizenzen, Aktualität, Änderungen, Freigaben:** Stufe 1: Jeder Fakt ist eine YAML-Datei mit Quellen und Lizenz; Änderungen laufen per Pull Request, Git ist die Revisionshistorie, das Review ist die Freigabe, das Build-Skript lehnt Fakten ohne Quelle oder mit unzulässiger Lizenz ab. Stufe 2: unveränderliche Revisionen in der Datenbank, Statusablauf mit Audit-Log, dieselben Prüfregeln.

**6. Fakten in der Nähe und entlang einer Route:** Stufe 1: Die App hat das ganze Regionspaket und prüft bei jedem Positions-Update lokal die Distanz (Haversine, Hysterese, Mindestverweildauer, „schon gehört“). Nur Punktorte mit Radius. Stufe 2: Kachel-API mit PostGIS (`ST_Intersects` mit der Kachelhülle, GiST-Index) für mehrere Regionen; für Routen liest die App die GPX-Datei lokal, bestimmt die berührten Kacheln und sortiert die Fakten entlang der Linie. Eine serverseitige Korridorabfrage ist als Ausweichlösung skizziert.

**7. Anonyme Nutzung, Konten, Beiträge, Moderation, Löschung:** Stufe 1 ist vollständig anonym. Ab Stufe 2: Konten nur für eigene Beiträge über einen externen OIDC-Anbieter ohne eigenes Passwortsystem; Beiträge als Entwurf mit Lizenz-Zustimmung, Vorab-Moderation mit Checkliste, Audit-Log, Begründung; Kontolöschung in der App und per Web-Link innerhalb von 30 Tagen. Was mit veröffentlichten Beiträgen nach der Löschung geschieht, ist eine offene Rechtsfrage.

**8. Prüfung von Standort und Audio auf Android und iOS:** Spikes S1–S5 auf echten Geräten mit messbaren Kriterien vor jeder Produktarbeit, danach eine Geräte-Checkliste pro Release. Details: [mobile-and-audio.md](mobile/mobile-and-audio.md) und WP-01 bis WP-04.

**9. Sicherheitsmaßnahmen, die im MVP Pflicht sind:** [threat-model-and-privacy.md](security/threat-model-and-privacy.md), Abschnitt 4.

**10. Offene Annahmen und Spikes:** [open-questions.md](roadmap/open-questions.md) und Phase A in [implementation-plan.md](roadmap/implementation-plan.md).

## 5. Größte Risiken

| Risiko | Wirkung | Gegenmaßnahme |
|---|---|---|
| Gute Inhalte sind zu teuer (H-Content) | Geschäftsmodell trägt nicht | Zeit pro Fakt ab dem ersten Fakt messen; Kill-Kriterium; später Community und Wikidata als Hebel prüfen |
| Kein wiederkehrendes Interesse (H-Wiederkehr), Inhalte im Stammgebiet schnell erschöpft | Produkt trägt nicht | Stufe 0 und Beta mit Entscheidungsregel; Annahme A7 zum Inhaltsnachschub |
| Auslösung bei gesperrtem Bildschirm auf iOS oder Samsung unzuverlässig | Kernfunktion bricht | Spike S1 zuerst; Ausweg Transistorsoft-Bibliothek (Lizenz) |
| Store-Prüfung lehnt Hintergrund-Standort oder -Audio ab | Verzögerung | Spike S5 mit geschlossenem Test, Review-Notiz mit Video, Session-Begrenzung |
| Lizenzverstöße durch vermischte Daten (ODbL, CC BY-SA) | Rechtsrisiko | Lizenzprüfung im Build; OSM und Wikipedia-Texte nicht im MVP |
| Akkuverbrauch zu hoch | Abbruch der Nutzung | Spike S3 |
| Personaldecke (1–2 Personen) | Alles dauert länger | Stufen klein halten; Plattform erst nach Validierung |
