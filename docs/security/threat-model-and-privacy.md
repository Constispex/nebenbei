# Bedrohungsmodell, Sicherheit und Datenschutz

Version 2 (nach Review). Technische Einordnung, **keine Rechtsprüfung**. Abschnitt 5 muss eine Fachperson für Datenschutz und IT-Recht klären: Frage 13 vor dem Feldtest (Stufe 0), die übrigen für Stufe 1 markierten Punkte vor der Beta, den Rest vor dem öffentlichen Start.

## 1. Leitprinzipien
1. **Der genaue Standort verlässt das Gerät nicht**, auch nicht indirekt über Ladezeitpunkte ([ADR-007](../architecture/decisions/ADR-007-location-on-device.md)).
2. **Kein Tracking:** keine Werbe- oder Analyse-SDKs, keine Geräte- oder Installations-ID, auch nicht versteckt in Bibliotheken.
3. **Kein eigener Passwort-Stack** ([ADR-009](../architecture/decisions/ADR-009-external-auth.md)).
4. **Fremde Inhalte gehen immer durch eine Prüfung**, bevor sie veröffentlicht werden.
5. **Sichere Defaults:** verboten ist, was nicht ausdrücklich erlaubt ist.

**Ehrliche Formulierung für Nutzer und Stores:** Nicht „wir verarbeiten keinen Standort“, sondern: „Dein Standort wird nur auf deinem Gerät ausgewertet. Unser Server erfährt nur, welche Region du herunterlädst. Wie bei jeder Internetverbindung sehen unsere Dienstleister deine IP-Adresse, die wir nur gekürzt und kurz speichern lassen.“

## 2. Angriffsflächen
Stufe 1: App (Bundle, lokaler Speicher), CDN und Objektspeicher, `config.json`, Messpunkt, Content-Repository und Build-Pipeline, TTS-Anbieter, CI/CD, Abhängigkeiten, Expo/EAS-Konto.
Stufe 2 zusätzlich: API, Redaktionswerkzeug, Datenbank, Konten, Beiträge, Meldungen, GPX-Dateien, externe Datenquellen.

## 3. Priorisiertes Bedrohungsmodell (STRIDE-light)

| # | Bedrohung | Komponente | Auswirkung | Wahrsch. | Maßnahme | Stufe |
|---|---|---|---|---|---|---|
| 1 | Positionsspur durch Ladezeitpunkte: Audio oder Details erst beim Auslösen laden; Kacheln beim Gehen nachladen; Startbereich verrät Wohnort | App, CDN | hoch | hoch | Ganzes Regionspaket mit Audio vorab laden; während des Spaziergangs keine inhaltsbezogenen Anfragen; Netzwerkmitschnitt als Abnahme | **1** |
| 2 | Logs (Reverse-Proxy in Coolify, Hoster, später CDN) speichern volle IP mit Pfad | Hosting | mittel | hoch | Zugriffslogs aus oder IP gekürzt, Frist ≤ 7 Tage; AV-Vertrag mit Hetzner; ein späteres CDN nur mit EU-Standort und derselben Regel | **1** |
| 3 | Versteckte Kennungen: Session-Tracking im Crash-SDK, Client-ID von `expo-updates`, Breadcrumbs mit URLs | App | mittel | hoch | Kein `expo-updates` in Stufe 1; Crash-SDK ohne Session-Tracking, ohne HTTP-Breadcrumbs, `sendDefaultPii=false`, EU-Region; SDK-Liste pro Release geprüft; im Crash-Dienst IP-Speicherung aus und Data Scrubbing an | **1** |
| 4 | Übernahme von Expo/EAS-Konto oder CI-Token → bösartiger Build oder Update mit Standortberechtigung | Lieferkette | hoch | niedrig–mittel | Kein OTA in Stufe 1; Expo-, GitHub- und Store-Konten mit Passkey oder zweitem Faktor; EAS-Robot-Token nur mit Build- und Submit-Rechten, nur in geschützter Environment mit Freigabe; App-Netzwerkzugriffe nur zu fester Domain-Liste | **1** |
| 5 | Lieferkette (kompromittiertes npm- oder Go-Paket, Action) | CI, App | hoch | mittel | Lockfiles; Renovate oder Dependabot mit 7 Tagen Wartezeit; govulncheck; `npm audit`; Actions auf Commit-SHA; minimale `permissions`; Secret-Scanning mit Push-Schutz | **1** |
| 6 | Manipulierte Inhalte über das Content-Repository (fremder Pull Request, gekapertes Konto) | Content-Build | hoch | niedrig | Branch-Schutz mit Pflicht-Review; nur Mitglieder dürfen pushen; Build läuft nur auf `main`; Konten mit zweitem Faktor | **1** |
| 7 | Injektion über Inhalte: `javascript:`/`intent:`-Links in Quellen, SSML-Tags im TTS-Text, Markup | Build, App | mittel | mittel | Build erlaubt nur `https:`-URLs und Klartext; TTS als Klartext ohne SSML; App öffnet nur `https:`-Links | **1** |
| 8 | Unveröffentlichte oder archivierte Inhalte bleiben abrufbar; Bucket-Listung; Hotlinking-Kosten | Objektspeicher, CDN | mittel | mittel | Nur veröffentlichte Audios im öffentlichen Bucket; Listung aus; Löschung archivierter Audios beim Build; harte Budgetgrenze bzw. Abschaltung beim CDN, nicht nur Alarm | **1** |
| 9 | Gefälschte oder massenhafte Messdaten verfälschen die Go/No-Go-Entscheidung | Messpunkt | mittel | mittel | Beta-Schlüssel im Build (schwach, reicht für geschlossene Beta); Plausibilitätsgrenzen; Rate-Limit am CDN; Auswertung mit Mindestschwelle und Abgleich mit Fragebögen | **1** |
| 10 | Re-Identifikation über Messdaten (Liste passierter Fakten, kleine Zellen) | Messpunkt | mittel | mittel | Keine Fakt-IDs, nur Klassen; nur Datum, kein Zeitstempel; keine IP gespeichert; Auswertung nur für Zellen mit ≥ 5 Spaziergängen | **1** |
| 11 | Inhalt lockt an gefährliche oder private Orte, betrifft Personen, Auslösebereich an Kreuzungen | Inhalte | hoch | mittel | Redaktions-Checkliste (Personenbezug, Privatgrund, Gefahr, Schutzgebiet, Verkehr, Quelle, Urheberrecht); Auslösebereiche nicht auf Straßenquerungen; Meldelink | **1** |
| 12 | Ablenkung im Straßenverkehr | App | hoch | mittel | Auto-Play pausiert ab Radfahrgeschwindigkeit (ADR-011); keine visuellen Aufforderungen während des Spaziergangs; Sicherheitshinweis im Onboarding; Session endet nach 3 h oder 20 min Stillstand | **1** |
| 13 | Übernahme eines Redaktions- oder Admin-Kontos | Redaktionswerkzeug | hoch | mittel | OIDC mit PKCE, `state`, `nonce`; MFA über `amr`/`acr` geprüft; Session-Neuvergabe; `__Host-`-Cookie; CSRF; CSP; Audit-Log | 2 |
| 14 | Fehlende Objektautorisierung | API | hoch | mittel | Zentrale Policy-Schicht, Tests pro Endpunkt (OWASP API1:2023), UUIDs | 2 |
| 15 | Spam, Falschinformation, XSS über Beiträge und Meldungen | Beiträge, Redaktion | mittel | hoch | Konto nötig, Tageslimit, Quelle Pflicht, Vorab-Moderation, Ausgabe escaped | 2 |
| 16 | Keine Nachvollziehbarkeit von Moderationsentscheidungen (DSA) | Moderation | mittel | mittel | Audit-Log mit Begründung und Rechtsbehelf | 2 |
| 17 | GPX/XML-Angriffe | Import | mittel | mittel | GPX auf dem Gerät lesen; Größen- und Punktlimits | 2 |
| 18 | Manipulierte externe Daten (Wikidata-Vandalismus) | Importe | mittel | mittel | Nur als Kandidat, Quell-Revision speichern, Markup entfernen | 2 |
| 19 | Rate-Limit am Ursprung wirkungslos hinter dem CDN | API | mittel | hoch | Rate-Limit am CDN/WAF; am Ursprung nur die vom CDN gesetzte Client-IP-Kopfzeile vertrauen, Ursprung nur für CDN erreichbar | 2 |

## 4. Pflicht für Stufe 1 (je Kriterium mit Prüfmethode)

| Kriterium | Prüfmethode |
|---|---|
| Während eines Spaziergangs keine Netzwerkanfragen mit Inhaltsbezug | Mitschnitt mit einem Proxy (z. B. mitmproxy) über einen 30-minütigen Testspaziergang auf iOS und Android; erlaubt sind nur der Messpunkt (beim App-Start) und der Crash-Dienst |
| App kontaktiert nur Domains aus der Liste (CDN, Messpunkt, Crash-Dienst) | derselbe Mitschnitt, dazu App-Start und Region-Download |
| Keine Bibliothek mit eigener Kennung oder Netzwerkzugriff außerhalb der Liste | SDK-Liste pro Release: alle direkten Abhängigkeiten mit nativen Modulen oder Netzwerkzugriff sind geprüft und begründet; `expo-updates` nicht enthalten |
| Standortberechtigung nur „Beim Verwenden“, kein `ACCESS_BACKGROUND_LOCATION` | Prüfung der erzeugten `AndroidManifest.xml` und `Info.plist` in CI |
| Erklärseite vor der Standortabfrage | Maestro-Flow des Onboardings |
| Messpunkt speichert keine IP, keinen Zeitstempel feiner als ein Tag, keine Fakt-ID | Integrationstest gegen die Datenbank; Code-Review des Schemas |
| CDN- und Hoster-Logs: IP gekürzt oder aus, Frist ≤ 7 Tage | Einstellung dokumentiert (Screenshot oder Vertrag) |
| Build lehnt Fakten ohne Quelle, mit gesperrter Lizenz, nicht-`https`-Links oder Markup ab | Unit-Tests mit Negativbeispielen |
| Öffentlicher Bucket ohne Listung, nur veröffentlichte Audios | Anfrage auf Bucket-Wurzel liefert 403; Build-Test |
| Harte Kostenbremse bei CDN und TTS | Einstellung dokumentiert |
| Secrets nur in Environments; gitleaks in CI; Secret-Scanning mit Push-Schutz | CI-Lauf, Repository-Einstellung |
| Actions auf Commit-SHA, Standard `permissions: contents: read` | CI-Prüfung (z. B. GitHub-Policy „SHA-Pinning erzwingen“) |
| Keine Funde von govulncheck (Go-Code, nur erreichbare Schwachstellen) und keine High/Critical-Funde von `npm audit --omit=dev` | CI-Gate; Ausnahmen nur mit dokumentierter Begründung und Ablaufdatum |
| Konten bei GitHub, Expo, Apple, Google, Hoster, CDN mit Passkey oder zweitem Faktor | Checkliste vor Beta-Start |
| Store-Angaben stimmen mit der Datenarten-Matrix überein | Matrix (Datenart → erhoben ja/nein → Zweck → Empfänger) in WP-04; Formulare dagegen abgeglichen |

## 5. Offene Rechts- und Datenschutzfragen (Fachperson)
Für Stufe 1 relevant: 1, 2, 3, 4, 7, 8, 11, 12, 13.

1. **Rechtsgrundlage der Standortverarbeitung auf dem Gerät**: Art. 6 Abs. 1 lit. b oder lit. a DSGVO? Greift § 25 TDDDG, und gilt die Ausnahme „unbedingt erforderlich“?
2. **Datenschutz-Folgenabschätzung (Art. 35)**: nötig, wenn der Standort nur lokal verarbeitet wird? Empfehlung: freiwillige Kurz-DSFA.
3. **Auftragsverarbeitung und Drittlandtransfer**: Hosting, CDN, Objektspeicher, Crash-Dienst, Expo/EAS (Build), Apple/Google (Beta-Verteilung), TTS, ab Stufe 2 Auth.
4. **TTS-Anbieter**: Nutzungsrechte am erzeugten Audio für kommerzielle Verbreitung; Kennzeichnung synthetischer Stimmen.
5. **Nutzungsrechte an Beiträgen** (Stufe 2): Lizenzmodell, Vertonung, Bearbeitung, Verbleib nach Kontolöschung, Namensnennung.
6. **DSA** (Stufe 2): Melde- und Abhilfeverfahren (Art. 16), Begründung (Art. 17), Einstufung als Online-Plattform, Ausnahme für Klein- und Kleinstunternehmen (Art. 19, [dejure](https://dejure.org/gesetze/DSA/19.html)), Kontaktstellen (Art. 11/12), AGB (Art. 14).
7. **Haftung für Ortsinhalte**: Verkehrssicherung, Betretungsrecht, Naturschutz, Persönlichkeitsrechte (Wohnhäuser, Gedenkorte).
8. **Impressum** nach § 5 DDG in App und Store; Datenschutzerklärung in der App.
9. **Lizenzen externer Daten** (Stufe 2): ODbL, CC BY-SA auch für vorgelesene Bearbeitungen, Attribution im Audio.
10. **Mindestalter** für Beitragskonten (Art. 8 DSGVO, 16 Jahre in Deutschland).
11. **KI-Kennzeichnung** nach Art. 50 AI Act: Die Texte werden von Claude recherchiert und geschrieben und von einem Menschen geprüft und verantwortet; gilt dafür eine Kennzeichnungspflicht oder die Ausnahme für redaktionell geprüfte Inhalte? Dazu die synthetische Stimme.
12. **Messpunkt in der Beta**: Reicht die anonyme Klassen-Zusammenfassung ohne Einwilligung, oder ist eine Einwilligung (§ 25 TDDDG) nötig? Vorsorglich: Einwilligung im Beta-Onboarding, abschaltbar.
13. **Beta-Teilnehmende**: Rechtsgrundlage und Löschung für Tester-E-Mails, Fragebögen und Interview-Notizen bzw. -Aufnahmen.

## 6. Lösch- und Aufbewahrungsfristen (Vorschlag)

| Daten | Wo | Frist |
|---|---|---|
| IP-Adressen in Zugriffslogs | CDN, Hoster, Load-Balancer | aus oder gekürzt; vollständige IP höchstens 7 Tage, vertraglich festgehalten |
| Anwendungslogs | eigener Dienst | 30 Tage, ohne IP |
| Spaziergang-Zusammenfassungen | Messpunkt | bis Ende der Auswertung von Stufe 1, höchstens 12 Monate |
| Crash-Berichte | Crash-Dienst | 30 Tage (falls einstellbar, sonst Anbieter-Standard dokumentieren) |
| Beta-Teilnehmende (E-Mail, Fragebögen, Interviews) | Projektablage | Löschung 3 Monate nach Ende der Beta; Aufnahmen nach Transkription |
| Fehlermeldungen per E-Mail | Postfach | 6 Monate nach Erledigung |
| Build-, Expo-, Store-Daten | Anbieter | Anbieter-Standard dokumentieren |
| Audit-Log Redaktion/Moderation | Stufe 2 | 3 Jahre [A, rechtlich prüfen] |
| Konten | Stufe 2 | Löschung ≤ 30 Tage nach Antrag; Backups laufen nach ≤ 35 Tagen aus |
| Backups | Stufe 2 | PITR 7 Tage, tägliche Dumps 30 Tage |
