---
title: 2D Lighting
description: Dynamic 2D lighting with point lights, spotlights, and ambient lighting
keywords: 2d, lighting, point light, spotlight, ambient, dynamic
---

# 2D Lighting

The `Light2D` and `LightingManager2D` classes provide dynamic 2D lighting. Support for point lights, spotlights, and ambient illumination with configurable falloff and cone angles.

## Setup

```typescript
import { LightingManager2D, LightType2D } from "@babylonjs/2d";

const lighting = new LightingManager2D();
lighting.ambientColor = new Color4(0.15, 0.15, 0.2, 1); // Dark ambient
```

## Light Types

### Point Light

Radiates light in all directions from a position:

```typescript
const torch = lighting.createPointLight(200, 300, new Color4(1, 0.8, 0.4, 1), 250);
torch.intensity = 1.5;
torch.falloff = 2; // Quadratic falloff (default)
```

### Spotlight

A cone-shaped light with inner (full) and outer (fade) angles:

```typescript
const spot = lighting.createSpotLight(
    400, 100,                        // position
    new Vector2(0, 1),               // direction (pointing down)
    new Color4(1, 1, 1, 1),         // white
    300,                             // radius
    Math.PI / 8,                     // inner angle
    Math.PI / 4                      // outer angle
);
```

### Ambient Light

Use `LightingManager2D.ambientColor` for base illumination, or create a `LightType2D.Ambient` light for layer-filtered ambient:

```typescript
const ambient = new Light2D(LightType2D.Ambient);
ambient.color = new Color4(0.1, 0.1, 0.2, 1);
ambient.intensity = 1;
lighting.addLight(ambient);
```

## Sampling Light

Compute the combined illumination at any world position:

```typescript
const color = lighting.computeLight(entityX, entityY);
// Apply to sprite tint
sprite.tint = color;
```

## Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `position` | `Vector2` | (0,0) | World position |
| `color` | `Color4` | white | Light color |
| `radius` | `number` | 200 | Maximum reach in pixels |
| `intensity` | `number` | 1 | Brightness multiplier |
| `falloff` | `number` | 2 | Falloff exponent (higher = sharper) |
| `enabled` | `boolean` | true | Toggle on/off |
| `layer` | `number` | 0xFFFFFFFF | Layer bitmask filter |

## Dynamic Lights

Animate lights for effects like flickering torches:

```typescript
// In game loop
torch.intensity = 1.2 + Math.sin(time * 8) * 0.3;
torch.position.x = baseX + Math.sin(time * 3) * 2;
```

## Layer Filtering

Control which sprites are affected by which lights:

```typescript
const playerLight = lighting.createPointLight(px, py, new Color4(1, 1, 0.8, 1), 150);
playerLight.layer = 0x01; // Only affects layer 1

const color = lighting.computeLight(x, y, 0x01); // Only sample layer 1 lights
```
