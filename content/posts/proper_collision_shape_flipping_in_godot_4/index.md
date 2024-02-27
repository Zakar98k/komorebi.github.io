+++
title = 'Proper Collision Shape Flipping in Godot 4'
date = 2024-02-27T10:42:13+08:00
draft = true
tags = ["godot","game-development"]
categories = ["tutorial", "youtube-video"]
+++

I made a Youtube Tutorial on how to implement a *Hitbox Flipping* mechanic in Godot. See embedded video below:

{{< youtube id="lHC3YqYB9mI" title="Proper Collision Shape Flipping in Godot 4 | Godot Intermediate Level Tutorial" >}}

A GitHub repository for this project is also available:
{{< github repo="Zakar98k/godot-hitbox-flipping" >}}

## Implementation
I used a **composition-based** approach to implement the Hitbox Flipping. By utilizing composition, we're able to define scenes called **"components"**, that we will later be able to reuse in other scenes: the player scene, any enemy scene, or even traps that need to be flipped.

> To implement *Hitbox Flipping*, we need to define 3 new scenes, each with one node only.

### Flippable Sprite Component
> The job of this component is to notify any [Flippable Collision Shapes]({{% relref "index.md#flippable-collision-shape-component" %}})
whenever the sprite is flipped, so that the Flippable Collision Shapes can update their positions and flip as well.

Make a new scene with the root node being a `Sprite2D`. Rename the `Sprite2D` to `FlippableSprite`. Save the scene anywhere as `flippable_sprite.tscn`

Attach a script to your newly renamed `FlippableSprite` node. Save it anywhere as `flippable_sprite.gd`
```gdscript
# flippable_sprite.gd

extends Sprite2D
class_name FlippableSprite

signal sprite_flipped(flip_value)

var flipped: bool = false:
	set(new_value):
		if new_value != flipped:
			flipped = new_value
			flip_h = flipped
			sprite_flipped.emit(new_value)
```

In the script, we define a new signal called `sprite_flipped(flip_value)`. This signal will be emitted in our setter function, `set(new_value)`, whenever the value of our variable `flipped` is set. (you can read more about Godot 4 setters and getters [here](https://shaggydev.com/2022/01/04/setget/))

In our setter function, we set the value of the variable `flipped` to the new flip value, and we also set the value of `Sprite2D`'s property `flip_h` to the new flip value

### Flippable Collision Shape Component
> This component will listen to the signal `sprite_flipped(flipped_value)` that we defined earlier in [Flippable Sprite Component]({{% relref "index.md#flippable-sprite-component" %}})

Make a new scene with the root node being a `CollisionShape2D`. Rename the `CollisionShape2D` to `FlippableShape`. Save the scene anywhere as `flippable_shape.tscn`.

> The `FlippableShape` node will throw 2 errors. Ignore them, as we'll resolve these errors later on by initializing this scene as a child of a parent HitboxComponent scene and giving it a shape.

Attach a script to your newly renamed `FlippableShape` node. Save it anywhere as `flippable_shape.gd`

```gdscript
# flippable_shape.gd

extends CollisionShape2D
class_name FlippableShape

@export var default_position: Vector2:
	set(new_position):
		default_position = new_position
		position = new_position

var current_flip_value: bool

func _on_sprite_flipped(flip_value):
	if current_flip_value != flip_value:
		default_position.x *= -1
		current_flip_value = flip_value
```