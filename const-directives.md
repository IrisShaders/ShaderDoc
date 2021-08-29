# Const Directives

## Table of Contents

1. [Ambient occlusion level](#ambient-occlusion-level)

## Ambient occlusion level

This value controls the strength of vanilla's ambient occlusion. By default, it is 1.0, corresponding to unmodified vanilla ambient occlusion.

### Declaration

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
* ❌ Iris

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
