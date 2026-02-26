---
title: Loading Sprite Atlases (JSON & XML)
description: Load pre-packed sprite atlases from TexturePacker, ShoeBox, Aseprite, and other tools using JSON and XML formats in Babylon.js 2D
keywords: 2d, sprite atlas, texture packer, json, xml, starling, sparrow, spritesheet, atlas loading
---

# Loading Sprite Atlases (JSON & XML)

`SpriteAtlas` can load pre-packed sprite atlases exported from industry-standard tools like **TexturePacker**, **ShoeBox**, **Aseprite**, **Leshy SpriteSheet Tool**, and others. Instead of packing images at runtime with `SpriteAtlasBuilder`, you supply a descriptor file (JSON or XML) and its companion texture image.

This is the recommended workflow for production games — pack your sprites offline with a dedicated tool, export the atlas, and load it directly at runtime.

## Supported Formats

| Format | File Extension | Description |
|--------|---------------|-------------|
| **JSON Hash** | `.json` | Each frame keyed by name. Default TexturePacker format. |
| **JSON Array** | `.json` | Frames stored as an array with `filename` properties. |
| **Starling / Sparrow XML** | `.xml` | XML with `<SubTexture>` elements. Used by Starling, Sparrow, and many other frameworks. |

All three formats describe the same data — a list of named rectangular regions within a single texture image.

## Loading from URLs with LoadJsonAsync

The most common approach is to load both the descriptor and texture from URLs. `SpriteAtlas.LoadJsonAsync` fetches the JSON file, loads the texture image, and returns a ready-to-use `SpriteAtlas`:

```typescript
import { SpriteAtlas, Sprite2D, Scene2D } from "@babylonjs/2d";
import { Engine } from "@babylonjs/core";

const engine = new Engine(canvas);
const scene = new Scene2D(engine);

// Load a TexturePacker JSON atlas
const atlas = await SpriteAtlas.LoadJsonAsync(
    "assets/characters.json",   // JSON descriptor URL
    "assets/characters.png",    // Texture image URL
    engine
);

// Use a frame from the atlas
const player = new Sprite2D("player", scene);
player.texture = atlas.texture;
player.sourceRect = atlas.getFrame("player_idle_0");
player.width = 64;
player.height = 64;
player.position.set(400, 300);
```

Both the JSON file and texture image are fetched in parallel for fastest loading.

### JSON Hash Format

In JSON Hash format, frames are keyed by name:

```json
{
    "frames": {
        "player_idle_0": {
            "frame": { "x": 0, "y": 0, "w": 32, "h": 48 }
        },
        "player_idle_1": {
            "frame": { "x": 32, "y": 0, "w": 32, "h": 48 }
        },
        "enemy_walk_0": {
            "frame": { "x": 64, "y": 0, "w": 24, "h": 32 }
        }
    }
}
```

### JSON Array Format

In JSON Array format, frames are stored in an array with `filename` properties:

```json
{
    "frames": [
        {
            "filename": "player_idle_0",
            "frame": { "x": 0, "y": 0, "w": 32, "h": 48 }
        },
        {
            "filename": "player_idle_1",
            "frame": { "x": 32, "y": 0, "w": 32, "h": 48 }
        }
    ]
}
```

`LoadJsonAsync` and `FromJson` detect the format automatically — no configuration needed.

## Loading XML with LoadXmlAsync

For Starling/Sparrow XML atlases, use `SpriteAtlas.LoadXmlAsync`:

```typescript
const atlas = await SpriteAtlas.LoadXmlAsync(
    "assets/characters.xml",    // XML descriptor URL
    "assets/characters.png",    // Texture image URL
    engine
);

const enemy = new Sprite2D("enemy", scene);
enemy.texture = atlas.texture;
enemy.sourceRect = atlas.getFrame("enemy_walk_0");
enemy.width = 24;
enemy.height = 32;
```

### Starling / Sparrow XML Format

```xml
<TextureAtlas imagePath="characters.png">
    <SubTexture name="player_idle_0" x="0" y="0" width="32" height="48" />
    <SubTexture name="player_idle_1" x="32" y="0" width="32" height="48" />
    <SubTexture name="enemy_walk_0" x="64" y="0" width="24" height="32" />
</TextureAtlas>
```

Each `<SubTexture>` element defines a named rectangular region in the atlas texture.

## Using FromJson and FromXml (Pre-Loaded Data)

If the atlas data is already loaded — for example, bundled with your code, fetched through a custom loader, or embedded as a constant — use the synchronous `FromJson` or `FromXml` methods:

### FromJson

```typescript
import { SpriteAtlas } from "@babylonjs/2d";
import { Texture } from "@babylonjs/core";

// Atlas data already available (e.g., imported from a JSON module)
const atlasData = {
    frames: {
        "coin_0": { frame: { x: 0, y: 0, w: 16, h: 16 } },
        "coin_1": { frame: { x: 16, y: 0, w: 16, h: 16 } },
        "coin_2": { frame: { x: 32, y: 0, w: 16, h: 16 } },
        "coin_3": { frame: { x: 48, y: 0, w: 16, h: 16 } },
    }
};

// Texture already loaded
const texture = new Texture("assets/coins.png", engine);

const atlas = SpriteAtlas.FromJson(atlasData, texture);
```

### FromXml

```typescript
import { SpriteAtlas } from "@babylonjs/2d";

// Parse XML from a string (e.g., loaded via XMLHttpRequest or bundled)
const xmlString = `<TextureAtlas imagePath="items.png">
    <SubTexture name="sword" x="0" y="0" width="32" height="32" />
    <SubTexture name="shield" x="32" y="0" width="32" height="32" />
</TextureAtlas>`;

const xmlDoc = new DOMParser().parseFromString(xmlString, "text/xml");
const atlas = SpriteAtlas.FromXml(xmlDoc, myTexture);
```

## Using with AnimatedSprite2D

Every `SpriteAtlas` exposes a `spriteSheet` property that is compatible with `AnimatedSprite2D`. Use it to play frame-based animations from your atlas:

```typescript
import { SpriteAtlas, AnimatedSprite2D } from "@babylonjs/2d";

const atlas = await SpriteAtlas.LoadJsonAsync(
    "assets/hero.json",
    "assets/hero.png",
    engine
);

// The atlas provides a SpriteSheet with all frames indexed
const hero = new AnimatedSprite2D("hero", atlas.spriteSheet, scene);

// Define animations using frame name arrays
hero.addAnimation("idle", ["hero_idle_0", "hero_idle_1", "hero_idle_2", "hero_idle_3"], 6);
hero.addAnimation("run", ["hero_run_0", "hero_run_1", "hero_run_2", "hero_run_3", "hero_run_4", "hero_run_5"], 12);
hero.addAnimation("attack", ["hero_atk_0", "hero_atk_1", "hero_atk_2"], 10);

hero.playAnimation("idle");
```

<Alert severity="info">
**Tip:** Name your frames with a consistent pattern in your atlas tool (e.g., `hero_idle_0`, `hero_idle_1`, ...). This makes it easy to define animation sequences in code.
</Alert>

## SpriteAtlas API Reference

### Static Methods

| Method | Returns | Description |
|--------|---------|-------------|
| `SpriteAtlas.LoadJsonAsync(jsonUrl, textureUrl, engine)` | `Promise<SpriteAtlas>` | Fetches a JSON descriptor and texture image from URLs. Supports both JSON Hash and JSON Array formats. |
| `SpriteAtlas.LoadXmlAsync(xmlUrl, textureUrl, engine)` | `Promise<SpriteAtlas>` | Fetches a Starling/Sparrow XML descriptor and texture image from URLs. |
| `SpriteAtlas.FromJson(atlasData, texture)` | `SpriteAtlas` | Creates an atlas from already-parsed JSON data and an existing texture. |
| `SpriteAtlas.FromXml(xmlDoc, texture)` | `SpriteAtlas` | Creates an atlas from an already-parsed XML `Document` and an existing texture. |

### Instance Properties and Methods

| Property / Method | Type | Description |
|---|---|---|
| `texture` | `BaseTexture` | The atlas texture — assign to `Sprite2D.texture` |
| `spriteSheet` | `SpriteSheet` | A `SpriteSheet` built from the atlas data, compatible with `AnimatedSprite2D` |
| `getFrame(key)` | `Rectangle2D \| undefined` | Returns the frame rectangle for a named sprite |
| `getFrameKeys()` | `string[]` | Returns all frame names in the atlas |
| `hasFrame(key)` | `boolean` | Checks if a frame name exists in the atlas |

## TexturePacker Export Settings

[TexturePacker](https://www.codeandweb.com/texturepacker) is the most popular sprite atlas tool. Use these settings for best compatibility with Babylon.js 2D:

| Setting | Recommended Value | Notes |
|---------|-------------------|-------|
| **Data Format** | JSON (Hash) | JSON Hash is the simplest to work with. JSON Array also works. |
| **Framework** | Generic / Custom | Any framework that outputs standard `{ frame: { x, y, w, h } }` fields. |
| **Trim mode** | None | Trim is **not yet supported** — trimmed frames will render at incorrect sizes. |
| **Rotation** | Disabled | Rotation is **not yet supported** — rotated frames will render incorrectly. |
| **Max size** | 2048 × 2048 | Safe for all platforms. Use 4096 only if targeting desktop. |
| **Padding** | 1–2 px | Prevents texture bleeding between adjacent frames. |
| **Power of 2** | Yes | Better GPU compatibility, especially on mobile. |

<Alert severity="warning">
**Important:** Disable **Trim** and **Rotation** in your atlas tool. The loader reads only `frame` rectangles (`x`, `y`, `w`, `h`). Trimmed sprite offsets and rotated frame transforms are not applied, so sprites will appear misaligned or distorted.
</Alert>

For **Starling/Sparrow XML** export, select the Starling or Sparrow data format in TexturePacker with the same settings above.

## Complete Example

```typescript
import { SpriteAtlas, Sprite2D, AnimatedSprite2D, Scene2D } from "@babylonjs/2d";
import { Engine } from "@babylonjs/core";

const engine = new Engine(canvas);
const scene = new Scene2D(engine);

// Load the atlas exported from TexturePacker
const atlas = await SpriteAtlas.LoadJsonAsync(
    "assets/game-sprites.json",
    "assets/game-sprites.png",
    engine
);

// --- Static sprites ---
const background = new Sprite2D("bg", scene);
background.texture = atlas.texture;
background.sourceRect = atlas.getFrame("sky_background");
background.width = 800;
background.height = 600;

const tree = new Sprite2D("tree", scene);
tree.texture = atlas.texture;
tree.sourceRect = atlas.getFrame("tree_oak");
tree.position.set(600, 400);

// --- Animated sprite using the atlas SpriteSheet ---
const player = new AnimatedSprite2D("player", atlas.spriteSheet, scene);
player.addAnimation("idle", [
    "player_idle_0", "player_idle_1",
    "player_idle_2", "player_idle_3"
], 8);
player.addAnimation("run", [
    "player_run_0", "player_run_1", "player_run_2",
    "player_run_3", "player_run_4", "player_run_5"
], 12);
player.playAnimation("idle");
player.position.set(400, 450);

// --- Verify atlas contents ---
console.log("Atlas frames:", atlas.getFrameKeys());
console.log("Has player_idle_0?", atlas.hasFrame("player_idle_0"));

// --- Render loop ---
engine.runRenderLoop(() => {
    const dt = engine.getDeltaTime() / 1000;
    scene.update(dt);
    scene.render();
});
```

## Performance Tips

- **One atlas per scene or level.** Pack all sprites for a given level into a single atlas. Sprites sharing one texture are batched into a single draw call by the `SpriteBatchRenderer`.
- **Pack related sprites together.** Group characters, tiles, and UI elements that appear on screen at the same time. Avoid mixing assets from different levels into one atlas.
- **Use the async loaders during loading screens.** `LoadJsonAsync` and `LoadXmlAsync` fetch and decode everything before returning — call them during an initial loading phase, not during gameplay.
- **Prefer JSON Hash.** It is the most widely supported format across tools and is slightly faster to parse than Array or XML.
- **Check frame existence.** Use `hasFrame()` before `getFrame()` in production code to avoid silent `undefined` values that can cause invisible sprites.

<Alert severity="info">
**When to use LoadJsonAsync vs. SpriteAtlasBuilder:** Use `LoadJsonAsync` / `LoadXmlAsync` when you have pre-packed atlases from an external tool — this is the standard production workflow. Use [SpriteAtlasBuilder](/features/featuresDeepDive/2d/sprite-atlas-builder) when you need to pack images dynamically at runtime (e.g., user-generated content or procedural sprites).
</Alert>

## Cross-References

- **[Sprites & Animation](/features/featuresDeepDive/2d/sprites-and-animation)** — Sprite2D, SpriteSheet, and AnimatedSprite2D fundamentals
- **[Sprite Atlas Builder](/features/featuresDeepDive/2d/sprite-atlas-builder)** — Runtime atlas packing with SpriteAtlasBuilder
- **[Object Pooling](/features/featuresDeepDive/2d/object-pooling)** — Pool sprites that share an atlas texture for maximum performance
- **[Tilemaps](/features/featuresDeepDive/2d/tilemaps)** — Tilemap rendering also benefits from atlas textures
