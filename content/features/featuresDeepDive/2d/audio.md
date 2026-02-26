---
title: Audio in 2D Games
description: Using Babylon.js AudioV2 for sound effects and music in 2D games
keywords: 2d, audio, sound, music, spatial audio, streaming
---

# Audio in 2D Games

Babylon.js 2D games use the core **AudioV2** system for sound effects and music. AudioV2 is fully standalone — it has no dependency on 3D scenes, making it a natural fit for 2D projects.

## Quick Start

```typescript
import { CreateAudioEngineAsync, CreateSoundAsync, CreateStreamingSoundAsync } from "@babylonjs/core/AudioV2";

// Create the audio engine
const audioEngine = await CreateAudioEngineAsync({
    volume: 0.8,
});

// Load a sound effect (fully buffered — ideal for short clips)
const hitSound = await CreateSoundAsync("hit", "assets/hit.mp3", {
    volume: 1.0,
});

// Load background music (streamed — ideal for long tracks)
const bgMusic = await CreateStreamingSoundAsync("music", "assets/music.ogg", {
    volume: 0.5,
    loop: true,
});

// Play
bgMusic.play();
hitSound.play();
```

## Static vs. Streaming Sounds

| | `CreateSoundAsync` (Static) | `CreateStreamingSoundAsync` (Streaming) |
|---|---|---|
| **Best for** | Short sound effects, UI clicks | Background music, ambient loops |
| **Loading** | Entire file loaded into memory | Streamed from source on demand |
| **Pitch control** | ✅ `pitch`, `playbackRate` | ❌ Not supported |
| **Loop points** | ✅ `loopStart`, `loopEnd` | ❌ Full track only |
| **Startup latency** | Instant (pre-buffered) | May need `preloadInstanceAsync()` |

## Controlling Playback

```typescript
// Volume (0 to 1)
hitSound.volume = 0.7;

// Smooth volume ramp
bgMusic.setVolume(0.2, { duration: 1.5 });

// Looping
hitSound.loop = true;

// Pause / Resume
bgMusic.pause();
bgMusic.resume();

// Stop
bgMusic.stop();

// Playback speed (StaticSound only)
hitSound.playbackRate = 1.5; // 1.5× speed
hitSound.pitch = 200;        // Pitch shift in cents
```

## Spatial Audio (2D Positioning)

AudioV2 supports 3D spatial audio. For 2D games, set the Z coordinate to 0:

```typescript
import { Vector3 } from "@babylonjs/core/Maths/math.vector";

const explosionSound = await CreateSoundAsync("explosion", "assets/boom.mp3", {
    volume: 1.0,
    spatialEnabled: true,
});

// Position the sound in 2D space (z = 0)
explosionSound.spatial.position = new Vector3(worldX, worldY, 0);

// Configure distance attenuation
explosionSound.spatial.distanceModel = "linear";
explosionSound.spatial.minDistance = 50;
explosionSound.spatial.maxDistance = 500;

// Update position each frame (e.g., following a sprite)
function gameLoop() {
    explosionSound.spatial.position = new Vector3(
        sprite.position.x,
        sprite.position.y,
        0
    );
}
```

### Listener Position

By default, the listener is at the origin. For a 2D game, update it to match the camera:

```typescript
audioEngine.listener.position = new Vector3(
    camera.position.x,
    camera.position.y,
    0
);
```

### SpatialAudio2D Utility

The `@babylonjs/2d` package includes a `SpatialAudio2D` helper that automates the coordinate mapping and listener sync described above. It maps 2D world coordinates `(x, y)` to 3D `(x, y, 0)` internally, and keeps the Web Audio listener in sync with a [Camera2D](/features/featuresDeepDive/2d/camera2d).

<Alert severity="info">
`SpatialAudio2D` does **not** create or manage sounds — you still create sounds with AudioV2 directly. It only handles positioning and listener updates.
</Alert>

```typescript
import { SpatialAudio2D } from "@babylonjs/2d/Audio/spatialAudio2D";

const spatial = new SpatialAudio2D(engine);

// Position a sound at 2D world coordinates
spatial.setSoundPosition(explosionSound, 400, 300);

// Attach a sound so it auto-tracks a Node2D's world position
spatial.attachToNode(engineSound, spaceship);

// Each frame: sync the listener to the camera and update all tracked sounds
engine.runRenderLoop(() => {
    const dt = engine.getDeltaTime() / 1000;
    spatial.update(camera);   // camera is a Camera2D instance

    scene.update(dt);
    scene.render();
});
```

To stop tracking a sound or clean up:

```typescript
// Stop tracking a single sound
spatial.detachFromNode(engineSound);

// Remove all attachments and release references
spatial.dispose();
```

#### SpatialAudio2D API

| Method / Property | Type | Description |
|---|---|---|
| `constructor(engine)` | `AbstractEngine` | Creates the utility, linked to the engine's audio context |
| `engine` | `AbstractEngine` (readonly) | The engine associated with this instance |
| `attachmentCount` | `number` (readonly) | Number of currently tracked sound-to-node attachments |
| `update(camera)` | `Camera2D` → `void` | Syncs the Web Audio listener to the camera position and updates all attached sound positions. Call once per frame |
| `setSoundPosition(sound, x, y)` | `Sound, number, number` → `void` | Sets a spatial sound's 3D position from 2D coordinates (maps to `(x, y, 0)`) |
| `attachToNode(sound, node)` | `Sound, Node2D` → `void` | Registers a sound to automatically follow a Node2D's world position each `update()`. Replaces any previous attachment for that sound |
| `detachFromNode(sound)` | `Sound` → `void` | Stops automatic position tracking for the given sound |
| `dispose()` | `void` | Clears all attachments and releases references |

## Audio Buses (Mixing)

Use buses to group sounds by category and control their volume independently:

```typescript
const musicBus = await audioEngine.createBusAsync("music");
const sfxBus = await audioEngine.createBusAsync("sfx");

// Route sounds through buses
const bgMusic = await CreateStreamingSoundAsync("music", "assets/music.ogg", {
    loop: true,
    outBus: musicBus,
});

const hitSound = await CreateSoundAsync("hit", "assets/hit.mp3", {
    outBus: sfxBus,
});

// Volume sliders for options menu
musicBus.volume = 0.3;  // Lower music
sfxBus.volume = 1.0;    // Full SFX volume
```

## Integration with Scene2D

A typical setup in a 2D game:

```typescript
import { Engine } from "@babylonjs/core/Engines/engine";
import { Scene2D } from "@babylonjs/2d/Scene2D/scene2D";
import { CreateAudioEngineAsync, CreateSoundAsync, CreateStreamingSoundAsync } from "@babylonjs/core/AudioV2";

// Standard 2D setup
const engine = new Engine(canvas, true);
const scene = new Scene2D(engine);

// Audio is independent — no Scene dependency
const audioEngine = await CreateAudioEngineAsync();

const jumpSound = await CreateSoundAsync("jump", "assets/jump.wav");
const bgm = await CreateStreamingSoundAsync("bgm", "assets/level1.ogg", {
    loop: true,
    volume: 0.4,
});

bgm.play();

// In the game loop
engine.runRenderLoop(() => {
    const dt = engine.getDeltaTime() / 1000;

    if (input.isActionPressed("jump")) {
        jumpSound.play();
    }

    scene.update(dt);
    scene.render();
});
```

## Tips

- Call `CreateAudioEngineAsync` once at startup and reuse the engine for all sounds.
- Use `StaticSound` for any effect that plays frequently (footsteps, UI clicks, hits).
- Use `StreamingSound` for background music and ambient tracks.
- Pre-load streaming sounds with `preloadInstanceAsync()` to avoid startup delay.
- AudioV2 handles browser autoplay restrictions automatically — set `resumeOnInteraction: true` (the default) and audio will unlock on the first user click/tap.
