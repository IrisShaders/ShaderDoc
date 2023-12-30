# Iris Features

### These are features either currently not added or not planned to be added to Iris.

# Entity motion vectors

## Status
- Not implemented, requires further design changes

## Information

Entity motion vectors are a tricky issue. In a vanilla game, each entity's moving part is stored as a ModelPart dealing with rendering that model. However, this model part is shared between entities, and as such you cannot just store velocity in it.
There are further issues, however preliminary testing has been done in a separate branch.

# World instancing / GL_ARB_geometry_shader4 support

## Status
- Currently unsupported, no plans

## Information

Mostly obsolete. This feature would tell the shader loader to duplicate the vertices used in any render using that specific program by a specific amount.
GL_ARB_geometry_shader4 is a legacy version of core geometry shaders.

# Boss Battle uniform

## Status
- Implemented poorly in Optifine, no intentions to replicate in Iris