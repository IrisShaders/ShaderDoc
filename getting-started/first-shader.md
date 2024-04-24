# Your first code

## Making the basics

Assuming you've followed Part 1 already, you should have your IDE of choice open with your shaders folder. Now, let's make a shader.

We'll start with making 2 files (make sure these files are in the shaders folder *inside* the main folder!):
- `gbuffers_terrain.fsh`
- `gbuffers_terrain.vsh`

`fsh` and `vsh` denote the Fragment and Vertex shaders respectively. Let's do the vertex shader first.

## The Vertex Shader

We won't get too into this right now, but we do need it, so let's open it up.

**Every GLSL shader must start with a version directive.**

For this guide, we will be using `#version 330 compatibility`, so put that as the first line in your shader file.

Note the *compatibility* line, this indicates we will be using the older compatibility profile instead of the newer core profile, which has some changes. This is due to the core profile not being well supported at this time.

Now, let's fill in the placeholders.

```glsl
#version 330 compatibility

out vec2 texCoord;
out vec2 lightCoord;
out vec4 vertexColor;

void main() {
    gl_Position = gl_ModelViewProjectionMatrix * gl_Vertex;
    texCoord = gl_MultiTexCoord0.xy;
    lightCoord = (gl_TextureMatrix[1] * gl_MultiTexCoord1).xy;
    vertexColor = gl_Color;
}
```

This will be our basic vertex shader for now, but let's go over what it does.

```glsl
out vec2 texCoord;
```

This notes that we will be sending a variable **out** of the vertex shader and **into** the fragment shader. This is a one way process.

```glsl
gl_Position = gl_ModelViewProjectionMatrix * gl_Vertex;
```
This line is the most critical; it changes where the terrain is relative to the screen. It does that by multiplying the modelview and projection matrices with the world space position.

These matrices will be critical later on, but for now let's ignore it.

```glsl
texCoord = gl_MultiTexCoord0.xy;
lightCoord = (gl_TextureMatrix[1] * gl_MultiTexCoord1).xy;
vertexColor = gl_Color;
```

These fill in the texture coordinates and light map coordinates using info Iris provides, as well as the vertex color. These are pretty much standard to all shaders.

Cool, we have our vertex shader, and in only 11 lines! Let's move on to the fragment shader.

## Your first modification

Time to make the fragment shader! Let's open it up and do a similar placeholder.

```glsl
#version 330 compatibility

in vec2 texCoord;
in vec2 lightCoord;
in vec4 vertexColor;

layout(location = 0) out vec4 pixelColor;

void main() {
    pixelColor = vec4(1, 0, 0, 1);
}
```

Assuming you followed all those guides, this should start looking pretty familiar.

Now, we can go back into the game, and press R to reload the shader!

Sorry for blinding you, but you've just made your first shader edit. This turns all terrain red, due to the change in pixel color.

## Texturing

Okay, that's cool; but we want something more than just red. We want our blocks.

Let's go back to our fragment shader, and define some *textures*.

```glsl
#version 330 compatibility

in vec2 texCoord;
in vec2 lightCoord;
in vec4 vertexColor;

// Our new textures!
uniform sampler2D gtexture;
uniform sampler2D lightmap;

layout(location = 0) out vec4 pixelColor;

void main() {
    pixelColor = texture(gtexture, texCoord);
}
```

We have now defined `gtexture`, the standard ground texture name in Iris. We can now sample this texture using the `texCoord` we passed in earlier.

Reload the shader again... and we can see the blocks! It's even more flat now.

Also, weirdly, the colors for leaves and water seem to have disappeared... but that's fine; we'll fix it later.

And by later, I mean right now.

## Adding some dimensionality

So, remember that vertex color attribute? First, let's use that.

```glsl
    pixelColor = texture(gtexture, texCoord) * vertexColor;
```

You will *immediately* notice a drastic difference. Along with leaves and water becoming colored again, everything has gained some dimensionality.

This isn't a mistake; the vertex color also contains the "ambient occlusion" of the blocks; the dark shadows around corners.

We still don't have lighting, so let's fix that.

```glsl
    vec4 lightColor = texture(lightmap, lightCoord);
    pixelColor = texture(gtexture, texCoord) * lightColor * vertexColor;
```

And now we have lighting! Things still look a little bright, though.

This is due to Minecraft's directional lighting being disabled by default when shaders are on. We *can* recreate this manually, but we can also just tell Iris to turn it back on.

To do this, make a new file called `shaders.properties` in the shaders folder and put this.

```
oldLighting = true
```

Reload again, and we now have our directional lighting.

## Alpha testing

Now, it may not be immediately obvious, so go look at some trees and grass. They're no longer transparent.

This is due to the shader not telling the GPU that they should be! Let's change our code a bit to handle this.

```glsl
    vec4 texColor = texture(gtexture, texCoord);
    if (texColor.a < 0.1) discard;
    vec4 lightColor = texture(lightmap, lightCoord);
    pixelColor = texColor * lightColor * vertexColor;
```

We've simply moved our texture calculation higher, and are checking if the alpha value of this pixel is less than 0.1; if it is, we tell the GPU to not render it.

Reload again, and we've fixed it!

We're now looking pretty close to how Vanilla looks, so it's time to start actually changing stuff.