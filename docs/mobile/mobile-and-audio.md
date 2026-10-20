# Mobile-App, Standort, Audio und UX

Legende: **[F]** belegt, **[A]** Annahme oder nicht abschließend verifiziert, **[E]** Empfehlung. docs.expo.dev, transistorsoft.com und support.google.com waren aus der Rechercheumgebung nicht direkt abrufbar; Aussagen dazu stammen aus Suchergebnis-Auszügen und werden in den Spikes verifiziert.

Die Code-Struktur der App beschreibt [ADR-011](../architecture/decisions/ADR-011-app-core.md), das Laden der Inhalte [ADR-007](../architecture/decisions/ADR-007-location-on-device.md).

## 1. Kernempfehlung in einem Satz

**[E]** Expo (Managed Workflow mit Development Builds, CNG/Prebuild, EAS) + **expo-location** (Vordergrund-Updates in einer bewusst gestarteten „Spaziergang“-Session, Nähe-Erkennung im App-Code statt OS-Geofences) + **expo-audio** (Hintergrundwiedergabe, Lock-Screen-Steuerung) + **vorproduzierte TTS-Audiodateien**, die mit dem Regionspaket vorab geladen werden. Eine Karte (MapLibre React Native) ist erst nach der Beta vorgesehen und nur, wenn die Interviews sie verlangen. Ein Hintergrundstandort im Sinne von „Always“ bzw. `ACCESS_BACKGROUND_LOCATION` wird im MVP **vermieden**.

## 2. Mobile-Architektur und Bibliotheken

### Expo vs. bare React Native
- **[F]** React Native 0.82 (Okt. 2025) läuft nur noch mit der New Architecture; `newArchEnabled=false` wird ignoriert. Expo SDK 54 war das letzte SDK mit der Legacy Architecture.
- **[F]** Expo SDK 57 (30.06.2026) bringt React Native 0.86. SDK 58 (React Native 0.88, iOS 27) war im September 2026 als Beta angekündigt; ob die stabile Version schon erschienen ist, widersprechen sich die Quellen.
- **[E] Expo.** Der frühere Hauptvorteil von „bare“ (eigene native Module) entfällt: Development Builds und Config Plugins erlauben jede native Bibliothek, und mit CNG/Prebuild lassen sich die nativen Ordner jederzeit neu erzeugen. EAS Build/Submit/Update spart einem Team aus 1–2 Personen die Pflege eigener CI-Signierung. Alternative: bare RN mit Fastlane und GitHub Actions, das bringt mehr Pflege und keinen belegbaren Vorteil. Risiko: Abhängigkeit von EAS-Kontingenten. `eas build --local` bzw. eigene Runner sind als Ausweg vorhanden. Kein Wechselbedarf gegenüber dem Vorschlag „React Native + TS“, Expo ist eine Ausprägung davon.
- **[F]** Expo Go reicht nicht: MapLibre ist nicht in Expo Go nutzbar, und Hintergrund-Audio und -Standort brauchen native Konfiguration. Deshalb ab Tag 1 Development Builds.

### Standort
| Option | Bewertung |
|---|---|
| **expo-location + expo-task-manager** [E] | Kostenlos, im SDK gepflegt. `watchPositionAsync` bzw. `startLocationUpdatesAsync` mit Android-Foreground-Service-Option. **[A]** Laut Suchergebnissen klappen Hintergrund-Geofences per `startGeofencingAsync` weniger zuverlässig als Location-Updates. Für das MVP wird das nicht gebraucht. |
| Transistorsoft react-native-background-geolocation | Ausgereift (Bewegungserkennung, Akku-Optimierung, Expo-Plugin). **[F]** Eine Lizenz ist nur für Android-Release-Builds nötig, v4-Keys gelten nicht für v5. **[A]** Preis laut Reseller ab ca. 399 USD einmalig, offizieller Shop nicht abrufbar. Lohnt sich erst, wenn Auto-Play ohne aktive Session (echter Hintergrund) kommen soll. |
| OS-Geofencing | **[F]** Android erlaubt höchstens 100 Geofences pro App und Nutzer. **[A]** iOS erlaubt höchstens 20 überwachte Regionen pro App. Das ist breit zitiert, die offizielle Seite war aber nicht lesbar. Android empfiehlt mindestens 100–150 m Radius, typische Latenz < 2 min, im Hintergrund 2–3 min, bei Stillstand bis ca. 6 min. Für dichte POI-Felder (alle 50 m ein Fakt) ist das **ungeeignet**. Erst später sinnvoll, als „Aufwecker“ für große Regionen. |

**[E] Nähe-Logik in der App:** Das Regionspaket mit allen Fakten und Audios liegt vor dem Spaziergang auf dem Gerät. Bei jedem Positions-Update prüft der App-Kern lokal die Distanz zu Punktorten (Haversine gegen Radius, mit Hysterese, Mindestverweildauer und „schon gehört“; Parameter in ADR-011). In Stufe 1 gibt es nur Punktorte, damit Build und App dieselbe Regel anwenden. Kein Regionen-Limit, keine Netzwerkanfragen unterwegs.

### Audio
- **[F] expo-audio:** Das Config-Plugin-Flag `enableBackgroundPlayback` setzt unter iOS den Background Mode `audio` und deklariert unter Android `FOREGROUND_SERVICE_MEDIA_PLAYBACK` sowie einen Media-Playback-Service. Zur Laufzeit kommen `setAudioModeAsync({ shouldPlayInBackground, playsInSilentMode, interruptionMode })` und `player.setActiveForLockScreen(true, metadata)` dazu. **Ohne aktive Lock-Screen-Steuerung beendet Android die Wiedergabe nach ca. 3 Minuten im Hintergrund.**
- **[F] react-native-track-player:** v5 (`@rntp/player`) ist ein New-Architecture-Rewrite unter **kommerzieller Lizenz** (Pro 99 €/Monat bzw. 999 €/Jahr pro App). v4 ist eingefroren (Apache-2.0, keine Updates mehr).
- **[E] expo-audio** mit eigener Warteschlange in JS (Zustandsautomat, ein Player). RNTP v4 nicht verwenden, weil es nicht mehr gepflegt wird. RNTP v5 kommt nur infrage, wenn der PoC zeigt, dass expo-audio bei Audio-Focus, Kopfhörer- oder Bluetooth-Tasten nicht reicht. Dann wären rund 1.000 €/Jahr einzuplanen.
- **[A]** Die genauen `interruptionMode`-Werte (Ducking anderer Apps) von expo-audio ließen sich nicht in der offiziellen Doku verifizieren. Das ist Teil von Spike S2.

### TTS
- **[F]** Es gibt einen Apple-Forum-Bericht, nach dem expo-speech (AVSpeechSynthesizer) beim Sperren des Bildschirms pausiert, obwohl `UIBackgroundModes` audio und location gesetzt sind und eine aktive Audio-Session läuft. System-TTS hat außerdem keine Lock-Screen-Steuerung, und die Stimmqualität hängt vom Gerät ab.
- **[F]** Google Cloud TTS kostet pro 1 Mio. Zeichen: Standard/WaveNet 4 USD, Neural2 16 USD, Chirp 3 HD 30 USD, jeweils mit Freikontingent.
- **[E] Serverseitig vorproduzierte TTS-Dateien** (AAC, Cache pro Text und Stimme, siehe ADR-008). Kostenbeispiel [A]: 10.000 Fakten × 600 Zeichen = 6 Mio. Zeichen, mit Chirp 3 HD also ca. 180 USD einmalig. Vorteile: einheitliche, hochwertige Stimme, derselbe Abspielweg wie für spätere echte Sprecheraufnahmen, Offline-Paketierung möglich, keine Hintergrundprobleme der System-TTS. expo-speech dient nur als Fallback im Vordergrund (z. B. für frische Community-Beiträge vor der Vertonung).

### Karten
- **[F]** MapLibre RN (ab v10) hat ein Expo-Plugin und läuft über eine Kompatibilitätsschicht auf der New Architecture, ist aber noch kein nativer Rewrite. Es läuft nicht in Expo Go.
- **[E] MapLibre** mit OSM-basierten Vektorkacheln (selbst gehostet oder EU-Anbieter): ohne Google-API-Key, mit eigenem Stil und späteren Offline-Regionen. react-native-maps (Apple/Google Maps) bleibt die Alternative mit weniger Setup-Aufwand, bringt aber Google-Abhängigkeit und Kosten auf Android und keine Offline-Kacheln. Risiko: Mit der Kompatibilitätsschicht sind Performance-Einbußen möglich, deshalb Teil des PoC.

## 3. Hintergrundstandort im MVP vermeiden: „Spaziergang“-Modus

**[E]** Die Nutzerin tippt auf „Spaziergang starten“, und zwar bei sichtbarer App. Danach gilt:
- **iOS:** Berechtigung „Beim Verwenden der App“, `UIBackgroundModes` = `location` und `audio`, `allowsBackgroundLocationUpdates` (expo-location setzt das intern) und der blaue Statusleisten-Indikator. **[F]** Apple: Für Hintergrund-Updates ist der Background Mode `location` nötig. **[A]** Dass „When In Use“ für Updates genügt, die im Vordergrund gestartet wurden (CLBackgroundActivitySession), belegen nur Forum- und Drittquellen. Das ist in S1 zu prüfen.
- **Android:** Foreground Service vom Typ `location` (plus `mediaPlayback` für die Wiedergabe) wird beim Tippen gestartet, mit dauerhafter Benachrichtigung „Spaziergang aktiv – Beenden“. **[F]** Die Berechtigung `FOREGROUND_SERVICE_LOCATION` und FINE/COARSE genügen. Ohne `ACCESS_BACKGROUND_LOCATION` darf der Service nur **nicht aus dem Hintergrund** gestartet werden.
- **Play-Policy [F/A]:** Google bevorzugt Standortzugriff im Vordergrund. Ein Foreground Service ist zulässig, wenn er durch eine Nutzeraktion gestartet und nach deren Ende beendet wird. Wird die Nutzung aber als „gleichwertig zu Hintergrundzugriff“ eingestuft, gelten die Hintergrundregeln mit Declaration und Video. Daher: Session-Ende automatisch (z. B. nach 3 h oder 20 min Stillstand) und explizit, keine Wiederaufnahme nach Neustart, klare Offenlegung im Play-Formular.
- **Auto-Play nur in aktiver Session.** Ein echter Hintergrund-Trigger („erinnere mich, wenn ich in der Nähe bin“) ist eine spätere Erweiterung mit „Always“, Declaration und eventuell Transistorsoft.

## 4. Plattformrisiken

| Risiko | Plattform | Wahrsch. | Wirkung | Gegenmaßnahme |
|---|---|---|---|---|
| App Review lehnt nach 2.5.4 ab (Hintergrund-Audio/-Standort „nicht nachvollziehbar“, es gibt dokumentierte Fälle mit TTS-Apps) | iOS | mittel | hoch | Review-Notiz mit Demo-Video und Testroute, Hintergrundmodi nur in der Session, Akkuhinweis |
| Play stuft den FGS als Hintergrundstandort ein, dann sind Declaration und Video nötig | Android | mittel | mittel | Session-Begrenzung, Video vorsorglich drehen, kein `ACCESS_BACKGROUND_LOCATION` |
| OEM-Akkusparmodi (Samsung, Xiaomi) beenden den FGS, dazu Doze | Android | mittel | hoch | Test auf Samsung-Gerät, Hinweis auf dontkillmyapp, Watchdog/Neustart-Hinweis |
| Wiedergabe stoppt im Hintergrund nach ca. 3 min ohne Lock-Screen-Steuerung | Android | hoch, wenn vergessen | hoch | `setActiveForLockScreen` während der gesamten Session |
| System-TTS pausiert bei Sperrbildschirm | iOS | hoch | hoch | Vorproduzierte Audiodateien |
| Positionsgenauigkeit im Wald oder in Altstadtgassen > 50 m | beide | hoch | mittel | Trigger-Radius 30–80 m je Fakt, Hysterese, Geschwindigkeitsfilter |
| Kollision mit Navigations- oder Musik-Apps (Audio Focus) | beide | hoch | mittel | Bei Navigationsansage pausieren statt mischen; Musik während eines Fakts pausieren lassen (siehe Kap. 5) |
| Lizenzkosten von RNTP v5 bzw. Transistorsoft | beide | niedrig | niedrig | Nur bei nachgewiesenem Bedarf |

## 5. Audio-Verhalten (Soll)
- **Anruf oder Siri:** pausieren, danach **nicht** automatisch fortsetzen, sondern den Fakt mit 2 s Rücksprung anbieten. Eine Ausnahme ist ein kurzer Transient (< 10 s).
- **Navigationsansage (transient, Ducking):** den eigenen Inhalt **pausieren** statt leiser weiterzuspielen, weil gesprochenes Wort beim Ducking unverständlich wird. Das entspricht Androids Vorgabe: kein Auto-Ducking bei Sprachinhalten.
- **Musik oder Podcast einer anderen App läuft:** Nebenbei fordert für die Dauer eines Fakts den Audio-Fokus kurzzeitig an (Android: transienter Fokus; iOS: Session ohne Mischen). Die andere App pausiert und setzt danach selbst fort, sofern sie das unterstützt. Ob das mit expo-audio so steuerbar ist, prüft S2.
- **Kopfhörer abgezogen (becoming noisy):** pausieren, nicht über den Lautsprecher weiterspielen.
- **App-Wechsel oder Sperre:** weiterspielen, Lock-Screen- und Kopfhörertasten: Play/Pause, Weiter = nächster Fakt, Zurück = wiederholen.
- **Warteschlange:** sortiert nach Distanz und Relevanz; Länge, Verfall und Wiederholsperre stehen in [ADR-011](../architecture/decisions/ADR-011-app-core.md). Zwischen Fakten ein kurzer Earcon plus Titelansage, damit man nicht auf das Display schauen muss.

## 6. Akku und Nutzerkontrolle
- **Stufe 1:** eine feste Einstellung, deren Werte S1 und S3 festlegen. Startwerte für den Spike: Genauigkeit „Balanced“ und „High“ im Vergleich, `distanceFilter` 10 m, Intervall 5 s; bei Stillstand über 2 min drosseln.
- Sichtbare Session-Anzeige mit Dauer und Beenden-Knopf; automatisches Ende nach 3 h oder 20 min Stillstand.
- Später, falls gewünscht: mehrere Akkustufen, Ruhemodus („nur manuell“), Lautlos-Zeiten, Themenfilter, maximale Fakten pro Stunde.

## 7. UX-Prinzipien (unterwegs, Barrierefreiheit, Sicherheit)
1. **Eyes-free first:** Alles Wesentliche lässt sich über Kopfhörer- und Lock-Screen-Tasten bedienen, jeder Fakt kündigt sich hörbar an.
2. **Große Ziele:** mindestens 44 pt (iOS) bzw. 48 dp (Android), ein Primärknopf pro Screen, Bedienung mit einer Hand.
3. **Screenreader:** VoiceOver und TalkBack mit Labels und Rollen, sinnvolle Fokusreihenfolge. Die App-Ausgabe darf nicht mit der Screenreader-Sprache kollidieren (bei aktivem Screenreader nur manuell starten). Dynamic Type und Kontraste nach WCAG AA, Transkript zu jedem Audio (auch für Gehörlose).
4. **Sicherheit:** Hinweis beim ersten Start, nicht aufs Display zu schauen und auf den Verkehr zu achten. Während eines Spaziergangs keine visuellen Aufforderungen. Auto-Play pausiert verpflichtend ab Radfahrgeschwindigkeit (über 12 km/h im Mittel über 60 s, Wiederaufnahme unter 8 km/h; ADR-011). Die Redaktion legt Auslösebereiche nicht auf Straßenquerungen.
5. **Transparenz:** eigene Erklärseite vor dem Berechtigungsdialog (Pflicht bei Android Prominent Disclosure, gute Praxis bei iOS). Standort bleibt auf dem Gerät, sofern nicht nötig.
6. **Offline:** In Stufe 1 ist die Region vollständig vorab geladen und damit offline nutzbar; Größe vor dem Download anzeigen. Karten-Offline-Pakete erst mit einer Karte.

## 8. Spike- und PoC-Plan

Maßgeblich für Aufbau, Messmethode und Abnahmekriterien sind die Arbeitspakete WP-01 bis WP-04 in [implementation-plan.md](../roadmap/implementation-plan.md). Kurzüberblick:

| # | Spike | Paket |
|---|---|---|
| S1 | Spaziergang mit gesperrtem Bildschirm, Auslösegenauigkeit | WP-01 |
| S2 | Audio-Unterbrechungen | WP-02 |
| S3 | Akkuverbrauch | WP-01 |
| S4 | TTS-Qualität und Kosten | WP-03 |
| S5 | Store-Vorprüfung | WP-04 |
| S6 | Karte (MapLibre), nur falls nach der Beta gewünscht | WP-17 |

Abbruch- bzw. Eskalationskriterium: Verfehlt S1 auf Samsung oder iOS das Ziel, wird Transistorsoft (Lizenz) evaluiert, bevor die Session-Architektur geändert wird.

## 9. Quellen
- React Native 0.82, nur noch New Architecture: https://reactnative.dev/blog/2025/10/08/react-native-0.82
- Expo SDK 57: https://expo.dev/changelog/sdk-57 · SDK 58 Beta: https://expo.dev/changelog/sdk-58-beta · New Architecture: https://docs.expo.dev/guides/new-architecture/
- expo-audio (Hintergrund, Lock Screen, 3-min-Grenze Android): https://docs.expo.dev/versions/latest/sdk/audio/
- expo-location: https://docs.expo.dev/versions/latest/sdk/location/ (nicht abrufbar, Details in S1 verifizieren)
- Android-FGS-Typen location/mediaPlayback: https://developer.android.com/develop/background-work/services/fgs/service-types · Android 14: https://developer.android.com/about/versions/14/changes/fgs-types-required
- Android-Geofencing (100 Geofences, Latenz): https://developer.android.com/develop/sensors-and-location/location/geofencing
- Android Audio Focus: https://developer.android.com/media/optimize/audio-focus
- Google-Play-Standortrichtlinie: https://support.google.com/googleplay/android-developer/answer/9799150
- Apple allowsBackgroundLocationUpdates: https://developer.apple.com/documentation/corelocation/cllocationmanager/allowsbackgroundlocationupdates · Region Monitoring: https://developer.apple.com/documentation/corelocation/monitoring-the-user-s-proximity-to-geographic-regions · CLBackgroundActivitySession (Forum): https://developer.apple.com/forums/thread/731823
- Ablehnung nach 2.5.4 bei TTS-Hintergrund (Forum): https://developer.apple.com/forums/thread/826849 · expo-speech pausiert bei Sperre: https://developer.apple.com/forums/thread/849627
- RNTP v5, Lizenz und Preise: https://rntp.dev/docs/ · https://www.rntp.dev/terms
- Transistorsoft-Lizenz (nur Android-Release): https://github.com/transistorsoft/react-native-background-geolocation
- Google Cloud TTS, Preise: https://cloud.google.com/text-to-speech/pricing
- MapLibre RN mit Expo: https://maplibre.org/maplibre-react-native/docs/setup/expo · v10: https://maplibre.org/news/2025-02-03-maplibre-newsletter-january-2025/
