# Compute Passes

Compute passes are substantially more flexible than [Composite](composite.md) passes, at the cost of additional complexity and performance overhead.

Compute passes enable implementing new approaches to rendering problems that can be faster than just using a composite pass, but trivially porting a composite pass to a compute shader probably won't lead to a performance improvement - in fact, if you're not clever about it, it could reduce performance!

A single composite pass can have up to 27 compute passes attached. For a given composite fragment shader named `composite1.fsh`, valid compute shader names are `composite1.csh`, `composite1_a.csh`, all through `composite1_z.csh`.

All compute passes attached to a given composite pass are dispatched before that composite pass.


## Platform support

Compute shaders require either OpenGL 4.3 or the [`GL_ARB_compute_shader`](http://www.opengl.org/registry/specs/ARB/compute_shader.txt) extension.

This requires either a GLSL 4.3 declaration:

```
#version 430
```

Or, enabling the ARB extension:

```
#extension GL_ARB_compute_shader enable
```

Unlike fragment and vertex shaders, the compatibility profile is not relevant to compute shaders and does not need to be enabled. The shader implementation SHOULD ignore any attempt to enable the compatibility profile on a compute shader, and MAY log a warning.

While this is available on most modern NVIDIA, AMD, and Intel hardware, due to Apple refusing to support OpenGL properly, compute shaders are not available on any version of macOS.

## Implementation Support

* ❌ ShadersMod
* ✔️ OptiFine
    - Available as of OptiFine HD U G8 for Minecraft 1.16.5, released May 15th 2021
* ✔️ Iris
    - Available as of Iris 1.4.0, released October 9th 2022

## Compute Space: Controlling how many times a compute shader executes

For a composite pass, it's overall fairly straightforward how many times the fragment shader runs: once per pixel, [except in certain edge cases along the diagonal line of the screen quad](https://wallisc.github.io/rendering/2021/04/18/Fullscreen-Pass.html). This is because a fragment shader is operating in the 2D space of an output viewport targeting a given framebuffer.

However, for a compute pass, things get a whole lot more complex. Compute shaders don't have the same rigid restrictions of an output framebuffer, so there's nothing that implicitly defines the space in which a compute shader runs, and thefore the number of times it executes.

While the [OpenGL Wiki](https://www.khronos.org/opengl/wiki/Compute_Shader#Compute_space) documents this topic in more detail, a summary is still relevant here. Essentially, compute space is formed by a number of work groups, and each work group itself has a number of local invocations. Local invocations are still executed in parallel with each other as well as in parallel with local invocations of other work groups, but local invocations within a single work group are able to communicate through special functions and variables, such as variables defined with the `shared` keyword. On the other hand, local invocations cannot communicate through work groups.

Defining a smaller number of local invocations and a larger number of work groups gives the driver more flexibility in executing the compute shader. This presents a cruical trade-off for a shader pack developer: there might be opportunities to implement more efficient and concurrent algorithms with a larger number of local invocations per work group (due to the ability to share data), but less-capable hardware might have to break up a work group and execute parts of that work group sequentially, rather than being able to execute all invocations in parallel.

Overall, this is one of the few areas where the underlying GPU microarchitecture shines through directly to the shader pack developer. Be prepared to do some digging into relevant architecture documentation if you're optimizing compute shaders!

### Declaration

In both cases, the number of local invocations and the number of work groups are defined as a 3-dimensional (X, Y, Z) quantity. This is **purely for developer convenience** and ultimately the number of actual work groups / local invocations is equal to the product of X, Y, and Z. So declaring a work group size of (8, 1, 1) has no performance difference to (1, 1, 8), and similarly has no performance difference to (2, 2, 2). Define the space in the way that makes your code the cleanest!


#### Declaring the local size

```glsl
layout(local_size_x = X, local_size_y = Y, local_size_z = Z) in;
```

Note that any omitted components default to 1, so the following is valid:

```glsl
layout(local_size_x = 3) in;
```

Which is equivalent to:

```glsl
layout(local_size_x = 3, local_size_y = 1, local_size_z = 1) in;
```

See the [OpenGL Wiki](https://www.khronos.org/opengl/wiki/Compute_Shader#Local_size) for more details.

#### Declaring the number of work groups

In OpenGL, the number of work groups is decided by the applicating issuing OpenGL commands. OptiFine and Iris allow the shader pack to control this using const directives in the compute shader source code.

There are two different methods to declare the number of work groups:

1. Constant / absolute:
    ```glsl
    const ivec3 workGroups = ivec3(1, 8, 1);
    ```
    
    In this case, the number of work groups is passed directly to OpenGL without modification.
2. Relative to screen size:
    ```glsl
    const vec2 workGroupsRender = vec2(1.0f, 0.5f);
    ```
    In this case, the number of work groups is calculated such that the total number of invocations of this compute shader is equal to the number of pixels that would result from scaling the X and Y axis of the screen by the given scale factors. Since this determines the total number of invocations, the calculation will also be adjusted by the localSize declaration. Note: If localSize.z > 1, then the total number of invocations is equal to the number of pixels times localSize.z.
    
    For example, for the following shader, running in a 1920x1080 screen:
    ```glsl
    const vec2 workGroupsRender = vec2(1.0f, 0.732f);
    layout(local_size_x = 19, local_size_y = 1, local_size_z = 5) in;
    ```
    
    First, the equivalent integral screen width is computed, rounding up fractional values:
    ```glsl
    scaledWidth = ciel(vec2(1.0f, 0.732f) * vec2(1920, 1080)) = ciel(vec2(1920.0, 790.56)) = ivec2(1920, 791)
    ```
    
    Then, the number of work groups is calculated by dividing and then rounding up:
    ```glsl
    workGroups = ivec3(ciel(scaledWidth / vec2(19.0, 1.0)), 1) = ivec3(ciel(101.05, 791.0), 1) = (102, 791, 1)
    ```

If the declaration is omitted, by default the following is inferred, meaning that the number of compute shader invocations will equal the number of pixels on the screen times localSize.z:

```glsl
const vec2 workGroupsRender = vec2(1.0f, 1.0f);
```


## Concurrency between compute passes

After all compute passes corresponding to a given composite pass have been dispatched, a single `glMemoryBarrier(GL_TEXTURE_FETCH_BARRIER_BIT | GL_SHADER_IMAGE_ACCESS_BARRIER_BIT)` command is issued **both before and after the compute pass**. This ensures that all `texture()` calls and all `imageLoad()` / `imageStore()` calls receive up-to-date values following the execution of all compute passes for a given composite pass, as opposed to out-of-date or undefined values.

NOTE: Notably, the `GL_SHADER_STORAGE_BARRIER_BIT` and `GL_FRAMEBUFFER_BARRIER_BIT` flags are omitted in this call. While omitting `GL_SHADER_STORAGE_BARRIER_BIT` is valid as no release version of Iris or OptiFine supports shader storage buffer objects in any context, it is unclear whether it is valid to omit `GL_FRAMEBUFFER_BARRIER_BIT`. TODO: More investigation is needed here.

There are two important corrolaries of this memory barrier behavior:

### The Sequential Dispatch Corrolary

A compute shader belonging to composite pass A is always executed before a compute shader belonging to composite pass B, if composite pass B is executed after composite pass A.

Sequential dispatch between two given compute passes is required if one of the given compute passes is dependent on the output of a the other given compute pass.

It is possible to define a compute pass without the corresponding composite pass being defined. As a result, it is possible to have both `composite.csh` and `composite1.csh` be defined without either `composite.fsh`/`composite.vsh` or `composite1.fsh`/`composite1.vsh` being defined. This allows exploiting the sequential dispatch corrolary without being restricted by the number of composite passes that the shader pack is utilizing, or the structuring of those composite passes relative to the compute passes.


### The Parallel Dispatch Corrolary

A compute shader belonging to a given composite pass MAY be executed at the same time as another compute pass belonging to the same composite pass.

Parallel dispatch can lead to significantly increased performance because it allows the GPU to execute multiple compute shaders at the same time.

However, care must be taken with parallel dispatch because you are **bypassing the conventional safety guarantees of composite passes**.

As a result, if two parallel compute passes attempt to access the same image without using atomic operations, inconsistent behavior MAY result. This is highly dependent on drivers, FPS, operating system, hardware, and all sorts of other factors.

There are two ways to mitigate this issue:

1. Elminate resource sharing: Ensure that no resources are used at the same time by two compute passes.
2. Atomic-only resource sharing: Only utilize atomic operations to access a shared resource.


## Further reading

- An in-depth pair of blog posts touching on using advanced concurrency techniques in compute shaders specifically in the context of Minecraft path-tracing shaders has been published by BruceKnowsHow:
    - [Good and Bad Conditional Branching in GPU Shaders](https://lavish-waitress-2da.notion.site/Good-and-Bad-Conditional-Branching-in-GPU-Shaders-17cca16e30be4e38baef9c9166ac4c24)
    - [Branchless Multi-Bounce Path-Tracing with a Ray-Buffer](https://lavish-waitress-2da.notion.site/Branchless-Multi-Bounce-Path-Tracing-with-a-Ray-Buffer-c9cd87ff41bd455b8dd8d629011167b0)
