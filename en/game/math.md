# Vectors, boxes, angles

`Vec`, `Box` and `Rotation` are immutable value types used everywhere in the API: positions and velocities in blocks, bounding boxes in world coordinates, yaw and pitch in degrees. They are records — every operation returns a new instance, and `equals`/`hashCode` compare components.

```kotlin
on<ClientTickEvent> {
    val eyes = player.eyePosition()
    val ahead = player.rotation().direction().multiply(3.0)   // 3 blocks forward
    val area = Box.around(eyes, 0.5).offset(ahead)
    chat.print("in front: " + world.entitiesIn(area).size)
}
```

## Vec

| Method | Type | Description |
|---|---|---|
| `Vec(x, y, z)` | `Vec` | canonical constructor, components in blocks, no normalization |
| `Vec.of(x, y, z)` | `Vec` | static factory, identical to the constructor |
| `Vec.ZERO` | `Vec` | constant (0, 0, 0) |
| `x()` | `double` | x component in blocks |
| `y()` | `double` | y component in blocks |
| `z()` | `double` | z component in blocks |
| `add(other)` | `Vec` | component-wise sum |
| `add(x, y, z)` | `Vec` | component-wise sum with literals |
| `subtract(other)` | `Vec` | component-wise difference |
| `multiply(factor)` | `Vec` | uniform scale |
| `normalize()` | `Vec` | unit vector, `ZERO` when the length is 0 |
| `dot(other)` | `double` | dot product |
| `length()` | `double` | euclidean length in blocks |
| `squaredLength()` | `double` | squared length, no sqrt |
| `distanceTo(other)` | `double` | euclidean distance in blocks |
| `squaredDistanceTo(other)` | `double` | squared distance, no sqrt |

Positions, eye points and velocities come back as `Vec` — see [Your player](player.md) and [Entities and filters](entities.md).

## Box

| Method | Type | Description |
|---|---|---|
| `Box(minX, minY, minZ, maxX, maxY, maxZ)` | `Box` | canonical constructor, swaps components so min ≤ max per axis |
| `Box.around(center, radiusBlocks)` | `Box` | cube of half-extent radiusBlocks around a point |
| `minX()` | `double` | lower x bound |
| `minY()` | `double` | lower y bound |
| `minZ()` | `double` | lower z bound |
| `maxX()` | `double` | upper x bound |
| `maxY()` | `double` | upper y bound |
| `maxZ()` | `double` | upper z bound |
| `center()` | `Vec` | midpoint of the box |
| `sizeX()` | `double` | extent along x in blocks |
| `sizeY()` | `double` | extent along y in blocks |
| `sizeZ()` | `double` | extent along z in blocks |
| `offset(offset)` | `Box` | translated copy |
| `offset(x, y, z)` | `Box` | translated copy with literals |
| `expand(blocks)` | `Box` | grown by blocks on all six faces |
| `expand(x, y, z)` | `Box` | grown per axis, on both faces |
| `contract(blocks)` | `Box` | shrunk by blocks on all six faces |
| `contains(point)` | `boolean` | point inside, bounds inclusive |
| `intersects(other)` | `boolean` | volumes overlap, touching faces excluded |
| `raycast(from, to)` | `Vec?` | entry point on the segment, null on a miss, `from` when it starts inside |

`entity.box()` and `block.box()` return one; `world.entitiesIn`, `collisionsIn` and `isFree` take one — see [World and blocks](world.md).

## Rotation

| Method | Type | Description |
|---|---|---|
| `Rotation(yawDegrees, pitchDegrees)` | `Rotation` | canonical constructor, clamps pitch to -90..90 degrees |
| `Rotation.of(yawDegrees, pitchDegrees)` | `Rotation` | static factory, identical to the constructor |
| `yawDegrees()` | `float` | yaw in degrees, not wrapped |
| `pitchDegrees()` | `float` | pitch in degrees, -90..90 |
| `yawDeltaTo(other)` | `float` | shortest signed yaw difference, -180..180 degrees |
| `pitchDeltaTo(other)` | `float` | signed pitch difference in degrees |
| `angleTo(other)` | `float` | hypotenuse of the yaw and pitch deltas, in degrees |
| `direction()` | `Vec` | unit look vector for this rotation |

Reading, building and applying rotations is on [Rotations](../actions/rotations.md); `raycast.from(rotation, ...)` is on [Rays and the crosshair](raycast.md).
