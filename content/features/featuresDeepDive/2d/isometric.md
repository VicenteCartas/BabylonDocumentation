---
title: Isometric Games
description: Building isometric games with IsometricGrid, depth sorting, and coordinate conversion
keywords: 2d, isometric, diamond, staggered, depth sorting, tiles
---

# Isometric Games

The `IsometricGrid` class provides coordinate conversion and utilities for building isometric games (SimCity, Baldur's Gate, Age of Empires style).

## Setup

```typescript
import { IsometricGrid, IsometricOrientation } from "@babylonjs/2d";

// Diamond isometric (most common: Diablo, AoE)
const isoGrid = new IsometricGrid(
    40, 40,    // grid size in tiles
    64, 32,    // tile size: 64px wide, 32px tall (2:1 ratio)
    IsometricOrientation.Diamond
);
```

## Orientations

| Orientation | Description | Example Games |
|------------|-------------|---------------|
| `Diamond` | Tiles rotated 45° in diamond layout | Diablo, Age of Empires |
| `Staggered` | Offset columns/rows | SimCity 2000 |

## Coordinate Conversion

Convert between tile coordinates and pixel positions:

```typescript
// Tile → World (returns center of tile)
const worldPos = isoGrid.tileToWorld(5, 3);

// World → Tile (returns nearest tile)
const tile = isoGrid.worldToTile(mouseWorldX, mouseWorldY);
```

### Diamond Coordinate System

In diamond isometric:
- (col+1) moves right and down on screen
- (row+1) moves left and down on screen
- The formula: `x = (col - row) * tileWidth/2`, `y = (col + row) * tileHeight/2`

## Depth Sorting

Isometric games require correct draw order (painter's algorithm). Use `getDepth()` to compute z-order:

```typescript
// Sort entities by depth before rendering
entities.sort((a, b) => {
    const tileA = isoGrid.worldToTile(a.position.x, a.position.y);
    const tileB = isoGrid.worldToTile(b.position.x, b.position.y);
    return isoGrid.getDepth(tileA.col, tileA.row) - isoGrid.getDepth(tileB.col, tileB.row);
});

// Or set zIndex on Node2D
for (const entity of entities) {
    const tile = isoGrid.worldToTile(entity.position.x, entity.position.y);
    entity.zIndex = isoGrid.getDepth(tile.col, tile.row);
}
```

## View Culling

Only render tiles visible on screen:

```typescript
const visible = isoGrid.getVisibleTiles(
    0, 0,                    // screen rect origin
    canvas.width, canvas.height,  // screen rect size
    camera.position.x - camera.viewportWidth / 2,  // camera offset
    camera.position.y - camera.viewportHeight / 2
);

for (const { col, row } of visible) {
    const tileId = tilemap.getTileAt("ground", col, row);
    if (tileId > 0) {
        renderTile(col, row, tileId);
    }
}
```

## Neighbors

```typescript
const neighbors = isoGrid.getNeighbors(5, 5);
// Returns 4 cardinal neighbors in iso space (N, E, S, W)
```

## Integration with Pathfinding

Combine `IsometricGrid` with `AStarPathfinder`:

```typescript
const pathfinder = new AStarPathfinder({
    width: isoGrid.width,
    height: isoGrid.height,
    isWalkable: (col, row) => !isoMap.isSolid(col, row),
    allowDiagonal: false, // Usually no diagonal in iso games
});

// Click to move
const target = isoGrid.worldToTile(clickWorldX, clickWorldY);
const unit = isoGrid.worldToTile(selectedUnit.position.x, selectedUnit.position.y);
const path = pathfinder.findPath(unit.col, unit.row, target.col, target.row);

// Convert path to world positions for movement
const worldPath = path.map(p => isoGrid.tileToWorld(p.col, p.row));
```

## Tips

- Use 2:1 tile ratio (e.g., 64×32, 128×64) for standard diamond isometric
- For entities taller than one tile (trees, buildings), offset the sprite anchor and adjust depth
- Camera panning works naturally — just update Camera2D position
- Z-ordering must be recalculated whenever entities move between tiles
