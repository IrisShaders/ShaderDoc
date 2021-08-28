# ShaderDoc

The ShaderDoc project seeks to provide an independent source of documentation for the ShadersMod / OptiFine / Iris shader pack format. Though it does not completely document the format, it seeks to provide a high level of detail in the areas that it does document.

There are two purposes of this project:

1. Providing high-quality documentation for shader pack developers seeking to develop shader packs for Iris.
2. Providing a relatively unambiguous description of critical yet complex elements of the shader format that are otherwise only briefly (or incorrectly!) described in the official OptiFine documentation, in order to facilitate implementing these elements in Iris without infringing copyright.

## Gathering information for ShaderDoc

There are a few ways to gather information in order to add new documentation to ShaderDoc:

1. Using the OptiFine documentation as a reference. Please note that the [official OptiFine documentation for shaders](https://github.com/sp614x/optifine/blob/master/OptiFineDoc/doc/shaders.txt) is unlicensed, making it All Rights Reserved. This means that it is forbidden to directly copy text from the OptiFine documentation to ShaderDoc. However, it is permitted to read the OptiFine documentation and then use it to write a description of a topic in your own words. Note that the OptiFine documentation is not necessarily a source of truth - there are many places where it is incorrect. Use with some caution.
2. Other third-party documentation. Shader pack developers have also created their own resources to document the shaders format. Similar rules and guidelines to working with OptiFine documentation apply.
3. Experience working with the shader pack format. Working with the shader pack format under OptiFine is a decent way to gather general knowledge about the format.
4. Reverse engineering. Though somewhat risky, this method can be the most accurate way to find or confirm information. The source code of ShadersMod v2.7.0 for Minecraft 1.12 is available here: http://www.karyonix.net/shadersmod/files/smc-2.7.0-mc1.12-src.7z. Decompiling OptiFine should be avoided where possible.

## Contributing to ShaderDoc

If you would like to contribute to ShaderDoc, feel free to open a pull request. Note that PRs may take a long time to be merged, as they are only merged once they've been fully verified, and whenever I have time to actually look at them.

## License

[LGPLv3 or later](LICENSE)
