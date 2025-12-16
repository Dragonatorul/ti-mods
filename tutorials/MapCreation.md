# Creating Custom Maps and Regions in Terra Invicta

A comprehensive guide for creating custom maps, regions, and nations in Terra Invicta.

## Table of Contents
- [Overview](#overview)
- [⚠️ Disclaimer](#️-disclaimer)
- [Required Tools](#required-tools)
- [Understanding the Map System](#understanding-the-map-system)
- [Method 1: Creating Custom Regions](#method-1-creating-custom-regions)
- [Method 2: Working with SVG Files](#method-2-working-with-svg-files)
- [Updating Template Files](#updating-template-files)
- [Testing Your Changes](#testing-your-changes)
- [Troubleshooting](#troubleshooting)
- [External Resources](#external-resources)

## Overview

Terra Invicta's map system consists of two main components:
1. **Region polygons** - The actual boundary coordinates stored in AssetBundles
2. **Region metadata** - Properties like population, GDP, ownership stored in JSON templates

Creating custom regions requires:
- Drawing or defining region boundaries
- Converting coordinates to Unity's Vector3 format
- Editing AssetBundle files
- Updating multiple JSON template files
- Ensuring all relationships (adjacencies, ownership, etc.) are properly configured

## ⚠️ Disclaimer

**This guide was researched, compiled, and formatted with the assistance of Claude Code AI.** While every effort has been made to ensure accuracy, the information presented here may contain errors or become outdated as the game is updated.

**Please verify critical information** by consulting the original references provided in the [External Resources](#external-resources) section, particularly the Steam Community guides and official documentation. If you encounter issues or inaccuracies, please refer to the source materials and the Terra Invicta modding community for the most current information.

## Required Tools

### Essential Tools

1. **UABEAvalonia (Unity Assets Bundle Extractor)**
   - For editing AssetBundle files containing region polygons
   - Download: [GitHub - nesrak1/UABEA](https://github.com/nesrak1/UABEA/releases)

2. **Unity Hub + Unity Editor 2020.3.48f1**
   - **CRITICAL:** Must use exactly version 2020.3.48f1
   - Wrong version will crash the game when loading modified bundles
   - Download: [Unity Archive](https://unity3d.com/get-unity/download/archive)

3. **Text Editor**
   - For editing JSON template files
   - VSCode, Notepad++, or similar recommended

### Optional But Helpful

4. **AssetStudio**
   - For viewing AssetBundle contents
   - Download: [GitHub - Perfare/AssetStudio](https://github.com/Perfare/AssetStudio/releases)

5. **Inkscape** (for SVG method)
   - Open-source vector graphics editor
   - Download: [Inkscape.org](https://inkscape.org/)

6. **Python or JavaScript**
   - For coordinate conversion scripts
   - Not required but makes batch processing easier

## Understanding the Map System

### File Locations

**AssetBundles (Polygon Data):**
```
\TerraInvicta_Data\StreamingAssets\AssetBundles\
├── regionoutlines (contains EarthRegionoutlines asset)
└── Earthbundle.earth (contains visual assets)
```

**Template Files (Metadata):**
```
\TerraInvicta_Data\StreamingAssets\Templates\
├── TIRegionTemplate.json
├── TIMapRegionTemplate.json
├── TIBilateralTemplate.json
├── TINationTemplate.json
└── TIMetaTemplate.json
```

### Coordinate System

Terra Invicta uses **3D Vector3 coordinates** representing points on a sphere (Earth):
- **x, y, z** - Cartesian coordinates
- **Normalized** to unit sphere radius
- **Left-handed** coordinate system (Unity standard)

Converting from latitude/longitude to Vector3:
```javascript
x = cos(latitude) * cos(longitude)
y = sin(latitude)
z = cos(latitude) * sin(longitude)
```

Converting from Vector3 to latitude/longitude:
```javascript
latitude = asin(y) * (180 / PI)
longitude = atan2(z, x) * (180 / PI)
```

## Method 1: Creating Custom Regions

This is the recommended method for creating new regions from scratch.

### Step 1: Draw Your Region Boundaries

1. **Go to [geojson.io](https://geojson.io)**
2. **Zoom to your area of interest**
3. **Use the "Draw a polygon" tool** (right sidebar)
4. **Draw your region boundary**
   - Click to place points
   - Double-click to complete the polygon
   - Be reasonably precise but don't use excessive points (50-300 points is typical)

5. **Save the GeoJSON**
   - Copy the JSON from the right panel
   - Save to a file (e.g., `my_region.geojson`)

### Step 2: Extract and Convert Coordinates

Your GeoJSON will look like:
```json
{
  "type": "FeatureCollection",
  "features": [{
    "type": "Feature",
    "geometry": {
      "type": "Polygon",
      "coordinates": [[
        [longitude, latitude],
        [longitude, latitude],
        ...
      ]]
    }
  }]
}
```

**Extract just the coordinate pairs:**
1. Find the `"coordinates"` array
2. Copy only the number pairs: `[[-122.5, 45.5], [-122.6, 45.6], ...]`

**Convert to Vector3 format:**

Use this Python script template:
```python
import json
import math

def latlon_to_vector3(lat, lon):
    """Convert latitude/longitude to Unity Vector3."""
    lat_rad = math.radians(lat)
    lon_rad = math.radians(lon)

    x = math.cos(lat_rad) * math.cos(lon_rad)
    y = math.sin(lat_rad)
    z = math.cos(lat_rad) * math.sin(lon_rad)

    return (x, y, z)

def geojson_to_vector3_dump(geojson_file, output_file, region_name):
    """Convert GeoJSON to Unity dump format."""
    with open(geojson_file, 'r') as f:
        data = json.load(f)

    coords = data['features'][0]['geometry']['coordinates'][0]

    with open(output_file, 'w') as f:
        f.write(f'0 string name = "{region_name}"\n')
        f.write(f'0 Vector3Array regionSurfacePoints\n')
        f.write(f' Array Array size: {len(coords)}\n')

        for i, (lon, lat) in enumerate(coords):
            x, y, z = latlon_to_vector3(lat, lon)
            f.write(f' [{i}]\n')
            f.write(f'  0 float x = {x}\n')
            f.write(f'  0 float y = {y}\n')
            f.write(f'  0 float z = {z}\n')

# Usage
geojson_to_vector3_dump('my_region.geojson', 'my_region_dump.txt', 'MyRegion')
```

### Step 3: Edit the AssetBundle

1. **Backup the original file**
   ```
   Copy regionoutlines to regionoutlines.backup
   ```

2. **Open UABEAvalonia**

3. **Load the regionoutlines file**
   - File → Open
   - Navigate to `\TerraInvicta_Data\StreamingAssets\AssetBundles\`
   - Select `regionoutlines`

4. **Export the current dump**
   - Click "Info" tab
   - Select **EarthRegionoutlines**
   - Click "Plugins" → "Export Dump"
   - Save as `regionoutlines_dump.txt`

5. **Add your region to the dump**
   - Open `regionoutlines_dump.txt` in text editor
   - Find the end of the regions list (before the closing structure)
   - Insert your region data from Step 2
   - **CRITICAL:** Ensure NO trailing whitespace or newlines at file end
   - Update any array counts if needed

6. **Import the modified dump**
   - In UABEAvalonia, select EarthRegionoutlines
   - Click "Plugins" → "Import Dump"
   - Select your modified `regionoutlines_dump.txt`
   - If UABEA crashes or errors, check for whitespace issues

7. **Save the modified AssetBundle**
   - File → Close (will prompt to save)
   - Click "Yes" to save
   - Close again and save again (must save TWICE)
   - **CRITICAL:** Save with a NEW filename (e.g., `regionoutlines_new`)
   - Do NOT overwrite the original (causes crashes)

8. **Replace the game file**
   - Rename original: `regionoutlines` → `regionoutlines.original`
   - Rename your file: `regionoutlines_new` → `regionoutlines`

### Step 4: Update Template Files

You need to update multiple JSON templates to make your region functional.

**See [Updating Template Files](#updating-template-files) section below.**

## Method 2: Working with SVG Files

This method is useful for importing vector graphics as region boundaries.

### SVG Requirements

**IMPORTANT:** The SVG importer built into Terra Invicta doesn't work with relative offset positions, so your SVG editor needs to save only in absolute coordinates.

In **Inkscape**, this can be accomplished via the setting:
![image](https://user-images.githubusercontent.com/11687023/194723480-c9377e60-2a11-409c-a39d-844d4793b05c.png)

Edit → Preferences → SVG output → Path data: **"Absolute"**

### SVG to Region Workflow

1. **Create your SVG**
   - Use Inkscape or similar tool
   - Draw region boundaries as paths
   - Ensure absolute coordinate mode is enabled
   - Keep complexity reasonable (avoid thousands of points)

2. **Extract path data**
   - Open SVG in text editor
   - Find `<path d="...">` elements
   - Extract the coordinate data

3. **Convert SVG coordinates to geographic coordinates**
   - Determine your SVG's coordinate system
   - Map SVG space to latitude/longitude space
   - This requires knowing your projection and bounds

4. **Convert to Vector3**
   - Use the same conversion as Method 1
   - Apply the lat/lon to Vector3 formulas

5. **Import using UABEAvalonia**
   - Follow steps from Method 1, Step 3

### SVG Tips

- **Simplify paths** before export to reduce point count
  - In Inkscape: Path → Simplify (Ctrl+L)
- **Use appropriate projection** - Mercator is common but distorts at poles
- **Test iteratively** - Start with simple shapes
- **Reference existing regions** - Extract an existing region to see expected format

## Updating Template Files

After adding region polygons, you must update JSON templates to make regions functional.

### Template File Hierarchy

Creating a new region requires updating (in order):

1. **TIRegionTemplate.json** - Core region properties
2. **TIMapRegionTemplate.json** - Map-specific properties
3. **TIBilateralTemplate.json** - Ownership and adjacencies
4. **TINationTemplate.json** - Add region to nation (optional)
5. **TIMetaTemplate.json** - Metadata (if needed)

### 1. TIRegionTemplate.json

Add a new entry for your region:

```json
{
  "dataName": "MyCustomRegion",
  "friendlyName": "My Custom Region",
  "displayName": "LOC_REGION_MYCUSTOMREGION",
  "area": 50000,
  "population": 1000000,
  "GDP": 50000000000,
  "democracyLevel": 7,
  "governmentCohesion": 5,
  "corruptionLevel": 3,
  "militaryTechLevel": 5,
  "unrest": 0,
  "regionalHQ": false
}
```

**Key fields:**
- `dataName` - Unique identifier (must match AssetBundle region name)
- `friendlyName` - Display name in game
- `area` - Square kilometers
- `population` - Number of people
- `GDP` - In dollars
- `democracyLevel`, `governmentCohesion`, etc. - Game balance values (0-10)

### 2. TIMapRegionTemplate.json

Add map-specific properties:

```json
{
  "dataName": "MyCustomRegion",
  "boostLatitude": 45.5,
  "mapOffset": {
    "x": 0,
    "y": 0
  }
}
```

**Key fields:**
- `boostLatitude` - Affects boost production (equator = buff)
- `mapOffset` - Visual offset for UI elements

### 3. TIBilateralTemplate.json

Define ownership and adjacencies:

**Initial ownership:**
```json
{
  "dataName": "ClaimUSAMyCustomRegion",
  "relationType": "Claim",
  "nation1": "USA",
  "region1": "MyCustomRegion",
  "initialOwner": true
}
```

**Adjacencies to other regions:**
```json
{
  "dataName": "AdjacentMyCustomRegionOregon",
  "relationType": "Adjacency",
  "region1": "MyCustomRegion",
  "region2": "Oregon"
},
{
  "dataName": "AdjacentMyCustomRegionWashington",
  "relationType": "Adjacency",
  "region1": "MyCustomRegion",
  "region2": "Washington"
}
```

**Important:** Define adjacency for EVERY neighboring region or they won't be connected in-game.

### 4. TINationTemplate.json (Optional)

If creating new nations or modifying existing ones:

```json
{
  "dataName": "USA",
  "regions": ["Alaska", "MyCustomRegion", "..."],
  ...
}
```

Add your region to the `regions` array of the owning nation.

### Localization (Optional)

To add proper display names:

**Location:** `\TerraInvicta_Data\StreamingAssets\Localization\en\`

Add to appropriate localization file:
```
LOC_REGION_MYCUSTOMREGION,My Custom Region
```

## Testing Your Changes

### Initial Test Process

1. **Create mod structure** (recommended approach)
   ```
   \Terra Invicta\Mods\Enabled\MyMapMod\
   ├── ModInfo.json
   ├── TIRegionTemplate.json (only your new regions)
   ├── TIMapRegionTemplate.json
   └── TIBilateralTemplate.json
   ```

2. **Create ModInfo.json**
   ```json
   {
     "Title": "MyMapMod",
     "Author": "YourName",
     "Description": "Adds custom regions to Terra Invicta"
   }
   ```

3. **Launch the game**
   - Go to Mods menu
   - Enable your mod
   - Check "Use Mods" checkbox
   - Restart game

4. **Verify merge**
   - Game creates backups of original templates
   - Merges your changes into Templates folder
   - Check `\Templates\` for merged files

5. **Start new campaign**
   - Load a new game (existing saves won't have new regions)
   - Check if your region appears on the map
   - Verify ownership, adjacencies, and properties

### Validation Checklist

✅ Region appears on map
✅ Region has correct owner
✅ Clicking region shows correct stats
✅ Adjacent regions are properly connected
✅ Can move units to/from region
✅ No crashes or errors in console
✅ Region responds to game mechanics (control points, etc.)

## Troubleshooting

### Common Issues

**Issue: Game crashes on startup**
- **Cause:** AssetBundle version mismatch or corrupted file
- **Fix:** Verify Unity version is exactly 2020.3.48f1, rebuild AssetBundle

**Issue: Region doesn't appear on map**
- **Cause:** dataName mismatch between AssetBundle and templates
- **Fix:** Ensure exact same `dataName` in all files (case-sensitive)

**Issue: UABEA crashes when importing dump**
- **Cause:** Trailing whitespace/newlines in dump file
- **Fix:** Remove ALL whitespace after last data entry

**Issue: Region appears but has no stats**
- **Cause:** Missing TIRegionTemplate.json entry
- **Fix:** Add complete entry with all required fields

**Issue: Cannot move between regions**
- **Cause:** Missing adjacency definitions
- **Fix:** Add TIBilateralTemplate.json entries for all borders

**Issue: Region belongs to wrong nation**
- **Cause:** Incorrect or missing bilateral claim
- **Fix:** Verify `initialOwner: true` in correct bilateral entry

**Issue: Regions overlap or are misaligned**
- **Cause:** Coordinate conversion errors
- **Fix:** Verify lat/lon to Vector3 conversion formula, check coordinate order

### Debug Tips

1. **Enable Unity Debug Console**
   - Look for error messages related to region loading
   - Check for missing references or null values

2. **Test with minimal changes**
   - Start with one simple region
   - Add complexity incrementally

3. **Compare with existing regions**
   - Export dumps of existing regions
   - Use as templates for structure

4. **Verify file integrity**
   - Use JSON validators for template files
   - Check AssetBundle loads without errors

5. **Check file permissions**
   - Ensure game can read modified files
   - Run as administrator if needed

## External Resources

### Official and Community Guides

- **[Steam Guide: Creating Regions and Nations](https://steamcommunity.com/sharedfiles/filedetails/?id=2965074107)**
  Comprehensive step-by-step tutorial with screenshots

- **[Steam Guide: Introduction to Editing Template Files](https://steamcommunity.com/sharedfiles/filedetails/?id=2905346265)**
  Overview of template system and JSON modding

- **[Creating Template JSON Mods](Create_Template_JSON_mod.md)**
  Local guide for template modification basics

- **[Map and Region Data Extraction Guide](Map%20and%20Region%20Data%20Extraction.md)**
  Detailed guide for extracting existing region data

### Tools and Resources

- **[geojson.io](https://geojson.io)**
  Web-based GeoJSON editor for drawing regions

- **[UABEAvalonia GitHub](https://github.com/nesrak1/UABEA)**
  Unity Assets Bundle Extractor tool

- **[AssetStudio GitHub](https://github.com/Perfare/AssetStudio)**
  Asset viewing and extraction tool

- **[Unity Hub](https://unity.com/download)**
  Download page for Unity (select Archive for 2020.3.48f1)

- **[Inkscape](https://inkscape.org/)**
  Free SVG editor

### Example Projects

- **[SolarInvicta Mod](https://github.com/slyhw4/SolarInvicta)**
  Example mod that creates custom solar system regions

- **[ti-mods Repository](https://github.com/TROYTRON/ti-mods)**
  Collection of Terra Invicta modding resources

### Technical Documentation

- **[GeoJSON Specification](https://geojson.org/geojson-spec.html)**
  Official GeoJSON format documentation

- **[Unity Manual - AssetBundles](https://docs.unity3d.com/Manual/AssetBundlesIntro.html)**
  Unity's documentation on AssetBundles

### Community Resources

- **[Terra Invicta Official Wiki](https://hoodedhorse.com/wiki/Terra_Invicta)**
  Official game documentation

- **[Terra Invicta Subreddit](https://reddit.com/r/TerraInvicta)**
  Community discussions and support

- **[Pavonis Interactive Discord](https://discord.gg/terrainvicta)**
  Official Discord for developer interaction

## Advanced Topics

### Optimizing Region Complexity

- Use polygon simplification algorithms
- Target 50-150 points for most regions
- Reserve 200+ points only for complex coastlines
- Test performance with many regions active

### Creating Formable Nations

Combine custom regions into formable nations:
1. Define regions in templates
2. Create unification requirements in TINationTemplate.json
3. Add localization strings
4. Test unification mechanics

### Planet-Scale Mods

For mods like SolarInvicta (adding Mars, Moon, etc.):
1. Create new AssetBundles for each celestial body
2. Define new coordinate systems
3. Add body-specific templates
4. Integrate with game mechanics

### Scripting Coordinate Conversion

Batch process many regions using Python/JavaScript:
- Parse GeoJSON collections
- Convert all features to Vector3
- Generate complete dump files automatically
- Validate output format

---

**Last Updated:** December 2024
**Game Version:** Build 0.3.111+
**Unity Version:** 2020.3.48f1
