---
title: Tilemaps and Tiled Integration
description: Working with tilemaps and loading Tiled editor maps in Babylon.js 2D
keywords: 2d, tilemap, tiled, tiles, level design
---

# Tilemaps and Tiled Integration

The tilemap system renders large tile-based worlds efficiently and integrates with the [Tiled](https://www.mapeditor.org/) level editor.

## Tilemap2D

A `Tilemap2D` holds one or more `TilemapLayer2D` layers, plus metadata about tile sizes and tilesets.

### Loading from Tiled

The recommended workflow is to design levels in Tiled, export as `.tmj` (JSON), and load at runtime:

```typescript
import { Tilemap2D } from "@babylonjs/2d";
import { Texture, Engine } from "@babylonjs/core";

// Load tileset textures
const tileset = new Texture("tiles.png", engine);
const textures = new Map([["tiles.png", tileset]]);

// Fetch and parse the Tiled JSON
const response = await fetch("level1.tmj");
const tiledData = await response.json();

const tilemap = Tilemap2D.FromTiled(tiledData, textures);
```

### Tiled Format Support

The loader supports:
- **Tile layers** — Grid-based tile data
- **Object layers** — Named points, rectangles for spawn positions, triggers, etc.
- **Collision layers** — Layers with a `collision: true` property are used for tile-based collision
- **Multiple tilesets** — Each tileset maps to a texture

### Tile Queries

```typescript
// Get tile ID at a grid position
const tileId = tilemap.getTileAt("ground", 3, 5);

// Convert between world coordinates and tile coordinates
const { col, row } = tilemap.worldToTile(worldX, worldY);
const { x, y } = tilemap.tileToWorld(col, row);
```

## Collision Layers

Mark a Tiled layer as a collision layer by adding a custom property `collision = true` in the Tiled editor. Then query solid tiles:

```typescript
// Is a specific tile position solid?
if (tilemap.isSolid(col, row)) {
    // Block movement
}

// Get all solid tiles in a region (useful for broad-phase)
const solidTiles = tilemap.getCollisionTilesInRegion(startCol, startRow, endCol, endRow);
```

## Object Layers

Access named objects placed in Tiled:

```typescript
const spawnPoint = tilemap.getObject("PlayerSpawn");
if (spawnPoint) {
    player.position.x = spawnPoint.x;
    player.position.y = spawnPoint.y;
}

// Get all objects of a type
const enemies = tilemap.getObjectsByType("enemy");
for (const obj of enemies) {
    spawnEnemy(obj.x, obj.y, obj.properties);
}
```

## TilemapLayer2D

Each layer stores a flat array of tile IDs and can be queried individually:

```typescript
const groundLayer = tilemap.layers[0];
console.log(groundLayer.name);         // "ground"
console.log(groundLayer.width);        // Width in tiles
console.log(groundLayer.height);       // Height in tiles
console.log(groundLayer.isCollision);  // true if collision layer
```

## Manual Construction

You can create tilemaps programmatically without Tiled:

```typescript
import { Tilemap2D, TilemapLayer2D } from "@babylonjs/2d";

const tiles = [
    1, 1, 1, 1, 1,
    1, 0, 0, 0, 1,
    1, 0, 0, 0, 1,
    1, 1, 1, 1, 1,
];

const layer = new TilemapLayer2D("walls", 5, 4, tiles);
layer.isCollision = true;

const tilemap = new Tilemap2D(5, 4, 32, 32, [layer], new Map(), new Map());
```

## Tips

- Keep tile sizes power-of-two (16, 32, 64) for best GPU texture sampling
- Use separate layers for visual tiles and collision — this lets artists iterate visuals without affecting gameplay
- Object layers are great for level metadata (spawn points, camera bounds, trigger zones)
- Large maps work well because only visible tiles need to be rendered

## Animated Tiles

Tiled supports per-tile animations (e.g., water, lava, torches). The 2D engine
parses this data automatically and provides an `update()` / `getDisplayTileId()`
workflow to drive frame cycling.

### Tiled Setup

In the Tiled editor, select a tile in your tileset and open the **Animation
Editor** (View → Tile Animation Editor). Add frames and set durations in
milliseconds. When you export to `.tmj`, each animated tile generates a
`tiles[].animation` array.

### Runtime Usage

Call `tilemap.update(dt)` once per frame to advance all tile animations. Then
use `getDisplayTileId(gid)` to resolve the current display GID when rendering:

```typescript
engine.runRenderLoop(() => {
    const dt = engine.getDeltaTime() / 1000;
    tilemap.update(dt);

    // When rendering each tile sprite:
    for (const tile of visibleTiles) {
        const displayGid = tilemap.getDisplayTileId(tile.gid);
        // Update the sprite's source rect based on displayGid...
        tile.sprite.sourceRect = getTileSourceRect(displayGid);
    }

    scene.render();
});
```

### Programmatic Animations

You can also define animations in code without a Tiled file:

```typescript
import { Tilemap2D } from "@babylonjs/2d";

// GID 5 cycles through GIDs 5, 6, 7 at 200ms per frame
tilemap.addTileAnimation(5, [
    { gid: 5, duration: 200 },
    { gid: 6, duration: 200 },
    { gid: 7, duration: 200 },
]);
```

### API

| Method | Description |
| ------ | ----------- |
| `update(deltaTime)` | Advance all animations by `deltaTime` seconds |
| `getDisplayTileId(gid)` | Returns current frame GID for animated tiles, or the input GID unchanged |
| `addTileAnimation(gid, frames)` | Register an animation programmatically |
| `tileAnimations` | `Map<number, ITileAnimation>` — read/inspect active animations |
