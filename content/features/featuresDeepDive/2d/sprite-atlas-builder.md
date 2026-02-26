---
title: Sprite Atlas Builder
description: Auto-pack multiple images into a single texture atlas to reduce draw calls
keywords: 2d, sprite atlas, texture packing, batching, performance, draw calls, spritesheet
---

# Sprite Atlas Builder

`SpriteAtlasBuilder` packs multiple images into a single texture atlas at load time. By reducing the number of individual textures, the `SpriteBatchRenderer` can batch more sprites into fewer draw calls — a significant performance win for games with many sprite types.

## Why Use Atlases?

The `SpriteBatchRenderer` batches sprites that share the same texture into a single draw call (up to 8 textures per batch). If your game has 50 individual sprite textures, the renderer must issue many separate draw calls with frequent GPU texture switches.

Packing those 50 images into one atlas means **one texture bind, one draw call** for all 50 sprite types.

## Basic Usage

```typescript
import { SpriteAtlasBuilder } from "@babylonjs/2d";

// Create a builder
const builder = new SpriteAtlasBuilder(engine, {
    maxWidth: 2048,
    maxHeight: 2048,
    padding: 2,
    powerOfTwo: true
});

// Add images from URLs
builder.addImage("player", "assets/player.png");
builder.addImage("enemy", "assets/enemy.png");
builder.addImage("bullet", "assets/bullet.png");

// Or add existing HTMLImageElements
builder.addImage("coin", coinImageElement);

// Build the atlas (async — loads images and packs them)
const atlas = await builder.buildAsync();

// Use with Sprite2D
const playerSprite = new Sprite2D("player", scene);
playerSprite.texture = atlas.texture;
playerSprite.sourceRect = atlas.getFrame("player");
```

## Configuration Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `maxWidth` | `number` | 2048 | Maximum atlas width in pixels |
| `maxHeight` | `number` | 2048 | Maximum atlas height in pixels |
| `padding` | `number` | 1 | Pixels between packed sprites (prevents texture bleeding) |
| `powerOfTwo` | `boolean` | true | Constrain atlas to power-of-two dimensions (better GPU compatibility) |

```typescript
// Compact atlas for a small game
const builder = new SpriteAtlasBuilder(engine, {
    maxWidth: 512,
    maxHeight: 512,
    padding: 1,
    powerOfTwo: true
});

// Large atlas for a content-rich game
const builder = new SpriteAtlasBuilder(engine, {
    maxWidth: 4096,
    maxHeight: 4096,
    padding: 2,
    powerOfTwo: false  // Use exact size to save memory
});
```

## Image Sources

`addImage()` accepts three source types:

```typescript
// 1. URL string — loaded asynchronously during buildAsync()
builder.addImage("player", "assets/player.png");

// 2. HTMLImageElement — used directly (must already be loaded)
const img = new Image();
img.src = "assets/enemy.png";
await img.decode();
builder.addImage("enemy", img);

// 3. Babylon.js Texture — extracted via its source URL
import { Texture } from "@babylonjs/core";
const tex = new Texture("assets/coin.png", engine);
builder.addImage("coin", tex);
```

Each image gets a unique string key used to retrieve its frame from the built atlas.

## SpriteAtlas Result

`buildAsync()` returns a `SpriteAtlas` with these APIs:

| Property / Method | Type | Description |
|---|---|---|
| `texture` | `BaseTexture` | The packed atlas texture — assign to `Sprite2D.texture` |
| `spriteSheet` | `SpriteSheet` | Compatible with `AnimatedSprite2D` and `SpriteSheet.FromAtlas` |
| `getFrame(key)` | `Rectangle2D \| undefined` | Frame rectangle for a specific image |
| `getFrameKeys()` | `string[]` | All image keys in the atlas |
| `hasFrame(key)` | `boolean` | Check if a key exists |

```typescript
const atlas = await builder.buildAsync();

// Check what's in the atlas
console.log(atlas.getFrameKeys()); // ["player", "enemy", "bullet", "coin"]

// Get a specific frame
const frame = atlas.getFrame("player");
if (frame) {
    console.log(`Player: ${frame.x}, ${frame.y}, ${frame.width}x${frame.height}`);
}
```

## Using with SpriteSheet and Animations

The atlas produces a `SpriteSheet` compatible with `AnimatedSprite2D`:

```typescript
// Build an atlas containing animation frame images
const builder = new SpriteAtlasBuilder(engine);
builder.addImage("walk_0", "assets/walk_0.png");
builder.addImage("walk_1", "assets/walk_1.png");
builder.addImage("walk_2", "assets/walk_2.png");
builder.addImage("walk_3", "assets/walk_3.png");

const atlas = await builder.buildAsync();

// Use the sprite sheet with AnimatedSprite2D
const character = new AnimatedSprite2D("hero", atlas.spriteSheet, scene);
character.addAnimation("walk", ["walk_0", "walk_1", "walk_2", "walk_3"], 10);
character.playAnimation("walk");
```

## Packing Algorithm

`SpriteAtlasBuilder` uses a **shelf-first-fit** algorithm:

1. Images are sorted by height (tallest first)
2. Each image is placed on the first shelf (horizontal row) that fits
3. If no shelf fits, a new shelf is created below the last one
4. The final atlas is sized to tightly fit all shelves

This simple algorithm works well for sprites of similar sizes. For highly varied sizes (e.g., mixing 16×16 icons with 512×512 backgrounds), consider splitting into separate atlases.

## Complete Example

```typescript
import { SpriteAtlasBuilder, Sprite2D, Scene2D } from "@babylonjs/2d";
import { Engine } from "@babylonjs/core";

const engine = new Engine(canvas);
const scene = new Scene2D(engine);

// Build atlas during loading
const builder = new SpriteAtlasBuilder(engine, {
    maxWidth: 1024,
    maxHeight: 1024,
    padding: 2,
    powerOfTwo: true
});

// Add all game sprites
builder.addImage("terrain_grass", "assets/tiles/grass.png");
builder.addImage("terrain_water", "assets/tiles/water.png");
builder.addImage("terrain_stone", "assets/tiles/stone.png");
builder.addImage("player_idle", "assets/player/idle.png");
builder.addImage("enemy_slime", "assets/enemies/slime.png");
builder.addImage("coin", "assets/items/coin.png");
builder.addImage("heart", "assets/ui/heart.png");

const atlas = await builder.buildAsync();

// All sprites share one texture — maximum batching
const player = new Sprite2D("player", scene);
player.texture = atlas.texture;
player.sourceRect = atlas.getFrame("player_idle");
player.position.set(400, 300);

const coin = new Sprite2D("coin", scene);
coin.texture = atlas.texture;
coin.sourceRect = atlas.getFrame("coin");
coin.position.set(500, 250);

// Create many terrain tiles — all batched in one draw call
for (let x = 0; x < 20; x++) {
    for (let y = 0; y < 15; y++) {
        const tile = new Sprite2D(`tile_${x}_${y}`, scene);
        tile.texture = atlas.texture;
        tile.sourceRect = atlas.getFrame("terrain_grass");
        tile.position.set(x * 32, y * 32);
    }
}

// Render loop
engine.runRenderLoop(() => {
    scene.render();
});
```

## Best Practices

### Atlas Sizing

- **Start with 1024×1024** — enough for most 2D games
- **Use 2048×2048** for content-rich games with many sprite types
- **Avoid 4096+** unless necessary — some mobile GPUs don't support large textures
- **Enable `powerOfTwo: true`** for maximum GPU compatibility

### When to Use Multiple Atlases

Group related sprites into separate atlases:

```typescript
// Atlas 1: UI elements (always visible)
const uiBuilder = new SpriteAtlasBuilder(engine);
uiBuilder.addImage("button", "assets/ui/button.png");
uiBuilder.addImage("health_bar", "assets/ui/health.png");
const uiAtlas = await uiBuilder.buildAsync();

// Atlas 2: Level 1 sprites (loaded per level)
const levelBuilder = new SpriteAtlasBuilder(engine);
levelBuilder.addImage("tree", "assets/level1/tree.png");
levelBuilder.addImage("rock", "assets/level1/rock.png");
const levelAtlas = await levelBuilder.buildAsync();
```

### Padding

- **Use `padding: 1`** minimum to prevent texture bleeding at edges
- **Use `padding: 2`** if you see artifacts at sprite boundaries
- **Use `padding: 0`** only for pixel-art at integer positions

### Performance Tips

- Build atlases during loading screens, not during gameplay
- Fewer, larger atlases = fewer draw calls = better performance
- Use `atlas.getFrameKeys()` to verify all sprites were packed
- Log the atlas dimensions to check packing efficiency

<Alert severity="info">
**Tip:** Combine atlas building with object pooling for maximum performance. Pool objects that share the same atlas texture to eliminate both GC spikes and texture switching overhead.
</Alert>

<Alert severity="warning">
**Warning:** When passing a Babylon.js `Texture` as a source, the texture must have a resolvable URL. GPU-only textures (e.g., render targets) cannot be extracted — use a URL string or HTMLImageElement instead.
</Alert>

## Cross-References

- **[Loading Sprite Atlases (JSON & XML)](/features/featuresDeepDive/2d/sprite-atlas-loading)** — Load pre-packed atlases from TexturePacker, ShoeBox, and other tools
- **[Sprites & Animation](/features/featuresDeepDive/2d/sprites-and-animation)** — Using Sprite2D and AnimatedSprite2D with atlas textures
- **[Object Pooling](/features/featuresDeepDive/2d/object-pooling)** — Combine with pooling for maximum performance
- **[Tilemaps](/features/featuresDeepDive/2d/tilemaps)** — Tilemap rendering also benefits from atlas textures
