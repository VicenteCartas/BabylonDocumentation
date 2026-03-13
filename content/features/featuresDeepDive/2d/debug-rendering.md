---
title: Debug Rendering
description: Visualizing collision shapes, physics bodies, spatial grids, and pathfinding grids with wireframe overlays
keywords: 2d, debug, wireframe, colliders, physics, spatial grid, pathfinding, overlay
---

# Debug Rendering

`DebugRenderer2D` draws colored wireframe overlays on top of your 2D scene. Use it to visualize collision shapes, physics bodies, spatial grid cells, and pathfinding walkability — all without affecting gameplay rendering.

## Setup

```typescript
import { DebugRenderer2D } from "@babylonjs/2d/Debug/debugRenderer2D";

const debug = new DebugRenderer2D(engine);
scene.debugRenderer = debug;
```

When `scene.debugRenderer` is set, `Scene2D` automatically calls `debug.render()` after sprite rendering each frame.

## Overlay Categories

Toggle each category independently:

```typescript
debug.showColliders = true;        // Collision shape outlines (default: true)
debug.showPhysicsBodies = true;    // Physics body outlines (default: true)
debug.showSpatialGrid = false;     // Spatial grid cell boundaries (default: false)
debug.showPathfindingGrid = false; // Pathfinding walkable/unwalkable cells (default: false)
```

Use the master toggle to disable all overlays at once:

```typescript
debug.enabled = false; // Skips all debug rendering
```

## Connecting Data Sources

Each overlay category reads from a data source that you assign:

```typescript
// Collision shapes from SpatialGrid
debug.spatialGrid = mySpatialGrid;

// Physics body outlines
debug.physicsEngine = myPhysicsEngine;

// Pathfinding grid overlay (requires both pathfinder AND grid)
debug.pathfinder = myPathfinder;
debug.pathfinderGrid = myGrid; // Grid2D or IsometricGrid
```

<Alert severity="info" title="Data sources are optional">
You only need to assign the sources for the overlays you want. For example, if you only care about physics debugging, just set `physicsEngine` — the other overlays will be silently skipped.
</Alert>

## Customizing Colors

Every overlay category has configurable colors:

```typescript
import { Color4 } from "@babylonjs/core/Maths/math.color";

// Collision shape outlines
debug.colliderColor = new Color4(0, 1, 0, 1);         // Green

// Physics bodies (colored by body type)
debug.physicsStaticColor = new Color4(0.5, 0.5, 0.5, 1);   // Gray
debug.physicsDynamicColor = new Color4(0, 0.7, 1, 1);      // Cyan
debug.physicsKinematicColor = new Color4(1, 0.5, 0, 1);    // Orange

// Spatial grid cells
debug.spatialGridColor = new Color4(0.3, 0.3, 0.3, 0.5);  // Faint gray

// Pathfinding grid
debug.walkableColor = new Color4(0, 0.4, 0, 0.3);          // Green tint
debug.unwalkableColor = new Color4(0.6, 0, 0, 0.3);        // Red tint
```

## Properties Reference

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `enabled` | `boolean` | `true` | Master toggle for all debug rendering |
| `showColliders` | `boolean` | `true` | Draw collision shape outlines |
| `showPhysicsBodies` | `boolean` | `true` | Draw physics body outlines |
| `showSpatialGrid` | `boolean` | `false` | Draw spatial grid cell boundaries |
| `showPathfindingGrid` | `boolean` | `false` | Draw pathfinding walkability |
| `spatialGrid` | `SpatialGrid \| null` | `null` | Data source for colliders and grid cells |
| `physicsEngine` | `IPhysicsEngine2D \| null` | `null` | Data source for physics bodies |
| `pathfinder` | `AStarPathfinder \| null` | `null` | Data source for pathfinding overlay |
| `pathfinderGrid` | `Grid2D \| IsometricGrid \| null` | `null` | Grid for pathfinding coordinate conversion |

## Drawing Custom Lines

`DebugRenderer2D` also exposes immediate-mode drawing methods you can call from `onBeforeRender` to draw custom debug shapes:

```typescript
scene.onBeforeRender.add(() => {
    // Draw a line between two points
    debug.drawLine(100, 100, 200, 200, new Color4(1, 1, 0, 1));

    // Draw a wireframe rectangle (center, half-extents)
    debug.drawRect(300, 200, 50, 30, new Color4(1, 0, 1, 1));

    // Draw a wireframe circle
    debug.drawCircle(400, 300, 40, new Color4(0, 1, 1, 1));

    // Draw a wireframe polygon
    debug.drawPolygon([
        new Vector2(100, 300),
        new Vector2(150, 280),
        new Vector2(160, 320),
        new Vector2(120, 340),
    ], new Color4(1, 1, 1, 1));
});
```

### Drawing Methods

| Method | Parameters | Description |
|--------|-----------|-------------|
| `drawLine` | `x1, y1, x2, y2, color` | Line segment between two points |
| `drawRect` | `x, y, halfWidth, halfHeight, color` | Wireframe rectangle from center |
| `drawCircle` | `cx, cy, radius, color, segments?` | Wireframe circle (default 32 segments) |
| `drawPolygon` | `vertices, color` | Wireframe polygon from `Vector2[]` |
| `drawCrossHatchRect` | `x, y, halfWidth, halfHeight, color` | Two diagonal lines (fill indicator) |

## Example: Full Debug Setup

```typescript
import { DebugRenderer2D } from "@babylonjs/2d/Debug/debugRenderer2D";
import { SpatialGrid } from "@babylonjs/2d/Collision/spatialGrid";
import { AStarPathfinder } from "@babylonjs/2d/Pathfinding/aStarPathfinder";
import { Grid2D } from "@babylonjs/2d/Grid/grid2D";

// Create the debug renderer
const debug = new DebugRenderer2D(engine);

// Wire up data sources
debug.spatialGrid = spatialGrid;
debug.physicsEngine = physicsEngine;
debug.pathfinder = pathfinder;
debug.pathfinderGrid = grid;

// Enable specific overlays
debug.showColliders = true;
debug.showPhysicsBodies = true;
debug.showSpatialGrid = true;
debug.showPathfindingGrid = true;

// Attach to scene
scene.debugRenderer = debug;

// Toggle with a key press
input.defineAction("toggleDebug", { key: "F3" });

engine.runRenderLoop(() => {
    if (input.isActionPressed("toggleDebug")) {
        debug.enabled = !debug.enabled;
    }
    scene.render();
});
```

<!-- Playground about debug rendering with colliders, physics bodies, and pathfinding grid goes here -->

## Performance

The debug renderer uses a single GPU draw call per frame (GL_LINES). The default line capacity is 16,384 segments per frame, which is sufficient for most scenes. If you have an extremely large number of shapes, you can increase the capacity via the constructor:

```typescript
const debug = new DebugRenderer2D(engine, 32768); // 32k line segments
```

## Cleanup

```typescript
debug.dispose(); // Releases GPU buffers and shaders
scene.debugRenderer = null;
```

## Related

- [Collision Detection](/features/featuresDeepDive/2d/collision) — SpatialGrid and collision shapes
- [Physics](/features/featuresDeepDive/2d/physics) — Physics bodies and the IPhysicsEngine2D interface
- [Pathfinding](/features/featuresDeepDive/2d/pathfinding) — AStarPathfinder and walkability grids
- [Grid System](/features/featuresDeepDive/2d/grid-system) — Grid2D coordinate conversion
