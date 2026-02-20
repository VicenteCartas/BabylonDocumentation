---
title: Grid System
description: Square and hexagonal grid utilities for 2D games
keywords: 2d, grid, hex, hexagonal, square, coordinate conversion
---

# Grid System

`Grid2D` provides coordinate conversion, neighbor queries, distance calculations, and range queries for square and hexagonal grids.

## Setup

```typescript
import { Grid2D, GridTopology } from "@babylonjs/2d";

// Square grid: 20x15 cells, 32px each
const squareGrid = new Grid2D(20, 15, 32, GridTopology.Square);

// Hex grid (flat-top): 10x10 cells, 32px radius
const hexGrid = new Grid2D(10, 10, 32, GridTopology.HexFlatTop);

// Hex grid (pointy-top)
const hexGridPointy = new Grid2D(10, 10, 32, GridTopology.HexPointyTop);
```

## Grid Topologies

| Topology | Description | Use Case |
|----------|-------------|----------|
| `Square` | Standard rectangular grid | Most tile-based games |
| `HexFlatTop` | Hexagonal, flat edge on top | Strategy games (Civ-style) |
| `HexPointyTop` | Hexagonal, vertex on top | Strategy games (alternate layout) |

## Coordinate Conversion

Convert between grid coordinates and world pixel positions:

```typescript
// Cell → World (returns center of cell)
const worldPos = grid.cellToWorld(3, 5);

// World → Cell (returns nearest cell)
const cell = grid.worldToCell(mouseX, mouseY);
```

## Neighbors

Get adjacent cells:

```typescript
const neighbors = grid.getNeighbors(5, 5);
// Square grid: 4 neighbors (up, right, down, left)
// Hex grid: 6 neighbors
```

Neighbors respect grid bounds — edge cells return fewer neighbors.

## Distance

Calculate grid distance between two cells:

```typescript
const dist = grid.distance(0, 0, 3, 4);
// Square: Manhattan distance = 7
// Hex: hex distance (cube coordinate distance)
```

## Range Queries

Get all cells within a distance:

```typescript
const cells = grid.getCellsInRange(5, 5, 3);
// Returns all cells within 3 steps of (5,5)
```

For square grids, this returns a diamond shape (Manhattan distance). For hex grids, this returns a hex-shaped region.

## Bounds Checking

```typescript
if (grid.inBounds(col, row)) {
    // Cell is valid
}
```

## Hex Grid Coordinate Systems

The Grid2D uses **offset coordinates** (col, row) for its public API, which is the most intuitive for storage and Tiled integration. Internally, it converts to **cube coordinates** for distance and range calculations on hex grids.

## Integration with Pathfinding

Use Grid2D with AStarPathfinder for complete grid-based gameplay:

```typescript
const grid = new Grid2D(20, 15, 32, GridTopology.Square);
const pathfinder = new AStarPathfinder({
    width: grid.width,
    height: grid.height,
    isWalkable: (col, row) => !blocked[row][col],
});

// Click handler: find path to clicked cell
const targetCell = grid.worldToCell(clickX, clickY);
const path = pathfinder.findPath(unitCol, unitRow, targetCell.col, targetCell.row);

// Highlight movement range
const range = pathfinder.getReachableCells(unitCol, unitRow, 4);
for (const cell of range) {
    const worldPos = grid.cellToWorld(cell.col, cell.row);
    drawHighlight(worldPos.x, worldPos.y);
}
```

## Integration with Tilemap

Grid2D and Tilemap2D complement each other — tilemaps handle rendering and tile data, while Grid2D provides game-logic queries:

```typescript
// Grid matches the tilemap dimensions
const grid = new Grid2D(tilemap.width, tilemap.height, tilemap.tileWidth);
```
