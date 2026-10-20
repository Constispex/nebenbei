# Umsetzungsplan

Stand: 20.10.2026 · Version 2 (nach Review) · Nichts davon ist begonnen. Jedes Arbeitspaket (WP) startet erst nach Freigabe.

**Aufwand** als grobe Klasse ohne Kalenderdaten: **S** ≤ 3 Personentage, **M** 4–10, **L** 11–20. Annahme: Umsetzung in Go und TypeScript (Q1), Inhalte recherchiert Claude, Constantin prüft. Unsicherheit steht beim Paket. **Agent?** sagt, ob ein Subagent bei der Umsetzung sinnvoll ist.

**Reihenfolge:** (1) Risiken mit hoher Unsicherheit, einschließlich Nachfrage und Inhaltskosten; (2) Produktvalidierung mit dem billigsten Mittel; (3) kleinster durchgängiger Ablauf; (4) Sicherheits- und Qualitätsgrundlagen im Umfang dieses Ablaufs; (5) Erweiterungen.

## Abhängigkeiten im Überblick

```
Phase A (parallel)                 Phase B          Phase C: Stufe 1 Beta                          Phase D
WP-05 Rahmen ─┬─ WP-06 Inhaltsprobe ──────────┐
              ├─ WP-01 Spike Standort/Akku ─┬─┤
              │                             ├─┼─ WP-04 Play-Vorprüfung ─┐
WP-02 Spike Audio ──────────────────────────┘ │                          │
WP-03 Spike TTS ──────────────────────────────┴──────────────────────────┴─► WP-07 Feldtest (Stufe 0) ─► Go?
                                                                                   │
WP-00 Repo-Zugang (jederzeit, nötig für WP-10)                                     ▼
WP-10 Gerüst ─┬─ WP-11 Content-Build ─┐
              ├─ WP-12 Messpunkt ─────┤
              ├─ WP-15 App-Kern ──────┤
              ├─ WP-16 App-Rahmen ────┼─► WP-21 Beta ─► Go? ─► Phase D (Stufe 2)
              ├─ WP-17 Karte ─────────┤
              ├─ WP-19 Betrieb ───────┤
WP-20 Inhalte ┴───────────────────────┤
WP-08 iOS nachholen (nach WP-07) ─────┘ (nur für iOS-Teilnehmende)
```

## Phase A: Risiken klären

### WP-00 Repository-Zugang und Bestandsabgleich
- **Ziel:** Planungsunterlagen ins Repo bringen; Annahme „leeres Repo“ bestätigen.
- **Bereich:** `docs/` · **Abhängig von:** Freigabe der Claude-GitHub-App für das Repo
- **Abnahme:** PR mit `docs/` gemergt; Branch-Schutz für `main` (Review, CI) und Secret-Scanning mit Push-Schutz aktiv; Abweichungen in `open-questions.md` eingetragen.
- **Tests:** keine · **Sicherheit:** Konten mit zweitem Faktor · **Risiken:** keine
- **Parallel:** ja · **Agent?** nein · **Aufwand:** S

### WP-05 Rahmen, Hypothesen, Entscheidungsregeln
- **Ziel:** Alles festlegen, woran spätere Entscheidungen gemessen werden, bevor Daten vorliegen.
- **Ergebnis:**
  - Q1–Q10 sind beantwortet ([decisions.md](../decisions.md)); Entscheidungsregeln stehen in [stages-and-hypotheses.md](../product/stages-and-hypotheses.md#entscheidungsregel).
  - Wettbewerbsrecherche: 5–8 Alternativen mit Stärken, Schwächen und Abgrenzung.
  - Erstberatung Datenschutz/Recht zu den für Stufe 1 markierten Fragen terminiert.
  - Verifikation der technischen Prüfpunkte, die Stufe 1 betreffen (Expo-SDK-Version, Google-Play-Policy, expo-audio-Fokus).
- **Abnahme:** Wettbewerbsabschnitt in `plan.md` ohne [A]-Markierungen; Termin für die Rechtsberatung steht; Prüfpunkte für Stufe 0/1 verifiziert.
- **Risiken:** Entscheidungen werden vertagt → Phase B startet ohne Messlatte.
- **Parallel:** ja · **Agent?** ja, für Wettbewerbs- und Anbieterrecherche · **Aufwand:** S

### WP-06 Inhaltsprobe (H-Content)
- **Ziel:** Feldtest-Bereich in Nienburg festlegen; Claude recherchiert und schreibt 30–60 Fakten mit belegbaren Quellen; Constantin prüft jeden Fakt; Prüfaufwand und Fehlerquote messen.
- **Bereich:** YAML-Dateien nach dem Schema in [data-model-and-sources.md](../data/data-model-and-sources.md); bis WP-00 erledigt ist, im Projektordner.
- **Abhängig von:** – (Q2 und Q5 sind beantwortet)
- **Ergebnis:** Fakten mit Quellen und Lizenz; Prüfzeit pro Fakt (`editor_minutes`, Constantin) und Zahl der in der Prüfung gefundenen Fehler; Zielwert X für H-Content nach den ersten zehn Fakten; Hochrechnung auf Kosten pro km² bei der angestrebten Dichte; kurze Checkliste „guter Fakt“ (Länge 45–90 s gesprochen, Ortsbezug sichtbar vor Ort, eine Überraschung, Quelle belegt).
- **Regel für KI-Recherche:** Jede Aussage braucht eine abrufbare Quelle (Stadtarchiv, Museum, Denkmalliste, Fachliteratur, seriöse Webquelle); was sich nicht belegen lässt, fliegt raus. Kein Text wird ohne Constantins Prüfung veröffentlicht.
- **Abnahme:** Median und Streuung der Prüfzeit aus mindestens 30 Fakten; Fehlerquote; Hochrechnung dokumentiert; Zielwert X in [decisions.md](../decisions.md) eingetragen.
- **Sicherheit:** Redaktions-Checkliste aus dem Sicherheitsdokument (Personenbezug, Privatgrund, Gefahr, Verkehr).
- **Parallel:** ja, mit WP-01 bis WP-03 · **Agent?** ja, Claude recherchiert · **Aufwand:** Constantins Prüfzeit ist das Messergebnis

### WP-01 Spike S1 und S3: Spaziergang mit gesperrtem Bildschirm, Akku
- **Ziel:** Nachweis, dass die Kernfunktion ohne Hintergrund-Standortberechtigung trägt. Der Build wird in WP-07 als Feldtest-App weiterverwendet.
- **Bereich:** `spikes/walk` (Expo Development Build, expo-location, expo-audio, Fakten aus WP-06 bzw. 20 Testorte eingebaut).
- **Abhängig von:** Testroute (aus WP-06)
- **Abnahme** (Messaufbau in [strategy.md](../testing/strategy.md) Abschnitt 5):
  - Geräte: Samsung S23 Ultra und Xiaomi 13T mit aktuellem Android ohne `ACCESS_BACKGROUND_LOCATION`. iOS folgt in WP-08.
  - Je Gerät drei Durchgänge über dieselbe Route in Nienburg mit 20 Orten (60 Auslösechancen je Gerät).
  - Je Gerät lösen ≥ 57 von 60 (95 %) bei gesperrtem Bildschirm aus, jeweils höchstens 30 m oder 15 s nach dem Referenz-Eintritt in den 50-m-Radius; keine Doppelauslösung.
  - Die Session überlebt 60 min mit gesperrtem Bildschirm auf beiden Geräten, auch mit den Standard-Akkueinstellungen von Samsung und Xiaomi.
  - Akku: Median ≤ 5 %/h (Ziel), ≤ 8 %/h (Grenze) auf beiden Geräten.
- **Tests:** Messprotokoll mit Rohdaten; keine automatischen Tests (Wegwerf-Code).
- **Datenschutz:** Spuren nur lokal auf Testgeräten, nach Auswertung gelöscht.
- **Risiken:** iOS-Verhalten mit „Beim Verwenden“ nur durch Foren belegt; Akkusparmodi der Hersteller. Bei Verfehlen: Transistorsoft evaluieren.
- **Parallel:** ja · **Agent?** nein (Gerätetest) · **Aufwand:** M, hohe Unsicherheit

### WP-02 Spike S2: Audio-Unterbrechungen
- **Ziel:** Prüfen, ob expo-audio das Soll-Verhalten aus [mobile-and-audio.md](../mobile/mobile-and-audio.md) Kap. 5 erfüllt; sonst Entscheidung für react-native-track-player v5 (kommerziell).
- **Bereich:** `spikes/walk` · **Abhängig von:** –
- **Abnahme:** Testfall-Matrix mit 7 Fällen (Anruf, Navigationsansage Google Maps, Navigationsansage zweite Navi-App (z. B. Komoot), Musik-App läuft, Kopfhörer abziehen, Bluetooth-Tasten, App-Wechsel) × 2 Android-Geräte: alle 14 Fälle verhalten sich wie im Soll; Lock-Screen-Steuerung sichtbar; Android-Wiedergabe > 3 min im Hintergrund.
- **Tests:** Video je Fall.
- **Risiken:** genaue `interruptionMode`-Werte nicht verifiziert.
- **Parallel:** ja · **Agent?** nein · **Aufwand:** S–M

### WP-03 Spike S4: TTS-Stimme, Qualität und Kosten
- **Ziel:** Anbieter und Stimme für [ADR-008](../architecture/decisions/ADR-008-pre-rendered-tts.md) vorwählen; die endgültige Wahl fällt im Feldtest.
- **Zusatz:** Constantin spricht 10 Fakten aus WP-06 selbst ein (Smartphone, ruhiger Raum) als Vergleich für den Feldtest.
- **Abnahme:** 20 Texte aus WP-06 (mit Eigennamen, Jahreszahlen, lateinischen Artnamen); mindestens drei Stimmen von mindestens zwei Anbietern, darunter ein Anbieter mit EU-Verarbeitung oder eine selbst gehostete Option; System-TTS als Referenz; Blindtest mit ≥ 5 Personen (Skala 1–5 für Verständlichkeit und Angenehmheit); die zwei besten Stimmen gehen in WP-07; Dateigröße ≤ 300 KB pro Minute; Kosten pro 1.000 Fakten und Lizenzbedingungen für kommerzielle Nutzung dokumentiert.
- **Sicherheit:** API-Schlüssel nur lokal.
- **Parallel:** ja · **Agent?** ja, für Anbieter- und Lizenzrecherche · **Aufwand:** S

### WP-04 Spike S5: Store-Vorprüfung
- **Ziel:** Frühe Rückmeldung von Google zu Hintergrund-Audio und Standort im Vordergrund-Service (Apple folgt in WP-08).
- **Abhängig von:** WP-01, WP-02
- **Ergebnis:** Datenarten-Matrix (Datenart, erhoben ja/nein, Zweck, Empfänger); Play-Track „Geschlossener Test“ (nicht „Interner Test“, der ohne Prüfung bleibt) mit ausgefülltem Standort-Formular, Data-Safety-Angaben, Review-Notiz und Demo-Video.
- **Abnahme:** Die Prüfung hat stattgefunden (Status in der Play Console) und endet ohne Ablehnung wegen Standort- oder Hintergrundnutzung; Store-Angaben entsprechen der Matrix.
- **Risiken:** Beta-Prüfungen sind weniger streng als die Produktion; Ergebnis ist ein Indiz.
- **Parallel:** nein · **Agent?** nein · **Aufwand:** S, unsichere Wartezeiten

## Phase B: Stufe 0

### WP-07 Feldtest (Nachfrage und Form)
- **Ziel:** H-Kern, A6 und die Form (Stimme, Länge, Dichte) mit echten Menschen beim Gehen prüfen.
- **Abhängig von:** WP-01, WP-02, WP-03, WP-04, WP-06
- **Aufbau:** 10–15 Personen mit Android-Handy, möglichst aus allen drei Gruppen (Einheimische, Touristen, Wanderer/Radler); App aus WP-01 mit 30–60 Fakten; Varianten: TTS gegen Constantins Aufnahme für 10 gleiche Fakten, kurz (~45 s) gegen lang (~120 s), hohe gegen niedrige Dichte auf zwei Abschnitten. Nur freies Gehen. Lokale Zählung in der App (gestartet, vollständig, übersprungen), gemeinsam im Interview abgelesen, nichts wird übertragen. Kurzinterview nach jedem Spaziergang, Abschlussfragebogen inklusive „Was hörst du sonst beim Spazierengehen?“.
- **Abnahme:** Auswertung je Variante; Entscheidung Go / Umsteuern / Stopp nach der in WP-05 festgelegten Regel; Festlegung von Stimme, Länge und Dichte für Stufe 1.
- **Datenschutz:** Einwilligung der Teilnehmenden, Interview-Notizen nach Rechtsfrage 13.
- **Parallel:** nein · **Agent?** ja, für die Auswertung der Notizen · **Aufwand:** M (Betreuung)

### WP-08 iOS nachholen
- **Ziel:** Spikes S1–S3 und die Store-Vorprüfung auf iOS, bevor die Beta auch iPhone-Nutzer einschließt.
- **Abhängig von:** WP-07 (Go), ein geliehenes oder gebrauchtes iPhone mit aktuellem iOS, Apple-Developer-Konto (99 USD pro Jahr)
- **Abnahme:** wie WP-01 und WP-02 auf dem iPhone mit Berechtigung „Beim Verwenden“; TestFlight-Build für externe Tester mit abgeschlossener Beta-Prüfung ohne Ablehnung.
- **Risiken:** Reicht „Beim Verwenden“ nicht, muss die Session-Architektur für iOS angepasst oder die Beta zunächst nur für Android geöffnet werden.
- **Parallel:** ja, mit Phase C · **Agent?** nein · **Aufwand:** M, hohe Unsicherheit

## Phase C: Stufe 1 Beta (nur nach Go aus WP-07)

### WP-10 Repository-Grundgerüst und CI
- **Ziel:** Gerüst für App, Content-Build und Messpunkt mit den CI-Gates aus [strategy.md](../testing/strategy.md) Abschnitt 3 „Pull Request“.
- **Bereich:** Wurzel, `apps/mobile/`, `tools/contentbuild/`, `backend/` (nur `platform`, `feedback`), `packages/schema/`, `testdata/geo/`, `infra/compose.yaml`, `.github/workflows/`, `CLAUDE.md`
- **Abhängig von:** WP-00, WP-07, Q1
- **Abnahme:** `docker compose up` startet Postgres und MinIO; ein PR durchläuft Lint, Typen, Tests, Schema-Prüfung, govulncheck, npm audit, gitleaks, Manifest-Prüfung und ist grün; ein absichtlich eingefügter Verstoß je Gate lässt die CI scheitern; Actions auf SHA; Renovate oder Dependabot aktiv.
- **Parallel:** nein (Grundlage) · **Agent?** nein · **Aufwand:** M

### WP-11 Content-Schema und Build
- **Ziel:** [ADR-012](../architecture/decisions/ADR-012-content-in-git.md) umsetzen.
- **Bereich:** `content/`, `tools/contentbuild/`, `packages/schema/`
- **Abhängig von:** WP-10, WP-03
- **Abnahme:** Prüfregeln mit Negativbeispielen getestet (fehlende Quelle, NC-Lizenz, `javascript:`-Link, Markup, Radius außerhalb 30–80 m, Koordinate außerhalb der Region); TTS nur für geänderte Texte (Cache-Treffer im Test); Regionspaket validiert gegen JSON Schema; `config.json` wird erst nach allen Uploads geschrieben (Test mit simuliertem Upload-Fehler: alte Version bleibt aktiv); Audios archivierter Fakten werden gelöscht; Bucket-Wurzel liefert 403.
- **Sicherheit:** TTS als Klartext; Build nur auf `main`; TTS-Schlüssel nur in der Environment.
- **Parallel:** ja, mit WP-12/15/16 · **Agent?** nein · **Aufwand:** M

### WP-12 Messpunkt
- **Ziel:** Ein Endpunkt `POST /v1/walk-summaries` für die Spaziergang-Zusammenfassung aus [stages-and-hypotheses.md](../product/stages-and-hypotheses.md).
- **Bereich:** `api/openapi.yaml`, `backend/internal/feedback`
- **Abhängig von:** WP-10
- **Abnahme:** Gespeichert werden nur Datum und Klassenfelder (Integrationstest gegen das Schema: keine IP-, Zeitstempel-, Fakt- oder Geräte-Spalten); Anfragen ohne Beta-Schlüssel oder mit unplausiblen Werten (z. B. mehr als 200 Fakten pro Spaziergang) werden abgelehnt; Rate-Limit am CDN (Startwert 30 Anfragen pro Minute und IP); Auswertungsskript liefert die Kennzahlen der Entscheidungsregel und unterdrückt Zellen mit < 5 Spaziergängen.
- **Parallel:** ja · **Agent?** nein · **Aufwand:** S

### WP-15 App-Kern: Spaziergang, Nähe-Logik, Player
- **Ziel:** [ADR-011](../architecture/decisions/ADR-011-app-core.md) umsetzen; Erkenntnisse aus WP-01/02 übernehmen, Code neu schreiben.
- **Bereich:** `apps/mobile/src/core`, `src/adapters`
- **Abhängig von:** WP-10, WP-07
- **Abnahme:** Golden-Fixtures (Fälle in strategy.md Abschnitt 1) liefern exakt die erwarteten Ereignisse; Regionspaket mit Audio wird vorab geladen, Größe vorher angezeigt; Mitschnitt eines 30-minütigen Spaziergangs zeigt keine inhaltsbezogenen Anfragen; Lock-Screen- und Kopfhörersteuerung; Auto-Ende nach 3 h oder 20 min Stillstand; Pause ab Radfahrgeschwindigkeit (Fixture und Feldprüfung auf dem Fahrrad); Echtgeräte-Checkliste erfüllt die Kriterien aus WP-01.
- **Datenschutz:** keine Positionsdaten in Logs, Crash-Berichten oder Anfragen.
- **Parallel:** ja, mit WP-16 nach Absprache der Ports · **Agent?** nein · **Aufwand:** L

### WP-16 App-Rahmen
- **Ziel:** Onboarding mit Erklärseite und Sicherheitshinweis, Frage nach der Gruppe, Einwilligung zur Messung (abschaltbar), Region herunterladen, Liste „Zuletzt gehört“ mit Transkript und Quelle, Auto-Play-Schalter, Meldelink (E-Mail mit Fakt-ID), Datenschutzerklärung, Impressum, Quellen- und Open-Source-Lizenzen.
- **Bereich:** `apps/mobile/src/ui`
- **Abhängig von:** WP-10, WP-11 (Paketformat)
- **Abnahme:** Kernpfad mit VoiceOver und TalkBack ohne Sicht bedienbar; Bedienelemente ≥ 44 pt / 48 dp; größte Schriftgröße ohne abgeschnittene Texte; App vollständig ohne Konto nutzbar; Links öffnen nur `https:`; Maestro-Kernpfad grün.
- **Parallel:** ja · **Agent?** nein · **Aufwand:** M

### WP-19 Betrieb Stufe 1
- **Ziel:** Auslieferung von `config.json` und Regionspaketen sowie Messpunkt auf dem eigenen Hetzner-Server über Coolify; Crash-Dienst; Konten-Härtung.
- **Abhängig von:** WP-12
- **Abnahme:** AV-Vertrag mit Hetzner abgeschlossen; Logs des Reverse-Proxys (Coolify) mit gekürzter IP oder aus, Frist ≤ 7 Tage (dokumentiert); Verzeichnis-Listing aus; harte Kostenbremse beim TTS-Anbieter; Bandbreite des Servers für 45 Downloads à bis zu 150 MB geprüft; Messpunkt mit täglichem Dump und einmal geprobtem Restore; Uptime-Check mit Alarm; Crash-Dienst in der EU ohne PII, ohne Session-Tracking, ohne HTTP-Breadcrumbs (Test mit absichtlichem Absturz und Sichtprüfung des Berichts); zweiter Faktor auf allen Betriebskonten.
- **Parallel:** ja · **Agent?** nein · **Aufwand:** S–M

### WP-17 Karte mit Offline-Kacheln
- **Ziel:** Einfache Karte mit den Fakten der Region; Kacheln für Nienburg und Umland als PMTiles-Datei im Regionspaket, damit beim Gehen keine Kachelanfragen entstehen.
- **Bereich:** `apps/mobile/src/ui/map`, `tools/contentbuild` (Kachel-Ausschnitt aus OSM-Daten, z. B. über Protomaps)
- **Abhängig von:** WP-11, WP-15; Spike S6 als erster Schritt
- **Abnahme:** S6: MapLibre React Native zeigt eine lokale PMTiles-Datei im Expo-Development-Build ohne manuelle Eingriffe; alle Fakten der Region als Cluster, ≥ 50 fps beim Schwenken auf dem Xiaomi 13T; „© OpenStreetMap-Mitwirkende“ sichtbar; Netzwerkmitschnitt zeigt keine Kachelanfragen; Kachelgröße für Nienburg dokumentiert.
- **Risiken:** Ob MapLibre RN lokale PMTiles unterstützt, ist nicht verifiziert [A]; Ausweg: MBTiles-Offline-Pack oder vorab geladene Kacheln. ODbL-Pflichten für die Kacheln prüfen (Produced Work).
- **Parallel:** ja · **Agent?** ja, für die Kachel-Erzeugung · **Aufwand:** M, mittlere Unsicherheit

### WP-20 Inhalte der Pilotregion
- **Ziel:** 150–300 Fakten für Nienburg und Umland (Zahl nach WP-06 und WP-07 festgelegt), recherchiert von Claude, geprüft von Constantin; auch Natur-Inhalte entlang der Weser für die Gruppe Wanderer/Radler.
- **Abhängig von:** WP-06, WP-07; ab WP-11 direkt im Repo.
- **Abnahme:** Auf drei vorab festgelegten Referenzrouten (je 3–5 km) liegt im Mittel ein Fakt pro 150–250 m Wegstrecke bzw. die in WP-07 ermittelte Dichte; alle Fakten bestehen den Build; Redaktions-Checkliste dokumentiert; Zeit pro Fakt fortlaufend erfasst.
- **Risiken:** größter Aufwandsblock, kein Technikthema.
- **Parallel:** ja · **Agent?** ja, zur Recherche mit menschlicher Prüfung · **Aufwand:** Redaktionszeit (Hochrechnung aus WP-06)

### WP-21 Beta und Auswertung
- **Abhängig von:** WP-11, WP-12, WP-15, WP-16, WP-17, WP-19, WP-20 (WP-08 für iOS-Teilnehmende)
- **Aufbau:** etwa 45 Teilnehmende, je etwa 15 Einheimische, Touristen und Wanderer/Radler; Android und, falls WP-08 bestanden, iOS; Laufzeit mindestens 4 Wochen; wöchentliches Kurz-Check-in; Abschlussfragebogen; mindestens 8 Interviews, je Gruppe mindestens zwei.
- **Abnahme:** Kennzahlen der Entscheidungsregel aus [stages-and-hypotheses.md](../product/stages-and-hypotheses.md#entscheidungsregel) ausgewertet; Entscheidung Go / Umsteuern / Stopp nach dieser Regel dokumentiert, Abweichungen begründet.
- **Aufwand:** M (Betreuung)

## Phase D: Stufe 2 (nur nach Go aus WP-21; Pakete werden dann verfeinert)

| WP | Titel | Abhängig von | Kernabnahme |
|---|---|---|---|
| WP-30 | Backend-Basis: PostGIS, Datenmodell Stufe 2, Import der YAML-Inhalte, `/v1/config`, Kachel-API | WP-21 | Import verlustfrei (Anzahl, Quellen, Lizenzen gleich); Golden-Fixtures für Kacheln grün; nur veröffentlichte Fassungen auslieferbar; p95 < 200 ms (Messaufbau strategy.md) |
| WP-31 | Veröffentlichungsablauf mit Jobs (ADR-005) | WP-30 | alte Fassung bleibt sichtbar, bis das Audio fertig ist (Integrationstest) |
| WP-32 | Redaktionswerkzeug (ADR-010), OIDC (ADR-009) | WP-30, Q4 | MFA über `amr`/`acr` geprüft; Autorisierungstests je Aktion; Audit-Log |
| WP-33 | Konten, Kontolöschung in App und Web | WP-32, Rechtsklärung | Löschung ≤ 30 Tage; Apple 5.1.1(v) und Google-Löschlink erfüllt |
| WP-34 | Beiträge, Vorab-Moderation, DSA-Meldeweg | WP-33 | Checkliste, Begründung an Einreichende, Tageslimit, Ausgabe escaped |
| WP-35 | GPX-Import auf dem Gerät, Modus „Route“ | WP-30 | Fixture-Satz (gültig, kaputt, sehr groß) korrekt; Route verlässt das Gerät nicht (Mitschnitt) |
| WP-36 | Wikidata-Kandidaten für die Redaktion | WP-32 | User-Agent mit Kontakt; Kandidaten nur über Prüfung veröffentlichbar |
| WP-37 | Betrieb und CI vollständig (strategy.md „Ab Stufe 2“) | WP-30 | Gates vor dem öffentlichen Start erfüllt |
| WP-38 | Rechtliche Freigaben, Store-Veröffentlichung | Fachperson | AGB, Datenschutzerklärung, Nutzungsrechte, Impressum freigegeben |

Später: Hintergrund-Auslösung ohne Session, OSM-Daten, Wetter, Artenbeobachtungen, Museen und Archive, KI-Zusammenfassungen, Bilderkennung, CarPlay und Android Auto, Mehrsprachigkeit.
