---
title: Object Pooling
description: Reuse frequently created objects to avoid garbage collection spikes in 2D games
keywords: 2d, object pool, pooling, gc, garbage collection, performance, bullets, particles
---

# Object Pooling

`ObjectPool<T>` provides generic object pooling to avoid garbage collection (GC) spikes in action games. Instead of creating and destroying objects every frame (bullets, enemies, effects), you acquire them from a pool and release them back when done.

## Why Pool?

In a 2D shooter with hundreds of bullets per second, `new Bullet()` on every shot and GC when bullets leave the screen causes frame hitches. Pooling pre-allocates objects and recycles them — zero allocation, zero GC.

## Basic Usage

```typescript
import { ObjectPool } from "@babylonjs/2d/ObjectPool/objectPool";

// Define how to create and reset bullets
const bulletPool = new ObjectPool({
    factory: () => {
        const bullet = new Sprite2D("bullet");
        bullet.texture = bulletTexture;
        bullet.visible = false;
        return bullet;
    },
    reset: (bullet) => {
        bullet.position.x = 0;
        bullet.position.y = 0;
        bullet.visible = false;
        bullet.parent = null; // Auto-handled by pool for Node2D objects
    },
    maxPoolSize: 200,
    name: "BulletPool",
});

// Pre-warm to avoid first-frame allocations
bulletPool.prewarm(50);
```

### Acquiring and Releasing

```typescript
// Fire a bullet
function fireBullet(x: number, y: number, vx: number, vy: number) {
    const bullet = bulletPool.acquire();
    bullet.position.x = x;
    bullet.position.y = y;
    bullet.visible = true;
    scene.addNode(bullet);
    // Store velocity somewhere for update loop...
}

// When bullet goes off screen
function destroyBullet(bullet: Sprite2D) {
    bulletPool.release(bullet);
    // Pool calls reset(), sets parent = null, and returns it to the free list
}
```

## Constructor Options

| Option | Type | Required | Description |
|--------|------|----------|-------------|
| `factory` | `() => T` | ✅ | Creates a new instance when the pool is empty |
| `reset` | `(obj: T) => void` | ✅ | Resets an object to initial state on release |
| `maxPoolSize` | `number` | — | Maximum free objects to keep (excess are GC'd) |
| `name` | `string` | — | Label for debugging (default: `"ObjectPool"`) |

## Properties

| Property | Type | Description |
|----------|------|-------------|
| `activeCount` | `number` | Objects currently in use (acquired but not released) |
| `freeCount` | `number` | Objects available in the pool |
| `totalCreated` | `number` | Total objects ever created by this pool |
| `name` | `string` | Pool name |
| `activeObjects` | `readonly T[]` | Snapshot of all currently active objects |
| `isDisposed` | `boolean` | Whether the pool has been disposed |

## Methods

| Method | Description |
|--------|-------------|
| `acquire(): T` | Gets an object from the pool (creates one if empty) |
| `release(obj: T): void` | Returns an object to the pool (calls `reset` and `onRelease`) |
| `prewarm(count: number): void` | Pre-allocates objects to avoid runtime allocation |
| `forEachActive(callback): void` | Iterates over all active objects |
| `clear(): void` | Empties the free list (active objects unaffected) |
| `dispose(): void` | Disposes the pool — no further acquire/release allowed |

## The IPoolable Interface

Objects can optionally implement `IPoolable` for lifecycle hooks:

```typescript
import type { IPoolable } from "@babylonjs/2d/ObjectPool/objectPool";

class Enemy extends Sprite2D implements IPoolable {
    public health: number = 100;

    onAcquire(): void {
        // Called when acquired from pool
        this.health = 100;
        this.visible = true;
    }

    onRelease(): void {
        // Called when released back to pool
        this.visible = false;
    }
}
```

When `IPoolable` is detected (duck-typed), the pool calls `onAcquire()` after acquiring and `onRelease()` before resetting.

## Scene Graph Integration

The pool automatically detaches objects from the scene graph on release. If a pooled object has a `parent` property (like `Node2D`), the pool sets `parent = null` during release — no need to manually call `removeChild()`.

## Max Pool Size

Setting `maxPoolSize` prevents the pool from growing unbounded:

```typescript
const pool = new ObjectPool({
    factory: () => new Particle(),
    reset: (p) => { p.active = false; },
    maxPoolSize: 100,
});
```

When the pool is full and you release an object, the excess object is discarded (garbage collected) instead of being returned to the pool. This caps memory usage while still avoiding allocation during gameplay.

## Pre-warming

Call `prewarm()` during a loading screen to allocate objects ahead of time:

```typescript
// During level load
bulletPool.prewarm(100);
enemyPool.prewarm(20);
effectPool.prewarm(50);
```

`prewarm()` respects `maxPoolSize` — it won't create more objects than the pool can hold.

## Iterating Active Objects

Use `forEachActive()` to update all live pooled objects:

```typescript
bulletPool.forEachActive((bullet) => {
    bullet.position.x += bulletVelocities.get(bullet)!.x * dt;
    bullet.position.y += bulletVelocities.get(bullet)!.y * dt;

    if (isOffScreen(bullet)) {
        bulletPool.release(bullet);
    }
});
```

<Alert severity="warning" title="Safe to release during iteration">
`forEachActive()` uses a `Set` iterator internally. Releasing objects during iteration is safe — the released object is removed from the active set, and iteration continues with remaining objects.
</Alert>

## Complete Example

```typescript
import { ObjectPool } from "@babylonjs/2d/ObjectPool/objectPool";
import { Sprite2D } from "@babylonjs/2d/Sprite2D/sprite2D";
import { Scene2D } from "@babylonjs/2d/Scene2D/scene2D";

const scene = new Scene2D(engine);

// Create a pool for explosion effects
const explosionPool = new ObjectPool<AnimatedSprite2D>({
    factory: () => {
        const fx = new AnimatedSprite2D("explosion");
        fx.spriteSheet = explosionSheet;
        fx.visible = false;
        return fx;
    },
    reset: (fx) => {
        fx.visible = false;
        fx.stop();
    },
    maxPoolSize: 30,
    name: "ExplosionPool",
});

explosionPool.prewarm(10);

// Spawn an explosion
function spawnExplosion(x: number, y: number) {
    const fx = explosionPool.acquire();
    fx.position.x = x;
    fx.position.y = y;
    fx.visible = true;
    scene.addNode(fx);
    fx.play("explode");
    fx.onAnimationEnd.addOnce(() => {
        explosionPool.release(fx);
    });
}
```

<!-- Playground about object pooling with bullet recycling goes here -->

## Related

- [Sprites & Animation](/features/featuresDeepDive/2d/sprites-and-animation) — Sprite2D and AnimatedSprite2D
- [Collision Detection](/features/featuresDeepDive/2d/collision) — SpatialGrid for broad-phase queries
- [Particles](/features/featuresDeepDive/2d/particles) — ParticleHelper2D for GPU particle effects
