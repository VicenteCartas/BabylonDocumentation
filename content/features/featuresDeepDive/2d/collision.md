---
title: 2D Collision System
description: Collision shapes, spatial partitioning, and collision queries in Babylon.js 2D
keywords: 2d, collision, shapes, aabb, circle, polygon, spatial grid, raycast
---

# 2D Collision System

The collision system provides shape-based overlap testing, spatial partitioning for performance, and raycast queries.

## Collision Shapes

Three shape types are available:

### BoxCollider2D

An axis-aligned bounding box (AABB):

```typescript
import { BoxCollider2D } from "@babylonjs/2d";

const box = new BoxCollider2D(32, 32);           // 32x32 pixels
const offset = new BoxCollider2D(16, 16, new Vector2(0, -8)); // with offset
```

### CircleCollider2D

```typescript
import { CircleCollider2D } from "@babylonjs/2d";

const circle = new CircleCollider2D(16);  // radius 16 pixels
```

### PolygonCollider2D

A convex polygon:

```typescript
import { PolygonCollider2D } from "@babylonjs/2d";

const triangle = new PolygonCollider2D([
    new Vector2(0, -16),
    new Vector2(16, 16),
    new Vector2(-16, 16),
]);
```

## Collider2D Component

Group shapes with layer/mask filtering:

```typescript
import { Collider2D } from "@babylonjs/2d";

const collider = new Collider2D(
    [new BoxCollider2D(32, 32)],
    1,          // layer bitmask
    0xFFFFFFFF  // mask bitmask (collides with everything)
);
```

**Layer/mask filtering**: Two colliders interact only when `(a.layer & b.mask) !== 0 && (b.layer & a.mask) !== 0`.

## Overlap Test Functions

Low-level functions for testing shape overlap:

```typescript
import { TestBoxBox, TestCircleCircle, TestCircleBox, TestPointBox, TestPointCircle } from "@babylonjs/2d";

// Box vs Box (center-based with half-widths)
TestBoxBox(ax, ay, aHalfW, aHalfH, bx, by, bHalfW, bHalfH);

// Circle vs Circle
TestCircleCircle(ax, ay, aRadius, bx, by, bRadius);

// Circle vs Box
TestCircleBox(cx, cy, cRadius, bx, by, bHalfW, bHalfH);

// Point vs Box
TestPointBox(px, py, bx, by, bHalfW, bHalfH);

// Point vs Circle
TestPointCircle(px, py, cx, cy, cRadius);
```

## Spatial Grid

`SpatialGrid` provides broad-phase spatial partitioning using a uniform grid. It dramatically reduces the number of narrow-phase collision tests needed.

```typescript
import { SpatialGrid } from "@babylonjs/2d";

const grid = new SpatialGrid(128); // 128px cell size
```

### Rebuilding Each Frame

The spatial grid is rebuilt each frame with all active entities:

```typescript
function updateCollision(entities: ICollisionEntry[]) {
    grid.clear();
    for (const entry of entities) {
        grid.insert(entry);
    }
}
```

### Queries

```typescript
// Point query — what is at this position?
const hits = grid.queryPoint(mouseX, mouseY);

// Box query — what overlaps this region?
const nearby = grid.queryBox(x, y, halfWidth, halfHeight);

// Circle query — what is within this radius?
const inRange = grid.queryCircle(x, y, radius);

// Raycast — first hit along a direction
const hit = grid.raycast(originX, originY, dirX, dirY, maxDistance);
if (hit) {
    console.log(`Hit at ${hit.point.x}, ${hit.point.y} dist=${hit.distance}`);
}
```

All query methods accept an optional `mask` parameter for layer filtering.

## Typical Game Loop Pattern

```typescript
// 1. Build spatial grid from all entities with colliders
grid.clear();
for (const entity of allEntities) {
    grid.insert({
        node: entity,
        shapes: entity.collider.shapes,
        layer: entity.collider.layer,
        mask: entity.collider.mask,
    });
}

// 2. Query for specific interactions
const playerHits = grid.queryBox(
    player.position.x, player.position.y,
    player.halfWidth, player.halfHeight,
    LAYER_ENEMY  // only check enemies
);

for (const hit of playerHits) {
    handlePlayerEnemyCollision(hit.node);
}
```
