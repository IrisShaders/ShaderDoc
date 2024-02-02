# Distant Horizons support

## Table of Contents

1. [Definitions/Properties](#definitionsproperties)
2. [Samplers/Uniforms](#samplersuniforms)
3. [Programs](#programs)
4. [Block ID's](#block-ids)

# Definitions/Properties

### Enabling the DH shadow pass

The DH shadow pass is **automatically enabled**. It's existence can be turned on or off with the shader property `dhShadow.enabled`.

**Distant Horizons has no set shadow distance; all chunks will be rendered.**

### Checking for Distant Horizons

All programs and properties have the `DISTANT_HORIZONS` define set if DH is enabled.

# Samplers/Uniforms

## Depth textures

There are two depth textures attached to Distant Horizons;

`dhDepthTex0` and `dhDepthTex1`. These behave identically to their non-DH counterparts, except they have different near/far planes (explained below) and only contain DH terrain/water.

## Uniforms

### dhNearPlane

#### Declaration

```glsl
uniform float dhNearPlane;
```

This uniform specifies the near plane of DH's projection. **This does not apply to the shadow pass, use normal projection.**

### dhFarPlane

#### Declaration

```glsl
uniform float dhFarPlane;
```

This uniform specifies the far plane of DH's projection. **This is not the render distance! Use dhRenderDistance for that.**

### dhRenderDistance

#### Declaration

```glsl
uniform int dhRenderDistance;
```

This uniform specifies the render distance currently set in DH.

### dhProjection and variants

#### Declaration

```glsl
uniform mat4 dhProjection;
uniform mat4 dhProjectionInverse;
uniform mat4 dhPreviousProjection;
```

This is the projection matrix for the non-shadow pass of Distant Horizons. This includes a different near and far plane, see the uniforms above.

# Programs

## Supported attributes

The following attributes are supported in DH programs:

`gl_Vertex`
`gl_MultiTexCoord2`
`gl_Normal`
`gl_Color`
`dhMaterialId`

The following built in uniforms are supported:

`gl_ModelViewMatrix(Inverse, etc)`
`gl_ProjectionMatrix(Inverse, etc)`
`gl_NormalMatrix`

### Terrain

Terrain gets the program `dh_terrain`. This runs before normal terrain.

### Water

Water gets the program `dh_water`.  This runs **before** normal water.

### Shadow

Shadow pass gets the program `dh_shadow`. This runs before shadow terrain and shadow water respectively. **The shadow pass retains the normal textures and projection.**

# Block ID's

**Normal block ID's (mc_Entity) are not supported.**

Iris provides "mini-IDs" in the form of `int dhMaterialId` automatically declared.

All Mini-ID's get their own definitions that can be used in any program.

```text
DH_BLOCK_UNKNOWN // Any block not in this list that does not emit light
DH_BLOCK_LEAVES // All types of leaves, bamboo, or cactus
DH_BLOCK_STONE // Stone or ore
DH_BLOCK_WOOD // Any wooden item
DH_BLOCK_METAL // Any block that emits a metal or copper sound.
DH_BLOCK_DIRT // Dirt, grass, podzol, and coarse dirt.
DH_BLOCK_LAVA // Lava.
DH_BLOCK_DEEPSLATE // Deepslate, and all it's forms.
DH_BLOCK_SNOW // Snow.
DH_BLOCK_SAND // Sand and red sand.
DH_BLOCK_TERRACOTTA // Terracotta.
DH_BLOCK_NETHER_STONE // Blocks that have the "base_stone_nether" tag.
DH_BLOCK_WATER // Water...
DH_BLOCK_AIR // Air. This should never be accessible/used.
DH_BLOCK_ILLUMINATED // Any block not in this list that emits light
```
