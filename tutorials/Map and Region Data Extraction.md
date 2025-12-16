# Terra Invicta Map and Region Data Extraction Guide

This guide provides comprehensive information about how Terra Invicta's in-game map works and where to find region boundary data for use in external applications like web visualizations.

## Table of Contents
- [Overview](#overview)
- [⚠️ Disclaimer](#️-disclaimer)
- [File Locations](#file-locations)
- [Data Structure](#data-structure)
- [Required Tools](#required-tools)
- [Extraction Process](#extraction-process)
- [Data Format Details](#data-format-details)
- [Converting for Web Applications](#converting-for-web-applications)
- [References](#references)

## Overview

Terra Invicta's map system stores region data in two distinct locations:
1. **Template JSON files** - Contains region metadata (population, GDP, ownership, adjacencies)
2. **AssetBundles** - Contains actual polygon coordinates defining region boundaries

The map uses a 3D coordinate system (Vector3) since regions are defined on a spherical globe representation of Earth.

## ⚠️ Disclaimer

**This guide was researched, compiled, and formatted with the assistance of Claude Code AI.** While every effort has been made to ensure accuracy, the information presented here may contain errors or become outdated as the game is updated.

**Please verify critical information** by consulting the original references provided in the [References](#references) section, particularly the Steam Community guides and official documentation. If you encounter issues or inaccuracies, please refer to the source materials and the Terra Invicta modding community for the most current information.

## File Locations

### Template Files (Metadata)
**Location:** `\TerraInvicta_Data\StreamingAssets\Templates\`

Key template files:
- **TIRegionTemplate.json** - Region metadata (population, GDP, area, etc.)
- **TIMapRegionTemplate.json** - Map-specific properties (boostLatitude, etc.)
- **TIBilateralTemplate.json** - Region adjacencies and ownership relationships
- **TINationTemplate.json** - Nation data including regions owned
- **TIMetaTemplate.json** - Additional metadata

### AssetBundles (Polygon Coordinates)
**Location:** `\TerraInvicta_Data\StreamingAssets\AssetBundles\`

Key asset files:
- **regionoutlines** - Contains the `EarthRegionoutlines` asset with actual polygon boundary coordinates
- **Earthbundle.earth** - Contains Earth map visual assets

## Data Structure

### TIRegionTemplate Structure
Located in JSON template files with fields including:
- `dataName` - Unique region identifier (e.g., "Alaska", "California")
- `friendlyName` - Display name for the region
- `boostLatitude` - Latitude value affecting boost production (regions closer to equator get buffs)
- Population, GDP, and other economic/demographic data

### EarthRegionoutlines Structure
Located in the `regionoutlines` AssetBundle:
- Each region is stored as a `TIRegionOutline` object
- Contains **Vector3Array regionSurfacePoints** - the actual polygon coordinates
- Coordinates are in 3D space (x, y, z) representing points on Earth's surface
- Array count indicates number of points in the polygon

Example structure in dump format:
```
0 string name = "Alaska"
0 Vector3Array regionSurfacePoints
  Array size: 150
  [0]
    0 float x = -2.345
    0 float y = 1.234
    0 float z = 0.567
  [1]
    ...
```

## Required Tools

### UABEAvalonia (Recommended for extraction)
- **Purpose:** Unity Assets Bundle Extractor - extracts and edits AssetBundle files
- **Download:** [GitHub - nesrak1/UABEA](https://github.com/nesrak1/UABEA)
- **Latest Release:** [UABEA Releases](https://github.com/nesrak1/UABEA/releases)
- **Compatibility:** Works with Unity 2020.3.x (Terra Invicta's engine version)

### AssetStudio (Alternative viewer)
- **Purpose:** View and export Unity AssetBundles
- **Download:** [GitHub - Perfare/AssetStudio](https://github.com/Perfare/AssetStudio/releases)
- **Note:** Currently the only known utility that works with Unity 2020
- **Use case:** Viewing assets, exporting textures and visual assets

### Unity Hub + Unity Editor (For asset creation/modification)
- **Version Required:** Unity 2020.3.48f1 (MUST match exactly)
- **Download:** [Unity Archive](https://unity3d.com/get-unity/download/archive)
- **Warning:** Using wrong version will crash game when loading modified bundles

### Supporting Tools
- **geojson.io** - Web tool for drawing/editing polygon regions
- **JSON to CSV converter** - For easier spreadsheet editing of template files
- **Python/JavaScript** - For converting between coordinate formats

## Extraction Process

### Step-by-Step Guide to Extract Region Boundaries

1. **Open UABEAvalonia**
   - Launch the application

2. **Load the regionoutlines file**
   - File → Open
   - Navigate to `\TerraInvicta_Data\StreamingAssets\AssetBundles\`
   - Select `regionoutlines`

3. **Access EarthRegionoutlines**
   - Click on "Info" tab
   - Find and select **EarthRegionoutlines** asset
   - Click "Plugins" → "Export Dump"

4. **Export the dump**
   - Choose a destination folder
   - Save the dump as a text file (e.g., `EarthRegionoutlines.txt`)

5. **Parse the dump file**
   - Open the exported text file
   - Search for `TIRegionOutline` entries
   - Each region will have a section like:
     ```
     0 string name = "RegionName"
     0 Vector3Array regionSurfacePoints
     ```
   - Extract the Vector3Array data for regions you need

### Important Notes on Working with Dumps

**When modifying and re-importing:**
- Ensure **NO whitespace or newlines** at the end of the file
- Update the `Array size` count if you modify point arrays
- UABEA will crash or error if formatting is incorrect
- Always **save under a NEW name** when re-importing (don't overwrite)
- Must close and save **twice** in UABEA after importing

## Data Format Details

### Vector3Array Format
Each region boundary is defined as an array of 3D points:
```
0 Vector3Array regionSurfacePoints
  Array size: N
  [0]
    0 float x = <x-coordinate>
    0 float y = <y-coordinate>
    0 float z = <z-coordinate>
  [1]
    ...
```

### Coordinate System
- **Type:** 3D Cartesian coordinates on a unit sphere
- **Origin:** Earth's center
- **Scale:** Normalized to sphere radius
- **Handedness:** Unity's left-handed coordinate system
- **Projection:** Points lie on the surface of a sphere representing Earth

### Converting to Geographic Coordinates
To convert Vector3 to latitude/longitude:
```javascript
// Pseudocode
latitude = Math.asin(y) * (180 / Math.PI)
longitude = Math.atan2(z, x) * (180 / Math.PI)
```

To convert lat/long to Vector3:
```javascript
// Pseudocode
x = Math.cos(lat) * Math.cos(lon)
y = Math.sin(lat)
z = Math.cos(lat) * Math.sin(lon)
```

## Converting for Web Applications

### Option 1: Convert to GeoJSON
GeoJSON is the standard format for web mapping libraries:

```json
{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "properties": {
        "name": "Alaska",
        "dataName": "Alaska"
      },
      "geometry": {
        "type": "Polygon",
        "coordinates": [
          [
            [longitude, latitude],
            [longitude, latitude],
            ...
          ]
        ]
      }
    }
  ]
}
```

### Option 2: Use 3D Globe Libraries
For maintaining spherical representation:
- **Cesium.js** - 3D globe visualization
- **Three.js with globe** - Custom 3D rendering
- **Google Earth API** - If still available

### Option 3: Project to 2D
Common map projections:
- **Web Mercator** (EPSG:3857) - Used by most web maps
- **WGS84** (EPSG:4326) - Standard lat/long
- Use libraries like **proj4js** for projection transformation

### Workflow for Creating Web Map

1. **Extract** regionoutlines dump from game files
2. **Parse** Vector3Array data for each region
3. **Convert** Vector3 coordinates to latitude/longitude
4. **Transform** to GeoJSON format
5. **Validate** using geojson.io
6. **Import** into web mapping library (Leaflet, Mapbox, etc.)

### Example Parsing Script (Pseudocode)
```python
import re
import json
import math

def parse_region_dump(dump_file):
    regions = {}
    current_region = None

    with open(dump_file, 'r') as f:
        for line in f:
            # Find region name
            if 'string name' in line:
                name = re.search(r'"([^"]+)"', line).group(1)
                current_region = name
                regions[name] = []

            # Find Vector3 coordinates
            if 'float x' in line:
                x = float(re.search(r'[-\d.]+', line).group(0))
            elif 'float y' in line:
                y = float(re.search(r'[-\d.]+', line).group(0))
            elif 'float z' in line:
                z = float(re.search(r'[-\d.]+', line).group(0))
                # Convert to lat/long
                lat = math.asin(y) * (180 / math.pi)
                lon = math.atan2(z, x) * (180 / math.pi)
                regions[current_region].append([lon, lat])

    return regions

def create_geojson(regions):
    features = []
    for name, coords in regions.items():
        feature = {
            "type": "Feature",
            "properties": {"name": name},
            "geometry": {
                "type": "Polygon",
                "coordinates": [coords]
            }
        }
        features.append(feature)

    return {
        "type": "FeatureCollection",
        "features": features
    }
```

## Modding Workflow (Reverse Process)

For creating custom regions (opposite of extraction):

1. **Draw regions** on geojson.io
2. **Extract coordinates** from GeoJSON (delete everything except numbers)
3. **Convert** lat/long to Vector3 format
4. **Edit** TIRegionTemplate.json with new region metadata
5. **Edit** regionoutlines dump with new Vector3Array
6. **Import** dump back into AssetBundle using UABEAvalonia
7. **Save** as new file and move to AssetBundles folder
8. **Update** related templates (TIBilateralTemplate, TINationTemplate, etc.)

## References

### Official Documentation
- [Terra Invicta Official Wiki - Regions](https://hoodedhorse.com/wiki/Terra_Invicta/Regions)
- [Terra Invicta Official Wiki - Nations](https://wiki.hoodedhorse.com/Terra_Invicta/Nations)
- [Terra Invicta Official Wiki - TIRegionTemplate](https://wiki.hoodedhorse.com/Terra_Invicta/TIRegionTemplate_EN)
- [Terra Invicta Official Wiki - Game Data](https://wiki.hoodedhorse.com/Terra_Invicta/Terra_Invicta_Official_Wiki:Game_Data)

### Community Guides
- [Steam Guide: Terra Invicta Modding Tutorial - Creating Regions and Nations](https://steamcommunity.com/sharedfiles/filedetails/?id=2965074107)
- [Steam Guide: Introduction to Editing Template Files](https://steamcommunity.com/sharedfiles/filedetails/?id=2905346265)

### Tools
- [GitHub - TROYTRON/ti-mods](https://github.com/TROYTRON/ti-mods) - This repository with modding resources
- [GitHub - nesrak1/UABEA](https://github.com/nesrak1/UABEA) - Unity Assets Bundle Extractor
- [GitHub - Perfare/AssetStudio](https://github.com/Perfare/AssetStudio) - AssetStudio for Unity
- [geojson.io](https://geojson.io) - GeoJSON editor and viewer

### Example Projects
- [GitHub - slyhw4/SolarInvicta](https://github.com/slyhw4/SolarInvicta) - Example mod that modifies regions

### Related Resources
- [Unity Archive Downloads](https://unity3d.com/get-unity/download/archive) - For Unity 2020.3.48f1
- [GeoJSON Specification](https://geojson.org/geojson-spec.html) - GeoJSON format documentation

## Additional Notes

### Game Version Compatibility
- Terra Invicta uses **Unity 2020.3.48f1**
- AssetBundles are version-specific - mismatch causes crashes
- Always backup original files before modifying

### Common Issues
- **UABEA crashes on import:** Check for trailing whitespace in dump file
- **Game crashes on load:** Verify Unity version matches exactly
- **Regions not appearing:** Check TIMetaTemplate.json and bilateral relationships
- **Incorrect boundaries:** Verify coordinate conversion and array counts

### Performance Considerations
- Terra Invicta has 700+ regions globally
- Each region can have 50-300+ polygon points
- Full extraction results in large data files
- Consider filtering to specific regions for web apps
- Use polygon simplification algorithms to reduce point count

### Legal and Ethical Notes
- Respect Pavonis Interactive's intellectual property
- Region boundary data may be derivative of real-world geographic data
- For public web apps, verify licensing terms
- Personal/educational use typically acceptable
- Commercial use may require permission

---

**Last Updated:** December 2024
**Game Version:** Build 0.3.111+
**Unity Version:** 2020.3.48f1
