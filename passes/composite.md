# Composite passes

TODO: This documentation page is incomplete and needs additional detail.

Composite passes form the core of any deferred rendering pipeline, and even in the context of forward-rendering pipelines are critical for almost all postprocessing effects.

A composite pass enables a shader pack developer to render to a specific region of one or more render targets using the content of existing render targets as inputs, as well as any of the currently-available uniforms.

## A trivial example: Passthrough

The simplest possible composite pass is one that simply reads from one buffer's render target, and writes the same value to the other render target in that buffer.


### Fragment shader

```glsl
#version 150 compatibility

uniform sampler2D colortex0;

in vec2 texcoord;

void main() {
    /* DRAWBUFFERS:5 */
    gl_FragData[0] = texture(colortex0, texcoord);
}
```


### Vertex shader

```glsl
#version 150 compatibility

out vec2 texcoord;

void main() {
	gl_Position = ftransform();
	texcoord = (gl_TextureMatrix[0] * gl_MultiTexCoord0).xy;
}
```


## Components of a composite pass

A composite pass contains the following elements:

- A vertex (`.vsh`) and fragment (`.fsh`) shader
- (Optional) a geometry (`.gsh`) shader
- A list of render targets and custom textures to read from (`uniform sampler2D <...>;`)
- A list of render targets to write to (`gl_FragData[...] = <...>;`)
- A viewport (region) of the output render targets to write to
    - This is controlled by the `scale.<program>` key in `shaders.properties`. Note that OptiFine and Iris do not currently allow adjusting the offset of the viewport, so it is always rooted at the corner of the render targets.
