---
title: 2D Physics
description: Physics simulation with Planck.js backend in Babylon.js 2D
keywords: 2d, physics, planck, rigid body, collision, gravity
---

# 2D Physics

The 2D engine provides a plugin-based physics system via the `IPhysicsEngine2D` interface. The default backend is **Planck.js** (a TypeScript port of Box2D).

## Setup

```typescript
import { PlanckPhysicsEngine } from "@babylonjs/2d";
import { Vector2 } from "@babylonjs/core";

// Gravity: Y-down, 980 pixels/sec² (≈ 9.8m/s² at 100px/m)
const physics = new PlanckPhysicsEngine(new Vector2(0, 980));
```

## Body Types

| Type | Description |
|------|-------------|
| `PhysicsBodyType2D.Static` | Immovable (terrain, walls). Zero mass. |
| `PhysicsBodyType2D.Dynamic` | Fully simulated (player, projectiles). Affected by gravity and forces. |
| `PhysicsBodyType2D.Kinematic` | Moves by velocity only (moving platforms). Not affected by forces. |

## Adding Bodies

Attach a physics body to any `Node2D`:

```typescript
import { PhysicsBodyType2D } from "@babylonjs/2d";

// Dynamic player body
const playerBody = physics.addBody(playerNode, {
    bodyType: PhysicsBodyType2D.Dynamic,
    shape: { type: "box", width: 24, height: 48 },
    density: 1,
    friction: 0.3,
    restitution: 0,
    fixedRotation: true,  // Prevent tumbling
});

// Static ground
physics.addBody(groundNode, {
    bodyType: PhysicsBodyType2D.Static,
    shape: { type: "box", width: 800, height: 32 },
});

// Circular projectile
physics.addBody(bulletNode, {
    bodyType: PhysicsBodyType2D.Dynamic,
    shape: { type: "circle", radius: 4 },
    density: 0.5,
    restitution: 0.8,  // Bouncy
});
```

### Shape Types

| Shape | Properties |
|-------|-----------|
| `box` | `width`, `height` (full dimensions in pixels) |
| `circle` | `radius` (in pixels) |
| `polygon` | `vertices` (array of Vector2, convex, clockwise) |

## Stepping the Simulation

Call `step()` with the delta time each frame. This advances the physics world and syncs body positions back to their Node2D transforms:

```typescript
engine.runRenderLoop(() => {
    const dt = engine.getDeltaTime() / 1000;
    physics.step(dt);
    scene.render();
});
```

## Controlling Bodies

```typescript
// Set velocity directly (good for character controllers)
playerBody.setLinearVelocity(new Vector2(200, playerBody.getLinearVelocity().y));

// Apply an impulse (instant force, good for jumps)
playerBody.applyImpulse(new Vector2(0, -500));

// Apply a continuous force
playerBody.applyForce(new Vector2(100, 0));

// Get mass
const mass = playerBody.getMass();
```

## Collision Callbacks

React to physics bodies touching:

```typescript
physics.onBeginContact((bodyA, bodyB) => {
    if (bodyA.node.name === "player" && bodyB.node.name === "spike") {
        handleDamage();
    }
});

physics.onEndContact((bodyA, bodyB) => {
    // Bodies separated
});
```

## Sensors

Sensors detect overlap without producing collision responses (useful for triggers, area detection):

```typescript
physics.addBody(triggerZone, {
    bodyType: PhysicsBodyType2D.Static,
    shape: { type: "box", width: 100, height: 100 },
    isSensor: true,
});
```

## Collision Layers

Control which bodies interact using bitmask layers:

```typescript
const LAYER_PLAYER = 1;
const LAYER_ENEMY = 2;
const LAYER_TERRAIN = 4;

physics.addBody(playerNode, {
    bodyType: PhysicsBodyType2D.Dynamic,
    shape: { type: "box", width: 24, height: 48 },
    layer: LAYER_PLAYER,
    mask: LAYER_TERRAIN | LAYER_ENEMY,  // Collide with terrain and enemies
});

physics.addBody(enemyNode, {
    bodyType: PhysicsBodyType2D.Dynamic,
    shape: { type: "box", width: 32, height: 32 },
    layer: LAYER_ENEMY,
    mask: LAYER_TERRAIN | LAYER_PLAYER,  // Collide with terrain and player
});
```

## Raycast

Cast a ray through the physics world:

```typescript
const hit = physics.raycast(
    new Vector2(100, 100),   // origin
    new Vector2(1, 0),       // direction (will be normalized)
    500                      // max distance in pixels
);

if (hit) {
    console.log(`Hit at ${hit.point.x}, ${hit.point.y}, distance: ${hit.distance}`);
}
```

## Coordinate System

The physics engine uses the same Y-down coordinate system as the rest of the 2D engine. Internally, Planck.js works in meters — the engine converts automatically using a configurable pixels-per-meter scale (default: 50).

## Cleanup

```typescript
// Remove a single body
physics.removeBody(playerBody);

// Dispose the entire physics world
physics.dispose();
```

## Plugin Architecture

The physics system uses an interface (`IPhysicsEngine2D`) to allow alternative backends. To implement a custom physics backend, implement all methods of `IPhysicsEngine2D`:

```typescript
class MyPhysicsEngine implements IPhysicsEngine2D {
    setGravity(gravity: Vector2): void { /* ... */ }
    getGravity(): Vector2 { /* ... */ }
    addBody(node: Node2D, options: IPhysicsBody2DOptions): IPhysicsBody2D { /* ... */ }
    removeBody(body: IPhysicsBody2D): void { /* ... */ }
    step(deltaTime: number): void { /* ... */ }
    raycast(origin: Vector2, direction: Vector2, maxDistance: number, mask?: number): IRaycastHit2D | null { /* ... */ }
    onBeginContact(callback: PhysicsContactCallback): void { /* ... */ }
    onEndContact(callback: PhysicsContactCallback): void { /* ... */ }
    dispose(): void { /* ... */ }
}
```
