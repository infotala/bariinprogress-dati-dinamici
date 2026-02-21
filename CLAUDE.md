# CLAUDE.md — Bari in Progress: Dati Dinamici

This file documents the repository structure, data conventions, and workflows for AI assistants working on this project.

## Project Overview

**Repository**: `infotala/bariinprogress-dati-dinamici`
**Purpose**: A geospatial data repository tracking dynamic traffic restrictions ("DIVIETI") across the city of Bari, Italy. The data is maintained as a KML file, likely exported from and consumed by Google Maps / Google My Maps.

The project name "Bari in Progress — Dati Dinamici" ("Bari in Progress — Dynamic Data") reflects its role: continuously updated geodata representing active construction sites, road closures, no-parking zones, and citizen-reported road issues across Bari's urban territory.

---

## Repository Structure

```
bariinprogress-dati-dinamici/
└── DIVIETI.kml          # Primary data file — all traffic restriction placemarks
```

This is a **data-only repository**. There is no application code, build system, package manager, or test suite. The single file `DIVIETI.kml` is the entire content.

---

## Primary File: DIVIETI.kml

### Format

KML (Keyhole Markup Language) — an XML-based format for geographic data, standardized by the OGC and used by Google Earth, Google Maps, QGIS, and many other GIS tools.

- Encoding: UTF-8
- KML namespace: `http://www.opengis.net/kml/2.2`
- Size: ~764 KB, ~6,378 lines
- Placemarks: ~408 geographic features

### Top-Level Structure

```xml
<kml>
  <Document>
    <name>DIVIETI</name>
    <!-- Style definitions -->
    <Style id="..."> ... </Style>
    <StyleMap id="..."> ... </StyleMap>
    <!-- Geographic features -->
    <Placemark> ... </Placemark>
    ...
  </Document>
</kml>
```

### Geometry Types

Two geometry types are used:

1. **Point** — a single coordinate marking a specific location:
   ```xml
   <Point>
     <coordinates>16.875046,41.1232699,0</coordinates>
   </Point>
   ```

2. **LineString** — a sequence of coordinates defining a road segment:
   ```xml
   <LineString>
     <tessellate>1</tessellate>
     <coordinates>16.8828295,41.1318308,0 ...</coordinates>
   </LineString>
   ```

Coordinates are in `longitude,latitude,altitude` order (decimal degrees, WGS84). All altitudes are `0`.

### Placemark Structure

Each `<Placemark>` contains:

| Field | Description |
|---|---|
| `<name>` | Short title of the restriction or report |
| `<description>` | Rich HTML content (wrapped in `<![CDATA[...]]>`) with details |
| `<styleUrl>` | Reference to a `<StyleMap>` defining the icon/line appearance |
| `<Point>` or `<LineString>` | Geographic geometry |

### Description Content

Descriptions are HTML strings (inside CDATA blocks) containing:
- Ordinance number (e.g. `2025/04135`)
- Google Drive link to the official ordinance PDF
- Date range of the restriction
- Nature of the restriction (no parking, road narrowing, no transit, etc.)
- Names of contractors and city departments involved
- Citizen report transcripts (for "SEGNALAZIONE" entries)
- Outcome notes and follow-up actions

---

## Data Categories

### Restriction Types (from `<name>` fields)

| Italian Term | English Meaning |
|---|---|
| Divieto di sosta | No parking |
| Divieto di fermata | No stopping |
| Divieto di transito | No through traffic / road closed |
| Restringimento carreggiata | Road narrowing |
| Senso unico alternato | Alternating one-way traffic (with traffic marshals) |
| Senso unico di marcia | One-way traffic |
| Area pedonale | Pedestrian zone |
| Inversione di marcia | Traffic direction reversal |
| SEGNALAZIONE | Citizen-reported issue (not yet a formal restriction) |
| Regolamentazione temporanea della circolazione | Temporary traffic regulation |

### Style / Color Coding

Styles follow the pattern `icon-{icon_id}-{color_hex}` or `line-{color_hex}-{width}`.

| Color (hex) | Color Name | Typical Use |
|---|---|---|
| `E65100` | Deep orange | Active traffic restrictions (most common, ~188 point markers) |
| `0F9D58` | Green | Restrictions with specific permits / completed or routine |
| `9C27B0` | Purple | Citizen reports (SEGNALAZIONE entries) |
| `C2185B` | Pink/Red | Urgent or notable restrictions |
| `880E4F` | Dark pink | Other restriction category |
| `795548` | Brown | Other restriction category |
| `FF5252` | Red | Other restriction category |
| `757575` | Gray | Other restriction category |

Line styles (`line-E65100-{width}`) represent road segments. The numeric suffix encodes line width in tenths of a pixel (e.g. `4100` = 410 pixels? Actually likely a different encoding — treat as an opaque identifier).

---

## Coordinate System

- **CRS**: WGS84 (EPSG:4326)
- **Bounding area**: City of Bari and its municipalities (e.g. Santo Spirito, Carbonara)
- **Approximate center**: longitude ~16.87, latitude ~41.12
- **Coordinate order in KML**: longitude first, then latitude (opposite of GeoJSON convention)

---

## Data Source and Workflow

The data is managed through **Google My Maps** by the project maintainer (`infotala` / `info@alessandrotalarico.it`) and exported as KML for version-controlled storage.

### Typical Update Cycle

1. Traffic restriction data is gathered from:
   - Official Bari municipal ordinances (published on the city's portal)
   - Citizen reports passed through municipal channels
2. Placemarks are added/updated in Google My Maps
3. The map is exported as KML
4. The exported KML replaces `DIVIETI.kml` in this repository
5. Changes are committed and pushed

### Linked Resources

- Ordinance documents: Hosted on Google Drive (`drive.google.com`)
- Municipal contact: Ripartizione Governo e Sviluppo Strategico del Territorio, Bari
- Google Maps icons: `https://www.gstatic.com/mapspro/images/stock/503-wht-blank_maps.png`

---

## Git Workflow

### Branches

| Branch | Purpose |
|---|---|
| `master` | Primary branch with KML data |
| `main` | Remote default branch on origin |
| `claude/*` | Branches used by AI assistants for documentation/changes |

### Commit Convention

Commits to date use the message `"Add files via upload"`, reflecting GitHub's web upload workflow. When making programmatic commits, prefer descriptive messages such as:

```
Update DIVIETI.kml with restrictions through YYYY-MM-DD
Add SEGNALAZIONE entries for via Foo and via Bar
Remove expired ordinance 2025/XXXXX from DIVIETI.kml
```

### Push Command

```bash
git push -u origin <branch-name>
```

---

## Working with KML Data

### Viewing

- **Google Earth** (desktop or web) — open the KML file directly
- **Google My Maps** — import the KML to view/edit interactively
- **QGIS** — open as a vector layer for GIS analysis
- **VS Code** with an XML extension — for raw editing

### Validation

KML files can be validated against the KML schema. A quick sanity check:

```bash
# Check the file is well-formed XML
xmllint --noout DIVIETI.kml && echo "Valid XML"

# Count placemarks
grep -c '<Placemark>' DIVIETI.kml

# List all restriction names
grep -o '<name>[^<]*</name>' DIVIETI.kml
```

### Editing

When editing `DIVIETI.kml` directly:
- Preserve the UTF-8 encoding and the XML declaration on line 1
- Keep CDATA sections intact for description fields — descriptions contain raw HTML
- Match existing `<StyleMap>` IDs when adding placemarks; do not invent new styles
- Coordinates must remain in `longitude,latitude,altitude` order
- The `<altitude>` component should remain `0`

### Adding a Placemark

Minimal template for a new point restriction:

```xml
<Placemark>
  <name>DIVIETO DI SOSTA — VIA EXAMPLE</name>
  <description><![CDATA[
    Ordinanza n. YYYY/NNNNN<br>
    Dal giorno GG/MM/AAAA al GG/MM/AAAA<br>
    <br>
    Descrizione della restrizione.
  ]]></description>
  <styleUrl>#icon-1541-E65100</styleUrl>
  <Point>
    <coordinates>16.XXXXXX,41.XXXXXX,0</coordinates>
  </Point>
</Placemark>
```

---

## Key Conventions for AI Assistants

1. **Data integrity first**: This is civic data affecting citizens' ability to navigate the city. Do not fabricate, estimate, or hallucinate coordinates, ordinance numbers, or dates.

2. **Preserve all existing placemarks** unless explicitly instructed to remove one. Silently dropping restrictions could cause real-world harm.

3. **Italian language**: Placemark names and descriptions are in Italian. Maintain this convention when adding or editing entries. Do not translate existing content.

4. **No code changes**: This repository has no source code. Do not add JavaScript, Python, or other code files unless explicitly requested and the purpose is clearly defined.

5. **KML structure**: Always produce well-formed XML. Use an XML validator before committing.

6. **Style IDs**: Reuse the existing `<StyleMap>` IDs. The most common for active restrictions is `#icon-1541-E65100`. For citizen reports use `#icon-1574-9C27B0`.

7. **Dates**: Italian date format is `GG/MM/AAAA` (day/month/year). Ordinance numbers follow the pattern `YYYY/NNNNN`.

8. **Coordinate verification**: Before adding a coordinate, verify it falls within the Bari metropolitan area (roughly: longitude 16.7–17.1, latitude 41.0–41.3).
