# Iris Features

### All the following features are exclusive to Iris.

## Table of Contents

1. [Entity shadow distance multiplier](#entity-shadow-distance-multiplier)

## Entity shadow distance multiplier (Iris 1.2.1)

This value controls the bounds for entity shadows to be rendered. By default, it is the value set for terrain. Any floating point number that is not 1 is multiplied by the terrain distance to get the final shadow distance multiplier.

### Declaration

```glsl
const float entityShadowDistanceMul = 1.0f;
```

### Value Range

* Minimum Value: 0.01
* Default Value: Terrain value
* Out-of-range values behavior: Disable

### Valid Declaration Locations

* ❌ Vertex Shader (*.vsh)
* ❌ Geometry Shader (*.gsh)
* ✔️ Fragment Shader (*.fsh)


## playerShadow directive (Iris 1.2.5)

This value controls if the player should have a shadow rendered. This is forced on if entityShadow (default true) is enabled.
This also controls shadows of any entities the player is riding.

### Declaration

```
playerShadow = true
```

### Valid Declaration Locations

* `shaders.properties`