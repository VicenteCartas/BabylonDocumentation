---
title: Masks & Clipping
description: Restrict rendering to specific regions using rectangular scissor masks and sprite-shaped stencil masks
keywords: 2d, mask, clipping, scissor, stencil, rect mask, sprite mask, ui, scroll, viewport
---

# Masks & Clipping

Masks restrict rendering of a node's subtree to a specific region. Any `Node2D` can have a mask assigned via its `mask` property — all children (and the node's own visuals if it's a `Sprite2D`) are clipped to the mask region.

Common use cases include:

| Use Case | Mask Type |
|----------|-----------|
| **Scroll regions / UI panels** | `RectMask2D` |
| **Minimap viewports** | `RectMask2D` |
| **Circular portraits / shaped windows** | `SpriteMask2D` |
| **Health bar fills** | `SpriteMask2D` |
| **Reveal / wipe effects** | `SpriteMask2D` |
| **Fog-of-war borders** | `SpriteMask2D` |

The 2D engine provides two mask types, each optimized for a different scenario:

- **`RectMask2D`** — GPU scissor test. The cheapest option for axis-aligned rectangular clipping.
- **`SpriteMask2D`** — Stencil buffer test. Uses a sprite's alpha channel to define an arbitrary mask shape.

## RectMask2D

`RectMask2D` clips all children of a node to an axis-aligned rectangle using the hardware scissor test. It has **zero stencil overhead** — the GPU simply skips pixels outside the rectangle with no extra draw calls or shader changes.

### Creating a Rect Mask

```typescript
import { Scene2D, Node2D, Sprite2D, RectMask2D } from "@babylonjs/2d";
import { Vector2 } from "@babylonjs/core/Maths/math.vector";
import { Engine } from "@babylonjs/core";

const engine = new Engine(canvas);
const scene = new Scene2D(engine);

// Create a panel that clips its children to a 400×300 region
const scrollPanel = new Node2D("scrollPanel", scene);
scrollPanel.position = new Vector2(50, 50);
scrollPanel.mask = new RectMask2D(0, 0, 400, 300);

// Content inside the panel — only visible within the 400×300 rect
const content = new Node2D("content", scene);
content.parent = scrollPanel;

for (let i = 0; i < 20; i++) {
    const item = new Sprite2D(`item-${i}`, scene);
    item.texture = listItemTexture;
    item.position.y = i * 40;
    item.parent = content;
}

// Scroll by moving the content node — the mask stays fixed to the panel
scene.onBeforeRender.add(() => {
    content.position.y -= scrollSpeed * deltaTime;
});
```

The rectangle is defined in the **local space** of the node it's assigned to (pixels, Y-down coordinate system). It transforms with the node's `worldTransform` automatically.

### Constructor

```typescript
new RectMask2D(x?: number, y?: number, width?: number, height?: number)
```

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `x` | `number` | `0` | Left edge in local space |
| `y` | `number` | `0` | Top edge in local space |
| `width` | `number` | `0` | Width in pixels (0 = use node bounds) |
| `height` | `number` | `0` | Height in pixels (0 = use node bounds) |

### Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `rect` | `Rectangle2D` | from constructor | The clipping rectangle in the owning node's local space |
| `padding` | `number` | `0` | Expands the clipping rectangle by this many pixels on each side. Useful for preventing edge-pixel artifacts when sprites extend slightly beyond the mask boundary. |
| `enabled` | `boolean` | `true` | Toggle the mask on/off. When `false`, children render normally. |
| `inverted` | `boolean` | `false` | When `true`, pixels **inside** the rectangle are hidden instead of visible. Inverted rect masks use a stencil fallback internally (scissor can only clip *to* a rect, not *away* from one). |

### Padding

Padding expands the effective clipping rectangle, which is helpful when sprite edges land exactly on the mask boundary:

```typescript
const mask = new RectMask2D(0, 0, 400, 300);
mask.padding = 2; // 2px extra on all sides → effective rect is (-2, -2, 404, 304)

scrollPanel.mask = mask;
```

### Performance

`RectMask2D` is the cheapest mask type:

- Uses the hardware scissor test — one `gl.enable(SCISSOR_TEST)` + `gl.scissor()` call on push, one `gl.disable(SCISSOR_TEST)` on pop
- No extra draw calls, no shader changes, no stencil buffer writes
- Cost: ~0.01ms per mask push/pop (dominated by the batch flush at the mask boundary)

## SpriteMask2D

`SpriteMask2D` uses a sprite's rendered alpha to define the visible region via the stencil buffer. Pixels where the mask sprite's alpha meets or exceeds a configurable threshold pass the stencil test; pixels below the threshold are clipped.

### Creating a Sprite Mask

```typescript
import { Sprite2D, SpriteMask2D, Node2D } from "@babylonjs/2d";

// Create the mask sprite — pass null for scene so it is NOT added to the display list
const maskSprite = new Sprite2D("circleMask", null);
maskSprite.texture = circleTexture; // A circle with transparent corners

const container = new Node2D("masked", scene);
container.mask = new SpriteMask2D(maskSprite);

// All children are clipped to the circle shape
const portrait = new Sprite2D("portrait", scene);
portrait.texture = characterTexture;
portrait.parent = container;
```

<Alert severity="info">
The mask sprite does **not** need to be in the scene's display list. Pass `null` for the scene parameter when creating it (`new Sprite2D("mask", null)`) so it exists solely as a mask and is never rendered as a visible sprite.
</Alert>

### Constructor

```typescript
new SpriteMask2D(sprite: Sprite2D, alphaThreshold?: number)
```

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `sprite` | `Sprite2D` | *(required)* | The sprite whose alpha defines the mask shape |
| `alphaThreshold` | `number` | `0.5` | Alpha cutoff (0–1) for the stencil test |

### Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `sprite` | `Sprite2D` | from constructor | The sprite whose rendered alpha defines the mask shape. Its texture, transform, and alpha are all used. |
| `alphaThreshold` | `number` | `0.5` | Pixels with alpha ≥ threshold are visible; pixels below are clipped. Lower values = more permissive (semi-transparent areas pass). Higher values = stricter (only fully opaque areas pass). |
| `enabled` | `boolean` | `true` | Toggle the mask on/off |
| `inverted` | `boolean` | `false` | When `true`, pixels where the mask sprite **is** opaque become hidden |

### Alpha Threshold

The `alphaThreshold` controls how much of the mask sprite's alpha is considered "solid":

```typescript
// Strict — only fully opaque pixels of the mask sprite pass
const mask = new SpriteMask2D(maskSprite, 0.9);

// Permissive — even semi-transparent pixels pass
const softMask = new SpriteMask2D(maskSprite, 0.1);
```

This is especially useful for reveal effects with gradient textures — adjust the threshold or slide the mask sprite to progressively reveal content.

### Example: Shaped Health Bar

```typescript
import { Sprite2D, SpriteMask2D, Node2D } from "@babylonjs/2d";

// Mask sprite: a white rectangle anchored at the left edge
const maskSprite = new Sprite2D("healthMask", null);
maskSprite.texture = whiteRectTexture;
maskSprite.width = 200;
maskSprite.height = 20;
maskSprite.pivot.x = 0; // Anchor at left edge

// Health bar container
const healthBar = new Node2D("healthBar", scene);
healthBar.position.set(100, 50);
healthBar.mask = new SpriteMask2D(maskSprite);

// The fill sprite (full-width gradient or solid color)
const fill = new Sprite2D("fill", scene);
fill.texture = healthBarTexture;
fill.width = 200;
fill.height = 20;
fill.parent = healthBar;

// Update health: scale the mask sprite to reveal only the filled portion
function setHealth(percent: number): void {
    maskSprite.scale.x = Math.max(0, Math.min(1, percent));
    maskSprite.position.copyFrom(healthBar.position);
}
```

### Example: Animated Reveal Wipe

```typescript
import { Sprite2D, SpriteMask2D, Node2D } from "@babylonjs/2d";

// A horizontal gradient texture (black → white) used as a wipe mask
const wipeMask = new Sprite2D("wipeMask", null);
wipeMask.texture = horizontalGradientTexture;
wipeMask.width = 800;
wipeMask.height = 600;

const revealContainer = new Node2D("reveal", scene);
revealContainer.mask = new SpriteMask2D(wipeMask, 0.5);

const hiddenContent = new Sprite2D("secret", scene);
hiddenContent.texture = secretImageTexture;
hiddenContent.parent = revealContainer;

// Animate: slide the mask from left to right to reveal the content
let wipeProgress = 0;
scene.onBeforeRender.add(() => {
    wipeProgress += deltaTime * 0.5;
    wipeMask.position.x = -800 + wipeProgress * 1600;
});
```

### Performance

`SpriteMask2D` is more expensive than `RectMask2D` because it involves stencil buffer operations:

- Batch flush + render mask sprite into the stencil buffer (1 draw call) + stencil state changes on push
- Batch flush + stencil decrement + stencil restore on pop
- Cost: ~0.1ms per mask push/pop, dominated by the batch flush

## Common Properties

Both `RectMask2D` and `SpriteMask2D` implement the `IMask2D` interface and share these properties:

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `enabled` | `boolean` | `true` | When `false`, the mask has no effect — children render normally |
| `inverted` | `boolean` | `false` | Show what would normally be hidden and vice versa |

### Enabling and Disabling

Toggle a mask without removing it from the node:

```typescript
// Temporarily disable
scrollPanel.mask!.enabled = false; // Children render without clipping

// Re-enable
scrollPanel.mask!.enabled = true;
```

### Inverting

Inversion flips which region is visible:

```typescript
// Normal: show content INSIDE the circle
container.mask = new SpriteMask2D(circleSprite);

// Inverted: show content OUTSIDE the circle (e.g., a vignette effect)
container.mask = new SpriteMask2D(circleSprite);
container.mask.inverted = true;
```

<Alert severity="warning">
Inverted `RectMask2D` masks use a stencil fallback internally — the hardware scissor test can only clip *to* a rectangle, not *away* from one. This means inverted rect masks have the same cost as a `SpriteMask2D`.
</Alert>

## Assigning Masks to Nodes

Masks are assigned via the `Node2D.mask` property:

```typescript
// Assign a mask
node.mask = new RectMask2D(0, 0, 300, 200);

// Replace with a different mask
node.mask = new SpriteMask2D(circleSprite);

// Remove the mask entirely
node.mask = null;
```

<Alert severity="info">
Masks are **not** auto-disposed when the node is disposed. If a `SpriteMask2D` holds a sprite reference you want cleaned up, dispose it explicitly. This allows masks to be shared across multiple nodes.
</Alert>

## Nested Masks

Masks on parent nodes affect all children. When masks are nested, the visible region is the **intersection** of all active masks in the hierarchy:

```typescript
// Outer mask: circular viewport (stencil)
const outerMask = new Sprite2D("outerMask", null);
outerMask.texture = circleTexture;

const viewport = new Node2D("viewport", scene);
viewport.mask = new SpriteMask2D(outerMask);

// Inner mask: rectangular scroll region inside the circular viewport (scissor)
const scrollRegion = new Node2D("scroll", scene);
scrollRegion.parent = viewport;
scrollRegion.mask = new RectMask2D(10, 10, 180, 180);

// Content is clipped to BOTH masks — visible only within the intersection
const content = new Sprite2D("content", scene);
content.parent = scrollRegion;
content.texture = mapTexture;
```

**How nesting works internally:**

- **Nested `RectMask2D`:** Uses scissor rect intersection. The child rect is clipped to the parent rect. No depth limit.
- **Nested `SpriteMask2D`:** Stencil reference value is incremented for each nested level. Supports up to 255 levels of nesting (8-bit stencil buffer).
- **Mixed nesting** (rect inside sprite or vice versa) works naturally — each mask type uses its own GPU mechanism independently.

## Works with All 2D Node Types

Both `NineSliceSprite2D` and `Text2D` work inside masks with no special handling needed. Their render data flows through the same collection and batching pipeline:

```typescript
import { Node2D, NineSliceSprite2D, Text2D, RectMask2D } from "@babylonjs/2d";

// Scrollable UI panel with 9-slice background + text, all clipped
const panel = new Node2D("uiPanel", scene);
panel.mask = new RectMask2D(0, 0, 320, 240);

const bg = new NineSliceSprite2D("bg", panelTexture);
bg.setUniformBorders(16);
bg.width = 320;
bg.height = 500; // Taller than mask — will be clipped
bg.parent = panel;

const title = new Text2D("title", "Inventory", { font: "24px Arial", color: "#fff" });
title.position.y = 10;
title.parent = panel;
// Both the 9-slice quads and the text quad are clipped to the 320×240 mask rect
```

## Performance Tips

1. **Prefer `RectMask2D` whenever possible.** It uses the hardware scissor test — zero stencil overhead, zero extra draw calls. If your clipping region is rectangular, always use `RectMask2D`.

2. **Masks cause batch breaks.** Every mask push/pop forces the sprite batch renderer to flush. If you have many small masked regions, each one adds a draw call boundary. Group masked content together to minimize flushes.

3. **Keep stencil mask counts reasonable.** Each `SpriteMask2D` renders its mask sprite into the stencil buffer (1 extra draw call). In a typical scene, 8–16 stencil masks are very affordable. Beyond that, profile to make sure you're within budget.

4. **Zero-cost default.** When no node in the scene has a mask, the render loop is completely unchanged — the same flat sprite collection + sort + batch path runs with zero overhead. A single boolean check gates the entire mask system.

5. **Sprites within a masked subtree cannot z-sort with sprites outside that subtree.** Mask boundaries act as sort barriers. This matches the expected visual behavior — if you mask a container, its contents are visually "inside" the mask regardless of `zIndex`.

## Limitations

- **`RectMask2D` is always axis-aligned.** If the owning node is rotated, the scissor rect uses the axis-aligned bounding box (AABB) of the transformed rectangle. The scissor test cannot clip to a rotated rect. If you need a rotated rectangular mask, use a `SpriteMask2D` with a rectangular texture instead.

- **Inverted `RectMask2D` falls back to stencil.** The hardware scissor can only clip *to* a rectangle, not *away* from one. When `inverted` is `true`, a `RectMask2D` internally uses the stencil path with the same cost as a `SpriteMask2D`.

- **`AnimatedSprite2D` as a mask requires manual `update()`.** If you use an `AnimatedSprite2D` as a mask sprite with `scene = null` (not in the display list), it won't receive automatic frame updates. You must call `maskSprite.update(deltaTime)` manually each frame to advance the animation.

- **No soft-edge alpha blending.** Both mask types produce binary clipping (visible or hidden). Per-pixel alpha gradient masks (soft edges) are not currently supported. This could be added in the future as an `AlphaMask2D` type using [Render Textures](/features/featuresDeepDive/2d/renderTexture).

## Cleanup

`RectMask2D` holds no GPU resources — its `dispose()` is a no-op. `SpriteMask2D.dispose()` clears the sprite reference but does **not** dispose the sprite itself (the caller owns the sprite's lifecycle):

```typescript
// Remove and clean up a sprite mask
const mask = container.mask as SpriteMask2D;
container.mask = null;

// Dispose the mask sprite if you're done with it
mask.sprite.dispose();
mask.dispose();
```

## API Reference

### IMask2D (Interface)

| Property | Type | Description |
|----------|------|-------------|
| `enabled` | `boolean` | Whether the mask is active |
| `inverted` | `boolean` | Whether to invert the mask region |
| `dispose()` | `void` | Release resources held by the mask |

### RectMask2D

| Member | Type | Description |
|--------|------|-------------|
| `rect` | `Rectangle2D` | The clipping rectangle in local space (Y-down) |
| `padding` | `number` | Pixels to expand the clipping rect by |
| `enabled` | `boolean` | Toggle on/off |
| `inverted` | `boolean` | Invert the mask (uses stencil fallback) |
| `dispose()` | `void` | No-op (no GPU resources) |

### SpriteMask2D

| Member | Type | Description |
|--------|------|-------------|
| `sprite` | `Sprite2D` | The sprite whose alpha defines the mask shape |
| `alphaThreshold` | `number` | Alpha cutoff for the stencil test (0–1) |
| `enabled` | `boolean` | Toggle on/off |
| `inverted` | `boolean` | Invert the mask |
| `dispose()` | `void` | Clears the sprite reference (does NOT dispose the sprite) |
