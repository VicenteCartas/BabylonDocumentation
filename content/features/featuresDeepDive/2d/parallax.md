---
title: Parallax Scrolling
description: Using scroll factors on Node2D to create parallax depth layers, fixed HUD elements, and multi-speed backgrounds
keywords: 2d, parallax, scrolling, scroll factor, depth layers, background, hud, camera
---

# Parallax Scrolling

Parallax scrolling creates an illusion of depth in a 2D scene by making background layers move slower than foreground layers as the camera pans. In the real world, distant objects appear to move less when you move — parallax replicates this effect to give flat 2D scenes a sense of three-dimensional space.

Babylon.js 2D implements parallax through **scroll factors** on `Node2D`. Every node has `scrollFactorX` and `scrollFactorY` properties that control how much camera movement affects it:

- **1** (default) — moves normally with the camera (foreground)
- **0** — fixed to the screen, unaffected by scrolling (HUD / UI)
- **0.1–0.9** — parallax depth layers (lower = further away, scrolls slower)

## Basic Usage

Set `scrollFactorX` and `scrollFactorY` on a `Node2D` (or any subclass like `Sprite2D`) to make it scroll at a fraction of the camera speed. Any children added to that node will inherit the scroll factor.

```typescript
import { Scene2D, Sprite2D, Node2D, Camera2D } from "@babylonjs/2d";
import { Engine, Texture } from "@babylonjs/core";

const engine = new Engine(canvas);
const scene = new Scene2D(engine);
const camera = new Camera2D();
camera.setViewport(engine.getRenderWidth(), engine.getRenderHeight());

// Create a distant background layer (scrolls at 20% of camera speed)
const bgLayer = new Node2D("background");
bgLayer.scrollFactorX = 0.2;
bgLayer.scrollFactorY = 0.2;

// Add sprites to the background layer
const cloudTexture = new Texture("clouds.png", null);
const clouds = new Sprite2D("clouds");
clouds.texture = cloudTexture;
bgLayer.addChild(clouds);

// Foreground objects use default scrollFactor = 1 (move normally)
const playerTexture = new Texture("player.png", null);
const player = new Sprite2D("player");
player.texture = playerTexture;
player.position.x = 400;
player.position.y = 300;
```

When the camera moves 100 pixels to the right, the player moves 100 pixels on screen (normal), while the clouds only move 20 pixels — creating the parallax depth effect.

## Multiple Depth Layers

For a convincing parallax effect, use 3–4 layers with progressively increasing scroll factors from back to front:

```typescript
// Layer 0 — Distant sky (barely moves)
const skyLayer = new Node2D("sky");
skyLayer.scrollFactorX = 0.1;
skyLayer.scrollFactorY = 0.1;

const sky = new Sprite2D("skySprite");
sky.texture = new Texture("sky.png", null);
sky.width = 2048;
sky.height = 600;
skyLayer.addChild(sky);

// Layer 1 — Far mountains
const mountainLayer = new Node2D("mountains");
mountainLayer.scrollFactorX = 0.3;
mountainLayer.scrollFactorY = 0.3;

const mountains = new Sprite2D("mountainSprite");
mountains.texture = new Texture("mountains.png", null);
mountains.width = 2048;
mountains.height = 400;
mountains.position.y = 200;
mountainLayer.addChild(mountains);

// Layer 2 — Near trees
const treeLayer = new Node2D("trees");
treeLayer.scrollFactorX = 0.6;
treeLayer.scrollFactorY = 0.6;

const trees = new Sprite2D("treeSprite");
trees.texture = new Texture("trees.png", null);
trees.width = 2048;
trees.height = 300;
trees.position.y = 300;
treeLayer.addChild(trees);

// Layer 3 — Foreground gameplay (default scrollFactor = 1)
const player = new Sprite2D("player");
player.texture = new Texture("player.png", null);
player.position.x = 400;
player.position.y = 400;
```

| Layer | scrollFactor | Visual Role |
|-------|-------------|-------------|
| Sky | 0.1 | Distant backdrop, barely moves |
| Mountains | 0.3 | Mid-distance scenery |
| Trees | 0.6 | Near scenery, noticeable movement |
| Gameplay | 1.0 (default) | Player, enemies, tiles — moves with camera |

## Fixed HUD Elements

Set both scroll factors to `0` to lock a node to the screen regardless of camera position. This is ideal for health bars, score counters, minimaps, and other UI overlays:

```typescript
import { Node2D, Text2D } from "@babylonjs/2d";

const hud = new Node2D("hud");
hud.scrollFactorX = 0;
hud.scrollFactorY = 0;

// Position in screen space (top-left corner)
hud.position.x = 16;
hud.position.y = 16;

const scoreLabel = new Text2D("score", "Score: 0");
scoreLabel.position.x = 0;
scoreLabel.position.y = 0;
hud.addChild(scoreLabel);

const healthBar = new Sprite2D("healthBar");
healthBar.texture = new Texture("health.png", null);
healthBar.position.x = 0;
healthBar.position.y = 40;
hud.addChild(healthBar);
```

<Alert severity="info">
When `scrollFactor` is 0, the node's position is effectively in **screen space** rather than world space. Position (0, 0) corresponds to the top-left of the viewport, consistent with the Y-down coordinate system.
</Alert>

## Scroll Factor Inheritance

Scroll factors are **multiplied through the node hierarchy**, just like `alpha`. Setting a scroll factor on a parent automatically applies to all of its children.

```typescript
const bgLayer = new Node2D("background");
bgLayer.scrollFactorX = 0.5;  // 50% camera speed

const nestedCloud = new Sprite2D("nestedCloud");
nestedCloud.scrollFactorX = 0.5;  // Local factor
bgLayer.addChild(nestedCloud);

// Effective (world) scroll factor: 0.5 × 0.5 = 0.25
console.log(nestedCloud.worldScrollFactorX); // 0.25
```

Use the read-only `worldScrollFactorX` and `worldScrollFactorY` getters to inspect the effective inherited value at any point in the hierarchy.

This multiplicative inheritance means you can organize your scene into layer groups and fine-tune individual children within them:

```typescript
// All background elements scroll at 30% speed
const bgGroup = new Node2D("bgGroup");
bgGroup.scrollFactorX = 0.3;
bgGroup.scrollFactorY = 0.3;

// This child scrolls at 0.3 × 1.0 = 0.3 (inherits parent)
const bgTrees = new Sprite2D("bgTrees");
bgGroup.addChild(bgTrees);

// This child scrolls at 0.3 × 0.5 = 0.15 (even slower)
const bgClouds = new Sprite2D("bgClouds");
bgClouds.scrollFactorX = 0.5;
bgClouds.scrollFactorY = 0.5;
bgGroup.addChild(bgClouds);
```

## Independent X/Y Scroll Factors

The X and Y scroll factors are independent, which is useful for genre-specific effects:

### Side-Scroller (Horizontal Parallax Only)

In a side-scrolling game, you typically want vertical movement to stay 1:1 (so jumping feels correct) while horizontal backgrounds scroll at a slower rate:

```typescript
const bgLayer = new Node2D("background");
bgLayer.scrollFactorX = 0.3;  // Slow horizontal parallax
bgLayer.scrollFactorY = 1.0;  // Normal vertical scrolling
```

### Top-Down (Uniform Parallax)

In a top-down game, both axes should usually match:

```typescript
const bgLayer = new Node2D("background");
bgLayer.scrollFactorX = 0.2;
bgLayer.scrollFactorY = 0.2;
```

### Vertical Scroller

For a vertical-scrolling game (e.g., a shoot-em-up), lock horizontal scrolling and slow down the vertical:

```typescript
const bgLayer = new Node2D("background");
bgLayer.scrollFactorX = 1.0;  // Normal horizontal
bgLayer.scrollFactorY = 0.3;  // Slow vertical parallax
```

## API Reference

### Node2D Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `scrollFactorX` | `number` | 1 | Horizontal scroll factor. 1 = normal scrolling, 0 = fixed to camera. |
| `scrollFactorY` | `number` | 1 | Vertical scroll factor. 1 = normal scrolling, 0 = fixed to camera. |
| `worldScrollFactorX` | `number` (readonly) | 1 | Effective horizontal scroll factor after multiplying through the parent hierarchy. |
| `worldScrollFactorY` | `number` (readonly) | 1 | Effective vertical scroll factor after multiplying through the parent hierarchy. |

## Performance

Sprites that keep the default scroll factor of `(1, 1)` have **zero overhead** from the parallax system. The renderer checks the scroll factor per sprite and only applies the parallax correction when at least one axis differs from 1. This means you can use parallax on a handful of background layers without paying any cost on the hundreds of foreground sprites in your scene.

## Tips

- **Tile your backgrounds** wider than the viewport. Because parallax layers scroll slower, they need less total width than the camera's travel distance — but they still need to cover the visible area at all camera positions.
- **Use sorting layers** to ensure parallax backgrounds render behind gameplay. Set `sortingLayer = 0` on background nodes and `sortingLayer = 1` (or higher) on foreground gameplay nodes.
- **Combine with camera zoom.** Parallax works correctly with [Camera2D](/features/featuresDeepDive/2d/camera2d) zoom — the scroll factor adjusts translation only, so zooming in still feels natural.
- **Values greater than 1** are allowed and create a foreground parallax effect (the layer scrolls *faster* than the camera), useful for very close decorative elements that whip past the player.

## Related Pages

- [Camera2D](/features/featuresDeepDive/2d/camera2d) — Viewport control, follow, zoom, bounds, and screen shake
- [Sprites & Animation](/features/featuresDeepDive/2d/sprites-and-animation) — Creating and animating Sprite2D nodes
- [Tilemaps](/features/featuresDeepDive/2d/tilemaps) — Tile-based level rendering (can be placed on parallax layers)
- [Text & UI](/features/featuresDeepDive/2d/text-and-ui) — In-world and HUD text labels
