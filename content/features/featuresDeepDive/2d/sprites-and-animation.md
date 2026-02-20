---
title: Sprites and Animation
description: Working with Sprite2D, SpriteSheet, and AnimatedSprite2D in Babylon.js 2D
keywords: 2d, sprites, animation, spritesheet, texture atlas
---

# Sprites and Animation

The 2D engine provides three classes for sprite rendering and animation:

- **Sprite2D** — A textured quad node in the 2D scene graph
- **SpriteSheet** — Defines frames from a texture (grid-based or JSON atlas)
- **AnimatedSprite2D** — Extends Sprite2D with frame-based animation playback

All sprites are rendered through a `SpriteBatchRenderer` with GPU instancing, multi-texture batching (up to 8 textures per draw call), and optional pixel-perfect mode for crisp pixel art. A typical 2D game renders all sprites in **1 draw call**.

## Sprite2D

`Sprite2D` extends `Node2D` and renders a textured rectangle. It supports tinting, flipping, and texture region selection.

```typescript
import { Scene2D, Sprite2D } from "@babylonjs/2d";
import { Engine, Texture } from "@babylonjs/core";

const engine = new Engine(canvas);
const scene = new Scene2D(engine);

const texture = new Texture("player.png", engine);
const player = new Sprite2D("player", scene.root, texture);
player.position.x = 100;
player.position.y = 200;
player.width = 64;
player.height = 64;
```

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `width` | `number` | Display width in pixels |
| `height` | `number` | Display height in pixels |
| `texture` | `Texture` | The source texture |
| `tint` | `Color4` | Multiplicative color tint |
| `flipX` | `boolean` | Mirror horizontally |
| `flipY` | `boolean` | Mirror vertically |
| `sourceRect` | `Rectangle2D` | Sub-region of texture to display |

## SpriteSheet

A `SpriteSheet` defines named frames and animations from a single texture. There are two ways to create one:

### Grid-Based

For textures arranged in a uniform grid (all frames same size):

```typescript
import { SpriteSheet } from "@babylonjs/2d";

const sheet = SpriteSheet.FromGrid(texture, {
    frameWidth: 32,
    frameHeight: 32,
    columns: 8,
    rows: 4,
});
```

### JSON Atlas

For textures packed with tools like TexturePacker:

```typescript
const sheet = await SpriteSheet.FromJSONAsync(texture, "sprites.json");
```

### Defining Animations

Animations are defined as named sequences of frame indices with a frame rate:

```typescript
sheet.defineAnimation("walk", [0, 1, 2, 3, 4, 5], 12);   // 12 fps
sheet.defineAnimation("idle", [8, 9, 10, 11], 6);          // 6 fps
sheet.defineAnimation("jump", [16, 17, 18], 10);
```

## AnimatedSprite2D

`AnimatedSprite2D` extends `Sprite2D` with animation playback. It uses a `SpriteSheet` to determine which frame to display.

```typescript
import { AnimatedSprite2D } from "@babylonjs/2d";

const player = new AnimatedSprite2D("player", scene.root, sheet);
player.play("walk");       // Loop the walk animation
player.play("jump", false); // Play jump once
```

### Playback Control

```typescript
player.play("idle");           // Start animation (looped by default)
player.play("attack", false);  // Play once, then stop
player.stop();                 // Stop and hold current frame
player.currentFrame = 0;       // Jump to specific frame
```

### Animation Events

AnimatedSprite2D fires an observable when a non-looping animation completes:

```typescript
player.onAnimationComplete.add((animName) => {
    if (animName === "attack") {
        player.play("idle");
    }
});
```

### Update Loop

Call `update(deltaTime)` each frame to advance animation:

```typescript
engine.runRenderLoop(() => {
    const dt = engine.getDeltaTime() / 1000;
    player.update(dt);
    scene.render();
});
```

## Tips

- Use `flipX` for left/right facing instead of duplicating animation frames
- SpriteSheets are reusable — create one and share across multiple AnimatedSprite2D instances
- Set `sourceRect` on a plain Sprite2D to display a sub-region without needing a full SpriteSheet
