---
title: Pathfinding
description: A* pathfinding on grids with movement range and line of sight queries
keywords: 2d, pathfinding, astar, grid, movement range, line of sight
---

# Pathfinding

The 2D engine provides an A* pathfinder optimized for grid-based 2D games.

## Setup

```typescript
import { AStarPathfinder } from "@babylonjs/2d";

const pathfinder = new AStarPathfinder({
    width: tilemap.width,
    height: tilemap.height,
    isWalkable: (col, row) => !tilemap.isSolid(col, row),
});
```

## Finding a Path

```typescript
const path = pathfinder.findPath(startCol, startRow, endCol, endRow);

if (path.length > 0) {
    // path is an array of { col, row } from start to end
    for (const point of path) {
        console.log(`Step: ${point.col}, ${point.row}`);
    }
} else {
    console.log("No path found!");
}
```

## Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `width` | `number` | required | Grid width in cells |
| `height` | `number` | required | Grid height in cells |
| `isWalkable` | `(col, row) => boolean` | required | Returns false for blocked cells |
| `getCost` | `(col, row) => number` | `() => 1` | Movement cost for entering a cell |
| `allowDiagonal` | `boolean` | `false` | Allow 8-directional movement |
| `heuristic` | `(ax, ay, bx, by) => number` | Manhattan/Octile | Custom heuristic function |

## Diagonal Movement

Enable diagonal movement for smoother paths:

```typescript
const pathfinder = new AStarPathfinder({
    width: 20,
    height: 20,
    isWalkable: (col, row) => !blocked[row][col],
    allowDiagonal: true,
});
```

When diagonal is enabled:
- Uses octile distance heuristic (instead of Manhattan)
- Diagonal moves cost √2 (≈1.41) instead of 1
- Corner-cutting is prevented (both adjacent cardinal cells must be walkable)

## Weighted Costs

Simulate terrain types with different movement costs:

```typescript
const pathfinder = new AStarPathfinder({
    width: mapWidth,
    height: mapHeight,
    isWalkable: (col, row) => terrain[row][col] !== "wall",
    getCost: (col, row) => {
        switch (terrain[row][col]) {
            case "road": return 0.5;   // Faster
            case "grass": return 1.0;  // Normal
            case "swamp": return 3.0;  // Slow
            default: return 1.0;
        }
    },
});
```

## Movement Range

For turn-based games, get all reachable cells within a movement budget:

```typescript
const reachable = pathfinder.getReachableCells(unitCol, unitRow, movementPoints);

for (const cell of reachable) {
    highlightCell(cell.col, cell.row, cell.cost);
}
```

This returns an array of `{ col, row, cost }` — useful for highlighting movement range on a grid.

## Line of Sight

Check if two cells have an unobstructed line between them (Bresenham's line algorithm):

```typescript
if (pathfinder.hasLineOfSight(archerCol, archerRow, targetCol, targetRow)) {
    // Can attack!
}
```

## Integration with Tilemap

Tilemaps and pathfinding work together naturally:

```typescript
// Create pathfinder from tilemap collision data
const pathfinder = new AStarPathfinder({
    width: tilemap.width,
    height: tilemap.height,
    isWalkable: (col, row) => !tilemap.isSolid(col, row),
});

// Convert world positions to tile coordinates for pathfinding
const start = tilemap.worldToTile(entity.position.x, entity.position.y);
const end = tilemap.worldToTile(targetX, targetY);
const path = pathfinder.findPath(start.col, start.row, end.col, end.row);

// Convert path back to world positions for movement
const worldPath = path.map(p => tilemap.tileToWorld(p.col, p.row));
```
