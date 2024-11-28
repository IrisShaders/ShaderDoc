# Making your shader look cool

## Directional lighting

Minecraft already has some directional lighting, but we can amplify it using a simple NdotL check.

In the process, we will restore some lighting to entities and such.

### Enhanced directional lighting

To pull this off, we will need two things: the view-space normals, and the normalized sun position. These are very easy to obtain.

In your vertex shader, you will need to add a new `out vec3 normal`, and initialize it in main with `normal = gl_NormalMatrix * gl_Normal`.

Add the corresponding `in` statement to the fragment shader, and now do this.

```glsl
// Add some new uniforms and your in statement
in vec3 normal;

uniform vec3 shadowLightPosition;

void main() {
    // all code before mixing fog
    
    // 0.2 will be the *minimum* light for objects facing away from the sun. You can mess with this value.
    // The sun/moon position must be normalized.
    float NdotL = max(0.2, dot(normal, normalize(shadowLightPosition)));
    
    finalColor.rgb *= NdotL;
}
```

Now you can see much more pronounced directional lighting. You can change `0.2` to a higher number to make the world overall brighter.

### Adding new programs

Let's make a copy of your `gbuffers_terrain` program and name it `gbuffers_textured`. This program will have all code for non-terrain.

Congrats! You now have cool shading on entities.

However, take a look at the sky. You may see the sun is a bit... wider than usual. This is because the vertex position for the sky is nonsense.

To fix this, we will need to make another copy `gbuffers_skytextured`, and remove the fog calculation. (Don't forget to remove directional lighting too; you don't want the sun shading itself!)
