# ADR-012: Inhalte in Git mit Build-Skript für Stufe 1

**Status:** vorgeschlagen

**Kontext:** Stufe 1 hat 150–300 Fakten und 1–2 Redakteure. Ein Backend mit Redaktionswerkzeug, Anmeldung, Revisionen und Jobs wäre der größte Bauaufwand vor der Validierung.

**Entscheidung:**
- Claude recherchiert und schreibt die Fakten mit belegbaren Quellen und öffnet dafür Pull Requests; Constantin prüft jeden Fakt vor dem Merge.
- Jeder Fakt ist eine YAML-Datei unter `content/<region>/` mit Ort (Punkt), Radius, Titel, Lesetext, TTS-Text, Kategorien, Quellen (Anbieter, URL, externe Revision, Abrufdatum), Lizenz, Status, Hinweis „nur vom Weg aus“.
- Änderungen per Pull Request; Git ist die Revisionshistorie, das Review ist die Freigabe (Branch-Schutz: mindestens ein Review, CI grün).
- `tools/contentbuild` (Go-CLI) prüft Schema und Regeln, erzeugt fehlende TTS-Audios über den Cache-Schlüssel (ADR-008), baut `regions/<id>/<content_version>/region.json`, lädt alles in den Objektspeicher und erhöht zuletzt `config.json` (`content_version`). Erst der letzte Schritt macht die neue Fassung sichtbar.
- Prüfregeln (Build schlägt fehl): jede Datei hat mindestens eine Quelle und eine Lizenz; nicht-kommerzielle Lizenzen sind erlaubt, weil Nebenbei nicht kommerziell ist (Q7), werden aber im Paket markiert, damit sie sich bei einer späteren Kommerzialisierung herausfiltern lassen; Share-Alike erzwingt Attributionstext; Quellen-URLs nur `https:`; Texte ohne Markup; Radius 30–80 m; Koordinaten innerhalb der Regionsgrenze.
- Das YAML-Schema entspricht dem späteren Datenmodell, damit ein Importskript die Inhalte in Stufe 2 in die Datenbank übernimmt.

**Begründung:** Spart Backend, Redaktionswerkzeug, Auth, Jobs und Betrieb bis zur Validierung; das Audit-Log liefert Git. Die App bleibt beim Umstieg gleich.

**Verworfen:** Tabelle in einem Cloud-Office-Dienst (keine Prüfung, keine Revision pro Feld, zusätzlicher Anbieter); sofortiges Backend (Aufwand vor Validierung).

**Risiken:** Redakteure ohne Git-Erfahrung. Gegenmittel: GitHubs Web-Editor reicht für YAML; bei Bedarf ein kleines Formular, das einen Pull Request erzeugt. Wird das zu eng, greift ADR-010.
