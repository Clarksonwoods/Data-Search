# Clarkson & Woods — Automated Desk Study Tool
**Version 0.1**

A browser-based ecological desk study tool built on the ArcGIS Maps SDK for JavaScript. Users select a red line boundary on a web map, apply a buffer, and the tool automatically interrogates national open access ecological datasets to produce a structured report ready for use in an Ecological Impact Assessment or Preliminary Ecological Appraisal.

---

## Live URL

```
https://charlie-fayers.github.io/Desktop-Survey/
```

---

## How to use

1. **Search** — type a site name or place name into the search bar at the top of the map. Red line boundary features can be searched by their `Label` field and will auto-select when chosen. Place names and postcodes are geocoded via the ArcGIS World Geocoder.

2. **Set buffer distance** — choose a buffer distance using the preset buttons (0.5, 2, 5, 10 km) or type a custom value. Default is 2 km.

3. **Select boundary** — click **Select red line boundary** then click a red line boundary polygon on the map, or use the search to auto-select one.

4. **Run Desk Study** — click **Run Desk Study**. The tool will:
   - Create a geodesic buffer around the selected boundary
   - Query all configured ecological layers against the buffer
   - Calculate distance and direction from the boundary to each intersecting feature
   - Generate a structured report

5. **Export to Word** — click **Export to Word** to download a `.docx` file formatted in Century Gothic (9pt body, 8pt tables) with green-headed tables, alternating row shading, and placeholder text highlighted for manual completion.

---

## Layers queried

### England — Statutory Designations
| Layer | Name field |
|---|---|
| Sites of Special Scientific Interest (England) | NAME |
| Special Areas of Conservation (England) | SAC_NAME |
| Special Protection Areas (England) | NAME |
| Ramsar (England) | NAME |
| National Nature Reserves (England) | NAME |
| Local Nature Reserves (England) | NAME |
| Areas of Outstanding Natural Beauty (England) | NAME |
| National Parks (England) | NAME |

### England — Habitats
| Layer | Field |
|---|---|
| Ancient Woodland (England) | NAME |
| Priority Habitats Inventory (England) | MainHabs (grouped by habitat type) |

### England — Watercourses
| Layer |
|---|
| Priority River Habitat — Rivers (England) |
| Priority River Habitat — Headwater Areas (England) |
| Chalk Rivers (England) |
| EA Statutory Rivers |

### Bat Consultation Zones
| Layer |
|---|
| Wiltshire |
| Trowbridge SPD |
| North Somerset and Mendips |
| Hestercombe House |
| Exmoor and Quantocks |
| Mells Valley |

### Biodiversity Records
| Layer | Field |
|---|---|
| iNaturalist Observations | common_name |
| C&W In-house Records | Common_Name |

### Wales (WMS — display only, not queryable)
Wales layers (SSSI, SAC, SPA, RAMSAR, NNR, LNR, AONB, National Park, Ancient Woodland, Priority Habitats) are shown on the map but cannot be queried programmatically as they are served as WMS. They are included in the Data Limitations section of the report only when the site is detected as being within Wales.

---

## Report structure

The generated report follows standard ecological appraisal structure:

```
Desk Study Findings
  Designated Sites
    Statutory Designated Sites        — Table 1 with size, distance, direction, importance
    Local and Non-Statutory Sites     — Table 2 (LNRs)
    Local Wildlife Sites              — placeholder
  Habitats                            — paragraphs with bullet lists
  Watercourses and River Habitats
  Bat Consultation Zones
  Biodiversity Records                — species lists
  Local BAP                           — placeholder
  Planning Policy                     — placeholder
  Data Limitations                    — WMS and unqueryable layers (England only if site not in Wales)
```

Importance is classified automatically:
- **International** — SAC, SPA, Ramsar
- **National** — SSSI, NNR, National Park, AONB
- **Local** — LNR

Each SSSI includes a direct link to its Natural England citation page (`designatedsites.naturalengland.org.uk`) so the Reason for Designation can be retrieved quickly.

Distance and direction are calculated from the nearest point of each intersecting feature to the red line boundary centroid. Features within 50m are described as "within or adjoining the site".

Feature counts for large datasets (Priority Habitat Inventory, EA Statutory Rivers, iNaturalist) are retrieved using `returnCountOnly` queries to give accurate totals. All other layers paginate in batches of 500 up to 2,000 features.

---

## Technical details

- **ArcGIS Maps SDK for JavaScript** 4.29 (loaded from Esri CDN)
- **Authentication** — OAuth 2.0 via ArcGIS Online App ID `pduqao2ad2vWm62Z` (QGIS Portal registration). Redirect URI must include `https://charlie-fayers.github.io/Desktop-Survey/`
- **Web Map ID** — `51a20531f9f44c4682f1bb246981ea86`
- **Buffer** — geodesic buffer using `geometryEngineAsync.geodesicBuffer()` in Web Mercator (WKID 102100)
- **Word export** — pure vanilla JavaScript OOXML builder using the native browser Compression Streams API (`CompressionStream("deflate-raw")`). No external libraries — avoids AMD conflicts with the ArcGIS/Dojo module system.
- **Hosted on** — GitHub Pages (`https://charlie-fayers.github.io/Desktop-Survey/`)

---

## Deployment

The tool is a single `index.html` file. To update:

1. Download the latest `index.html`
2. Go to `https://github.com/Charlie-Fayers/Desktop-Survey`
3. Click `index.html` → pencil icon (Edit) → select all → paste new content → **Commit changes**

Or drag and drop via **Add file → Upload files** and commit.

GitHub Pages republishes within ~60 seconds. Hard refresh (`Ctrl+Shift+R`) to clear the browser cache.

---

## Configuration

All configurable settings are in the `CONFIG` block at the top of the `<script>` section in `index.html`:

```javascript
const CONFIG = {
  portalUrl:           "https://www.arcgis.com",
  webMapId:            "51a20531f9f44c4682f1bb246981ea86",
  appId:               "pduqao2ad2vWm62Z",
  redLineTitlePattern: /red.?line.?boundar/i,
  layers: [ ... ]
};
```

### Adding a new layer

Add an entry to `CONFIG.layers`:

```javascript
{ title: "Exact layer title from web map",
  shortName: "Text used in the report",
  nameField: "FIELD_NAME",      // attribute field for feature names; null = count only
  type: "statutory",            // statutory | habitat | river | bat | records
  maxList: 4,                   // max names listed before "and N others"
  groupByField: true,           // optional: group by unique nameField values (e.g. PHI)
  wms: true }                   // optional: mark as WMS (display only, not queried)
```

### Changing the buffer presets

Edit the preset buttons in the HTML:

```html
<button class="preset-btn" data-km="0.5">0.5 km</button>
<button class="preset-btn active" data-km="2">2 km</button>
<button class="preset-btn" data-km="5">5 km</button>
<button class="preset-btn" data-km="10">10 km</button>
```

---

## Known limitations

- **Wales layers** are WMS services and cannot be queried via the REST API. They are shown on the map and noted in the report only when the site is detected as being in Wales. Wales detection uses an approximate bounding box — sites close to the England/Wales border should have Welsh layer results checked manually in the map.
- **Local Wildlife Sites (LWS)** are not available as open access data. The report includes a placeholder directing the ecologist to check with the local LERC.
- **EA Statutory Rivers** is hosted on a third-party ArcGIS server which may have CORS restrictions on some networks.
- **Feature counts** are paginated up to 2,000 features per layer. Where this cap is reached, the true count is retrieved via a separate `returnCountOnly` query and reported as "at least N".
- **Reason for Designation** cannot be retrieved automatically (no public API from Natural England). Each statutory designation in Table 1 links to its NE citation page for manual lookup.

---

## Version history

| Version | Date | Notes |
|---|---|---|
| 0.1 | April 2026 | Initial release |
