# Uniforms

ShadersMod & OptiFine define many uniforms that allow shaders to access the current state of the world, the player, and the renderer.

Note: Some of these uniforms are not supported by ShadersMod. They will take on an initial/default value of `0`, per the OpenGL Specification.

## Player

### Held item ID

Item ID of the item currently held in the main hand. 

For OptiFine, the item ID is defined by the mapping in `items.properties`.

> TODO
> * OptiFine: What about when nothing is held? (probably air)
> * OptiFine: What about when the currently held item isn't defined in `items.properties`?

For ShadersMod, which does not support `item.properties`, the ID is defined by the Anvil (pre-1.13 world format) ID mapping of the currently held item. If the player is currently not holding an item, an ID of `-1` is provided. For example, if the player was holding a stone block, the value would be `1`.

#### Declaration

```glsl
uniform int heldItemId;
```

#### Implementation Support

* ✔️ ShadersMod
* ✔️ OptiFine


### Held block light value

Light value of the block that the player is currently holding in their hand.

> TODO:
> * For OptiFine, the exact behavior is unknown. It's probably similar to ShadersMod.

For ShadersMod, the light value, ranging from 0 (no light) to 15 (brightest), of the block's default block state is provided. If the player is not currently holding an item, or the currently held item is not a block, a value of 0 is provided.

For example, if the player is currently holding a block of Glowstone, a value of 15 will be provided, but if they are holding a block of Stone, a value of 0 will be provided.

#### Declaration

```glsl
uniform int heldBlockLightValue;
```

#### Implementation Support

* ✔️ ShadersMod
* ✔️ OptiFine


### Held item ID (off hand)

This is the same as the main [Held item ID](#held-item-id), but instead referring to the off hand (most commonly the left hand).

#### Declaration

```glsl
uniform int heldItemId2;
```

#### Implementation Support

* ❌ ShadersMod
* ✔️ OptiFine

### Held block light value (off hand)

This is the same as the main [Held block light value](#held-block-light-value), but instead referring to the off hand (most commonly the left hand).

#### Declaration

```glsl
uniform int heldBlockLightValue2;
```

#### Implementation Support

* ❌ ShadersMod
* ✔️ OptiFine


## Sky and fog

### OpenGL fog mode

Holds the current OpenGL fog mode, or 0 if there is no fog. This is equivalent to the value of `glGetInteger(GL_FOG_MODE)`.

The following values are valid:

| Name      | Decimal | Hex    |
| --------- | ------- | -------|
| GL_LINEAR | 9279    | 0x2601 |
| GL_EXP    | 2048    | 0x0800 |
| GL_EXP2   | 2049    | 0x0801 |

For more information on fog modes, see the [official OpenGL documentation](https://www.khronos.org/registry/OpenGL-Refpages/gl2.1/xhtml/glFog.xml#description).


#### Declaration

```glsl
uniform int fogMode;
```

#### Implementation Support

* ✔️ ShadersMod
* ✔️ OptiFine


### Fog density

The current fog density. Ranges from 0.0 to 1.0.

> TODO:
> * Find out more about this value

#### Declaration

```glsl
uniform float fogDensity;
```

#### Implementation Support

* ❌ ShadersMod
* ✔️ OptiFine


### Fog color

The current color of the fog. Corresponds to the current value of `glGetFloatv(GL_FOG_COLOR)`.

Each color channel has a float value of 0.0 to 1.0, corresponding to RGB values from 0 to 255. Note that the values are encoded in the non-linear sRGB color space: to receive linear RGB values, apply the function `pow(x, 2.4)` as described [on Wikipedia](https://en.wikipedia.org/wiki/SRGB#The_reverse_transformation).

> TODO:
> * Make sure that the values are actually sRGB!

#### Declaration

```glsl
uniform vec3 fogColor;
```

#### Implementation Support

* ✔️ ShadersMod
* ✔️ OptiFine


### Sky / clear color

The current color of the sky. Corresponds to the current value of `glGetFloatv(GL_COLOR_CLEAR_VALUE)` or the value passed to `glClearColor`.

Each color channel has a float value of 0.0 to 1.0, corresponding to RGB values from 0 to 255. Note that the values are encoded in the non-linear sRGB color space: to receive linear RGB values, apply the function `pow(x, 2.4)` as described [on Wikipedia](https://en.wikipedia.org/wiki/SRGB#The_reverse_transformation).

> TODO:
> * Make sure that the values are actually sRGB!

#### Declaration

```glsl
uniform vec3 skyColor;
```

#### Implementation Support

* ✔️ ShadersMod
* ✔️ OptiFine


## World time

### Current world day time

Represents the current time of day, in ticks, ranging from 0 to 23999 (inclusive), corresponding to the Minecraft [daylight cycle](https://minecraft.gamepedia.com/Daylight_cycle). This is the value of the DayTime property as stored in `level.dat`, modulo 24000.

#### Declaration

```glsl
uniform int worldTime;
```

#### Implementation Support

* ✔️ ShadersMod
* ✔️ OptiFine


### Current world day

Represents the current day of the world. As opposed to the time of day, this is the number of the current day: 0 for the first day, 1 for the second day, and so on. This is the value of the DayTime property as stored in `level.dat`, divided by 24000.

#### Declaration

```glsl
uniform int worldDay;
```

#### Implementation Support

* ❌ ShadersMod
* ✔️ OptiFine


### Moon phase

Holds the current moon phase, ranging from 0 to 7 (inclusive). Normally this is equivalent to `worldDay % 8`, however, custom (modded) dimensions may define their own moon phase values. The actual appearance of the moon at each phase is in the same order that the [Minecraft Wiki](https://minecraft.gamepedia.com/Moon#Phases) depicts them:

0. Full moon
1. Waning gibbous
2. Third quarter
3. Waning crescent
4. New moon
5. Waxing crescent
6. First quarter
7. Waxing gibbous

#### Declaration

```glsl
uniform int moonPhase;
```

#### Implementation Support

* ✔️ ShadersMod
* ✔️ OptiFine


## System time

### Frame counter

An arbitrary frame index, ranging from 0 to 720719 (inclusive). Once this counter reaches 720719, it will reset to 0 on the next frame. At 60 FPS, this counter will reset approximately every 3 hours and 20 minutes.

#### Declaration

```glsl
uniform int frameCounter;
```

#### Implementation Support

* ❌ ShadersMod
* ✔️ OptiFine


### Frame time

The time that the last frame took to render, in seconds.

> TODO:
> * What is the resolution of this value? Presumably milliseconds.

#### Declaration

```glsl
uniform float frameTime;
```

#### Implementation Support

* ❌ ShadersMod
* ✔️ OptiFine


### Frame time counter

An indicator of the run time, in seconds. Starts at 0.0 on the first frame, and on each frame is updated according to the system time. Has a resolution of 1 millisecond (0.001).

On OptiFine, this counter resets every hour (3600 seconds), while on ShadersMod, it resets every 100000 seconds.

#### Declaration

```glsl
uniform float frameTimeCounter;
```

#### Implementation Support

* ✔️ ShadersMod
* ✔️ OptiFine


## Celestial bodies

### Sun angle

A float, ranging from 0.0 to 1.0, that represents the current angle of the sun (and, also, moon) in the sky. To get the angle in radians, multiply by 2π, and to get the angle in degrees, multiply by 360.0.

A value of 0.0 represents the sun directly on the east horizon and the moon on the west horizon, 0.25 represents the sun directly above, 0.5 represents the sun at the west horizon and the moon on the east horizon, and 0.75 represents the moon directly above.

For the purposes of determining the angle of the celestial body in the sky that is currently casting a shadow, it's recommended to use the [shadow angle](#shadow-angle) instead.

#### Declaration

```glsl
uniform float sunAngle;
```

#### Implementation Support

* ✔️ ShadersMod
* ✔️ OptiFine


### Sun position

Holds the sun position in "eye space" - the result of transforming the vector (0, 100, 0) by the celestial modelview matrix. In other words, this is the position of the sun, rotated along its path.

Minecraft note: the celestial modelview matrix refers to the content of the modelview matrix right before the sun and moon are rendered.

#### Declaration

```glsl
uniform vec3 sunPosition;
```

#### Implementation Support

* ✔️ ShadersMod
* ✔️ OptiFine


### Moon position

Holds the moon position in "eye space" - the result of transforming the vector (0, -100, 0) by the celestial modelview matrix. In other words, this is the position of the moon, rotated along its path.

Minecraft note: the celestial modelview matrix refers to the content of the modelview matrix right before the sun and moon are rendered.

#### Declaration

```glsl
uniform vec3 moonPosition;
```

#### Implementation Support

* ✔️ ShadersMod
* ✔️ OptiFine


### Shadow angle

The angle in the sky of the current shadow-casting celestial body. During the day, this is the sun angle, and during the night, this is the moon angle (sun angle - 0.5). Thus, this value has a range of 0.0 to 0.5, inclusive. At a sun angle of 0.5, the shadow angle will be 0.5, and on the next tick, it will reset.

> TODO:
> * The OptiFine documentation states that this value has a range of 0.0-1.0, not 0.0-0.5. Is this a documentation error, or a difference between ShadersMod and OptiFine?

#### Declaration

```glsl
uniform float shadowAngle;
```

#### Implementation Support

* ✔️ ShadersMod
* ✔️ OptiFine


### Shadow light position

The position of the celestial body that is currently casting shadows on the world. When `sunAngle ≤ 0.5`, this is equal to `sunPosition`, and when `sunAngle > 0.5`, this is equal to `moonPosition`.

#### Declaration

```glsl
uniform vec3 shadowLightPosition;
```

#### Implementation Support

* ✔️ ShadersMod
* ✔️ OptiFine


### Up position

An upwards vector of length 100. The value of this uniform is equal to the value of `sunPosition` at midday (6000 ticks), or, alternatively, `moonPosition` at midnight (18000 ticks). The value of this uniform is independent of the current in-game time, unlike the other celestial position values.

#### Declaration

```glsl
uniform vec3 upPosition;
```

#### Implementation Support

* ✔️ ShadersMod
* ✔️ OptiFine


## Weather

### Rain strength

A value from 0.0 to 1.0 representing the current strength of the rain. A value of 0.0 indicates no rain at all, while a value of 1.0 indicates full rain. Values from 0.0 to 0.2 indicate that no rain is falling, but that rainy weather exists.

For every tick that the world exists in a state of rainy weather, the strength increases by 0.01, up to 1.0. For every tick that rainy weather does not exist, the strength decreases by 0.01, down to 0.0. Thus, transitions to and from rainy weather take 5 seconds (100 ticks).

The [wetness](#wetness) value, calculated based on the rain strength, provides a smoother and more delayed transition between dry and wet states.

#### Declaration

```glsl
uniform float rainStrength;
```

#### Implementation Support

* ✔️ ShadersMod
* ✔️ OptiFine


## Viewport

These uniforms refer to properties of the rendering viewport / framebuffer, not the window itself. Though the window and viewport are often the same size, it's possible to modify the rendering resolution multiplier in the Shaders configuration. With a multiplier of 0.5x, for example, the width and height of the viewport will be half of the width and height of the window.

### Viewport width

The width of the viewport. This is the width of the color & depth framebuffers (colortex0-7 and depthtex0-2). Even though this is a float, it should always have a value that is a positive integer.

#### Declaration

```glsl
uniform float viewWidth;
```

#### Implementation Support

* ✔️ ShadersMod
* ✔️ OptiFine


### Viewport height

The height of the viewport. This is the height of the color & depth framebuffers (colortex0-7 and depthtex0-2). Even though this is a float, it should always have a value that is a positive integer.

#### Declaration

```glsl
uniform float viewHeight;
```

#### Implementation Support

* ✔️ ShadersMod
* ✔️ OptiFine


### Aspect ratio

The aspect ratio of the viewport. This is the aspect ratio of the color & depth framebuffers (colortex0-7 and depthtex0-2). This is equivalent to `viewWidth / viewHeight`. 

#### Declaration

```glsl
uniform float aspectRatio;
```

#### Implementation Support

* ✔️ ShadersMod
* ✔️ OptiFine


## Camera

### Near viewing plane

This uniform should always have a value of `0.05`. See the `GameRenderer#applyCameraTransformations` method in yarn-mapped Minecraft for confirmation.

> TODO: Check OptiFine behavior

#### Declaration

```glsl
uniform float near;
```

#### Implementation Support

* ✔️ ShadersMod
* ✔️ OptiFine


### Far viewing plane

ShadersMod sets this value to the render distance in blocks / meters. However, note that the `GameRenderer#applyCameraTransformations` method in yarn-mapped Minecraft actually sets the far plane to the render distance in blocks times the square root of 2.

> TODO: Check OptiFine behavior

#### Declaration

```glsl
uniform float far;
```

#### Implementation Support

* ✔️ ShadersMod
* ✔️ OptiFine
