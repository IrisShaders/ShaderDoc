# Iris Features

### All the following features are exclusive to Iris.

## Table of Contents

1. [New Programs](#new-programs)
2. [Defines / Feature Flags](#defines--feature-flags)
3. [Uniforms](#uniforms)
4. [Shader Properties](#shader-properties)
5. [Custom Entity ID's](#custom-entity-ids)
6. [Item and Armor Detection](#item-and-armor-detection)
7. [Light Block Voxelization](#light-block-voxelization)
8. [Hybrid-Deferred Entities](#hybrid-deferred-entities)
9. [Separate Hardware Shadow Samplers](#separate-hardware-shadow-samplers)
10. [Shader Storage Buffer Objects](#shader-storage-buffer-objects)
11. [Custom Images](#custom-images)
12. [Extended Shadowcolor](#extended-shadowcolor)

# New Programs

## Entity Translucent (Iris 1.5)

This program renders all entities that are marked translucent and have blending enabled. If this program is not used, all entities will be rendered with `gbuffers_entities`.

### Declaration

```
gbuffers_entities_translucent
```

## Block Entity Translucent (Iris 1.6)

This program renders all block entities that are marked translucent and have blending enabled. If this program is not used, all block entities will be rendered with `gbuffers_block`.

### Declaration

```
gbuffers_block_translucent
```

## Particles (Iris 1.6)

This program renders all non-translucent particles. If this program is not used, all particles will be rendered with `textured_lit`.

### Declaration

```
gbuffers_particles
```

## Particles Translucent (Iris 1.6)

This program renders all translucent particles with blending enabled. If this program is not used, all translucent particles will be rendered with `particles`.

This can be combined with [Particle Ordering](#particle-ordering-iris-15) to render translucent particles after the deferred pass.

### Declaration

```
gbuffers_particles_translucent
```

# Defines / Feature Flags

## IS_IRIS (Iris 1.6)

You can check for the define `IS_IRIS` to check for Iris support.

## Feature Flags

Feature flags are a new system in Iris to query the existence of certain features. To activate them use `iris.features.required` to show an error if your PC or Iris doesn't support a feature, or use `iris.features.optional` to get a define with the feature name `IRIS_FEATURE_X` if the feature is supported.

The currently added feature flags are:

`SEPARATE_HARDWARE_SAMPLERS` (required for [Separate Hardware Shadow Samplers](#separate-hardware-shadow-samplers))

`PER_BUFFER_BLENDING`

`COMPUTE_SHADERS`

`ENTITY_TRANSLUCENT` (recommended, but not required for [Entity Translucent](#entity-translucent--iris-15-))

`SSBO` (required for [Shader Storage Buffer Objects](#shader-storage-buffer-objects))

`CUSTOM_IMAGES` (required for [Custom Images](#custom-images))

`HIGHER_SHADOWCOLOR` (required for [Extended Shadowcolor](#extended-shadowcolor))

# Uniforms

## Lightning bolt position (Iris 1.2.5)

This value reads the position of a lightning bolt currently being rendered. If there are none, zero is returned. If there is at least one, `w` is set to 1. If there are multiple,
a random one is chosen.

### Declaration

```glsl
uniform vec4 lightningBoltPosition;
```

## Thunder strength (Iris 1.3)

This value controls the "thunder strength", equivalent to Optifine's rainStrength for thunder.
### Declaration

```glsl
uniform float thunderStrength;
```

## Player health, air, and hunger (Iris 1.2.7)

These are multiple declarations to read player health, air, and hunger.

The uniforms marked with `current` are 0-1. To get them as a full value, multiply them by their `max` values.

### Declaration

```glsl
uniform float currentPlayerHealth;
uniform float maxPlayerHealth;
uniform float currentPlayerAir;
uniform float maxPlayerAir;
uniform float currentPlayerHunger;
uniform float maxPlayerHunger;
```

## Camera uniforms & Eye Position (Iris 1.4)

These uniforms read multiple aspects of the camera.

`eyePosition` stores the world space position of the player's head model. When in first person view, this is equivalent to `cameraPosition`. However in third person mode the two will differ as the camera and player's head are now in different locations.

### Declaration

```glsl
uniform bool firstPersonCamera;
uniform bool isSpectator;
uniform vec3 eyePosition;
```

## Additional Player Model Uniforms (Iris 1.6.11)

These uniforms read multiple aspects of the player model and camera.

`relativeEyePosition` reads the world space offset from the player model's head position to the camera's position(ie `cameraPosition` - `eyePosition`).

`playerLookVector` reads the world alligned direction the player model's head is facing. This facing direction is not affected by animations such as swimming.

`playerBodyVector` reads the world alligned direction the player model's body is facing, although this behavior is currently broken and reads the same value as `playerLookVector`.

### Declaration
```glsl
uniform vec3 relativeEyePosition;
uniform vec3 playerLookVector;
uniform vec3 playerBodyVector;
```

## World Info Uniforms (Iris 1.5)

These uniforms read many aspects of the dimension, such as the minimum/maximum height and ambient lighting.


### Declaration

```glsl
uniform int bedrockLevel;
uniform int heightLimit;
uniform bool hasCeiling;
uniform bool hasSkylight;
uniform float ambientLight;
```

## Player sneaking, sprinting, hurt, invisible, and burning uniforms (Iris 1.5)

These boolean uniforms are `true` while the condition they are named after is active.

`is_hurt` is `true` for a short time after the player is hurt for any reason, then returns to `false`.

`is_invisible` is `true` both when using an invisibility potion and when in spectator mode.

### Declaration

```glsl
uniform bool is_sneaking;
uniform bool is_sprinting;
uniform bool is_hurt;
uniform bool is_invisible;
uniform bool is_burning;
```

## Player is on ground (Iris 1.6.5)

This boolean uniform is `true` when the player is not flying is an on the ground, and `false` otherwise.

### Declaration

```glsl
uniform bool is_on_ground;
```

# Shader Properties

## Particle ordering (Iris 1.5)

This controls how particles will be rendered. There are 3 possible options for this value.

`mixed`: Opaque particles are rendered before the deferred pass, and translucent particles are rendered after.

This is the default if **no deferred passes** are present.


`after`: All particles are rendered after the deferred pass.

This is the default if there are any deferred passes.

`before`: All particles are rendered before the deferred pass.

### Declaration

```
particles.ordering = mixed
```

### Valid Declaration Locations

* `shaders.properties`

## Entity shadow distance multiplier (Iris 1.2.1)

This value controls the bounds for entity shadows to be rendered. By default, it is the value set for terrain. Any floating point number that is not 1 is multiplied by the terrain distance to get the final shadow distance multiplier.

### Declaration

```glsl
const float entityShadowDistanceMul = 1.0f;
```

### Value Range

* Minimum Value: 0.01
* Default Value: Terrain value
* Out-of-range values behavior: Disable

### Valid Declaration Locations

* ❌ Vertex Shader (*.vsh)
* ❌ Geometry Shader (*.gsh)
* ✔️ Fragment Shader (*.fsh)


## playerShadow directive (Iris 1.2.5)

This value controls if the player should have a shadow rendered. This is forced on if entityShadow (default true) is enabled.
This also controls shadows of any entities the player is riding.

### Declaration

```
playerShadow = true
```

### Valid Declaration Locations

* `shaders.properties`

# The following are exclusive to Iris 1.6.

# Custom Entity ID's

Iris hardcodes some custom entity ID's to detect specific things.

`minecraft:entity_shadow`: The circular shadow under an entity when there is no shadow map.
`minecraft:entity_flame`: The flame when an entity is on fire.
`minecraft:zombie_villager_converting`: A zombie villager undergoing conversion.
`minecraft:player_cape`: Player cape (without elytra).
`minecraft:elytra_with_cape`: Player cape (with elytra).

# Item and Armor Detection

Iris allows detecting items and armor during rendering on *anything*. 

Using `uniform int currentRenderedItemId;`, you can detect items and armor rendered in the level at the point of render.

There are some new ID's that can be detected alongside items and armor:

`trim_material` to detect armor trims on armor. (For example, `trim_emerald`).

# Light Block Voxelization

Using `voxelizeLightBlocks` in shaders.properties, you can now voxelize light blocks in the shadow or main pass.

Light blocks will be rendered as a single (degenerate) invisible quad with all points centered on the middle of the block. (`at_midBlock` will be 0.) The ID will correspond as normal to the light block,
and UV will be 0. `lmcoord.xy` will both be the value of the light made by the light block.

# Hybrid Deferred Entities

Using the new `gbuffers_entities_translucent` and `gbuffers_block_translucent` programs, you can now render entities and blocks in a hybrid deferred manner.

If `separateEntityDraws` is true in `shaders.properties`, entity draws will behave differently. During the main render, translucent entities and block entities will wait to be drawn until **after** the deferred pass,
and then will be drawn in the `gbuffers_entities_translucent` and `gbuffers_block_translucent` programs.

# Separate Hardware Shadow Samplers

Separate hardware shadow samplers can be enabled using the [feature flag](#feature-flags) for it.

When enabled and shadow hardware filtering is enabled via the hardware constant, `shadowtex0` and `shadowtex1` will no longer function as hardware samplers.

Instead, you can use `shadowtex0HW` and `shadowtex1HW` to sample using hardware shadow filtering and software at the same time

# Shader Storage Buffer Objects

Shader Storage Buffer Objects (or SSBO's) are buffers that can store large amounts of data between shader invocations, and even between frames.

For more information about the limits of SSBO's in OpenGL, read https://www.khronos.org/opengl/wiki/Shader_Storage_Buffer_Object.

SSBO's can hold a guaranteed maximum of 128MB, and on most drivers can hold as much as the GPU physically alllows.

Iris will give an error and fail to load a shader if allocating an SSBO would otherwise use up too much VRAM.

To allocate an SSBO for a shaderpack, put the following in shaders.properties, where index can be between 0 and 8:

`bufferObject.<index> = byteSize`

To use an SSBO in a shader, you must define it's layout. Here is an example definition of a SSBO, where bufferName can be any name:

**This layout must be the same across all shaders, otherwise the data will get corrupted.**

```glsl
layout(std430, binding = index) buffer bufferName {
    vec4 someData; // 16 bytes
    float someExtraData; // 4 bytes
};

void main() {
    someData = vec4(0); // Other shaders will see this
    gl_FragColor = vec4(someExtraData); // Read from any previous shaders, or the previous frame if it was never overwritten
}
```

You can optionally make all the variables of an SSBO local to avoid redefinitions. To do this, declare an accessor for the SSBO, as the following.

```glsl
layout(std430, binding = index) buffer bufferName {
    vec4 someData; // 16 bytes
    float someExtraData; // 4 bytes
} bufferAccess;

void main() {
    bufferAccess.someData = vec4(0); // Other shaders will see this
    gl_FragColor = vec4(bufferAccess.someExtraData); // Read from any previous shaders, or the previous frame if it was never overwritten
}
```

# Custom Images

Custom images allow to write up to 8 custom full images of data.

For more info on allowed formats and restrictions, along for their use in shaders, read https://www.khronos.org/opengl/wiki/Image_Load_Store.

To declare a custom image, use the following in shaders.properties:

```
image.cimage1 = samplerAccess format internalFormat pixelType <shouldClearOnNewFrame> <isRelative> <relativeX/absoluteX> <relativeY/absoluteY> <absoluteZ>
```

For example, to declare a RGBA32F custom image half the screen size that does not clear, use the following:

```
image.cimage1 = cSampler1 rgba rgba32f float false true 0.5 0.5
```

And to declare a RGBA8 3D custom image with dimensions of 512x512x512 that clears every frame:

```
image.cimage1 = cSampler1 rgba rgba8 float true false 512 512 512
```

This image can be accessed in 2 ways:

The image can be written and read per-pixel by declaring it as an `image`:

```glsl
uniform image2D cimage1;

void main() {
    vec4 previousValue = imageLoad(cimage1, vec2(1, 1)); // Reads from first pixel in the image
    imageStore(cimage1, vec2(1, 1), vec4(1, 0, 0, 1)); // Writes to first pixel in the image
}
```

Or the image can be read with filtering as a `sampler`:

```glsl
uniform sampler2D cSampler1;

void main() {
    gl_FragColor = texture2D(cSampler1, gl_FragCoord.xy); // Samples the first pixel of the image with smooth linear filtering.
}
```

# Extended Shadowcolor

If the [Feature Flag](#feature-flags) for extended shadowcolor is set, `shadowcolor2` through `shadowcolor7` is enabled.

These can be drawn to via `DRAWBUFFERS` or `RENDERTARGETS` in the shadow pass or shadow composites.
