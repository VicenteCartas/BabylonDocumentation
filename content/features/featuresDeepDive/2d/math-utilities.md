---
title: Math Utilities
description: Matrix2D and Rectangle2D utility classes for 2D transformations and spatial queries
keywords: 2d, math, matrix, rectangle, transform, affine, collision, AABB
---

# Math Utilities

`@babylonjs/2d` provides two core math classes used throughout the engine: `Matrix2D` for affine transformations and `Rectangle2D` for axis-aligned bounding boxes. These complement Babylon.js core's `Vector2` (which is used directly for positions, scales, and directions).

## Matrix2D

A 3×2 affine transformation matrix stored as 6 floats in column-major order:

```
| a  c  tx |
| b  d  ty |
| 0  0   1 |
```

Stored as `Float32Array [a, b, c, d, tx, ty]`.

### Creating Matrices

```typescript
import { Matrix2D } from "@babylonjs/2d/Math/matrix2D";

const identity = Matrix2D.Identity();
const translate = Matrix2D.Translation(100, 50);
const rotate = Matrix2D.Rotation(Math.PI / 4);    // 45° clockwise (Y-down)
const scale = Matrix2D.Scaling(2, 2);
```

### Composing Transforms

Build a full transform from position, rotation, scale, and pivot:

```typescript
import { Vector2 } from "@babylonjs/core/Maths/math.vector";

const worldTransform = Matrix2D.Compose(
    new Vector2(200, 150),    // position
    Math.PI / 6,              // rotation (30° clockwise)
    new Vector2(1.5, 1.5),   // scale
    new Vector2(16, 16)       // pivot (rotation/scale center)
);
```

The `Compose` method applies the transform as: translate to position → rotate around pivot → scale around pivot.

Skew is also supported:

```typescript
const skewed = Matrix2D.Compose(
    position, rotation, scale, pivot,
    0.2,   // skewX in radians
    0.1    // skewY in radians
);
```

### Multiplying Matrices

```typescript
const combined = parentTransform.multiply(childTransform);

// In-place multiplication (avoids allocation)
const result = new Matrix2D();
parentTransform.multiplyToRef(childTransform, result);

// Multiply into self
parentTransform.multiplyToSelf(childTransform);
```

### Transforming Points

```typescript
const localPoint = new Vector2(10, 20);
const worldPoint = worldTransform.transformPoint(localPoint);
```

### Inverting

```typescript
const inv = worldTransform.invert();
const screenPoint = new Vector2(400, 300);
const localFromScreen = inv.transformPoint(screenPoint);

// In-place version
const invRef = new Matrix2D();
worldTransform.invertToRef(invRef);
```

If the matrix is not invertible (determinant ≈ 0), the result is reset to identity.

### Copying and Cloning

```typescript
const clone = matrix.clone();
const target = new Matrix2D();
target.copyFrom(matrix);
matrix.reset(); // Sets to identity
```

### Methods Reference

| Method | Returns | Description |
|--------|---------|-------------|
| `Matrix2D.Identity()` | `Matrix2D` | Creates an identity matrix |
| `Matrix2D.Translation(x, y)` | `Matrix2D` | Creates a translation matrix |
| `Matrix2D.Rotation(angle)` | `Matrix2D` | Creates a rotation matrix (clockwise in Y-down) |
| `Matrix2D.Scaling(x, y)` | `Matrix2D` | Creates a scaling matrix |
| `Matrix2D.Compose(pos, rot, scale, pivot, skewX?, skewY?)` | `Matrix2D` | Composes a full transform |
| `Matrix2D.ComposeToRef(...)` | `Matrix2D` | Compose into an existing matrix (no allocation) |
| `multiply(other)` | `Matrix2D` | Returns `this × other` |
| `multiplyToRef(other, ref)` | `Matrix2D` | Stores `this × other` into `ref` |
| `multiplyToSelf(other)` | `Matrix2D` | Stores `this × other` into `this` |
| `transformPoint(point)` | `Vector2` | Transforms a Vector2 by this matrix |
| `invert()` | `Matrix2D` | Returns the inverse matrix |
| `invertToRef(ref)` | `Matrix2D` | Stores the inverse into `ref` |
| `clone()` | `Matrix2D` | Deep copy |
| `copyFrom(other)` | `Matrix2D` | Copy values from another matrix |
| `reset()` | `Matrix2D` | Sets to identity |
| `get(index)` | `number` | Returns value at index (0–5) |

### How Node2D Uses Matrix2D

Every `Node2D` has a `worldTransform: Matrix2D` that is composed each frame from its `position`, `rotation`, `scale`, and `pivot`. The `localToWorld()` and `worldToLocal()` methods use this matrix and its inverse to convert between coordinate spaces. You typically don't create `Matrix2D` instances directly — the engine handles them — but they're useful for custom rendering, hit testing, or tools.

---

## Rectangle2D

An axis-aligned rectangle defined by position and size:

```typescript
import { Rectangle2D } from "@babylonjs/2d/Math/rectangle2D";

const rect = new Rectangle2D(10, 20, 200, 100);
// x=10, y=20, width=200, height=100
```

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `x` | `number` | Left edge X coordinate |
| `y` | `number` | Top edge Y coordinate |
| `width` | `number` | Width |
| `height` | `number` | Height |
| `right` | `number` | Right edge (`x + width`) — read-only |
| `bottom` | `number` | Bottom edge (`y + height`) — read-only |

### Point Containment

```typescript
const rect = new Rectangle2D(0, 0, 800, 600);
rect.contains(400, 300); // true — inside
rect.contains(900, 300); // false — outside
```

### Rectangle Intersection

```typescript
const a = new Rectangle2D(0, 0, 100, 100);
const b = new Rectangle2D(50, 50, 100, 100);

a.intersects(b); // true — they overlap

// Compute the overlapping region
const overlap = new Rectangle2D();
Rectangle2D.IntersectToRef(a, b, overlap);
// overlap = { x: 50, y: 50, width: 50, height: 50 }
```

If the rectangles don't overlap, `IntersectToRef` produces a rectangle with zero width and height.

### Cloning

```typescript
const copy = rect.clone();
```

### Methods Reference

| Method | Returns | Description |
|--------|---------|-------------|
| `contains(px, py)` | `boolean` | True if the point is inside the rectangle |
| `intersects(other)` | `boolean` | True if two rectangles overlap |
| `Rectangle2D.IntersectToRef(a, b, ref)` | `Rectangle2D` | Computes the overlapping region |
| `clone()` | `Rectangle2D` | Deep copy |

### Where Rectangle2D Is Used

- **Camera2D.bounds** — Constrains the camera to a region (`camera.bounds = new Rectangle2D(0, 0, levelW, levelH)`)
- **Sprite2D.sourceRect** — Defines the texture sub-region to render
- **RectMask2D** — Clip rectangle for masking children
- **SpatialGrid** — Internal AABB queries for broad-phase collision

---

## Vector2 (from core)

`@babylonjs/2d` reuses Babylon.js core's `Vector2` for all positions, velocities, scales, and directions:

```typescript
import { Vector2 } from "@babylonjs/core/Maths/math.vector";

const pos = new Vector2(100, 200);
const velocity = new Vector2(3, -1);

// Common operations
const sum = pos.add(velocity);
const scaled = velocity.scale(deltaTime);
const dist = Vector2.Distance(a, b);
const lerped = Vector2.Lerp(a, b, 0.5);
```

There is no custom `Vector2` in `@babylonjs/2d` — always import from `@babylonjs/core`.

## Related

- [Introduction](/features/featuresDeepDive/2d/intro) — Coordinate system overview (Y-down, top-left origin)
- [Camera2D](/features/featuresDeepDive/2d/camera2d) — Uses Matrix2D for view transforms and Rectangle2D for bounds
- [Sprites & Animation](/features/featuresDeepDive/2d/sprites-and-animation) — Source rectangles and transforms
