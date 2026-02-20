---
title: Camera2D
description: Using Camera2D for viewport control, follow targets, zoom, and screen shake
keywords: 2d, camera, viewport, follow, zoom, shake
---

# Camera2D

`Camera2D` provides viewport management for 2D scenes. It handles scrolling, following a target node, zoom, bounded regions, screen shake, and coordinate conversion between screen and world space.

## Basic Setup

```typescript
import { Scene2D, Camera2D } from "@babylonjs/2d";
import { Engine } from "@babylonjs/core";

const engine = new Engine(canvas);
const scene = new Scene2D(engine);

const camera = new Camera2D(engine.getRenderWidth(), engine.getRenderHeight());
camera.position.x = 0;
camera.position.y = 0;
```

## Following a Target

The camera can smoothly follow a `Node2D` with configurable smoothing:

```typescript
camera.follow(playerNode, {
    smoothing: 0.1,     // 0 = instant, 1 = no movement. Default: 0.1
    offset: { x: 0, y: -50 },  // Optional offset from target
});
```

Call `camera.update(deltaTime)` each frame to process the follow:

```typescript
engine.runRenderLoop(() => {
    const dt = engine.getDeltaTime() / 1000;
    camera.update(dt);
    scene.render();
});
```

## Zoom

```typescript
camera.zoom = 2.0;    // 2x zoom in
camera.zoom = 0.5;    // Zoom out
camera.zoom = 1.0;    // Default: no zoom
```

## Bounds

Restrict the camera to a region (e.g., the level boundaries):

```typescript
camera.setBounds(0, 0, levelWidth, levelHeight);
```

The camera will never scroll beyond these bounds. Call `clearBounds()` to remove the restriction.

## Screen Shake

Add a brief shake effect (e.g., on hit or explosion):

```typescript
camera.shake(10, 0.3);   // intensity: 10 pixels, duration: 0.3 seconds
```

## Coordinate Conversion

Convert between screen pixels and world coordinates:

```typescript
// Screen position (e.g., mouse click) → world position
const worldPos = camera.screenToWorld(mouseScreenPosition);

// World position → screen position
const screenPos = camera.worldToScreen(entityWorldPosition);
```

These methods account for the camera's position, zoom, and viewport size.

## Properties

| Property | Type | Description |
|----------|------|-------------|
| `position` | `Vector2` | Camera center in world coordinates |
| `zoom` | `number` | Zoom factor (>1 = zoom in, <1 = zoom out) |
| `viewportWidth` | `number` | Viewport width in pixels |
| `viewportHeight` | `number` | Viewport height in pixels |

## Integration with Input

Use Camera2D with `InputMap2D` for pointer-to-world conversion:

```typescript
const input = new InputMap2D(engine, camera);
// input.pointerWorldPosition automatically uses the camera
```
