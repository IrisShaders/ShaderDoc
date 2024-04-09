# Const Directives

## Table of Contents

1. [Texture Format](#texture-format)
2. [Texture Clear](#texture-clear)
3. [Texture Mipmaps](#texture-mipmaps)
4. [Shadow Texture Mipmaps](#shadow-texture-mipmaps)
5. [Instance Count](#instance-count)
6. [Ambient occlusion level](#ambient-occlusion-level)
7. [Super sampling level](#super-sampling-level)

## Texture Format

This directive allows a shader pack to specify the format and precision of a colortex or shadowcolor color attachment.

The available texture formats are as follows:

#### Normalized

Normalized buffers can store "floating point" values in a fixed range, between 0.0 and 1.0 (inclusive).  The values are not actually stored as floating point internally, but as an integer value. However, the result is converted to a floating point during read/write to a color attachment using a normalized format, so the shader will read/write floating point values to the attachments, however any values outside of the range will be clamped to fit in the range.

| 8-bit | 16-bit | mixed    |
| ----- | ------ | -------- |
| R8    | R16    | RGB5_A1  |
| RG8   | RG16   | RGB10_A2 |
| RGB8  | RGB16  |          |
| RGBA8 | RGBA16 |          |

#### Signed Normalized

Signed Normalized buffers are identical to normalized buffers, however the range they can represent is expanded to -1 to 1 inclusive, allowing them to store negative values.

| 8-bit       | 16-bit       |
| ----------- | ------------ |
| R8_SNORM    | R16_SNORM    |
| RG8_SNORM   | RG16_SNORM   |
| RGB8_SNORM  | RGB16_SNORM  |
| RGBA8_SNORM | RGBA16_SNORM |

#### Floating Point

Floating point buffers allow the attachment to store full floating point values. They are only available with larger than default precision due to the store needed for floating point values. However they offer a significantly larger range of storage, allowing the storing of HDR values.

The `RGB9_E5` format uses a 5-bit exponent for all three terms (R, G, and B), where each component has a 9 bit mantissa. This allows significantly more precision than the `R11F_G11F_B10F`, which only has 6 to 5 bits of precision per component, however `R11F_G11F_B10F` has individual exponents for each component. For more information on these format types, see the [OpenGL Wiki](https://www.khronos.org/opengl/wiki/Small_Float_Formats)

| 16-bit  | 32-bit  | mixed          |
| ------- | ------- | -------------- |
| R16F    | R32F    | RGB9_E5        |
| RG16F   | RG32F   | R11F_G11F_B10F |
| RGB16F  | RGB32F  |                |
| RGBA16F | RGBA32F |                |

#### Signed Integer

Signed Integer buffers allow storing signed integers, just like the tin says. These are standard two's complement integers, which means they can store negative values.

| 8-bit  | 16-bit  | 32-bit  |
| ------ | ------- | ------- |
| R8I    | R16I    | R32I    |
| RG8I   | RG16I   | RG32I   |
| RGB8I  | RGB16I  | RGB32I  |
| RGBA8I | RGBA16I | RGBA32I |

#### Unsigned Integer

Unsigned Integer buffers allow storing of unsigned integers, which will not be interpreted as negative values (although they can technically be cast to signed integers). As such they do not directly allow the storage of negative integers, but have double the range of their signed cousins in the positive domain.

| 8-bit   | 16-bit   | 32-bit   |
| ------- | -------- | -------- |
| R8UI    | R16UI    | R32UI    |
| RG8UI   | RG16UI   | RG32UI   |
| RGB8UI  | RGB16UI  | RGB32UI  |
| RGBA8UI | RGBA16UI | RGBA32UI |


This directive only needs to be defined once in the shader pack, can can be defined in (mostly) any shader file. Because the format names are not defined as part of glsl, the directive must either be put in a block comment, or the format name must be otherwise manually defined to any value to avoid compilation errors. Do not put the directive in a single line comment, or have any other text in the line of the directive, as this may cause it to not be detected properly.

### Declaration

Scope: Entire Pack

```glsl
/*
const int <bufferName>Format = <format>;

Ex:
const int colortex4Format = RGB9_E5;
*/
```

### Valid Declaration Locations

* ❌ Vertex Shader (*.vsh)
* ❌ Geometry Shader (*.gsh)
* ✔️ Fragment Shader (*.fsh)

### Implementation Support

* ❓ ShadersMod
* ✔️ OptiFine
* ✔️ Iris

## Texture Clear

This Clear directive allows a shader pack to disable clearing for a colortex or shadowcolor color attachment. This means that the values written to the texture will be retained through the next frame instead of being cleared after every frame.

The ClearColor directive is only used when clearing is enabled, and it defined the values stored in the buffer during the clear operation. By default, all colortex buffers are cleared to 0s (black), except colortex0, which is cleared to the current fog color, and colortex1, which is cleared to white. All shadowcolor buffers are cleared with white by default. This directive allows changing this clear color for each attachment.

Currently the only option for setting clear color is with a vec4 value. If the buffer stores less than four components, the leftmost components will be used and the rest discarded. For integer buffers, any positive value will be stored as the maximum positive integer, 0.0 will be stored as 0, and for signed integer buffers any negative value will be stored as the largest magnitude negative number.

Both of these directives only need to be defined once in the shader pack, and can be defined in (mostly) any shader file.

### Declaration

Scope: Entire Pack

```glsl
const bool <bufferName>Clear = false;
const vec4 <bufferName>ClearColor = <value>;
```

### Value Range

* Clear
	* Possible Values: true/false
	* Default Value: false
	* Out-of-range values behavior: default value (false)
* ClearColor
	* Possible Values: vec4(\<r\>, \<g\>, \<b\>, \<a\>)
	* Default Value: vec4(0.0, 0.0, 0.0, 0.0)
	* Out-of-range values behavior: default value

### Valid Declaration Locations

* ❌ Vertex Shader (*.vsh)
* ❌ Geometry Shader (*.gsh)
* ✔️ Fragment Shader (*.fsh)

### Implementation Support

* ❓ ShadersMod
* ✔️ OptiFine
* ✔️ Iris

## Texture Mipmaps

This directive allows shaders to generate a mipmap chain for a colortex attachment. The mipmap chain will be generated before any program where the directive is included, if there are multiple programs it will be re-generated before each program containing the directive. This directive is only valid for composite, deferred, and final programs.

### Declaration

Scope: Per-program

```glsl
const bool <bufferName>MipmapEnabled = true;
```

### Value Range

* Possible Values: true/false
* Default Value: false
* Out-of-range values behavior: default value (false)

### Valid Declaration Locations

* ❌ Vertex Shader (*.vsh)
* ❌ Geometry Shader (*.gsh)
* ✔️ Fragment Shader (*.fsh)

### Implementation Support

* ❓ ShadersMod
* ✔️ OptiFine
* ✔️ Iris

## Shadow Texture Mipmaps

There are several directives for enabling mipmaps for shadow buffers, including shadowtex and shadowcolor attachments. These directives only need to be included once, as after the shadow pass no other programs can write to shadow attachments. The mipmaps are not accessible during the shadow pass.

`shadowtexMipmap` is equivalent to `shadowtex0Mipmap`, only enabling mipmaps on shadowtex0. The other directives enable mipmaps on only the specific attachment listed in the name.

### Declaration

Scope: Entire Pack

```glsl
const bool shadowtexMipmap = true;
const bool shadowtex0Mipmap = true;
const bool shadowtex1Mipmap = true;
const bool shadowcolor0Mipmap = true;
const bool shadowcolor1Mipmap = true;
```

### Value Range

* Possible Values: true/false
* Default Value: false
* Out-of-range values behavior: default value (false)

### Valid Declaration Locations

* ❌ Vertex Shader (*.vsh)
* ❌ Geometry Shader (*.gsh)
* ✔️ Fragment Shader (*.fsh)

### Implementation Support

* ❓ ShadersMod
* ✔️ OptiFine
* ✔️ Iris
## Instance count

This directive allows a shader pack to enable a form of instanced rendering for a given program. Shaders may read the instanceId uniform in order to figure out what instance is currently being rendered. For the first instance, instanceId will be zero, and for all other instances, it will be a non-zero value corresponding to the instance index. So instance 0 will be drawn, then 1, then 2, etc.

Instanced rendering was introduced in order to allow shader packs to perform voxelization without geometry shaders.

Note that this mode of instanced rendering operates separately from the instanced rendering commands supported by OpenGL. Attempting to read gl_InstanceID will always result in a value of zero, and glDrawArraysInstanced will not be used. Instead, glDrawArrays is called multiple times in a loop, updating the instanceId uniform each time.

⚠️ Note that even on implementations where this directive is supported, support for this directive is still on a best-effort basis. Terrain and full-screen passes are guaranteed to properly draw all instances. Other content should draw all instances in vanilla. However, in the presence of mods, outside of terrain and fullscreen passes only the first instance is guaranteed to be drawn.

Note that no matter what, instanced rendering is all-or-nothing. If instanced rendering does work for some given content, then all instances will be drawn. If it does not work, then only one instance will be drawn.

**Shader packs aiming for full compatibility with mods should only use instancing for terrain and fullscreen passes.**

### Declaration

Scope: Per-program

```glsl
const int countInstances = 1;
```

### Value Range

* Minimum Value: 1 (normal behavior, no instancing)
* Maximum Value: none
* Default Value: 1
* Out-of-range values behavior: Disable instancing

### Implementation Support

* ❌ ShadersMod
* ✔️ OptiFine
    * ❌ 1.13 and below: not supported
    * ✔️ 1.14.4: HD U F3 and above
    * ✔️ 1.15 and above: all versions
* ❌ Iris

## Ambient occlusion level

This value controls the strength of vanilla's ambient occlusion. By default, it is 1.0, corresponding to unmodified vanilla ambient occlusion.

### Declaration

Scope: Entire Pack

```glsl
const float ambientOcclusionLevel = 1.0f;
```

### Value Range

* Minimum Value: 0.0 (no ambient occlusion)
* Maximum Value: 1.0 (vanilla ambient occlusion)
* Default Value: 1.0
* Out-of-range values behavior: Clamp

### Valid Declaration Locations

* ❌ Vertex Shader (*.vsh)
* ❌ Geometry Shader (*.gsh)
* ✔️ Fragment Shader (*.fsh)

### Implementation Support

* ❓ ShadersMod
    * Though supporting code is present, it is unused / disabled in ShadersMod v2.7.0 for 1.12, the version used for analysis. Other versions of ShadersMod might support it properly.
* ✔️ OptiFine
* ✔️ Iris

### Implementation Details

Existing valid implementations intercept and tweak the return value of the method `BlockState#getAmbientOcclusionLightLevel` (Yarn 1.16.5) / `BlockStateBase#getShadeBrightness` (Mojmap 1.16.5). For blocks that do not induce ambient occlusion, this method returns `1.0`. Full cubes induce ambient occlusion, with the exception of glass, barrier blocks, and structure void blocks. For all vanilla blocks that induce ambient occlusion, this method returns `0.2`.

Modifying the return value of this method with the following function is sufficient to properly implement support for values of ambientOcclusionLevel other than `1.0`:

```
F(x) = 1 - (ambientOcclusionLevel) (1 - x)
```

The implementation in ShadersMod / OptiFine uses a different version of the above function that is less general, only properly handling inputs of `0.2` and `1.0`. While this is adequate for unmodified Minecraft and most mods, if a mod tries to use a value other than 0.2 or 1.0 there will be incorrect ambient occlusion values.

## Super sampling level

Allows shader packs to use super sampling. A super sampling level of 2 means that the width and height of the shader framebuffers are doubled, meaning that there will be 4x the pixels. Super sampling is an extremely expensive method of anti aliasing and should probably be avoided!

⚠️ This directive is not known to be supported on any implementation of the shaders format! Any attempts to use it will almost certainly be ignored.

### Declaration

Scope: Entire Pack

```glsl
const int superSamplingLevel = 1;
```

### Value Range

* Minimum Value: 1 (super sampling disabled)
* Maximum Value: none
* Out-of-range values behavior: Disable super sampling

### Valid Declaration Locations

* ❌ Vertex Shader (*.vsh)
* ❌ Geometry Shader (*.gsh)
* ✔️ Fragment Shader (*.fsh)

### Implementation Support

* ❓ ShadersMod
    * Though supporting code is present, it is unused / disabled in ShadersMod v2.7.0 for 1.12, the version used for analysis. Other versions of ShadersMod might support it properly.
* ❌ OptiFine
* ❌ Iris

### Implementation Details

ShadersMod and OptiFine both appear to contain stub supporting code for this feature, but neither one actually enables super sampling when requested through this directive.
