# 🛰️ Universal GEE Satellite Scene Finder & Downloader

A reusable **Google Colab + Google Earth Engine** workflow for finding satellite scenes over a geographic location or vector AOI, inspecting acquisition metadata, exporting the results to Excel, and optionally downloading selected scenes.

> **Concept / inspiration:** Dharmendra Kumar Pandey Sir.

## What this project does

The workflow is designed to make satellite-scene discovery more reusable instead of writing a separate script for every sensor or study area.

It can:

- Query a Google Earth Engine `ImageCollection`.
- Search scenes using a **latitude/longitude + buffer**.
- Search using a **ZIP Shapefile, `.shp`, GeoJSON, GeoPackage (`.gpkg`) or KML** AOI in the interactive downloader.
- Automatically detect commonly used metadata fields for:
  - cloud cover
  - orbit pass / flight direction
  - orbit or track number
- Apply optional:
  - date filters
  - cloud-cover filters for optical collections
  - ascending/descending orbit filters
- Extract scene-level metadata including acquisition date/time and available collection properties.
- Export a formatted **Excel acquisition report**.
- Download selected scenes as GeoTIFF/other supported Earth Engine download formats.
- Clip downloads to a point-buffer AOI or uploaded vector AOI.
- Fall back to a Google Earth Engine batch export to Google Drive when a direct download is too large.

## Files

| File                                      | Purpose                                 |
| ----------------------------------------- | --------------------------------------- |
| `Universal_GEE_Scene_Finder_Public.ipynb` | Main Google Colab notebook              |
| `universal_gee_scene_finder.py`           | Python source copy of the notebook code |
| `README.md`                               | Documentation and usage instructions    |

## Requirements

You need:

1. A Google account.
2. Access to **Google Earth Engine**.
3. Google Colab (recommended).
4. A valid Earth Engine collection ID for the data you want to search.
5. For vector-AOI searches, one of the supported vector formats listed above.

The notebook installs the Python packages it needs for Excel export. Earth Engine authentication is handled in the first cell.

---

# Quick Start

## 1. Open the notebook in Google Colab

Upload:

`Universal_GEE_Scene_Finder_Public.ipynb`

to Google Colab.

## 2. Authenticate Earth Engine

Run the first cell.

The public version does **not** contain a personal Earth Engine project ID.

If your Earth Engine Python environment requires a Cloud project, edit:

```python
GEE_PROJECT = None
```

to:

```python
GEE_PROJECT = "your-ee-project-id"
```

Use **your own** project ID. Do not commit private credentials or authentication tokens to GitHub.

## 3. Select a dataset

In the **Universal Satellite Scene Acquisition Explorer** cell, change:

```python
COLLECTION_ID = "COPERNICUS/S1_GRD"
```

Examples included in the notebook include:

```text
COPERNICUS/S2_SR_HARMONIZED
COPERNICUS/S2_HARMONIZED
COPERNICUS/S1_GRD
LANDSAT/LC09/C02/T1_L2
LANDSAT/LC08/C02/T1_L2
LANDSAT/LE07/C02/T1_L2
LANDSAT/LT05/C02/T1_L2
JAXA/ALOS/PALSAR-2/Level2_2/ScanSAR
MODIS/061/MOD09GA
```

You can also enter another Google Earth Engine `ImageCollection` ID, provided it is accessible from your account.

## 4. Set the location

For point/buffer scene search:

```python
LATITUDE = 20.5937
LONGITUDE = 78.9629
BUFFER_METERS = 100
```

Replace these with your study location.

The coordinates in the public notebook are only generic example values.

## 5. Set the date range

To search a specific period:

```python
FETCH_ALL_ACQUISITIONS = False
START_DATE = "2025-01-01"
END_DATE = "2025-12-31"
```

To search the available archive:

```python
FETCH_ALL_ACQUISITIONS = True
```

Use a date range where possible because very large collections can require substantial Earth Engine processing.

## 6. Optional filters

For optical data:

```python
CLOUD_COVER_MAX = 5
```

For Sentinel-1 or another radar collection, the cloud filter is automatically bypassed when the workflow identifies the collection as radar.

For orbit direction:

```python
ORBIT_PASS = "ALL"
```

or:

```python
ORBIT_PASS = "ASCENDING"
```

or:

```python
ORBIT_PASS = "DESCENDING"
```

## 7. Run the acquisition search

The notebook returns a Pandas DataFrame containing scene information and exports a formatted Excel workbook.

The output includes fields such as:

- Scene ID
- GEE Asset ID
- Acquisition date
- Acquisition time (UTC)
- Day of year
- Day of week
- Cloud cover, where available
- Orbit pass, where available
- Orbit/track number, where available
- Other available image metadata
- Query latitude/longitude
- Query buffer

The exact metadata columns depend on the collection.

---

# Scene Download

After obtaining the acquisition table, the notebook provides a direct scene-download section.

Example:

```python
SCENES_TO_DOWNLOAD = "1"
```

Supported selection styles include:

```python
"1"
```

for one scene,

```python
[1, 5, 10]
```

for several scene numbers,

```python
"1, 3, 5"
```

for a comma-separated selection,

```python
"FIRST"
```

for the oldest scene,

```python
"LATEST"
```

for the newest scene, and:

```python
"ALL"
```

for all scenes returned by the search.

You can also provide a specific scene ID where supported.

### Clip to the search AOI

```python
CLIP_TO_AOI = True
BUFFER_METERS = 1000
```

When enabled, the selected scene is downloaded for the target area rather than the complete satellite scene footprint.

### Band selection

You can specify bands when needed:

```python
BANDS = ["B4", "B3", "B2"]
```

or for a radar product where those bands exist:

```python
BANDS = ["HH", "HV"]
```

Setting:

```python
BANDS = None
```

allows the downloader to use its automatic/default band handling.

---

# Vector AOI Scene Downloader

The final notebook section provides an interactive vector-AOI workflow.

Supported inputs include:

- ZIP Shapefile
- Loose Shapefile files (`.shp`, `.shx`, `.dbf`, `.prj`)
- GeoJSON
- GeoPackage (`.gpkg`), including multi-layer files
- KML

This is useful when the study area is not adequately represented by a point and buffer.

The workflow can:

1. Upload the vector AOI.
2. Read the AOI geometry.
3. Search for scenes intersecting the AOI.
4. Present available scenes for selection.
5. Select bands.
6. Clip the selected scene to the AOI.
7. Download the result.
8. Submit a Google Earth Engine Drive export when a direct download is too large.

---

# Important Notes

## 1. Collection compatibility

The workflow is intended to be broadly reusable across Earth Engine `ImageCollection`s, but metadata conventions differ between datasets.

The script therefore attempts to **detect common property names automatically**.

If a particular collection uses unusual metadata names, cloud/orbit filtering may not be available automatically.

Always inspect the returned metadata before using the results for scientific analysis.

## 2. Cloud filtering

Cloud filtering is intended primarily for optical imagery.

Radar collections such as Sentinel-1 do not normally use optical cloud-cover percentages, so the workflow bypasses the cloud filter for radar collections.

## 3. Scene availability

A zero-scene result does not necessarily mean the satellite has no imagery in the area. Possible causes include:

- the collection does not cover the location
- the selected dates contain no acquisitions
- the cloud threshold is too restrictive
- the orbit-direction filter is too restrictive
- the collection ID is incorrect
- the AOI does not intersect the image footprint
- the Earth Engine account does not have access to the requested collection

## 4. Earth Engine quotas

Large searches and downloads consume Earth Engine resources. Avoid unnecessarily large date ranges, huge AOIs, or downloading complete satellite scenes when a clipped AOI is sufficient.

## 5. Scientific use

The script is an **acquisition and metadata discovery tool**. Finding a scene does not guarantee that it is appropriate for a particular scientific analysis.

Before using imagery in research, check the dataset documentation, processing level, acquisition geometry, calibration, spatial resolution, polarization/bands, cloud or quality flags, and other mission-specific requirements.

---
# Acknowledgement

If you use or adapt this workflow in academic work, presentations, or other public projects, please acknowledge the conceptual source:

Inspiration: Dr. Dharmendra Kumar Pandey Sir.   
Ceated by: Shivansh Dutt Shukla


# Disclaimer

This is a community/research-oriented workflow for Google Earth Engine scene discovery and downloading. Dataset availability, Earth Engine APIs, collection metadata, access permissions, and download limits can change over time.

Users should verify the current documentation and terms for the datasets and platforms they use.
