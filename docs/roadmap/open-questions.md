# Offene Entscheidungen, Annahmen und Prüfpunkte

Stand: 20.10.2026.

Beantwortete Entscheidungen stehen in [decisions.md](../decisions.md), Hypothesen und Zielwerte in [stages-and-hypotheses.md](../product/stages-and-hypotheses.md).

## Noch offen
- Zielwert X für die Prüfzeit pro Fakt (nach den ersten zehn Fakten, WP-06).
- Genauer Feldtest-Bereich in Nienburg (WP-06).
- Bestätigung, dass der Server bei Hetzner steht, und Abschluss des AV-Vertrags (WP-19).
- iPhone für WP-08 leihen oder kaufen, Apple-Developer-Konto (vor der Beta).

## Annahmen (gelten, bis jemand widerspricht)
- A1: erledigt. Das Repository enthielt am 20.10.2026 nur eine `.gitignore` (Visual-Studio-Vorlage, wird in WP-00 ersetzt).
- A2: Zielmarkt zuerst Deutschland, Sprache Deutsch.
- A3: Team aus 1–2 Personen.
- A4: 150–300 Fakten reichen für eine aussagekräftige Beta (WP-07 kann die Zahl ändern).
- A5: Ein Fakt pro 150–250 m Wegstrecke ist eine angenehme Dichte (Stufe 0 variiert die Dichte).
- A7: Ein Stammgebiet braucht laufend neue Fakten, damit es interessant bleibt; wie viele pro Monat, zeigt die Beta (Wiederkehr und Interviews).
- A8: Ein Regionspaket von bis zu etwa 150 MB (mit Kartenkacheln) wird akzeptiert, wenn die Größe vorher angezeigt wird.

## Technische Prüfpunkte (direkt an der offiziellen Quelle verifizieren)
Vor Stufe 0/1:
- iOS: Reicht „Beim Verwenden“ plus Background Mode `location` für eine im Vordergrund gestartete Session mit gesperrtem Bildschirm? (WP-01)
- Google Play: aktuelle Policy zu Standort im Vordergrund-Service; Prüfung im geschlossenen Test (WP-04)
- expo-audio: `interruptionMode`-Werte, Audio-Fokus gegenüber Musik- und Navigations-Apps (WP-02)
- Aktuelle stabile Expo-SDK-Version zum Projektstart
- MapLibre React Native auf der New Architecture mit lokalen PMTiles (WP-17)
- Ob `expo-updates` bzw. andere Expo-Module Kennungen senden, wenn sie nicht eingebunden sind (SDK-Liste)
- CDN- und Hoster-Optionen für Log-Kürzung und harte Kostenbremse

Vor Stufe 2:
- PostGIS beim gewählten Hoster; River (Version, Reife, Lizenz); sqlc-Version; oapi-codegen und OpenAPI 3.1
- iOS-Grenze von 20 überwachten Regionen (nur für Hintergrund-Auslösung)
- Apple 4.8 bei reinem Magic-Link- und Passkey-Login über einen OIDC-Mandanten
- Wortlaut OSM-Tile- und Nominatim-Policy, Wikidata-Query-Service-Limits, DWD-Quellenvermerk

## Rechtsfragen
Siehe [threat-model-and-privacy.md](../security/threat-model-and-privacy.md) Abschnitt 5. Für Stufe 0/1: 1, 2, 3, 4, 7, 8, 11, 12, 13.
