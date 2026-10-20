# Datenmodell, Geo-Abfragen, Quellen und Lizenzen

## 1. Datenmodell

Grundsätze: Ort und Fakt sind getrennt; Texte sind unveränderliche Revisionen; Quellen und Lizenz hängen an der Revision, weil die Lizenz an der konkreten Textfassung hängt; externe Daten landen nie direkt in veröffentlichten Inhalten.

### Stufe 1: Inhalte als Dateien ([ADR-012](../architecture/decisions/ADR-012-content-in-git.md))

```yaml
# content/<region>/<slug>.yaml
id: 3f0c…            # UUID, beim Anlegen vergeben
name: Alte Stadtmühle # Ortsname (Redaktion, Dubletten)
location: { lat: 50.1234, lon: 8.5678 }   # nur Punktorte in Stufe 1
radius_m: 50          # 30–80
access_note: nur vom Weg aus   # optional
categories: [geschichte]
title: …
body_text: …          # Lesetext = Transkript
tts_text: …           # sprechoptimiert, Klartext
lang: de
license: CC-BY-4.0    # SPDX, Lizenz der Textfassung
sources:
  - provider: book | archive | website | wikidata | interview
    title: …
    url: https://…
    external_revision: …   # optional, z. B. Wikidata lastrevid
    retrieved_at: 2026-10-20
    role: Faktbasis | Textvorlage
attribution_text: …   # Pflicht bei Share-Alike
status: draft | published | archived
editor_minutes: 75    # Arbeitszeit (H-Content)
```

Das Build-Skript erzeugt daraus `regions/<id>/<content_version>/region.json` (Fakten mit Ort, Radius, Titel, Transkript, Kategorien, vollständigen Quellen, Lizenz und Attributionstext, Audio-Schlüssel, Dauer, `schema_version`) und `config.json` (`min_app_version`, `content_version`, `schema_version`, Regionsliste mit Paketgröße).

### Stufe 2: Datenbank (PostgreSQL + PostGIS)

| Tabelle | Kernfelder | Hinweise |
|---|---|---|
| `place` | id (uuid), name, geom `geography(Geometry,4326)`, geom_kind (point/line/area), trigger_radius_m, trigger_geom `geography` (abgeleitet), access_note | Auslösebereich beim Speichern berechnet. Linien und Flächen erst, wenn die App sie auswerten kann (dann vereinfachte `trigger_geom` mitliefern) |
| `fact` | id, place_id, status, pending_revision_id, published_revision_id, published_audio_id, priority, valid_from, valid_to, season | mehrere Fakten pro Ort |
| `category`, `fact_category` | id, slug, parent_id; n:m | |
| `fact_revision` | id, fact_id, version, title, body_text, tts_text, lang, author_id, license_id, created_at, change_note, editor_minutes | nur anhängen |
| `audio_asset` | id, content_hash, kind (tts/recorded), voice, engine_version, duration_s, storage_key, license_id | `content_hash` dient der Wiederverwendung bei gleichem Text und gleicher Stimme |
| `source`, `revision_source` | wie in Stufe 1 | n:m mit Rolle |
| `license` | id, spdx, share_alike, attribution_required, commercial_ok | Prüfregeln |
| `status_event` | id, target_type, target_id, from_status, to_status, actor_id, reason, created_at | Audit-Log |
| `content_version` | eine Zeile, Zähler | jede Veröffentlichung erhöht ihn |
| `staff_user`, `staff_session` | OIDC-Subjekt, Rolle; Sitzungen | ADR-010 |
| später | `account`, `contribution`, `review_task`, `moderation_action`, `candidate`, `notice` | Beiträge, Moderation, Importe, DSA-Meldungen |

Routen werden nicht serverseitig gespeichert ([ADR-007](../architecture/decisions/ADR-007-location-on-device.md)).

### Veröffentlichung (Stufe 2)
1. Freigabe setzt `fact.pending_revision_id` und reiht in derselben Transaktion einen TTS-Job ein. Die bisher veröffentlichte Fassung bleibt sichtbar.
2. Der Job erzeugt das Audio (oder findet es über `content_hash`) und setzt dann **in einer Transaktion** `published_revision_id`, `published_audio_id` und erhöht `content_version`.
3. Archivieren entfernt die veröffentlichte Fassung und erhöht `content_version`; ein Job löscht das Audio, wenn es nicht mehr verwendet wird.
4. Zeitliche Gültigkeit (`valid_from`, `valid_to`, `season`) wertet die App aus, damit Kachelantworten unveränderlich bleiben.

In Stufe 1 gilt dasselbe sinngemäß im Build-Skript: Erst wenn alle Audios hochgeladen sind, wird `config.json` mit der neuen `content_version` geschrieben.

### Prüfregeln vor Veröffentlichung (beide Stufen)
- Mindestens eine Quelle und eine Lizenz je Fassung.
- `commercial_ok = false` ist erlaubt, weil Nebenbei nicht kommerziell ist (Q7); solche Inhalte werden markiert und lassen sich bei einer späteren Kommerzialisierung herausfiltern.
- `share_alike = true` erzwingt dieselbe Lizenz für Text und Audio sowie einen Attributionstext.
- Links nur mit `https:`; Texte ohne Markup; Radius 30–80 m.

## 2. Geo-Abfragen

[F] `<->` liefert indexgestützte KNN-Sortierung für geometry und geography, wenn eine Seite konstant ist; für geography auf der Kugel ([PostGIS](https://postgis.net/docs/manual-3.7/de/geometry_distance_knn.html)). [F] `ST_DWithin` auf geography arbeitet in Metern und nutzt Indizes ([PostGIS](https://postgis.net/docs/ST_DWithin.html)).

```sql
CREATE INDEX place_trigger_gix ON place USING gist (trigger_geom);

-- Kachel-Abfrage (Stufe 2): alle veröffentlichten Fakten, deren Auslösebereich die Kachel berührt
SELECT f.id, r.title, ST_AsGeoJSON(p.geom) AS geom, p.trigger_radius_m, a.storage_key, ...
FROM place p
JOIN fact f            ON f.place_id = p.id AND f.status = 'published'
JOIN fact_revision r   ON r.id = f.published_revision_id
JOIN audio_asset a     ON a.id = f.published_audio_id
WHERE ST_Intersects(p.trigger_geom, ST_Transform(ST_TileEnvelope($z, $x, $y), 4326)::geography);

-- Umkreis (Redaktionswerkzeug, Dublettenprüfung)
SELECT f.id, ST_Distance(p.geom, $1::geography) AS d
FROM place p JOIN fact f ON f.place_id = p.id
WHERE ST_DWithin(p.geom, $1::geography, $2)
ORDER BY p.geom <-> $1::geography
LIMIT 50;

-- Ausweichlösung Stufe 2: Korridor entlang einer Route, sortiert nach Position auf der Linie
WITH r AS (SELECT ST_Transform($1::geometry, 3035) AS line)
SELECT f.id,
       ST_LineLocatePoint(r.line, ST_ClosestPoint(r.line, ST_Transform(p.geom::geometry, 3035))) AS frac
FROM r, place p JOIN fact f ON f.place_id = p.id AND f.status = 'published'
WHERE ST_DWithin(p.geom, ST_Transform(r.line, 4326)::geography, $2)
ORDER BY frac;
```

[F] `ST_TileEnvelope` liefert Web-Mercator (EPSG:3857), daher die Umwandlung ([PostGIS](https://postgis.net/docs/ST_TileEnvelope.html)). Hinweise [A]: Die Abfragen werden mit den gemeinsamen Golden-Fixtures (`testdata/geo/`) getestet. Lange Routen in Abschnitte zerlegen (`ST_Subdivide`), MultiLineStrings vorher mit `ST_LineMerge` zusammenführen. 

### Kachel-API (Stufe 2)
- `GET /v1/config` → `min_app_version`, `content_version`, `schema_version`; kurze Cache-Dauer.
- `GET /v1/tiles/13/{x}/{y}?v={content_version}` → Fakten der Kachel (wie `region.json`); `Cache-Control: public, max-age=31536000, immutable`. Kein ETag, kein Purge nötig, weil jede Änderung eine neue URL ergibt.
- Kachel-Antworten enthalten alle Details (Transkript, vollständige Quellen, Lizenz), damit die App nie einzelne Fakten beim Auslösen nachlädt. Ein Endpunkt `GET /v1/facts/{id}` ist nur für das Redaktionswerkzeug und Teilen-Links vorgesehen, nicht für die Wiedergabe; er liefert nur veröffentlichte Fassungen (Test).
- Laden mit Vorlauf und mitsamt Audio, siehe ADR-007.

## 3. GPX-Import (Stufe 2)

- [F] Komoot bietet offiziell nur den manuellen GPX-Export an; die Datei enthält nur die Geometrie ([Komoot-Hilfe](https://support.komoot.com/hc/articles/10115477099674)). Eine öffentliche API wurde nicht gefunden. Das Produktversprechen lautet daher „GPX-Datei importieren“ (Teilen-Menü oder Dateiauswahl).
- Auf dem Gerät lesen: Track vor Route vor Waypoints; mehrere Segmente getrennt behandeln, Lücken nicht überbrücken; fehlende Zeiten und Höhen tolerieren, Zeitstempel verwerfen; Nullpunkte und Sprünge filtern; Grenzen z. B. 10 MB und 200.000 Punkte; Douglas-Peucker-Vereinfachung mit etwa 10 m Toleranz.
- Die Route bleibt auf dem Gerät. Die App lädt die berührten Kacheln.

## 4. Quellen und Lizenzen

| Quelle | Lizenz | Attribution | Limits, Policy | Stufe | Risiko |
|---|---|---|---|---|---|
| Eigene Redaktion | eigene Wahl | eigene | – | **0, 1** | Durchsatz der Redaktion |
| Wikidata | [F] CC0 ([Wikidata](https://www.wikidata.org/wiki/Wikidata:Licensing)) | nicht nötig, Link empfohlen | Query Service mit Zeit- und CPU-Limits [F?]; User-Agent mit Kontakt [F] ([Policy](https://www.mediawiki.org/wiki/Wikimedia_APIs/Access_policy)) | 2 (Kandidaten) | Datenqualität |
| Wikipedia-Text | [F] CC BY-SA 4.0 ([ToU](https://meta.wikimedia.org/wiki/Terms_of_use/Creative_Commons_4.0)) | Titel, Link, Lizenz, Änderungshinweis | User-Agent | nur Recherche | Share-Alike auch für Umschreibungen, Übersetzungen und KI-Zusammenfassungen |
| OSM-Daten | [F] ODbL | „© OpenStreetMap-Mitwirkende“ | [F] Overpass < 10.000 Abfragen und < 1 GB pro Tag, für alle Nutzer einer App zusammen, bei Dauernutzung durch 100 teilen; kommerziell selbst hosten; Nominatim höchstens 1 Anfrage/s ([Wiki](https://wiki.openstreetmap.org/wiki/Overpass_API)) | später | Share-Alike für abgeleitete Datenbanken |
| Kartenkacheln | je Anbieter, OSM-Attribution | ja | [F] tile.openstreetmap.org: Apps nur mit Erlaubnis, kein Bulk ([Policy](https://operations.osmfoundation.org/policies/tiles/)) | nach Beta (falls Karte) | Kosten bei kommerziellem Anbieter |
| DWD (über Bright Sky) | [F] CC BY 4.0 ([DWD](https://opendata.dwd.de/climate_environment/CDC/Terms_of_use.pdf)) | „Quelle: Deutscher Wetterdienst“ | Bright Sky ohne SLA [A] | später | Ausfall |
| GBIF | [F] je Datensatz CC0, CC BY oder CC BY-NC ([GBIF](https://www.gbif.org/citation-guidelines)) | DOI zitieren | | später | NC-Daten |
| iNaturalist | [F] Standard CC BY-NC ([Hilfe](https://help.inaturalist.org/en/support/solutions/articles/151000175695)) | Beobachter + Lizenz | [F] ≤ 60 Anfragen/min, kein Scraping ([Dev](https://www.inaturalist.org/pages/developers)) | später | NC, Personenbezug |
| Deutsche Digitale Bibliothek | [F] Metadaten CC0 mit Ausnahmen ([DDB](https://www.deutsche-digitale-bibliothek.de/content/nutzungsbedingungen?lang=en)) | je Objekt | API-Schlüssel | später | schwacher Geo-Bezug |
| Europeana | [F] Metadaten CC0, Medien je `edm:rights` | je Objekt | API-Schlüssel | später | uneinheitliche Geo-Daten |

### Lizenzfolgen
- **OSM/ODbL** [F]: Ein „Produced Work“ ist frei lizenzierbar, eine abgeleitete Datenbank muss bei öffentlicher Nutzung unter ODbL bereitgestellt werden ([OSMF](https://osmfoundation.org/wiki/Licence/Community_Guidelines/Produced_Work_-_Guideline)). Unsere API liefert Koordinaten maschinenlesbar aus und ist eher eine Datenbank. **[E]** OSM-Daten später nur in getrennten Tabellen halten, Fakten nur per Referenz verknüpfen, Koordinaten redaktionell selbst setzen; vorher rechtlich prüfen.
- **Wikipedia/CC BY-SA** [F]: Bearbeitungen müssen attribuiert und unter gleicher Lizenz weitergegeben werden. **[E]** Redaktion schreibt eigene Texte aus Fakten und Primärquellen; Wikipedia nur zur Recherche.
- **NC-Lizenzen**: erlaubt, solange Nebenbei nicht kommerziell ist; Achtung, auch Werbung oder Spenden mit Gegenleistung können als kommerziell gelten. Markierung im Paket ermöglicht späteres Herausfiltern.

### Attribution in der App
Pro Fakt eine Quellenzeile im Player und im Transkript (Stufe 1: Liste „Zuletzt gehört“) (Anbieter, Titel, Link, Lizenz, Änderungshinweis). Eine Seite „Quellen & Lizenzen“ listet alle verwendeten Anbieter und die Open-Source-Lizenzen der App. Bei Share-Alike-Inhalten optional ein kurzer akustischer Abspann.

## 5. Integrationsstrategie (ab Stufe 2)
```
Abruf-Job (User-Agent mit Kontakt, Rate-Limit, ETag/If-Modified-Since)
 → candidate (Rohdaten, Lizenz, retrieved_at, external_revision)
 → Dublettenabgleich (Distanz < 50 m + Namensähnlichkeit) [A]
 → review_task (Redaktion übernimmt, verwirft, schreibt Text)
 → fact_revision + revision_source → TTS → Objektspeicher/CDN
 Neuabruf markiert Kandidaten als veraltet, wenn sich die Quelle geändert hat
```
Die App fragt nie externe Dienste direkt an. Pro neuer Quelle: Adapter im Modul `imports`, Lizenz-Eintrag, Attributionstext, Abruf-Limits dokumentiert.
