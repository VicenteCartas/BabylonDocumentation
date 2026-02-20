---
title: Input Mapping
description: Action-based input mapping for keyboard, mouse, and gamepad in Babylon.js 2D
keywords: 2d, input, keyboard, mouse, gamepad, action mapping
---

# Input Mapping

`InputMap2D` provides action-based input mapping that abstracts physical inputs (keyboard keys, mouse buttons, touch, gamepad buttons/axes) into named game actions. Built on Babylon.js core's `DeviceSourceManager`, it automatically handles both mouse and touch input — any `mouseButton` binding responds to touch as well.

## Setup

```typescript
import { InputMap2D } from "@babylonjs/2d";

const input = new InputMap2D(engine);
```

Optionally pass a `Camera2D` for automatic screen-to-world pointer conversion:

```typescript
const input = new InputMap2D(engine, camera);
```

## Defining Actions

Map abstract action names to physical input bindings:

```typescript
// Multiple bindings per action (keyboard + gamepad)
input.defineAction("jump",
    { type: "key", key: "Space" },
    { type: "gamepadButton", button: 0 }
);

input.defineAction("moveRight",
    { type: "key", key: "ArrowRight" },
    { type: "key", key: "KeyD" },
    { type: "gamepadAxis", axis: 0, direction: 1 }
);

input.defineAction("moveLeft",
    { type: "key", key: "ArrowLeft" },
    { type: "key", key: "KeyA" },
    { type: "gamepadAxis", axis: 0, direction: -1 }
);

input.defineAction("attack",
    { type: "mouseButton", button: 0 },
    { type: "gamepadButton", button: 2 }
);
```

### Binding Types

| Type | Properties | Description |
|------|-----------|-------------|
| `key` | `key: string` | Keyboard key code (e.g., `"Space"`, `"KeyW"`, `"ArrowUp"`) |
| `mouseButton` | `button: number` | Mouse button (0=left, 1=middle, 2=right) |
| `gamepadButton` | `button: number` | Gamepad button index |
| `gamepadAxis` | `axis: number, direction: 1\|-1` | Gamepad axis with direction |

## Querying Actions

Call `update()` once per frame before querying:

```typescript
engine.runRenderLoop(() => {
    input.update();  // Must be called first!

    if (input.isActionPressed("jump")) {
        // Just pressed this frame (rising edge)
        player.jump();
    }

    if (input.isActionDown("moveRight")) {
        // Held down
        player.velocity.x = speed;
    }

    if (input.isActionReleased("attack")) {
        // Just released this frame (falling edge)
        player.endAttack();
    }
});
```

### Query Methods

| Method | Returns | Description |
|--------|---------|-------------|
| `isActionDown(name)` | `boolean` | True while any binding is active |
| `isActionPressed(name)` | `boolean` | True only on the frame the action starts |
| `isActionReleased(name)` | `boolean` | True only on the frame the action stops |
| `getAxis(name)` | `number` | -1 to 1 value (analog for gamepad, 0 or 1 for digital) |

## Pointer Position

```typescript
// Screen-space position
const screenPos = input.pointerScreenPosition;

// World-space position (requires camera)
const worldPos = input.pointerWorldPosition;

// Mouse button state
if (input.isPointerDown(0)) { /* left button held */ }
```

## Cleanup

```typescript
input.dispose(); // Removes all event listeners
```

## Example: Full Movement Setup

```typescript
const input = new InputMap2D(engine, camera);

// Define movement axes
input.defineAction("moveRight",
    { type: "key", key: "ArrowRight" },
    { type: "key", key: "KeyD" }
);
input.defineAction("moveLeft",
    { type: "key", key: "ArrowLeft" },
    { type: "key", key: "KeyA" }
);

// In game loop
input.update();
let moveX = 0;
if (input.isActionDown("moveRight")) moveX += 1;
if (input.isActionDown("moveLeft")) moveX -= 1;
player.position.x += moveX * speed * dt;
```
