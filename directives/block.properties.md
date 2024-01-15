# `block.properties`

## Implementation Support

* ❌ ShadersMod
* ✔️ OptiFine
* ✔️ Iris

## Block Rendering Options

### MC_Entity

### Layer Changes

Layer Changes allow you to change what layer a block renders on.

#### Representation

```java
layer.solid=<blocks>
layer.cutout=<blocks>
layer.cutout_mipped=<blocks>
layer.translucent=<blocks>
```

#### Example

```
# Disable clouds with this shader pack
clouds=fancy
```

#### Default

No change. If this option is not present then the clouds render in the way that the user has chosen.

#### Implementation Support

* ❌ ShadersMod
* ✔️ OptiFine
* ✔️ Iris

Note: Optifine Only Has Partial Support
