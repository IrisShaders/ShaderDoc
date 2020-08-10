# Uniforms

ShadersMod & OptiFine define many uniforms that allow shaders to access the current state of the world, the player, and the renderer.

## Player

### Held item ID

Item ID of the item currently held in the main hand. 

For OptiFine, the item ID is defined by the mapping in `items.properties`.

> TODO
> * OptiFine: What about when nothing is held? (probably air)
> * OptiFine: What about when the currently held item isn't defined in `items.properties`?

For ShadersMod, which does not support `item.properties`, the ID is defined by the Anvil (pre-1.13 world format) ID mapping of the currently held item. If the player is currently not holding an item, an ID of `-1` is provided. For example, if the player was holding a stone block, the value would be `1`.

#### Declaration

```glsl
uniform int heldItemId;
```

#### Implementation Support

* ✔️ ShadersMod
* ✔️ OptiFine

