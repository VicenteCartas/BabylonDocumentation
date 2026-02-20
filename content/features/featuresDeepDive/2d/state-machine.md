---
title: State Machine
description: Finite state machine for animation and AI in Babylon.js 2D
keywords: 2d, state machine, fsm, ai, animation, transitions
---

# State Machine

The `StateMachine2D` is a generic finite state machine for managing game logic — enemy AI behaviors, animation states, game flow, and UI modes. It supports lifecycle hooks, automatic transitions with guard conditions, named trigger transitions, and priority ordering.

## Creating a State Machine

A state machine is parameterized by a **context** object that all states and transitions share:

```typescript
import { StateMachine2D } from "@babylonjs/2d";

interface EnemyContext {
    health: number;
    playerDistance: number;
    speed: number;
}

const context: EnemyContext = { health: 100, playerDistance: 500, speed: 60 };
const fsm = new StateMachine2D<EnemyContext>(context);
```

## Defining States

Each state has a unique name and optional lifecycle hooks:

- **onEnter** — called once when the state becomes active
- **onUpdate** — called every frame while the state is active
- **onExit** — called once when leaving the state

```typescript
fsm.addState({
    name: "patrol",
    onEnter: (ctx, previousState) => {
        // Start patrol animation
    },
    onUpdate: (ctx, dt) => {
        // Move along patrol path
    },
    onExit: (ctx, nextState) => {
        // Clean up patrol state
    },
});

fsm.addState({
    name: "chase",
    onEnter: (ctx) => {
        ctx.speed = 120; // Run faster when chasing
    },
    onUpdate: (ctx, dt) => {
        // Move toward player
    },
    onExit: (ctx) => {
        ctx.speed = 60;
    },
});

fsm.addState({ name: "dead" });
```

Methods return `this` for chaining:

```typescript
fsm.addState({ name: "idle" })
   .addState({ name: "walk" })
   .addState({ name: "run" });
```

## Automatic Transitions

Transitions with a `condition` function are evaluated every frame. When the condition returns `true`, the transition fires automatically:

```typescript
fsm.addTransition({
    from: "patrol",
    to: "chase",
    condition: (ctx) => ctx.playerDistance < 100,
});

fsm.addTransition({
    from: "chase",
    to: "patrol",
    condition: (ctx) => ctx.playerDistance > 200,
});

fsm.addTransition({
    from: "chase",
    to: "dead",
    condition: (ctx) => ctx.health <= 0,
});
```

Only **one** transition fires per `update()` call. If multiple conditions are true simultaneously, use **priority** to control which fires first (higher value = checked first):

```typescript
fsm.addTransition({
    from: "patrol",
    to: "chase",
    condition: (ctx) => ctx.playerDistance < 100,
    priority: 1,
});

fsm.addTransition({
    from: "patrol",
    to: "flee",
    condition: (ctx) => ctx.health < 20,
    priority: 10, // Flee takes priority over chase
});
```

## Named Triggers

Transitions without a condition (or with one used as a guard) can be fired manually via `trigger()`:

```typescript
fsm.addTransition({ from: "idle", to: "attack", name: "doAttack" });
fsm.addTransition({
    from: "idle",
    to: "special",
    name: "doSpecial",
    condition: (ctx) => ctx.health > 50, // Guard: only if healthy
});

// In game logic:
if (input.isActionPressed("attack")) {
    fsm.trigger("doAttack"); // Returns true if transition fired
}
```

`trigger()` returns `false` if:
- The named transition doesn't exist
- The current state doesn't match the transition's `from` state
- The guard condition returns `false`

## Starting and Updating

```typescript
fsm.start("patrol");

// In your game loop:
engine.runRenderLoop(() => {
    const dt = /* delta time */;
    fsm.update(dt); // Evaluates transitions, then calls current state's onUpdate
});
```

## Force State

To immediately switch states bypassing all guards:

```typescript
fsm.forceState("dead"); // Calls onExit on current, onEnter on "dead"
```

## Observing State Changes

```typescript
fsm.onStateChange.add((event) => {
    console.log(`${event.previousState} → ${event.currentState}`);
});
```

## Example: Animation State Machine

```typescript
interface AnimContext {
    velocityX: number;
    velocityY: number;
    isGrounded: boolean;
    animatedSprite: AnimatedSprite2D;
}

const animFsm = new StateMachine2D<AnimContext>(context);

animFsm.addState({
    name: "idle",
    onEnter: (ctx) => ctx.animatedSprite.play("idle", true),
});

animFsm.addState({
    name: "run",
    onEnter: (ctx) => ctx.animatedSprite.play("run", true),
});

animFsm.addState({
    name: "jump",
    onEnter: (ctx) => ctx.animatedSprite.play("jump", false),
});

animFsm.addState({
    name: "fall",
    onEnter: (ctx) => ctx.animatedSprite.play("fall", true),
});

animFsm
    .addTransition({ from: "idle", to: "run", condition: (ctx) => Math.abs(ctx.velocityX) > 10 })
    .addTransition({ from: "run", to: "idle", condition: (ctx) => Math.abs(ctx.velocityX) < 5 })
    .addTransition({ from: "idle", to: "jump", condition: (ctx) => ctx.velocityY < -10 })
    .addTransition({ from: "run", to: "jump", condition: (ctx) => ctx.velocityY < -10 })
    .addTransition({ from: "jump", to: "fall", condition: (ctx) => ctx.velocityY > 0 })
    .addTransition({ from: "fall", to: "idle", condition: (ctx) => ctx.isGrounded });

animFsm.start("idle");
```

## Example: Game Flow

```typescript
const gameFlow = new StateMachine2D<{ level: number }>({ level: 1 });

gameFlow
    .addState({ name: "menu", onEnter: () => showMainMenu() })
    .addState({ name: "playing", onEnter: (ctx) => loadLevel(ctx.level) })
    .addState({ name: "paused", onEnter: () => showPauseMenu() })
    .addState({ name: "gameOver", onEnter: () => showGameOver() });

gameFlow
    .addTransition({ from: "menu", to: "playing", name: "startGame" })
    .addTransition({ from: "playing", to: "paused", name: "pause" })
    .addTransition({ from: "paused", to: "playing", name: "resume" })
    .addTransition({ from: "playing", to: "gameOver", name: "die" })
    .addTransition({ from: "gameOver", to: "menu", name: "backToMenu" });

gameFlow.start("menu");

// Later: gameFlow.trigger("startGame");
```

## Cleanup

```typescript
fsm.dispose(); // Clears all states, transitions, and observables
```
