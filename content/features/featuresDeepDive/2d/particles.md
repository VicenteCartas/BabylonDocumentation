---
title: 2D Particles
description: GPU-accelerated particles in 2D games using Babylon.js core ParticleSystem
keywords: 2d, particles, emitter, effects, fire, smoke, explosion, gpu, node particle editor
---

# 2D Particles

`ParticleHelper2D` bridges Babylon.js core's GPU-accelerated `ParticleSystem` into your 2D game. You get the full power of the core particle engine — gradients, sub-emitters, noise, sprite sheets, and the **Node Particle Editor** — all rendering in the same Y-down coordinate space as your 2D sprites.

## Setup

```typescript
import { ParticleHelper2D } from "@babylonjs/2d";
import { Vector3, Color4, ParticleSystem, Texture } from "@babylonjs/core";

// Create the helper (uses the same engine as Scene2D)
const particles = new ParticleHelper2D(engine);
particles.camera = camera; // Sync with your Camera2D

// Create a particle system with full core capabilities
const fire = particles.createParticleSystem("fire", 2000);
fire.particleTexture = new Texture("textures/flare.png", particles.scene);
fire.emitter = new Vector3(400, 300, 0); // 2D world position (x, y, 0)

fire.minLifeTime = 0.3;
fire.maxLifeTime = 1.5;
fire.emitRate = 100;
fire.minSize = 0.5;
fire.maxSize = 1.5;

fire.color1 = new Color4(1, 0.6, 0, 1);
fire.color2 = new Color4(1, 0, 0, 0);
fire.colorDead = new Color4(0, 0, 0, 0);

fire.start();
```

## Render Loop

Render particles **after** your Scene2D so they appear on top:

```typescript
engine.runRenderLoop(() => {
    scene2D.render();      // 2D sprites, tiles, etc.
    particles.render();    // GPU particles (auto-syncs camera)
});
```

The helper automatically syncs its orthographic camera with your `Camera2D` — matching position, zoom, rotation, and design resolution.

## Camera Sync

`ParticleHelper2D` creates an internal core `Scene` with an orthographic camera that mirrors your `Camera2D`:

- **Position** matches Camera2D center
- **Ortho bounds** derived from viewport size and effective scale (zoom × design resolution)
- **Y-axis flipped** so Y-down coordinates match Camera2D convention
- **Rotation** synced each frame

You can also call `particles.sync()` manually or access `particles.scene` for full control.

## Using the Node Particle Editor

Export a particle system from the [Node Particle Editor](https://npe.babylonjs.com/) as JSON, then load it:

```typescript
import { ParticleSystem } from "@babylonjs/core";

// Load the exported JSON into the helper's scene
const response = await fetch("particles/explosion.json");
const data = await response.json();
const ps = ParticleSystem.Parse(data, particles.scene, "");
ps.emitter = new Vector3(hitX, hitY, 0);
ps.start();
```

## Examples

### Fire Effect

```typescript
const fire = particles.createParticleSystem("fire", 1000);
fire.particleTexture = new Texture("flare.png", particles.scene);
fire.emitter = new Vector3(200, 400, 0);
fire.minEmitBox = new Vector3(-10, 0, 0);
fire.maxEmitBox = new Vector3(10, 0, 0);

fire.direction1 = new Vector3(-0.5, -1, 0); // Upward (Y-down)
fire.direction2 = new Vector3(0.5, -1, 0);
fire.minLifeTime = 0.2;
fire.maxLifeTime = 0.8;
fire.emitRate = 80;
fire.gravity = new Vector3(0, -100, 0); // Float up in Y-down

fire.addColorGradient(0, new Color4(1, 1, 0.3, 1));
fire.addColorGradient(0.5, new Color4(1, 0.3, 0, 1));
fire.addColorGradient(1, new Color4(0.2, 0, 0, 0));

fire.start();
```

### Explosion Burst

```typescript
const explosion = particles.createParticleSystem("explosion", 200);
explosion.particleTexture = new Texture("spark.png", particles.scene);
explosion.emitter = new Vector3(hitX, hitY, 0);

explosion.minLifeTime = 0.1;
explosion.maxLifeTime = 0.5;
explosion.minEmitPower = 5;
explosion.maxEmitPower = 15;
explosion.minSize = 0.3;
explosion.maxSize = 1.0;
explosion.targetStopDuration = 0.1; // Short burst

explosion.addColorGradient(0, new Color4(1, 1, 0.5, 1));
explosion.addColorGradient(1, new Color4(1, 0.2, 0, 0));

explosion.start();
```

### Trail (Attach to Moving Object)

```typescript
const trail = particles.createParticleSystem("trail", 500);
trail.particleTexture = new Texture("dot.png", particles.scene);
trail.emitter = new Vector3(0, 0, 0); // Updated each frame

trail.emitRate = 40;
trail.minLifeTime = 0.2;
trail.maxLifeTime = 0.6;
trail.minSize = 0.2;
trail.maxSize = 0.5;
trail.minEmitPower = 0.1;
trail.maxEmitPower = 0.5;
trail.color1 = new Color4(0.5, 0.8, 1, 0.8);
trail.colorDead = new Color4(0.2, 0.4, 1, 0);

trail.start();

// In game loop — move emitter with the object
(trail.emitter as Vector3).x = projectile.position.x;
(trail.emitter as Vector3).y = projectile.position.y;
```

## Cleanup

```typescript
particles.dispose(); // Disposes the internal scene and all particle systems
```

## Why Core ParticleSystem?

The 2D helper uses Babylon.js core's `ParticleSystem` directly, giving you:

- **GPU-accelerated** rendering (WebGL/WebGPU)
- **Visual authoring** via the Node Particle Editor
- **Rich features**: color/size gradients, sub-emitters, noise textures, sprite animations, drag, flow maps
- **Battle-tested**: same particle engine used by thousands of Babylon.js projects
