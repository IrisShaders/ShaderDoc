# Improving your shader

## Border Fog

### The vertex shader

During the switch to your shader, you lost a minor, yet critical graphical feature: Border fog, the fog that hides chunk loading. Let's add it back.

First, we will need the distance between the vertex and the player. To do this, let's go back to our **vertex shader** and define a new output.

```glsl
out float vertexDistance;
```

Now, let's go to our main program and fill in the data.

```glsl
void main() {
// previous stuff
vertexDistance = length((gl_ModelViewMatrix * gl_Vertex).xyz);
}
```

### The Fragment shader

Let's do the corresponding in declaration;

```glsl
in float vertexDistance;
```

Now, we need to use some ***uniforms***. What are those?

### Uniforms

Uniforms are values that are static across a given draw, provided by Iris. Once defined, they can be used in the vertex or fragment shader.

A full list can be found in `uniforms.md` on this repo, but for now, we're going to use these.

```glsl
uniform float fogStart;
uniform float fogEnd;
uniform vec3 fogColor;
```

Once we have those uniforms, let's go back and calculate our fog.

### Back to the Fragment shader (again)

Let's plug in some basic fog code.

```glsl
void main() {
    vec4 texColor = texture(gtexture, texCoord);
    if (texColor.a < 0.1) discard;
    vec4 lightColor = texture(lightmap, lightCoord);
    
    // Calculate our new fog color!
    float fogValue = vertexDistance < fogEnd ? smoothstep(fogStart, fogEnd, vertexDistance) : 1.0;

    vec4 finalColor = texColor * lightColor * vertexColor;
    pixelColor = vec4(mix(finalColor.xyz, fogColor, fogValue), finalColor.a);
}
```

Note the use of the `mix` function, which mixes the first and second color based on the third value.

Once you reload, you should now see you have border fog!
