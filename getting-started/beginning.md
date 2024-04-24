# Getting Started with Shaders

## What is a shader?

So; you want to make a shader. What is a shader pack?

A shader pack is a zip file containing many shader "programs". In Optifine, these programs have unique names describing when they are used.

So, what are these programs...?

A shader program is essentially code that runs on the GPU. A "vertex shader" is responsible for taking world position information, and convering it into a position on screen. (These shaders can do much more, but it's not important right now.)

The fragment shader is what we'll be focusing on. It's responsible for taking texture and location information on a per-pixel basis, and using it to determine the color of the pixel.
In essence, it's what the "shader" part of a shader is.

### "Wow, I made a shader!"
To get started, let's first go into our `shaderpacks` folder and make a new folder with your shader's name. Inside that folder, make a new folder called `shaders`.

Also, while you're here, let's turn on debug mode. This won't be helpful right now, but will be later.

**To turn on debug mode, simply press Control and D while in the shader list. You should get a confirmation.**

This is all you need to get a loadable shader. Now, let's go into our game, and load up the shader pack!

Except... why doesn't it look quite right? Sure, you have the game, but everything looks just a little... off.

### "Wow, I ~~didn't~~ make a shader!"

So. You loaded a shader pack. However, things don't look quite right.

Take a look at a set of blocks quickly. Sure, it has some lighting, but a lot of what makes the game pop out has suddenly... disappeared.
This isn't a mistake; you've essentially deleted the code responsible for this, and now have to make it yourself.

However, don't worry, *it's very simple*! Simply get your shader properties, turn on old lighting; then get your fragment shader, and combine the ambient occlusion and lightmap!

...What? You didn't get that?

## Taking a step back

So, in order to write shaders, you actually need to learn how to code.
Shader coding involves a lot of algebra, as well as concepts universal to coding like variables and functions.

GLSL has a few unique functions like swizzling and vector math, and I won't be going over them here, rather linking helpful interactive tutorials.

The first great resource is **[The Book of Shaders](https://thebookofshaders.com/)**. Not all concepts here translate to Minecraft, notably the uniform components, however it gives a basic understanding of GLSL.

Once you're confident with that, you can test your skills in **[Hugh Kennedy's interactive course](https://hughsk.io/fragment-foundry/chapters/01-hello-world.html)**.

You got it? Cool.

These guides should have given you a basic understanding of how GLSL functions. However, each usage of it is slightly different, such as the one used in Minecraft.

## Setting up your workspace

Now, assuming you're working in Notepad or something, you want to get off that as soon as possible.

What IDE people choose is usually pretty subjective, however many people have chosen [Visual Studio Code](https://code.visualstudio.com/) as their favorite.

I won't get into how to install it here, but once you have that, you want to ***immediately*** go to the plugins section and download [GLSL Syntax for VS Code
](https://marketplace.visualstudio.com/items?itemName=GeForceLegend.vscode-glsl) to make it actually useful. Now you can go ahead and open your shader folder up in it.