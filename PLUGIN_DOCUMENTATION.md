# InteractiveGrandStrategyMap Plugin - User Documentation

## Table of Contents
1. [Overview](#overview)
2. [Features](#features)
3. [Plugin Installation](#plugin-installation)
4. [Map Generation Workflow](#map-generation-workflow)
5. [Loading Your Own Data Files and Lookup Texture](#loading-your-own-data-files-and-lookup-texture)
6. [Integrating Map into Project](#integrating-map-into-project)
7. [Setting Up UI](#setting-up-ui)
8. [Plugin Hooks & Delegates](#plugin-hooks--delegates)

## Overview

The **InteractiveGrandStrategyMap** plugin provides a framework for creating interactive maps similar to those in grand strategy games like Europa Universalis 4 and Crusader Kings. It enables procedural map generation using Voronoi diagrams, interactive tile systems, dynamic textures, and data-driven gameplay.

### Supported Versions
- **Unreal Engine:** 5.4+
- **Platform:** Windows 64-bit


## Features

- **Procedural Map Generation** - Voronoi-based tile layouts from heightmaps
- **Interactive Tiles** - Clickable provinces with custom data structures
- **Dynamic Textures** - Real-time texture updates via compute shaders
- **Map Modes** - Switch between visual representations (Political, Terrain, etc.)
- **Camera Controllers** - Pre-configured BirdEye and Globe controllers
- **Blueprint Integration** - Full Blueprint support for integration into projects

## Key Concepts

| Name | Type | Purpose |
|------|------|---------|
| **Map Object** | Asset (`UMapObject`) | Stores generated tile data, lookup data, and visual property setup |
| **ClickableMap** | Runtime actor | Uses a Map Object asset to render and interact with the map at runtime |
| **Visual Property** | UObject/class | Defines how tile data is translated into colors/material behavior for a map mode |
| **HUD Widget** | UI widget | Displays tile data and map mode controls at runtime |


## Plugin Installation

### Step 1: Enable the Plugin

1. Open your Unreal Engine project
2. Go to **Edit → Plugins**
3. Search for **"InteractiveGrandStrategyMap"**
4. Check **Enabled** checkbox
5. The plugin will be loaded

### Step 2: Configure Project Settings

The plugin requires the following project settings to be enabled:

#### Enable Support UV from Hit Results
1. Go to **Edit → Project Settings**
2. Search for **"Support UV from Hit Results"**
3. Check the **Enabled** checkbox
4. This is required for tile selection via raycasting

#### Enable Virtual Texturing (Optional but Recommended)
1. In **Project Settings**, search for **"Virtual Texture"**
2. Enable **Enable Virtual Texture Support**
3. This is used for large-scale map terrain textures
4. Recommended if using the example maps or large textures

### Step 3: Restart the Editor

Restart the editor to apply all changes and load the plugin fully.

## Map Generation Workflow

This section covers the process of generating a map in the editor before integrating it into your game. This step is optional if you already have the following files:
- Lookup table: a JSON file containing IDs and a unique color for each tile
- Data table (JSON) holding the data for each tile in map (data and the struct type name)
- Lookup texture

For information on importing existing files instead, check [Loading your own data files and lookup texture](#loading-your-own-data-files-and-lookup-texture)

### Prerequisites

Before you can generate a map, you need to prepare the following assets:

#### 1. Heightmap Texture (Required)
A 2D grayscale texture that defines the land/water distribution:
- **CompressionSettings**: UserInterface2D
- **SRGB:** Disabled (unchecked in texture settings)
- **Size:** Ideally should be power of 2 (512, 1024, 2048, 4096)
- **Content:** Grayscale values where brightness determines land/ocean (configurable via Cutoff Height)

#### 2. Border Texture (Optional)
If you want to upload custom borders for land or ocean tiles:
- **Same format requirements as heightmap**
- **Purpose:** The algorithm will generate tiles for one type (land or ocean) based on your heightmap, then use the border texture to define the borders
- **Note:** You can only upload borders for either land OR ocean, not both

### Step 1: Create the Custom Tile Data Structure

Define what data each tile will contain:

1. In Content Browser: **Right-click → Structure**
2. Create a new Blueprint Structure (e.g., `S_ProvinceData`)
3. Add property ID (as an integer)
4. Add any other properties your tiles will have

Define another struct if you wish for ocean tiles.
These structures can also be defined in C++.

> **Note:** If using a C++ struct as map data, do not use Live Coding when adding or removing fields. Restart the editor instead.

### Step 2: Create a Map Object Asset

The Map Object stores the generated map data:

1. In Content Browser: **Right-click → MapEditor**
2. Select **Map Object** (class `UMapObject`) from the dropdown
3. Name it (e.g., `DA_MyMapData`)
4. Open the asset

### Step 3: Configure Map Generation Parameters

All map generation is controlled through parameters available in the MapObject:

#### General Settings

| Parameter | Type | Default | Purpose |
|-----------|------|---------|---------|
| **Origin Texture** | Texture2D | None | **REQUIRED** - Your heightmap texture |
| **Cutoff Height** | Float (0.0-1.0) | 0.5 | Brightness threshold for land/ocean separation |
| **Land Below Cutoff** | Boolean | False | If true, darker areas = land; if false, brighter areas = land |
| **Upload Border** | Boolean | False | Whether to use a custom border texture |
| **Border Texture** | Texture2D | None | Custom border texture (if Upload Border is enabled) |

#### Land Details

Configure how land tiles are generated:

| Parameter | Type | Range | Purpose |
|-----------|------|-------|---------|
| **Number Of Tiles** | Int32 | 1-100000 | Target number of land tiles to generate |
| **Seed** | Int32 | Any | Random seed (change for different layouts with same heightmap) |
| **Lloyd Iteration** | Int32 | 0-20 | Relaxation iterations - higher = more regular, uniform tiles |
| **Search Radius** | Int32 | 1-1000 | Area to search before adding a new tile |

#### Ocean Details

Same structure as Land Details, controls water tile generation separately. You can have different numbers of land and ocean tiles.

#### Noise Details

Controls the roughness and appearance of tile borders:

| Parameter | Type | Range | Purpose |
|-----------|------|-------|---------|
| **Seed** | Int32 | Any | Noise randomization seed |
| **Octaves** | Int32 | 0-16 | Noise complexity (higher = more detailed borders) |
| **Frequency** | Float | 0.0-1.0 | Noise scale/zoom level |
| **Scale** | Float | 0.5-3.0 | Noise amplitude (how extreme the variations are) |
| **Line Thickness** | Float | 0.1-7.0 | Visual thickness of tile borders |

### Step 4: Generate the Map
1. Decide if you want to upload borders or not
2. Set the heightmap texture
3. Set cutoff height; this determines which pixels are considered land or ocean
4. Set data structures for ocean and land tiles. If you do not need this separation, set cutoff height to 0 so that all pixels are treated as land and only land tiles are generated
5. In the Debug category, you can set the log level of map generation. Increasing it will reduce log output and speed up map generation
6. In the Map Object editor, use the Generate Map action to run generation. This may take some time depending on the number of tiles.

There is no guarantee that the number of tiles will match the provided value exactly; it is best treated as a target.

### Step 5: Customize Tile Data (Optional)

After generation, you can edit individual tile data:

1. Open your **Map Object** asset
2. The asset editor shows all tiles and their data, you can filter by land tiles and ocean tiles by interacting witht the preview images.
3. Select individual tiles (through the list or by clicking tiles) and modify their properties. You can also select multiple tiles and batch edit them

Tile map data can also be updated from a json file, by using the button `Load Data File` 

### Step 6: Set Up Visual Properties

Visual Properties define how tile data is represented visually:

1. Create a **Visual Property** definition (a blueprint with base class `VisualProperty`)
2. Map data properties to visual outputs:
   - **Example:** Temperature value (float) → Color gradient
   - **Example:** TerrainType (enum) → Specific colors per terrain

3. Add these Visual Properties to your Map Object

> **Important:** You must add at least one visual property.

Some common types are provided, such as a string-to-color map (`StringMapVisualProperty`) etc.

#### Material Requirements

When creating custom materials for visual properties, ensure they include the required parameters:

**Required Parameters:**
- **LookupTexture** - Reference texture used for tile data lookup
- **Threshold** - This param controls the tolerance when you click on a tile, it should be more than 0 but small. From experience, 0.3 is fine and does not lead to false positives. For reference experiment with the provided materials.

**Optional Features:**
- **Tile Highlighting** - To highlight hovered tiles, use the provided `MF_Highlight` material function. This enables visual feedback when the player hovers over tiles.
- **Tile Borders** - Custom border visualization between tiles can be implemented using provided border functions. Refer to example materials for implementation patterns.

**Dynamic Map Modes:**
For materials intended for dynamic map modes (e.g., political maps that update at runtime), ensure they include:
- **DynamicTexture** - Parameter for runtime texture updates via compute shaders

Refer to the example materials provided in the plugin for proper usage patterns.

#### Material Quality & Performance Note

The example materials provided with this plugin are basic implementations offered as starting points. They prioritize functionality over visual polish or performance optimization. You are encouraged to create custom materials tailored to your project's visual requirements and performance targets. Modify these materials as needed while maintaining the required parameters mentioned above.

## Loading your own data files and lookup texture

To populate your `UMapObject` with custom data from JSON files and a specific lookup texture, you can use the **Asset Action** provided by the plugin. 
### Prerequisite

Before running the action, ensure you have:
1.  **Custom Structs:** The struct definitions for your Land and Ocean data types.
2.  **JSON Files:**
   *   A **Map Data** JSON file containing your tile configurations.
   *   A **Lookup Table** JSON file containing the tile IDs and colors
3.  **Lookup Texture:** A `UTexture2D` asset used to map colors to your data.

### Steps

1.  **Select Asset:**
In the Content Browser, select the `UMapObject` asset you wish to update.

2.  **Access Context Menu:**
Right-click the selected asset to open the context menu.

3.  **Select Asset Action:**
Navigate to the **Asset Actions** section and select **Upload Map Files**.

## Integrating Map into Project

### Choosing Your Map Type

The plugin supports two types of interactive maps:

#### 1. Flat Map (FlatMapController)
- **View:** Top-down, orthographic view
- **Use Case:** Traditional strategy game style (Europa Universalis, Civilization)
- **Controller:** FlatMapController

#### 2. Globe Map (GlobeMapController)
- **View:** 3D spherical/orbital view
- **Use Case:** Planetary maps, global perspectives
- **Controller:** GlobeMapController

### Step 1: Choose Controller, Pawn, HUD, and GameMode

The plugin provides the following Blueprints:

**HUD:** `BP_Core_HUD`

#### Flat Map
- **Controller:** `BP_FlatMapController`
- **GameMode:** `BP_Core_FlatMapGameMode`
- **Pawn:** `BP_Core_MapPawn`

#### Globe Map
- **Controller:** `BP_GlobeMapController`
- **GameMode:** `BP_Core_GlobeMapGameMode`
- **Pawn:** `BP_Core_GlobePawn`

Since this is provided as an engine plugin, it is recommended to inherit from these. You can use the provided ones to get started.
For the controllers, the relevant inputs are implemented via input actions in the `Inputs` folder.

### Step 2: Create a Runtime ClickableMap Blueprint

Create the blueprint that will be used in your actual game levels:

1. Create a Map Blueprint and set the base type depending on the map type you want
2. Open the Blueprint
3. In the Details panel, under **Map Parameters**:
   - **Set Map Object:** Select your generated `DA_MyMapData` asset
   - This links your blueprint to the pre-generated map data
4. Set your map mesh on the gameplay map mesh property for your chosen map actor
5. **IMPORTANT** Set the start map mode name. It must match the name of the visual property that you want to display by default

### Step 3: Configure Level Settings

Set up your level to use the correct controller and pawn:

1. Open your game level
2. Under **Game Mode**, select one of the defined game modes or create a Game Mode Blueprint
3. In the Game Mode settings:
   - **Player Controller Class:** Set to your desired controller
     - Use `FlatMapController` for flat maps
     - Use `GlobeMapController` for globe maps
   - **Default Pawn Class:** Set to the pawn compatible with your controller

### Step 4: Place the Map Actor

1. In your level, place an instance of your runtime ClickableMap blueprint
2. Ensure the **Map Object** property is set to your generated map asset
3. The map will be ready to use at runtime

For an example of a level where this is already set up, check the provided levels.

## Setting Up UI

The plugin includes the **PropertyDisplayGenerator** plugin to automatically generate UI for your data structures.
For detailed information on PropertyDisplayGenerator, check [PropertyDisplayGenerator Doc](https://docs.google.com/document/d/1Atnt15vsg1AsZ2PlTOnDwjJgdw_dszwZl24JnJ3NNWo/edit?usp=sharing)

### Asset Actions for UI Generation

To quickly generate all UI widgets required for your interactive map, use the automated asset action:

1. Select your `DA_MyMapData` in the Content Browser
2. Right-click and look for **Asset Actions**
3. Choose **Create Map UI Elements → Create All**

This will automatically create the following widgets using the PropertyDisplayGenerator plugin:

- **Data Display Widgets**: Two widgets (one for land tiles, one for ocean tiles) that display the properties of your custom data structures
- **Tile Selected Widget**: A container widget that combines the land and ocean data displays for showing selected tile information
- **Map Mode Selector Widget**: A widget for switching between different map visualization modes (based on your visual properties)
- **HUD Widget**: A main HUD widget that integrates the map mode selector and tile selected widget

These widgets provide a complete starting point for your map's UI. You can customize them further by modifying the generated blueprints. For more information on customizing the data display widgets, refer to the PropertyDisplayGenerator documentation.

4. Open the HUD widget created by the action and set its layout.
5. In your HUD class, set the `Hud Widget Class` field to the HUD widget created by the action.

This UI setup is a quick starting point, if you wish you can completely ignore this and set up your own UI system. Since this is a very general solution, for your project you probably want something more customizable.

## Plugin Hooks & Delegates

The plugin exposes the following delegates for gameplay, UI, and map data integration.

### AClickableMap Delegates

The map broadcasts these events that you can listen to:

#### OnMapInitialized
Fires when map initialization completes and is ready for use.
```
Signature: void(AClickableMap* Map)
Usage: Bind to this to know when map data is available
```

#### OnMapTileChanged
Fires when one or more tiles are modified.
```
Signature: void(const TArray<int>& TileIDs)
Usage: Bind to update UI or gameplay when provinces change
Parameters: Array of tile IDs that were modified
```

#### OnMapModeChanged
Fires when the map mode switches.
```
Signature: void(FName OldMode, FName NewMode)
Usage: Bind to know when visualization changes
Parameters: Old and new mode names
```

### UMapDataComponent Delegates

The map data component exposes delegates for save/load integration and runtime tile data updates.

#### OnSaveMapData
Fires when the component is ending play and wants to hand off the current runtime tile data to your save system.
```
Signature: void(UMapSaveData* MapSaveData)
Usage: Bind to serialize or store the current runtime map data before the actor/component is destroyed
Parameters: A `UMapSaveData` object containing the current `SavedMapData` map
```

#### OnLoadMapData
Called when the component is initialized with a map object and wants to restore saved runtime tile data.
```
Signature: UMapSaveData*()
Usage: Provide a previously saved `UMapSaveData` instance to restore runtime state instead of using the asset defaults
Return Value: Return `nullptr` to fall back to the `UMapObject` asset data
```

### ABirdEyeController Delegates

The base map controller broadcasts these events for input-driven interactions:

#### TileClickedDelegate
Fires when the player clicks a valid tile.
```
Signature: void(int TileID, FInstancedStruct Data)
Usage: Bind to react to tile selection and update UI/gameplay with tile data
Parameters: Tile ID and the clicked tile's instanced data payload
```

#### TileHoveredDelegate
Fires when the player hovers a tile.
```
Signature: void(FColor Color, int TileID)
Usage: Bind to show hover previews, tooltips, or highlight-related UI
Parameters: Hovered tile lookup color and tile ID
```

#### MapClickedDelegate
Fires when the player clicks the map area but not on a valid tile.
```
Signature: void()
Usage: Bind to clear selection state or hide tile-specific UI panels
```



