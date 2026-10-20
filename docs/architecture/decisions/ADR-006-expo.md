# ADR-006: Expo statt bare React Native

**Status:** vorgeschlagen

**Entscheidung:** Expo mit Development Builds, Continuous Native Generation und EAS Build/Submit. Expo Go wird nicht verwendet.

**Begründung:** [F] Das React-Native-Team empfiehlt ein Framework und nennt Expo ([Blog](https://reactnative.dev/blog/2024/06/25/use-a-framework-to-build-react-native-apps)). Config Plugins erlauben jede native Bibliothek. EAS nimmt die Signierung ab. [F] React Native läuft seit 0.82 nur noch mit der New Architecture ([Blog](https://reactnative.dev/blog/2025/10/08/react-native-0.82)); Bibliotheken müssen das unterstützen.

**Verworfen:** bare React Native mit Fastlane (mehr Pflege ohne Gegenwert).

**Over-the-Air-Updates:** In Stufe 0 und 1 wird `expo-updates` nicht eingebunden. Begründung: Wer das Expo-Konto oder den CI-Token übernimmt, könnte sonst Code an der Store-Prüfung vorbei an alle Geräte ausliefern, und dieser Code liefe mit der erteilten Standortberechtigung; außerdem sendet expo-updates eine dauerhafte Client-Kennung an Expo [A, prüfen]. Ab Stufe 2 nur mit Code-Signing der Updates (Schlüssel außerhalb der CI), Vier-Augen-Freigabe und Aufnahme von Expo als Auftragsverarbeiter.

**Risiken:** Abhängigkeit von EAS-Kontingenten (Ausweg: lokale Builds). Eine native Bibliothek ohne Config Plugin erfordert ein eigenes Expo-Modul.
