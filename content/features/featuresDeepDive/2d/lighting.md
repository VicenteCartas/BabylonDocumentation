---
title: 2D GPU Lighting
description: GPU-accelerated 2D lighting with point lights, spotlights, and ambient illumination
keywords: 2d, lighting, point light, spotlight, ambient, gpu, forward lighting, shader
---

# 2D GPU Lighting

The `LightingManager2D` provides GPU-accelerated forward lighting for 2D scenes. Lights are packed into shader uniforms and evaluated per-pixel in the fragment shader — no CPU per-sprite sampling needed. Up to 16 simultaneous lights are supported (point, spot, and ambient).

## Setup

```typescript
import { LightingManager2D } from "@babylonjs/2d";
import { Color4 } from "@babylonjs/core/Maths/math.color";

const lighting = new LightingManager2D();
lighting.ambientColor = new Color4(0.1, 0.08, 0.15, 1); // Dark ambient

// Attach to the scene — the renderer auto-switches to the lit shader
scene.lightingManager = lighting;
```

When `scene.lightingManager` is set, `Scene2D` automatically:
1. Packs light data into uniforms each frame (positions transformed to view space)
2. Tells `SpriteBatchRenderer` to use the lit shader variant
3. Applies per-pixel lighting to every sprite in the batch

## Light Types

### Point Light

Radiates light in all directions from a position:

```typescript
const torch = lighting.createPointLight(200, 300, new Color4(1, 0.8, 0.4, 1), 250);
torch.intensity = 1.5;
torch.falloff = 1.5; // Higher = sharper edge (default 2)
```

### Spotlight

A cone-shaped light with inner (full intensity) and outer (fade to zero) angles:

```typescript
const spot = lighting.createSpotLight(
    400, 100,                        // position
    new Vector2(0, 1),               // direction (pointing down)
    new Color4(1, 1, 1, 1),         // white
    300,                             // radius
    Math.PI / 8,                     // inner angle (full brightness)
    Math.PI / 4                      // outer angle (fades to zero)
);
```

### Ambient Light

The `ambientColor` property sets the global base illumination. Areas outside any light's reach will be tinted by this color:

```typescript
lighting.ambientColor = new Color4(0.08, 0.06, 0.12, 1); // Very dark purple
```

You can also create per-light ambient contributions via `LightType2D.Ambient`:

```typescript
import { Light2D, LightType2D } from "@babylonjs/2d";

const ambient = new Light2D(LightType2D.Ambient);
ambient.color = new Color4(0.1, 0.1, 0.2, 1);
ambient.intensity = 0.5;
lighting.addLight(ambient);
```

## How It Works

The lighting system uses **forward rendering** — light calculations happen in the same fragment shader that draws sprites:

1. **CPU side (once per frame):** `LightingManager2D.packLightUniforms(cameraMatrix)` packs all enabled lights into a flat array of vec4s and transforms positions/directions into view space.
2. **GPU side (per pixel):** The `computeLighting(worldPos)` function in the fragment shader loops over active lights, computing distance attenuation and cone angles, then multiplies the sprite color by the accumulated light.

Light data is packed as 4 `vec4`s per light (16 floats):

| Slot | Contents |
|------|----------|
| `[i*4+0]` | posX, posY, radius, type (0=point, 1=spot, 2=ambient) |
| `[i*4+1]` | colorR, colorG, colorB, intensity |
| `[i*4+2]` | dirX, dirY, innerAngle, outerAngle |
| `[i*4+3]` | falloff, 0, 0, 0 |

## Light Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `position` | `Vector2` | (0,0) | World position |
| `color` | `Color4` | white | Light color (RGB) |
| `radius` | `number` | 200 | Maximum reach in pixels |
| `intensity` | `number` | 1 | Brightness multiplier |
| `falloff` | `number` | 2 | Falloff exponent — `pow(1-t, falloff)` where t = dist/radius |
| `enabled` | `boolean` | true | Toggle on/off (disabled lights are skipped during packing) |

## Dynamic Lights

Animate lights for effects like flickering torches or a player-following glow:

```typescript
// Player-follow light
const playerLight = lighting.createPointLight(0, 0, new Color4(1, 0.95, 0.7, 1), 260);
playerLight.intensity = 1.8;

// In game loop
playerLight.position.x = player.position.x;
playerLight.position.y = player.position.y;
```

```typescript
// Flickering torch
torch.intensity = 1.2 + Math.sin(time * 8) * 0.3;
torch.position.x = baseX + Math.sin(time * 3) * 2;
```

## Removing and Disabling Lights

```typescript
// Temporarily disable (still in the list, but skipped during packing)
torch.enabled = false;

// Permanently remove
lighting.removeLight(torch);
```

## Limits

- **Maximum 16 forward lights** — additional lights beyond 16 are silently clamped. Only enabled lights count toward the limit.
- The lit shader is automatically compiled and cached on first use. The renderer seamlessly falls back to the unlit shader when `lightingManager` is null or has no active lights.

## Example: Side-Scroller Atmosphere

```typescript
const lighting = new LightingManager2D();
lighting.ambientColor = new Color4(0.08, 0.06, 0.12, 1);

// Warm glow following the player
const glow = lighting.createPointLight(0, 0, new Color4(1, 0.95, 0.7, 1), 260);
glow.intensity = 1.8;
glow.falloff = 1.2;

// Golden collectible lights
for (const item of collectibles) {
    const light = lighting.createPointLight(item.x, item.y, new Color4(1, 0.9, 0.3, 1), 100);
    light.intensity = 1.0;
}

scene.lightingManager = lighting;

// Each frame: move the glow with the player
engine.runRenderLoop(() => {
    glow.position.x = player.position.x;
    glow.position.y = player.position.y;
    scene.update(dt);
    scene.render();
});
```
