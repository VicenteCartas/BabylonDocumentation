---
title: Render Texture
description: Render a Scene2D to an offscreen texture for minimaps, trails, snapshots, and transition effects
keywords: 2d, render texture, offscreen, minimap, trail, screenshot, dynamic texture
---

# Render Texture

`RenderTexture2D` renders a `Scene2D` into an offscreen GPU texture. The resulting texture can be used as a sprite source in the 2D rendering pipeline — enabling minimaps, dash trails / afterimages, transition effects, blur snapshots, and dynamic textures without any extra shader work.

## Use Cases

| Use Case | Description |
|----------|-------------|
| **Minimap** | Render a zoomed-out camera to a small texture and display it as a HUD sprite |
| **Trail / Afterimage** | Accumulate previous frames at low alpha to create motion blur or dash trails |
| **Transition effects** | Snapshot the current scene before transitioning to the next |
| **Blur / Snapshot** | Capture a frame, then display it behind a pause menu |
| **Dynamic textures** | Procedurally render shapes, text, or effects into a texture at runtime |

## Basic Usage

Create a render texture, render a scene into it, then assign the result to a sprite:

```typescript
import { Scene2D, Sprite2D, Camera2D, RenderTexture2D } from "@babylonjs/2d";
import { Engine, Texture } from "@babylonjs/core";

const engine = new Engine(canvas);
const scene = new Scene2D(engine);

// Set up your scene content (sprites, camera, etc.)
const camera = new Camera2D(engine.getRenderWidth(), engine.getRenderHeight());
scene.camera = camera;

// Create a 256×256 offscreen render texture
const rt = new RenderTexture2D("snapshot", engine, 256, 256);

// Render the scene into the texture
rt.renderScene(scene);

// Use the result as a sprite texture
const display = new Sprite2D("display", scene);
display.texture = rt.texture;
display.width = 256;
display.height = 256;
display.position.x = 400;
display.position.y = 300;
```

<Alert severity="info">
`renderScene()` binds the offscreen framebuffer, calls `scene.renderContent()`, then restores the default framebuffer. The scene's camera and viewport are automatically adjusted to match the render texture dimensions.
</Alert>

## Minimap Example

A common pattern is to render a second camera — zoomed out — into a small render texture and display it as a HUD element.

```typescript
import { Scene2D, Sprite2D, Camera2D, RenderTexture2D } from "@babylonjs/2d";
import { Engine } from "@babylonjs/core";

const engine = new Engine(canvas);

// --- Main scene ---
const gameScene = new Scene2D(engine);
const mainCamera = new Camera2D(engine.getRenderWidth(), engine.getRenderHeight());
mainCamera.follow(playerSprite, { smoothing: 0.1 });
gameScene.camera = mainCamera;

// --- Minimap scene (shares the same sprites) ---
const minimapScene = new Scene2D(engine);
const minimapCamera = new Camera2D(engine.getRenderWidth(), engine.getRenderHeight());
minimapCamera.zoom = 0.25; // Zoom out 4×
minimapCamera.follow(playerSprite, { smoothing: 0 }); // Instant follow
minimapScene.camera = minimapCamera;

// Render texture for minimap (128×128 pixels)
const minimapRT = new RenderTexture2D("minimap", engine, 128, 128);

// HUD sprite to display the minimap in the corner
const minimapDisplay = new Sprite2D("minimapHUD", gameScene);
minimapDisplay.texture = minimapRT.texture;
minimapDisplay.width = 128;
minimapDisplay.height = 128;
minimapDisplay.position.x = engine.getRenderWidth() - 80;
minimapDisplay.position.y = 80;

// --- Render loop ---
let frameCount = 0;

engine.runRenderLoop(() => {
    const dt = engine.getDeltaTime() / 1000;

    // Update cameras
    mainCamera.update(dt);
    minimapCamera.update(dt);

    // Render minimap every 3 frames to save GPU time
    if (frameCount % 3 === 0) {
        minimapRT.renderScene(minimapScene);
    }
    frameCount++;

    // Render main scene to screen
    gameScene.update(dt);
    gameScene.render();
});
```

## Trail / Afterimage Effect

Render the scene into a render texture each frame, then display it behind the main scene at low alpha. Because the RT is not cleared between frames, previous content accumulates into a "trail."

```typescript
import { Scene2D, Sprite2D, Camera2D, RenderTexture2D } from "@babylonjs/2d";
import { Engine } from "@babylonjs/core";
import { Color4 } from "@babylonjs/core/Maths/math.color";

const engine = new Engine(canvas);
const gameScene = new Scene2D(engine);
const camera = new Camera2D(engine.getRenderWidth(), engine.getRenderHeight());
gameScene.camera = camera;

// Full-screen render texture for trail accumulation
const trailRT = new RenderTexture2D(
    "trail",
    engine,
    engine.getRenderWidth(),
    engine.getRenderHeight(),
);

// Sprite that displays the trail behind the main scene
const trailDisplay = new Sprite2D("trailDisplay", gameScene);
trailDisplay.texture = trailRT.texture;
trailDisplay.width = engine.getRenderWidth();
trailDisplay.height = engine.getRenderHeight();
trailDisplay.tint = new Color4(1, 1, 1, 0.85); // Slight fade each frame
trailDisplay.position.x = engine.getRenderWidth() / 2;
trailDisplay.position.y = engine.getRenderHeight() / 2;

engine.runRenderLoop(() => {
    const dt = engine.getDeltaTime() / 1000;
    gameScene.update(dt);

    // Render into RT *without* clearing — accumulates previous frames
    trailRT.renderScene(gameScene, false);

    // Render the main scene on top
    gameScene.render();
});
```

<Alert severity="warning">
Pass `clear = false` to `renderScene()` to accumulate frames. If you always clear, you get a standard snapshot instead of a trail.
</Alert>

## Resizing

Call `resize()` when the canvas size changes or you need a different resolution. This disposes the old GPU resources and creates new ones:

```typescript
window.addEventListener("resize", () => {
    engine.resize();

    // Match the render texture to the new canvas size
    trailRT.resize(engine.getRenderWidth(), engine.getRenderHeight());
});
```

If the requested size matches the current size, `resize()` is a no-op.

## Reading Pixels

`readPixelsAsync()` reads RGBA pixel data from the render texture. Use this for screenshots, hit-testing, or pixel-based game logic:

```typescript
async function takeScreenshot(rt: RenderTexture2D): Promise<void> {
    const pixels = await rt.readPixelsAsync();
    // pixels is an ArrayBufferView (Uint8Array) of RGBA data
    // pixels.length === rt.width * rt.height * 4

    // Example: create an ImageData for use with a 2D canvas
    const imageData = new ImageData(
        new Uint8ClampedArray(pixels.buffer),
        rt.width,
        rt.height,
    );

    const offscreen = document.createElement("canvas");
    offscreen.width = rt.width;
    offscreen.height = rt.height;
    offscreen.getContext("2d")!.putImageData(imageData, 0, 0);

    // Download as PNG
    const link = document.createElement("a");
    link.download = "screenshot.png";
    link.href = offscreen.toDataURL("image/png");
    link.click();
}
```

<Alert severity="info">
`readPixelsAsync()` is an async GPU readback. Avoid calling it every frame — it stalls the pipeline. Use it for one-off screenshots or infrequent logic checks.
</Alert>

## Cleanup

Always dispose render textures when you're done to free GPU memory:

```typescript
minimapRT.dispose();
trailRT.dispose();

// After dispose, isDisposed returns true and the RT cannot be reused
console.log(minimapRT.isDisposed); // true
```

## Tips

- **Don't render every frame if you don't need to.** A minimap updated every 3–5 frames looks identical to the player but costs a fraction of the GPU time.
- **Use small RT sizes.** A 128×128 minimap is 16× fewer pixels than 512×512. Match the RT size to its on-screen display size.
- **`clear = false` enables accumulation** for trails and afterimages. Use `clear = true` (the default) for snapshots and minimaps.
- **Wrap modes are CLAMP by default.** The `texture` getter sets `wrapU` / `wrapV` to `TEXTURE_CLAMP_ADDRESSMODE` to prevent bleeding at edges.
- **Coordinate system is Y-down** — same as the rest of the 2D engine. Content rendered into the RT follows the same [Camera2D](/features/featuresDeepDive/2d/camera2d) conventions.
- **Combine with [Scene Transitions](/features/featuresDeepDive/2d/scene-transitions)** — snapshot the outgoing scene to an RT, then animate it as a sprite during the transition.

## API Reference

### RenderTexture2D

#### Constructor

```typescript
new RenderTexture2D(
    name: string,
    engine: AbstractEngine,
    width: number,
    height: number,
    options?: IRenderTexture2DOptions,
)
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `name` | `string` | Display name for debugging |
| `engine` | `AbstractEngine` | The Babylon.js engine instance |
| `width` | `number` | Width of the render texture in pixels |
| `height` | `number` | Height of the render texture in pixels |
| `options` | `IRenderTexture2DOptions` | Optional creation options (see below) |

#### Properties

| Property | Type | Description |
|----------|------|-------------|
| `name` | `string` | Read-only display name |
| `width` | `number` | Width in pixels |
| `height` | `number` | Height in pixels |
| `texture` | `ThinTexture` | A `ThinTexture` wrapping the render target — use as a sprite texture. Created lazily on first access; wrap modes set to CLAMP. |
| `renderTarget` | `RenderTargetWrapper` | The underlying render target for low-level access |
| `isDisposed` | `boolean` | `true` after `dispose()` has been called |

#### Methods

| Method | Returns | Description |
|--------|---------|-------------|
| `renderScene(scene, clear?)` | `void` | Renders a `Scene2D` into this texture. `clear` defaults to `true`. |
| `resize(width, height)` | `void` | Resizes to new dimensions. Disposes old GPU resources. No-op if size unchanged. |
| `readPixelsAsync()` | `Promise<ArrayBufferView>` | Reads RGBA pixel data asynchronously from the GPU. |
| `dispose()` | `void` | Releases all GPU resources. The RT cannot be used after this call. |

### IRenderTexture2DOptions

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `generateMipMaps` | `boolean` | `false` | Generate mip-map levels for the texture |
| `samplingMode` | `number` | `TEXTURE_BILINEAR_SAMPLINGMODE` | Texture filtering mode |
| `format` | `number` | `TEXTUREFORMAT_RGBA` | Pixel format |
| `type` | `number` | `TEXTURETYPE_UNSIGNED_BYTE` | Data type per channel |
| `generateDepthBuffer` | `boolean` | `false` | Allocate a depth buffer |
| `generateStencilBuffer` | `boolean` | `false` | Allocate a stencil buffer |

For most 2D use cases the defaults are sufficient. Override `samplingMode` with `TEXTURE_NEAREST_SAMPLINGMODE` for crisp pixel-art rendering at non-native sizes.
