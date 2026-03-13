---
title: Text and UI in 2D Games
description: Rendering text and UI elements in Babylon.js 2D games using Text2D and the GUI system
keywords: 2d, text, label, ui, hud, gui, textblock, button
---

# Text and UI in 2D Games

Babylon.js 2D provides two approaches for text and UI:

- **Text2D** — Lightweight in-world text that lives in the 2D scene graph. Use for damage numbers, NPC names, score labels, floating text.
- **GUI (AdvancedDynamicTexture)** — Full-featured UI system for HUDs, menus, buttons, sliders, and panels. Renders as a screen-space overlay.

## Text2D — In-World Text

`Text2D` extends `Sprite2D`, so it inherits position, rotation, scale, alpha, tint, z-index, and parent-child hierarchy. Text is rasterized to a texture using the browser's canvas 2D API.

### Basic Usage

```typescript
import { Text2D } from "@babylonjs/2d/Text2D/text2D";

const label = new Text2D("score", "Score: 0", {
    font: "24px Arial",
    color: "#ffffff",
});
label.position = new Vector2(10, 10);
scene.addNode(label);
```

### Updating Text

Setting the `text` property automatically re-renders the texture on the next frame:

```typescript
label.text = "Score: 100";
```

### Styling Options

```typescript
const damage = new Text2D("dmg", "-25", {
    font: "bold 32px monospace",
    color: "#ff4444",
    textAlign: "center",     // "left", "center", "right"
    textBaseline: "middle",  // "top", "middle", "bottom"
    padding: 4,              // Pixels of padding around the text
});
```

### Scene Graph Integration

Because `Text2D` is a `Node2D`, it works naturally with the scene graph:

```typescript
// Attach to a sprite — label follows the sprite automatically
const npc = new Sprite2D("npc", npcTexture);
const nameTag = new Text2D("name", "Merchant", {
    font: "14px Arial",
    color: "#aaffaa",
});
nameTag.position = new Vector2(0, -30); // Offset above the sprite
npc.addChild(nameTag);

// Alpha, rotation, scale inherited from parent
npc.alpha = 0.5; // Name tag also becomes semi-transparent

// Z-index for layering
nameTag.zIndex = 100; // Render above other sprites
```

### Tinting

Since `Text2D` inherits from `Sprite2D`, you can tint the rendered text:

```typescript
label.tint = new Color4(1, 0.8, 0, 1); // Gold tint over white text
```

## GUI — Screen-Space UI

For HUD elements, menus, buttons, and interactive UI, use the core GUI package (`@babylonjs/gui`). The GUI renders as a screen overlay independent of the 2D scene.

### Setup

```typescript
import { AdvancedDynamicTexture } from "@babylonjs/gui/2D/advancedDynamicTexture";
import { TextBlock } from "@babylonjs/gui/2D/controls/textBlock";
import { Button } from "@babylonjs/gui/2D/controls/button";
import { StackPanel } from "@babylonjs/gui/2D/controls/stackPanel";
import { Control } from "@babylonjs/gui/2D/controls/control";

// Create a fullscreen UI layer
const ui = AdvancedDynamicTexture.CreateFullscreenUI("ui");
```

### HUD Text

```typescript
const scoreText = new TextBlock("score");
scoreText.text = "Score: 0";
scoreText.color = "white";
scoreText.fontSize = 24;
scoreText.textHorizontalAlignment = Control.HORIZONTAL_ALIGNMENT_LEFT;
scoreText.textVerticalAlignment = Control.VERTICAL_ALIGNMENT_TOP;
scoreText.left = "10px";
scoreText.top = "10px";
ui.addControl(scoreText);

// Update in game loop
scoreText.text = `Score: ${score}`;
```

### Buttons

```typescript
const pauseBtn = Button.CreateSimpleButton("pause", "Pause");
pauseBtn.width = "100px";
pauseBtn.height = "40px";
pauseBtn.color = "white";
pauseBtn.background = "rgba(0,0,0,0.5)";
pauseBtn.horizontalAlignment = Control.HORIZONTAL_ALIGNMENT_RIGHT;
pauseBtn.verticalAlignment = Control.VERTICAL_ALIGNMENT_TOP;
pauseBtn.top = "10px";
pauseBtn.left = "-10px";
pauseBtn.onPointerClickObservable.add(() => {
    paused = !paused;
});
ui.addControl(pauseBtn);
```

### Layout Panels

```typescript
const panel = new StackPanel("hud");
panel.isVertical = false;
panel.height = "40px";
panel.verticalAlignment = Control.VERTICAL_ALIGNMENT_BOTTOM;

const hpBar = new TextBlock("hp");
hpBar.text = "HP: 100";
hpBar.color = "#44ff44";
hpBar.width = "100px";
panel.addControl(hpBar);

const ammoText = new TextBlock("ammo");
ammoText.text = "Ammo: 30";
ammoText.color = "#ffff44";
ammoText.width = "100px";
panel.addControl(ammoText);

ui.addControl(panel);
```

## When to Use What

| Need | Use | Why |
|------|-----|-----|
| Damage numbers, floating labels | **Text2D** | Moves with the game world, respects camera zoom/scroll |
| NPC names above sprites | **Text2D** | Parent-child hierarchy, automatic positioning |
| HUD (score, health, ammo) | **GUI** | Fixed on screen, rich layout options |
| Menus, buttons, sliders | **GUI** | Interactive controls, event handling |
| Dialog boxes | **GUI** | Text wrapping, panels, scroll viewers |
