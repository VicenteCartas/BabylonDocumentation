---
title: Turn-Based Games
description: Building turn-based strategy games with grid, pathfinding, and movement range
keywords: 2d, turn-based, strategy, tactics, grid, movement range
---

# Turn-Based Games

This guide covers building turn-based strategy games (Fire Emblem, Civilization, Into the Breach style) using the 2D engine's grid system, pathfinding, and sprite rendering.

## Core Systems

Turn-based games combine several 2D engine features:

| System | Classes | Purpose |
|--------|---------|---------|
| Grid | `Grid2D` | Coordinate conversion, neighbors, range queries |
| Pathfinding | `AStarPathfinder` | Movement paths, movement range display |
| Rendering | `Sprite2D`, `Camera2D` | Unit rendering, map scrolling |
| Input | `InputMap2D` | Tile selection, unit commands |

## Grid Setup

```typescript
import { Grid2D, GridTopology, AStarPathfinder } from "@babylonjs/2d";

// Square grid for tactics
const grid = new Grid2D(20, 15, 48, GridTopology.Square);

// Or hex grid for Civ-style
const hexGrid = new Grid2D(30, 20, 36, GridTopology.HexPointyTop);
```

## Turn Management

A simple turn manager pattern:

```typescript
interface ITurnState {
    currentTeam: number;
    phase: "select" | "move" | "attack" | "end";
    selectedUnit: IUnit | null;
    reachableCells: Array<{ col: number; row: number; cost: number }>;
}

function nextTurn(state: ITurnState): void {
    state.currentTeam = state.currentTeam === 0 ? 1 : 0;
    state.phase = "select";
    state.selectedUnit = null;
    state.reachableCells = [];
    // Reset all units' moved/attacked flags for new team
}
```

## Movement Range

Show where a unit can move using `getReachableCells`:

```typescript
const pathfinder = new AStarPathfinder({
    width: grid.width,
    height: grid.height,
    isWalkable: (col, row) => !isOccupied(col, row) && terrain[row][col] !== "wall",
    getCost: (col, row) => terrainCost[terrain[row][col]] ?? 1,
});

// When a unit is selected, compute reachable cells
const reachable = pathfinder.getReachableCells(unit.col, unit.row, unit.movePoints);

// Highlight reachable cells
for (const cell of reachable) {
    const worldPos = grid.cellToWorld(cell.col, cell.row);
    drawHighlight(worldPos.x, worldPos.y, "blue");
}
```

## Attack Range

Use `Grid2D.getCellsInRange` for attack range:

```typescript
const attackRange = grid.getCellsInRange(unit.col, unit.row, unit.attackRange);

for (const cell of attackRange) {
    if (hasEnemy(cell.col, cell.row)) {
        const worldPos = grid.cellToWorld(cell.col, cell.row);
        drawHighlight(worldPos.x, worldPos.y, "red");
    }
}
```

## Click-to-Move

```typescript
input.update();
if (input.isActionPressed("select")) {
    const worldPos = input.pointerWorldPosition;
    const cell = grid.worldToCell(worldPos.x, worldPos.y);

    if (state.phase === "select") {
        const unit = getUnitAt(cell.col, cell.row);
        if (unit && unit.team === state.currentTeam && !unit.hasMoved) {
            state.selectedUnit = unit;
            state.reachableCells = pathfinder.getReachableCells(
                unit.col, unit.row, unit.movePoints
            );
            state.phase = "move";
        }
    } else if (state.phase === "move") {
        const isReachable = state.reachableCells.some(
            c => c.col === cell.col && c.row === cell.row
        );
        if (isReachable) {
            moveUnit(state.selectedUnit!, cell.col, cell.row);
            state.phase = "attack";
        }
    }
}
```

## Line of Sight

Check if a ranged unit can see its target:

```typescript
if (pathfinder.hasLineOfSight(archer.col, archer.row, target.col, target.row)) {
    // Can attack
}
```

## Animated Movement

Smoothly move units along a path:

```typescript
async function animateMovement(unit: Sprite2D, path: IPathPoint[]): Promise<void> {
    for (const step of path) {
        const target = grid.cellToWorld(step.col, step.row);
        // Lerp position over a few frames
        await tweenTo(unit, target, 0.15);
    }
}
```

## Hex Grid Tactics

For hex-based tactics (Battle for Wesnoth style):

```typescript
const hexGrid = new Grid2D(15, 12, 40, GridTopology.HexPointyTop);
const hexPathfinder = new AStarPathfinder({
    width: hexGrid.width,
    height: hexGrid.height,
    isWalkable: (col, row) => terrain[row][col] !== "mountain",
});

// Hex neighbors for adjacency checks
const adjacent = hexGrid.getNeighbors(unit.col, unit.row);
const adjacentEnemies = adjacent.filter(n => hasEnemy(n.col, n.row));
```

## Tips

- Pre-compute movement ranges when a unit is selected, not every frame
- Cache pathfinding results — the grid doesn't change during a unit's turn
- Use `Grid2D.distance()` for quick range checks before expensive pathfinding
- For AI, evaluate all possible moves and use scoring heuristics to pick the best
