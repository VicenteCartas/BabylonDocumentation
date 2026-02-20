---
title: Tween & Easing
description: Smooth value interpolation and standard easing curves for Babylon.js 2D
keywords: 2d, tween, easing, animation, interpolation, lerp
---

# Tween & Easing

The tween system provides smooth value interpolation for any numeric property — positions, alpha, rotation, scale, colors. Pair it with standard easing functions for polished, natural-feeling animations.

## Easing Functions

The `Easing` class provides 16 standard easing curves. Each is a pure function `(t: number) => number` mapping normalized time (0–1) to eased output (0–1):

```typescript
import { Easing } from "@babylonjs/2d";

Easing.Linear(0.5);    // 0.5 — constant speed
Easing.QuadIn(0.5);    // 0.25 — accelerating
Easing.QuadOut(0.5);   // 0.75 — decelerating
Easing.CubicInOut(0.5); // 0.5 — smooth S-curve
```

### Available Easings

| Category | In | Out | InOut |
|----------|-------|---------|-----------|
| Quad     | `QuadIn` | `QuadOut` | `QuadInOut` |
| Cubic    | `CubicIn` | `CubicOut` | `CubicInOut` |
| Sine     | `SineIn` | `SineOut` | `SineInOut` |
| Expo     | `ExpoIn` | `ExpoOut` | — |
| Back     | `BackIn` | `BackOut` | — |
| Elastic  | — | `ElasticOut` | — |
| Bounce   | — | `BounceOut` | — |

You can also pass any custom function matching the `EasingFunction` type: `(t: number) => number`.

## Basic Tween

A `Tween` interpolates a value from `from` to `to` over a duration:

```typescript
import { Tween, Easing } from "@babylonjs/2d";

const tween = new Tween({ from: 0, to: 300 }, 0.5, Easing.CubicOut)
    .onUpdate((value) => {
        sprite.position.x = value;
    })
    .onComplete(() => {
        console.log("Movement complete!");
    })
    .start();

// In your game loop:
tween.update(deltaTime);
```

### Static Factory

For quick one-liners:

```typescript
const tw = Tween.CreateAsync(0, 1, 0.3, Easing.QuadOut, (v) => {
    sprite.alpha = v;
});
```

## Delay

Start the animation after a delay:

```typescript
new Tween({ from: 0, to: 100 }, 0.5)
    .setDelay(0.2) // Wait 200ms before starting
    .onUpdate((v) => { sprite.position.y = v; })
    .start();
```

## Looping & Repeat

```typescript
// Loop forever
new Tween({ from: 0, to: 10 }, 1, Easing.SineInOut)
    .setLoop(true) // yoyo: true = ping-pong
    .onUpdate((v) => { sprite.position.y = baseY + v; })
    .start();

// Repeat 3 times (plays 4 total) with yoyo
new Tween({ from: 0, to: 1 }, 0.5)
    .setRepeat(3, true)
    .onUpdate((v) => { sprite.alpha = v; })
    .start();
```

## Chaining

Sequence multiple tweens:

```typescript
const moveRight = new Tween({ from: 0, to: 300 }, 0.5, Easing.QuadOut)
    .onUpdate((v) => { sprite.position.x = v; });

const moveDown = new Tween({ from: 0, to: 200 }, 0.3, Easing.QuadOut)
    .onUpdate((v) => { sprite.position.y = v; });

// moveDown starts automatically when moveRight completes
moveRight.chain(moveDown).start();
```

## TweenManager

The `TweenManager` batch-updates all active tweens and automatically removes completed ones:

```typescript
import { TweenManager, Tween, Easing } from "@babylonjs/2d";

const tweens = new TweenManager();

// Add tweens as needed
tweens.add(
    new Tween({ from: 0, to: 100 }, 0.5, Easing.QuadOut)
        .onUpdate((v) => { unit.sprite.position.x = v; })
        .start()
);

// In your game loop — one call updates everything:
tweens.update(deltaTime);

// Check how many are active:
console.log(tweens.count);

// Clean up:
tweens.stopAll();  // Stop without completing
tweens.dispose();  // Full cleanup
```

## Controlling Tweens

```typescript
// Jump to end immediately (fires onComplete)
tween.complete();

// Freeze at current value (does NOT fire onComplete)
tween.stop();

// Check state
tween.isComplete;   // boolean
tween.state;        // TweenState.Pending | Running | Complete
tween.progress;     // 0–1
tween.currentValue; // current interpolated value
```

## Real-World Example: Tactics Unit Movement

```typescript
function moveUnit(unit: IUnit, targetCol: number, targetRow: number): void {
    const targetPos = grid.cellToWorld(targetCol, targetRow);
    const startX = unit.sprite.position.x;
    const startY = unit.sprite.position.y;

    const twX = new Tween({ from: startX, to: targetPos.x }, 0.3, Easing.QuadInOut)
        .onUpdate((v) => { unit.sprite.position.x = v; });

    const twY = new Tween({ from: startY, to: targetPos.y }, 0.3, Easing.QuadInOut)
        .onUpdate((v) => { unit.sprite.position.y = v; })
        .onComplete(() => { isAnimating = false; });

    tweenManager.add(twX.start());
    tweenManager.add(twY.start());
}
```
