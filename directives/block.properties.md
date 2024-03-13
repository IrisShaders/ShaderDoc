# `block.properties`

## Implementation Support

* ❌ ShadersMod
* ✔️ OptiFine
* ✔️ Iris

## Block Rendering Options

### Render Layer Changes

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
# Change Stone To A Translucent Block
layer.translucent=stone
```

#### Default

No change. If this option is not present then the blocks render in the way that mojang has chosen.

#### Implementation Support

* ❌ ShadersMod
* ✔️ OptiFine
* ✔️ Iris

Note: Optifine Only Has Partial Support
