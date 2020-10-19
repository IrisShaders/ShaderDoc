# Vertex Format Extensions

ShadersMod extends the vertex format of Minecraft in order to enable additional features. It adds the following attributes to the vertex format:

* Vertex normals
* Tangent vector
* "MidUV" vectors
* Block state information (vec4)

## Vertex normals

For each quad, ShadersMod for 1.12.2 computes the surface normal, and then attaches that computed surface normal to each vertex of said quad.

Modern versions of Minecraft (such as 1.16.3) appear to include already-generated vertex normals in the vertex formats used for blocks and items, so using the below algorithm is not necessary for rendering vanilla blocks and items. In fact, it takes advantage of the block model format in order to simplify the vertex normal calculations.

However, it may still be necessary for things like entity rendering. This needs to be investigated.

### Algorithm

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

To compute the surface normal vector, the algorithm first computes the vector between `v0` and `v2` as well as the vector between `v1` and `v3`. Then, it computes the cross product of those two vectors, and then normalizes that cross product. This normalized vector is the *unpacked* surface normal.

The surface normal is then packed into a 32-bit integer by first scaling the unpacked surface normal vector by `127.0`, thus ensuring that x, y, and z of the vector all lie in the range [-127.0, 127.0]. Then, each element of the vector is converted into a signed byte using [Java conversion rules](https://docs.oracle.com/javase/specs/jls/se8/html/jls-5.html#jls-5.1.3). Finally, each of these bytes is then packed into the integer like so:

```
i << 32   i << 24   i << 16   i << 8    i
+---------+---------+---------+---------+
| zero    | z coord | y coord | x coord |
+---------+---------+---------+---------+
```

For example, the normalized vector (-0.75, 0.25, 0.612) would result in a packed vector of `0x004D1FA1`. Note that the most significant byte, a padding byte, will always be zero here.


### Notes

* These vertices are arranged in counterclockwise order, however, the algorithm will work for vertices arranged in clockwise order as well. In practice, Minecraft uses a counterclockwise winding order, because that is the default of OpenGL.
* It is possible for the cross product to be zero-length, so **ensure that you are not dividing by zero** in your implementation! You will notice that these two vectors form an "X" shape, and they are at 90 degree angles to each other, so for any quad with an area greater than zero, the cross product will be greater than zero as well. However, it is possible that `v0 == v1 == v2 == v3`, which is why this special check exists.
