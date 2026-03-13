---
title: Camera2D
description: Using Camera2D for viewport control, follow targets, zoom, and screen shake
keywords: 2d, camera, viewport, follow, zoom, shake, design resolution
---

# Camera2D

`Camera2D` provides viewport management for 2D scenes. It handles scrolling, following a target node, zoom, bounded regions, screen shake, design resolution scaling, and coordinate conversion between screen and world space.

<!-- Playground about Camera2D follow, zoom, and shake goes here (id: #JKVCPP) -->

## Basic Setup

```typescript
import { Camera2D } from "@babylonjs/2d/Camera2D/camera2D";
import { Scene2D } from "@babylonjs/2d/Scene2D/scene2D";
import { Engine } from "@babylonjs/core/Engines/engine";

const engine = new Engine(canvas);
const scene = new Scene2D(engine);

const camera = new Camera2D();
camera.setViewport(engine.getRenderWidth(), engine.getRenderHeight());
scene.camera = camera;
```

`Camera2D` takes no constructor arguments. Set the viewport dimensions with `setViewport()`, and assign the camera to `scene.camera`.

## Following a Target

Set the `lockedTarget` property to a `Node2D` and the camera will smoothly follow it each frame:

```typescript
camera.lockedTarget = playerNode;
camera.lerpSpeed = 0.15;                        // Smoothing time constant (0 = instant)
camera.followOffset = new Vector2(0, -50);       // Offset from the target
```

Call `camera.update(deltaTime)` each frame to process the follow. When using `scene.render()`, update is called automatically:

```typescript
engine.runRenderLoop(() => {
    scene.render(); // Calls camera.update(dt) and scene.update(dt) internally
});
```

For compositing with a 3D scene, use `renderContent()` instead:

```typescript
engine.runRenderLoop(() => {
    scene3D.render();
    scene2D.renderContent(); // Auto-updates camera and nodes, no beginFrame/endFrame
});
```

Set `lockedTarget = null` to stop following.

## Zoom

```typescript
camera.zoom = 2.0;    // 2x zoom in
camera.zoom = 0.5;    // Zoom out
camera.zoom = 1.0;    // Default: no zoom
```

## Design Resolution

For pixel-art games or resolution-independent rendering, set a design resolution. The camera auto-scales to fit the viewport:

```typescript
import { ScaleMode } from "@babylonjs/2d/Camera2D/camera2D";

camera.setDesignResolution(480, 270, ScaleMode.FIT);
// On a 1920×1080 screen → 4× zoom, letterboxed if aspect doesn't match
```

| Scale Mode | Behavior |
|------------|----------|
| `ScaleMode.FIT` | Uniform scale to fit within viewport (may letterbox) |
| `ScaleMode.FILL` | Uniform scale to fill viewport (may crop edges) |
| `ScaleMode.STRETCH` | Non-uniform stretch to exactly match viewport |

When a design resolution is active, `camera.zoom` acts as an additional multiplier on top of the auto-computed scale.

## Bounds

Restrict the camera to a world region (e.g., the level boundaries):

```typescript
import { Rectangle2D } from "@babylonjs/2d/Math/rectangle2D";

camera.bounds = new Rectangle2D(0, 0, levelWidth, levelHeight);
```

The camera will never scroll beyond these bounds. Set `camera.bounds = null` to remove the restriction.

## Screen Shake

Add a brief shake effect (e.g., on hit or explosion):

```typescript
camera.shake(10, 0.3);   // intensity: 10 pixels, duration: 0.3 seconds
```

An optional third parameter provides a seed for deterministic shake sequences (useful for replays):

```typescript
camera.shake(10, 0.3, 42); // Deterministic shake with seed 42
```

## Coordinate Conversion

Convert between screen pixels and world coordinates:

```typescript
// Screen position (e.g., mouse click) → world position
const worldPos = camera.screenToWorld(mouseScreenPosition);

// World position → screen position
const screenPos = camera.worldToScreen(entityWorldPosition);
```

These methods account for the camera's position, zoom, rotation, and viewport size.

## Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `position` | `Vector2` | (0, 0) | Camera center in world coordinates |
| `zoom` | `number` | 1 | Zoom factor (&gt;1 = zoom in, &lt;1 = zoom out) |
| `rotation` | `number` | 0 | Camera rotation in radians |
| `lockedTarget` | `Node2D \| null` | `null` | Target node to follow |
| `followOffset` | `Vector2` | (0, 0) | Offset from the locked target |
| `lerpSpeed` | `number` | 0 | Follow smoothing time constant (0 = instant) |
| `bounds` | `Rectangle2D \| null` | `null` | World-space bounds constraint |
| `viewportWidth` | `number` | 0 | Viewport width in pixels (read-only) |
| `viewportHeight` | `number` | 0 | Viewport height in pixels (read-only) |
| `effectiveScale` | `{ scaleX, scaleY }` | — | Combined design-resolution + zoom scale (read-only) |

## Methods

| Method | Description |
|--------|-------------|
| `setViewport(width, height)` | Updates viewport dimensions |
| `setDesignResolution(width, height, scaleMode?)` | Sets design resolution and scale mode |
| `update(deltaTime)` | Updates follow, bounds clamping, and shake |
| `shake(intensity, duration, seed?)` | Triggers screen shake |
| `screenToWorld(screenPos)` | Converts screen → world coordinates |
| `worldToScreen(worldPos)` | Converts world → screen coordinates |
| `getViewTransform()` | Returns the view Matrix2D (used by the renderer) |

## Integration with Input

Use Camera2D with `InputMap2D` for pointer-to-world conversion:

```typescript
const input = new InputMap2D(engine, camera);
// input.pointerWorldPosition automatically uses the camera
```

## Related

- [Introduction](/features/featuresDeepDive/2d/intro) — Coordinate system and scene setup
- [Parallax Scrolling](/features/featuresDeepDive/2d/parallax) — Depth-based scrolling layers
- [Input Mapping](/features/featuresDeepDive/2d/input-mapping) — Camera-aware pointer positions
- [Math Utilities](/features/featuresDeepDive/2d/math-utilities) — Matrix2D and Rectangle2D details
