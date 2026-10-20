# ADR-011: App-Kern als reines TypeScript, Zustand außerhalb von React

**Status:** vorgeschlagen

**Kontext:** Standort-Updates kommen über eine Hintergrundaufgabe (`expo-task-manager`) außerhalb des React-Baums. Nähe-Logik, Warteschlange und Player müssen auch bei gesperrtem Bildschirm laufen und testbar sein.

**Entscheidung:**
- `apps/mobile/src/core/`: reines TypeScript ohne React- oder Expo-Importe:
  - `proximity`: Positionsfolge → Ereignisse „betreten“/„verlassen“ je Fakt
  - `queue`: Zustandsautomat der Warteschlange und Wiedergabe
  - `walkSession`: Start, Ende, Auto-Ende, Geschwindigkeitsregel
  - Ports: `RegionStore`, `HeardStore`, `AudioPort`, `LocationPort`, `Clock`
- `src/adapters/`: expo-location, expo-audio, Dateisystem und Schlüssel-Wert-Speicher.
- Ein Store außerhalb von React (z. B. Zustand im Vanilla-Modus), den Hintergrundaufgabe und UI gemeinsam nutzen; die UI liest nur.
- Persistenz: Regionspaket und Audio als Dateien; „schon gehört“, Einstellungen und lokale Zähler in einer kleinen Schlüssel-Wert-Ablage.
- Neustart des JS-Kontexts: Session-Zustand wird persistiert; nach einem Neustart läuft keine Session automatisch weiter (Store-Policy, siehe mobile §3).

**Parameter der Nähe-Logik (Startwerte, im Feldtest justieren):**
- Positionsfixes mit gemeldeter Genauigkeit schlechter als 40 m werden verworfen.
- „Betreten“: zwei aufeinanderfolgende Fixes innerhalb des Radius (Standard 50 m, je Fakt 30–80 m) oder ein Fix mit mindestens 3 s Verweildauer.
- „Verlassen“: Abstand größer als Radius + 20 m.
- Ein Fakt wird höchstens einmal in 24 h ausgelöst; Warteschlange höchstens 3; ein Eintrag verfällt, wenn der Ort mehr als 200 m entfernt ist.
- Auto-Play pausiert, wenn die Geschwindigkeit über 60 s im Mittel mehr als 12 km/h beträgt, und setzt erst nach 60 s unter 8 km/h wieder ein.

**Begründung:** Die Kernlogik wird mit Golden-Fixtures (Positionsfolgen) als reine Unit-Tests prüfbar; Spike-Code wird nicht „übernommen“, sondern gegen diese Struktur neu geschrieben.

**Verworfen:** Zustand in React-Context (läuft nicht ohne UI); Redux (mehr Zeremonie ohne Nutzen).
