# Vertex Format Extensions

## Table of contents

ShadersMod extends the vertex format of Minecraft in order to enable additional features. It adds the following 4 attributes to the vertex format:

1. [Surface normal vector](#surface-normal-vector)
2. [Surface tangent vector](#surface-tangent-vector)
3. [Center texture coordinate](#center-texture-coordinate)
4. [Block identification](#block-identification)


## Surface normal vector

For each quad, ShadersMod for 1.12.2 computes the surface normal, and then attaches that computed surface normal to each vertex of said quad.

Modern versions of Minecraft (such as 1.16.3) appear to include already-generated vertex normals in the vertex formats used for blocks and items, so it may seem like using the below algorithm is not necessary for rendering vanilla blocks and items. In fact, the vanilla algorithm takes advantage of the block model format in order to simplify the vertex normal calculations!

Unfortunately, for things like tall grass, the vertex normals that Minecraft generates are improperly rotated and remain aligned to major axes (X, Y, or Z) even though they should be rotated by 45 degrees. As a result, it is often still necessary to calculate vertex normals using the below algorithm. Not to mention, it's also possible that Fabric mods will use custom models as well. The point is, we still need to calculate vertex normals here.

### Declaration

This is a built-in attribute and **does not need to be declared**. However, for reference, this is the declaration that would be used if it was *not* a built-in attribute:

```glsl
attribute vec3 gl_Normal;
```


### Algorithm

Consider the following quad, where `v0`, `v1`, `v2`, and `v3` each refer to the position of a single vertex:

```
v0 --------- v3
| \        / |
|   \    /   |
|     \/     |
|     /\     |
|   /    \   |
| /        \ |
v1 --------- v2
```

To compute the surface normal vector, the algorithm first computes the vector between `v0` and `v2` as well as the vector between `v1` and `v3`. Then, it computes the cross product of those two vectors, and then normalizes that cross product. This normalized vector is the *unpacked* surface normal. This operation can be expressed as: `normal = normalize((v2 - v0) × (v3 - v1))`.

The surface normal is then packed into a 32-bit integer by first scaling the unpacked surface normal vector by `127.0`, thus ensuring that x, y, and z of the vector all lie in the range [-127.0, 127.0]. Then, each element of the vector is converted into a signed byte using [Java conversion rules](https://docs.oracle.com/javase/specs/jls/se8/html/jls-5.html#jls-5.1.3). Finally, each of these bytes is then packed into the integer like so:

```
i << 32   i << 24   i << 16   i << 8    i
+---------+---------+---------+---------+
| zero    | z coord | y coord | x coord |
+---------+---------+---------+---------+
```

For example, the normalized vector (-0.75, 0.25, 0.612) would result in a packed vector of `0x004D1FA1`. Note that the most significant byte, a padding byte, will always be zero here.


### Notes

* These vertices are arranged in counterclockwise order, however, the algorithm might work for vertices arranged in clockwise order as well, though this is not verified. In practice, Minecraft uses a counterclockwise winding order, because that is the default of OpenGL.
* It is possible for the cross product to be zero-length, so **ensure that you are not dividing by zero** in your implementation! You will notice that these two vectors form an "X" shape, and they are at 90 degree angles to each other, so for any quad with an area greater than zero, the cross product will be greater than zero as well. However, it is possible that `v0 == v1 == v2 == v3`, which is why this special check exists.


## Surface tangent vector

This attribute in conjunction with the surface normal is very useful for calculating the TBN matrix used in [normal mapping](https://learnopengl.com/Advanced-Lighting/Normal-Mapping). Given the normal vector and tangent vector, you can compute the binormal vector (using cross product) in order to create a TBN matrix.

### Declaration

```glsl
attribute vec4 at_tangent;
```


### Algorithm

[LearnOpenGL](https://learnopengl.com/Advanced-Lighting/Normal-Mapping) has an excellent explanation for the algorithm behind computing the tangent vector, make sure to read the "Tangent Space" section first.

Consider the following quad, where `v0`, `v1`, `v2`, and `v3` are each vertices:

```
v0 --------- v3
| \        / |
|   \    /   |
|     \/     |
|     /\     |
|   /    \   |
| /        \ |
v1 --------- v2
```

Here is a simple expansion of the [LearnOpenGL algorithm](https://github.com/JoeyDeVries/LearnOpenGL/blob/443f16db600ffb62ecfa244a8638819a39335952/src/5.advanced_lighting/4.normal_mapping/normal_mapping.cpp#L178-L193) for the first triangle of the quad `(v0, v1, v2)`:

```java
Vector3f edge1 = v1.xyz - v0.xyz;
Vector3f edge2 = v2.xyz - v0.xyz;
Vector2f deltaUV1 = v1.uv - v0.uv;
Vector2f deltaUV2 = v2.uv - v0.uv;  

float f = 1.0f / (deltaUV1.x * deltaUV2.y - deltaUV2.x * deltaUV1.y);

tangent.x = f * (deltaUV2.y * edge1.x - deltaUV1.y * edge2.x);
tangent.y = f * (deltaUV2.y * edge1.y - deltaUV1.y * edge2.y);
tangent.z = f * (deltaUV2.y * edge1.z - deltaUV1.y * edge2.z);

bitangent.x = f * (-deltaUV2.x * edge1.x + deltaUV1.x * edge2.x);
bitangent.y = f * (-deltaUV2.x * edge1.y + deltaUV1.x * edge2.y);
bitangent.z = f * (-deltaUV2.x * edge1.z + deltaUV1.x * edge2.z);
```

The ShadersMod implementation is essentially the same as above, however it first checks whether it is about to divide by zero when calculating `f` and if so, it simply sets `f` to `1`.

Then, these two vectors (`tangent` and `bitangent`) are normalized. When normalizing a vector, if a vector has a length of zero, it is left unchanged, to avoid dividing by zero. The bitangent computed here will be known as the *actual bitangent*.

Next, the algorithm computes takes the cross product `tangent × normal`, where `normal` refers to the *unpacked* surface normal computed previously. The result of this cross product is known as the *predicted bitangent*, because it is the value that the shader will compute to determine the bitangent.

If all is well, the *actual bitangent* should match the *predicted bitangent*. However, it is possible that the predicted bitangent will not match the actual bitangent. The underlying issue is that depending on the order of values in the cross product, the predicted bitangent can be pointing in the exact opposite direction of the actual bitangent.

In order to compensate for this, the algorithm computes the dot product of the actual bitangent and predicted bitangent vectors. This takes advantage of a useful property of the dot product: it is also equivalent to multiplying the magnitudes of the two vectors together, then multiplying those magnitudes with the cosine of the angle between the vectors. If the vectors are pointing in the exact same direction, then the angle will be zero, cosine will be 1, and the dot product will be positive. However, if the vectors are pointing in exact opposite directions, the angle between them will be 180 degrees, and the cosine of 180 degrees is -1, meaning that the dot product will be negative.

If the dot product is negative, the `w` value of tangent is set to `-1`. Otherwise, if it is zero or positive, the `w` value of tangent is set to `+1`. This means that shaders will always be able to recalculate a correct value of the actual bitangent, because simply multiplying the `x`, `y`, and `z` values of the predicted tangent with `w` will result in the actual bitangent.

### Notes

* The algorithm does not compute the tangent for the second triangle `(v0, v2, v3)`, it just uses this same tangent for all vertices of the quad. This seems to work just fine for the geometry in Minecraft.

## Center texture coordinate

TODO: Overview

### Declaration

```glsl
attribute vec2 mc_midTexCoord;
```


### Algorithm

For each quad, the center texture coordinate is just the average of the texture coordinate for each vertex. That is, `mc_midTexCoord.x` is the average of all of the `x` / `u` values, and `mc_midTexCoord.y` is the average of all of the `y` / `v` values. It's as simple as that!


## Block identification

ShadersMod allows you to identify blocks based on their block id. This can be used to render certain blocks differently or give effects to specific blocks.

### Declaration

```glsl
attribute vec2 mc_Entity;
```

### Configuration

To define a block id group, use `block.n=<id>` where `n` is the group's id, and `<id>` is the space-separated list of block id's you want to add to the group.

```properties
block.1=grass_block dirt coarse_dirt gravel sand
block.2=water flowing_water lava flowing_lava # Fluids
```

Additionally, you can specify the namespace of the block and define blocks added by other mods.

```properties
# Both birch_leaves and minecraft:birch_leaves work but custom_leaves doesn't!
block.3=oak_leaves minecraft:birch_leaves modid:custom_leaves
```

### Example

```glsl
attribute vec2 mc_Entity;

void main() {
  switch(mc_Entity.x) {
    case 1: // if rendered block is grass, dirt, etc.
      // do something
    case 2: // if rendered block is a fluid
      // do something
    default:
      // do something else
  }
}
```
