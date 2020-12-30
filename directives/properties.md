# `shaders.properties`

## Implementation Support

* ❌ ShadersMod
* ✔️ OptiFine
* ❌ Iris

## Rendering details toggles

### Clouds mode

Allows shader packs to control the cloud rendering.

#### Representation

```java
enum CloudsMode {
	fast,
	fancy,
	off
}

CloudsMode clouds;
```

#### Example

```
# Disable clouds with this shader pack
clouds=fancy
```

#### Default

No change. If this option is not present then the clouds render in the way that the user has chosen.


## Backwards compatibility

### Old Hand Lighting

Allows old pre-1.9 shader packs to explicitly opt in to the old hand lighting mode. See the documentation for ["Held block light value" in Uniforms](../uniforms.md#held-block-light-value):

> If the "Old Hand Lighting" option is set, and the light value of the block in the off hand is higher than the light value of the block in the main hand, then this uniform takes on the value of `heldBlockLightValue2`. This is to support older (pre-1.9 or non-OptiFine) shader packs that do not support the off hand.

Users may override this setting in the shader configuration menu.

#### Representation

```java
boolean oldHandLight;
```


#### Example

```
# Enable old hand lighting since this is a legacy 1.8.x shaderpack
oldHandLight=true
```


#### Default

TODO


## Mod compatibility

### Dynamic Hand Lighting

Allows a shader pack to control whether dynamic hand lighting mods are enabled or disabled.

With OptiFine, this controls its Dynamic Hand Lighting feature. However, for Iris it would be good to have LambDynamicLights integration so that shader packs can disable 

#### Representation

```java
boolean dynamicHandLight;
```


#### Example

```
# Since this shader pack 
dynamicHandLight=false
```


#### Default

TODO
