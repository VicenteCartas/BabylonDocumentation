---
title: Scene Transitions
image:
description: Animated transitions between Scene2D instances — fade and slide effects.
keywords: scene, transition, fade, slide, 2d
further-reading:
video-overview:
video-content:
---

## Overview

`SceneTransition2D` provides animated transitions between two `Scene2D` instances.
It ships with two built-in effects:

| Effect | Description |
| ------ | ----------- |
| **Fade** | Fades the old scene to a solid color, then fades in the new scene |
| **Slide** | Old scene slides out while new scene slides in from the opposite edge |

## Fade Transition

Fade transitions cover the screen with a solid color (default: black), then reveal
the new scene. Duration is split equally between fade-out and fade-in.

```typescript
import { SceneTransition2D } from "@babylonjs/2d";
import { Color4 } from "@babylonjs/core/Maths/math.color";

const transition = SceneTransition2D.fade({
    from: currentScene,
    to: newScene,
    duration: 0.8,               // seconds (total)
    color: new Color4(0, 0, 0, 1), // black (default)
    easing: Easing.SineInOut,    // default
    onComplete: () => {
        console.log("Transition done!");
    },
});
```

### Render Loop Integration

While a transition is active, call `transition.update(dt)` and
`transition.render()` instead of your normal `scene.render()`:

```typescript
let activeScene = menuScene;

engine.runRenderLoop(() => {
    const dt = engine.getDeltaTime() / 1000;

    if (transition && transition.isActive) {
        transition.update(dt);
        transition.render();
    } else {
        activeScene.render();
    }
});
```

When the transition completes, `isActive` becomes `false` and `isDone` becomes
`true`. The `onComplete` callback fires at that moment — use it to update your
`activeScene` reference.

## Slide Transition

Slide transitions move the old scene off-screen in one direction while the new
scene enters from the opposite edge. Both scenes are rendered simultaneously in
a single frame using `renderContent()`.

```typescript
const transition = SceneTransition2D.slide({
    from: currentScene,
    to: newScene,
    duration: 0.6,
    direction: "left",           // "left" | "right" | "up" | "down"
    easing: Easing.CubicInOut,   // default
    onComplete: () => {
        activeScene = newScene;
    },
});
```

Supported directions (where the *old* scene moves toward):

| Direction | Old scene exits | New scene enters from |
| --------- | --------------- | -------------------- |
| `"left"` | ← left edge | right edge → |
| `"right"` | → right edge | ← left edge |
| `"up"` | ↑ top edge | bottom edge ↓ |
| `"down"` | ↓ bottom edge | ↑ top edge |

> **Note:** Slide transitions modify camera positions during the animation and
> restore them when complete. Both scenes must have a `Camera2D` assigned.

## Scene2D.renderContent()

Scene transitions introduced `renderContent(clear?)`, which renders a scene's
sprites without calling `engine.beginFrame()` / `engine.endFrame()`. This
enables compositing multiple scenes in a single GPU frame:

```typescript
engine.beginFrame();
sceneA.renderContent(true);   // clear + draw
sceneB.renderContent(false);  // draw on top (no clear)
engine.endFrame();
```

The existing `scene.render()` method calls `renderContent()` internally and
remains the recommended API for normal (non-transition) rendering.

## Custom Transitions

You can build custom transitions by combining `renderContent()` with the
`Tween` system. For example, a "zoom" transition:

```typescript
import { Tween, Easing } from "@babylonjs/2d";

function zoomTransition(from: Scene2D, to: Scene2D) {
    const cam = from.camera!;
    const originalZoom = cam.zoom;

    const tween = new Tween({ from: 1, to: 3 }, 0.4, Easing.CubicIn)
        .onUpdate((v) => { cam.zoom = v; })
        .onComplete(() => {
            cam.zoom = originalZoom;
            // Switch to new scene...
        })
        .start();

    // Call tween.update(dt) each frame
}
```

## API Reference

### SceneTransition2D

| Property | Type | Description |
| -------- | ---- | ----------- |
| `isActive` | `boolean` | `true` while the transition is running |
| `isDone` | `boolean` | `true` after the transition completes |
| `activeScene` | `Scene2D` | The scene currently being rendered |

| Method | Description |
| ------ | ----------- |
| `update(dt)` | Advance the transition by `dt` seconds |
| `render()` | Render the current transition frame |
| `static fade(options)` | Create a fade transition |
| `static slide(options)` | Create a slide transition |

### IFadeTransitionOptions

| Option | Type | Default | Description |
| ------ | ---- | ------- | ----------- |
| `from` | `Scene2D` | *required* | Source scene |
| `to` | `Scene2D` | *required* | Destination scene |
| `duration` | `number` | `0.5` | Total duration in seconds |
| `color` | `Color4` | black | Fade color |
| `easing` | `EasingFunction` | `SineInOut` | Easing function |
| `onComplete` | `() => void` | — | Completion callback |

### ISlideTransitionOptions

| Option | Type | Default | Description |
| ------ | ---- | ------- | ----------- |
| `from` | `Scene2D` | *required* | Source scene |
| `to` | `Scene2D` | *required* | Destination scene |
| `duration` | `number` | `0.5` | Total duration in seconds |
| `direction` | `string` | `"left"` | Slide direction |
| `easing` | `EasingFunction` | `CubicInOut` | Easing function |
| `onComplete` | `() => void` | — | Completion callback |
