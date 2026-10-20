# Hypothesen, Stufen und Entscheidungsregel

Quelle für Hypothesen, MVP-Umfang je Stufe, Messung und Go/No-Go-Werte. Constantins Grundsatzentscheidungen stehen in [decisions.md](../decisions.md).

## Hypothesen
- **H-Kern:** Menschen starten bewusst einen Spaziergang mit Nebenbei und hören zu, ohne aufs Handy zu schauen. Feldtest-Go: mindestens die Hälfte will weitermachen.
- **H-Wiederkehr:** Einheimische nutzen die App in mehreren Wochen wieder.
- **H-Content:** Ein von Claude recherchierter Fakt lässt sich in höchstens X Minuten verlässlich prüfen; X wird nach den ersten zehn Fakten festgelegt.
- **H-Segment:** Welche der drei Gruppen (Einheimische, Touristen, Wanderer/Radler) den größten Nutzen hat, zeigt die getrennte Auswertung.
- **A6 (Stimme):** Eine gute synthetische Stimme wird akzeptiert. Geprüft im Feldtest gegen 10 von Constantin eingesprochene Fakten. Ziel: Die Mehrheit findet TTS mindestens „in Ordnung“.
- Nicht verfolgt: H-Differenz (Touren gegen freies Gehen, Touren bleiben spätere Option) und H-Revenue (nicht kommerziell).

## Stufe 0: Feldtest (Nachfrage und Form)
- Die Spike-App aus WP-01/02 mit 30–60 eingebauten Fakten für einen Bereich in Nienburg, nur Android (geschlossener Play-Test).
- 10–15 Personen, möglichst aus allen drei Gruppen.
- Bewusst variiert: TTS gegen Constantins Aufnahme (10 gleiche Fakten), kurz (~45 s) gegen lang (~120 s), hohe gegen niedrige Dichte. Nur freies Gehen.
- Messung: Interviews nach jedem Spaziergang, kurzer Fragebogen, lokale Zählung in der App (wird im Interview gemeinsam abgelesen, nicht übertragen).
- Ergebnis: Go/No-Go nach der Regel unten und Festlegung von Stimme, Länge und Dichte für Stufe 1.

## Stufe 1: Beta (Wiederkehr, Inhaltskosten, Zielgruppe)

### Muss
- Hören ohne Konto, ohne Tracking, ohne Geräte-ID
- Region herunterladen (Größe wird angezeigt), danach offline nutzbar
- Spaziergang starten und beenden; automatische Auslösung bei Annäherung, auch bei gesperrtem Bildschirm, solange die Session läuft; sichtbare Benachrichtigung mit „Beenden“
- Lock-Screen- und Kopfhörersteuerung (Pause, Überspringen, Wiederholen)
- Liste „Zuletzt gehört“ mit manuellem Abspielen und Transkript inklusive Quelle
- Auto-Play an/aus; Auto-Play pausiert ab Fahrradgeschwindigkeit
- Onboarding mit Erklärung vor der Standortabfrage; Sicherheitshinweis; Datenschutzerklärung; Impressum; Seite mit Quellen- und Open-Source-Lizenzen
- Fehler melden per Link (E-Mail mit Fakt-ID)
- Anonyme Spaziergang-Zusammenfassung an einen kleinen Messpunkt (ohne Fakt-IDs, ohne Standort, ohne Kennung), mit Einwilligung im Onboarding und abschaltbar
- Einfache Karte mit den Fakten der Region; Kartenkacheln liegen offline im Regionspaket
- Onboarding fragt einmal nach der Gruppe (Einheimisch, zu Besuch, Wandern/Rad)
- Inhalte: Git-versionierte Dateien, Build-Skript prüft Quellen und Lizenzen, erzeugt TTS und ein Regionspaket, ausgeliefert vom eigenen Server

### Nach der Beta, falls die Interviews es verlangen
 Themenfilter, Liste „In der Nähe“, mehrere Akkustufen.

### Messung in der Beta
- Die App zählt lokal und sendet pro Spaziergang eine Zusammenfassung mit Klassen: laufende Nummer des Spaziergangs (1/2/3–5/6+), Wochen seit dem ersten Spaziergang (0/1/2/3+), Dauer (Klassen), Zahl ausgelöster, vollständig gehörter und übersprungener Fakten, Auto-Play an/aus, Gruppe. Gesendet wird verzögert beim nächsten App-Start.
- Wöchentliches Kurz-Check-in (2 Fragen) und Abschlussfragebogen mit Sean-Ellis-Frage („Wie enttäuscht wärst du, wenn es Nebenbei nicht mehr gäbe?“) und Frage nach Weiterempfehlung; mindestens 8 Interviews, aus jeder Gruppe mindestens zwei.
- Claude protokolliert die Recherchezeit, Constantin seine Prüfzeit pro Fakt (H-Content); Fehler, die erst in der Prüfung auffallen, werden gezählt.

## Entscheidungsregel
Festgelegt am 20.10.2026, bewusst locker.

| Kennzahl | Go | Umsteuern | Stopp |
|---|---|---|---|
| Anteil Einheimischer mit Spaziergängen in ≥ 2 verschiedenen Wochen | ≥ 30 % | 15–29 % | < 15 % |
| Sean-Ellis „sehr enttäuscht“ (alle Gruppen) | ≥ 30 % | 20–29 % | < 20 % |
| Skip-Rate ausgelöster Fakten | ≤ 30 % | 31–55 % | > 55 % |
| Prüfzeit pro Fakt (Constantin) | ≤ Zielwert H-Content | bis 2 × Ziel | > 2 × Ziel |
Bei Touristen und Wanderern zählen statt Wiederkehr Sean-Ellis-Wert und Weiterempfehlung. Mit etwa 15 Personen je Gruppe ist jede Quote auf etwa ±20 Prozentpunkte unsicher; die Interviews entscheiden mit.

## Stufe 2: öffentlicher Start
Go-Backend mit PostGIS und Kachel-API für mehrere Regionen, Redaktionswerkzeug, Konten (Magic Link, Passkey), Kontolöschung, Beiträge mit Vorab-Moderation und DSA-Meldeweg, GPX-Import auf dem Gerät mit Modus „Route“, Wikidata als Kandidatenquelle, vollständige CI-Härtung, rechtliche Freigaben, Store-Veröffentlichung. Auslöser: positives Beta-Ergebnis und eines von: Community-Beiträge gewollt, mehr als etwa 1.000 Fakten, mehr als drei Redakteure, mehr als eine Region.

## Ausdrücklich nicht im MVP (Stufen 0–1)
Konten, Community-Beiträge, Backend-Datenbank für Inhalte, Redaktionswerkzeug, Touren, Hintergrund-Auslösung ohne Session, OSM-Daten, Wikipedia-Texte, Wetter-, Arten-, Museums- und Archivdaten, Bilderkennung, automatische KI-Zusammenfassungen in der App (KI-gestützte Recherche durch Claude mit menschlicher Prüfung ist dagegen der Redaktionsweg), Empfehlungssystem, Social Login, CarPlay und Android Auto, Mehrsprachigkeit, Over-the-Air-Updates, Monetarisierung, iOS im Feldtest.
