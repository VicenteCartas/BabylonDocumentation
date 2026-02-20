---
title: 9-Slice Sprites
description: Resizable UI panels and buttons with undistorted borders using NineSliceSprite2D
keywords: 2d, nine-slice, 9-patch, ui, panel, button, border, resize
---

# 9-Slice Sprites

`NineSliceSprite2D` renders a texture as a resizable panel whose corners stay fixed, edges stretch in one direction, and center stretches in both directions. This is essential for UI elements like dialog boxes, buttons, and health bars.

```
┌────────┬────────────┬────────┐
│ Corner  │  Top Edge  │ Corner │  ← fixed height
├────────┼────────────┼────────┤
│  Left  │   Center   │  Right │  ← stretches vertically
│  Edge  │ (stretches)│  Edge  │
├────────┼────────────┼────────┤
│ Corner  │  Bot Edge  │ Corner │  ← fixed height
└────────┴────────────┴────────┘
  fixed     stretches    fixed
```

## Setup

```typescript
import { NineSliceSprite2D } from "@babylonjs/2d";

const panel = new NineSliceSprite2D("panel", panelTexture);
panel.setBorders(16, 16, 16, 16); // 16px border on all sides
panel.width = 400;
panel.height = 200;
panel.position = new Vector2(400, 300);
scene.root.addChild(panel);
```

The border values refer to pixels in the **source texture**, not the display size. A 64×64 texture with 16px borders has a 32×32 stretchable center.

## Border Configuration

```typescript
// Different borders per side
panel.setBorders(12, 12, 20, 8); // left, right, top, bottom

// Uniform borders (shorthand)
panel.setUniformBorders(16); // 16px on all sides

// Direct property access
panel.borderLeft = 10;
panel.borderRight = 10;
panel.borderTop = 20;
panel.borderBottom = 20;
```

## How It Works

The texture is divided into a 3×3 grid based on the border insets:

| Region | Source Area | Display Behavior |
|--------|------------|-----------------|
| 4 corners | Fixed-size corner patches | Never scaled |
| Top/Bottom edges | Horizontal strip between corners | Stretched horizontally |
| Left/Right edges | Vertical strip between corners | Stretched vertically |
| Center | Inner rectangle | Stretched in both directions |

When you resize the panel (change `width`/`height`), only the stretchable regions change. Corners keep their pixel-perfect appearance at any panel size.

## With Source Rects

Use `sourceRect` to select a region within a texture atlas:

```typescript
import { Rectangle2D } from "@babylonjs/2d";

const button = new NineSliceSprite2D("button", atlasTexture);
button.sourceRect = new Rectangle2D(0, 128, 64, 32); // region in atlas
button.setBorders(8, 8, 8, 8);
button.width = 200;
button.height = 48;
```

## Edge Cases

**Panel smaller than borders**: If the panel width/height is smaller than the combined border sizes, borders are proportionally scaled down so they still fit. For example, a 20px-wide panel with 16px left + 16px right borders scales each to 10px.

**Zero-width center**: If width equals borderLeft + borderRight, the center column has zero width and only left/right edges render (3 quads per row instead of full 9).

## Examples

### Dialog Box

```typescript
const dialog = new NineSliceSprite2D("dialog", dialogBgTexture);
dialog.setUniformBorders(24);
dialog.width = 500;
dialog.height = 300;
dialog.position = new Vector2(screenWidth / 2, screenHeight / 2);
```

### Health Bar Background

```typescript
const hpBar = new NineSliceSprite2D("hpBg", barTexture);
hpBar.setBorders(4, 4, 4, 4);
hpBar.width = playerHP / maxHP * 200; // Dynamic width
hpBar.height = 20;
```

### Button with Different States

```typescript
function makeButton(name: string, srcY: number): NineSliceSprite2D {
    const btn = new NineSliceSprite2D(name, uiAtlas);
    btn.sourceRect = new Rectangle2D(0, srcY, 64, 32);
    btn.setBorders(8, 8, 8, 8);
    btn.width = 160;
    btn.height = 40;
    return btn;
}

const normal = makeButton("btn_normal", 0);
const hover = makeButton("btn_hover", 32);
const pressed = makeButton("btn_pressed", 64);
```

## Performance

`NineSliceSprite2D` renders as 9 quads (or fewer when edges have zero size). Thanks to multi-texture batching in the `SpriteBatchRenderer`, all 9 quads share the same texture and batch into a single draw call alongside other sprites using the same texture.
