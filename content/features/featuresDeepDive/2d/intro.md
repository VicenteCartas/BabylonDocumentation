---
title: Introduction to Babylon.js 2D
description: Getting started with Babylon.js 2D game engine capabilities
keywords: 2d, game engine, sprites, tilemap, camera, lighting, physics, state machine
---

# Introduction to Babylon.js 2D

Babylon.js 2D (`@babylonjs/2d`) is a dedicated 2D game engine package that provides GPU-accelerated sprite rendering, tilemaps, physics, collision, camera, input mapping, lighting, tweens, and more. It is designed for building complete 2D games — from side-scrollers to isometric RPGs and turn-based strategy.

## Key Features

- **Sprite rendering** with texture atlas support, animation, tinting, and flip
- **Tilemap system** with Tiled editor integration and animated tiles
- **2D camera** with follow, zoom, bounds, parallax, and screen shake
- **2D collision detection** with AABB, circle, and polygon shapes
- **2D physics** via plugin architecture (Planck.js default)
- **Input mapping** for keyboard, mouse, touch, and gamepad
- **Pathfinding** with A* on grids and movement range queries
- **GPU lighting** with point lights, spotlights, and ambient (forward rendered per-pixel)
- **Tween & easing** with 16 easing curves, chaining, and looping
- **State machine** for game AI and logic with condition-based transitions
- **Text2D** for in-world text labels and HUD elements
- **9-slice sprites** for resizable UI panels and buttons
- **Scene transitions** with fade and slide effects
- **Grid system** for square and hex grids with coordinate helpers
- **Audio** integration with Babylon.js AudioV2

## Installation

```bash
npm install @babylonjs/2d
```

`@babylonjs/2d` depends on `@babylonjs/core` for the rendering engine, textures, audio, and GPU access. It does not affect the bundle size of projects that don't import it.

## Coordinate System

Babylon.js 2D uses a **Y-down, top-left origin** coordinate system with pixel units. This matches the convention used by other 2D frameworks (Phaser, PixiJS, Excalibur) and the HTML Canvas.

- `position(0, 0)` is the top-left corner
- Positive X goes right, positive Y goes down
- Rotation is clockwise (in radians)

> **Note:** This differs from Babylon.js 3D, which uses Y-up. The 2D package has its own scene graph (`Scene2D`, `Node2D`) separate from the 3D `Scene` and `TransformNode`.

## Hello World

<!-- <Playground id="#B9J5GF" title="Hello World 2D" description="Minimal 2D scene with colored sprite quads arranged along a diagonal." isMain={true} category="2D" /> -->
Playground **Hello World 2D** will go here

```typescript
import { Engine } from "@babylonjs/core/Engines/engine";
import { Texture } from "@babylonjs/core/Materials/Textures/texture";
import { Scene2D, Sprite2D, Camera2D } from "@babylonjs/2d";

// Create the engine (shared with 3D)
const canvas = document.getElementById("renderCanvas") as HTMLCanvasElement;
const engine = new Engine(canvas, true);

// Create a 2D scene
const scene = new Scene2D(engine);
const camera = new Camera2D();
camera.setViewport(engine.getRenderWidth(), engine.getRenderHeight());

// Create a sprite
const texture = new Texture("player.png", null);
const player = new Sprite2D("player", texture);
player.position.x = 400;
player.position.y = 300;
scene.addNode(player);

// Game loop
engine.runRenderLoop(() => {
    const dt = engine.getDeltaTime() / 1000;
    camera.update(dt);
    scene.update(dt);
    scene.render();
});
```

## Architecture

### Separate from Core

`@babylonjs/2d` is a standalone package with its own class hierarchy:

| 2D Class | Purpose |
|----------|---------|
| `Scene2D` | Manages 2D nodes, update loop, rendering, and lighting |
| `Node2D` | Base class for all 2D entities (transform, hierarchy, z-ordering) |
| `Sprite2D` | Textured quad with tint, flip, and alpha blending |
| `AnimatedSprite2D` | Sprite with frame-based animation playback |
| `Camera2D` | Viewport control with follow, zoom, bounds, parallax, and shake |
| `SpriteSheet` | Defines frames and animations from a texture atlas |
| `Tilemap2D` | Tile-based level rendering with animated tiles |
| `Text2D` | In-world text labels rendered to dynamic textures |
| `NineSliceSprite2D` | Resizable UI panels with undistorted borders |
| `LightingManager2D` | GPU forward lighting (point, spot, ambient — up to 16 lights) |
| `Tween2D` | Smooth value interpolation with 16 easing curves |
| `StateMachine2D` | Finite state machine for AI and game logic |
| `InputMap2D` | Action-based input mapping (keyboard, mouse, touch, gamepad) |
| `PlanckPhysicsEngine` | 2D physics via Planck.js (Box2D port) |

### Sharing the Engine

Both 2D and 3D scenes use the same `Engine` instance, sharing the GPU context, textures, and shaders. You can render a 3D scene and a 2D scene to the same canvas:

```typescript
engine.runRenderLoop(() => {
    scene3D.render();   // 3D background
    scene2D.render();   // 2D gameplay on top
});
```

<!-- <Playground id="#03W08W" title="2D + 3D Compositing" description="Renders a 3D rotating box with a 2D sprite overlay on the same canvas." category="2D" /> -->
Playground **2D + 3D Compositing** will go here

## Next Steps

- [Sprites and Animation](/features/featuresDeepDive/2d/sprites-and-animation) — Working with Sprite2D, SpriteSheet, and AnimatedSprite2D
- [Camera2D](/features/featuresDeepDive/2d/camera2d) — Follow targets, zoom, bounds, and screen shake
- [Tilemaps](/features/featuresDeepDive/2d/tilemaps) — Loading and rendering tile-based levels
- [GPU Lighting](/features/featuresDeepDive/2d/lighting) — Per-pixel forward lighting with point lights and spotlights
- [Physics](/features/featuresDeepDive/2d/physics) — Planck.js integration for rigid body physics
- [Input Mapping](/features/featuresDeepDive/2d/input-mapping) — Keyboard, mouse, touch, and gamepad
- [State Machine](/features/featuresDeepDive/2d/state-machine) — FSM for game AI and logic
- [Tween & Easing](/features/featuresDeepDive/2d/tween-easing) — Smooth animations and transitions
