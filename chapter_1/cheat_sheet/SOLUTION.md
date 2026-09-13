# Chapter 1 Solution: Walk Around Home

[Course overview](../../OVERVIEW.md) | [Back to the lesson](../LESSON.md)

This reference reconstructs the complete Chapter 1 result in Godot **4.7.2 stable**, using Godot 4 GDScript. It uses generated textures, no imported assets, and no wall scripts. Compare one checkpoint at a time rather than replacing work you already understand.

## Project Settings

Create or open the empty **Garden Below the Sky** project with **Compatibility** selected. This renderer and the presentation settings below are provisional. `res://` is your project folder; create its `scenes` folder through the FileSystem dock or save dialog.

Open Project > Project Settings; enable Advanced Settings to reveal hidden fields. Use the search box for these property names.

| Setting | Value |
| --- | --- |
| Application > Config > Name | `Garden Below the Sky` |
| Application > Run > Main Scene | `res://scenes/home.tscn` |
| Display > Window > Size > Viewport Width / Height | `640` / `360` |
| Display > Window > Size > Window Width / Height Override | `1280` / `720` |
| Display > Window > Stretch > Mode | `viewport` |
| Display > Window > Stretch > Aspect | `keep` |
| Display > Window > Stretch > Scale Mode | `integer` |
| Rendering > Textures > Canvas Textures > Default Texture Filter | `Nearest` |
| Rendering > Environment > Defaults > Default Clear Color | `#202020`, alpha `1.0` |

Save Home before assigning it as the main scene. F5 also lets you choose Home when no main scene is set. An embedded preview can fit its panel rather than show the exact requested desktop window size. No Camera2D is needed.

## Input Map

In Project > Project Settings > Input Map, type each action name and click Add. Use the plus button on each action, choose a keyboard event, activate listening/detection, and press its key. Enable Physical, confirm, and check the listed event is physical. Leave deadzones at `0.5`.

| Action | Physical key |
| --- | --- |
| `move_left` | A |
| `move_right` | D |
| `move_up` | W |
| `move_down` | S |

## Player Scene

Create a scene with CharacterBody2D as its root, name it Player, and save it as `res://scenes/player.tscn`. Add the two children below directly under Player.

```text
Player (CharacterBody2D; script: res://scenes/player.gd)
|-- Sprite2D
`-- CollisionShape2D
```

| Node or resource | Exact setup |
| --- | --- |
| Player | Position `(0, 0)`; Motion Mode **Floating**; exported Speed `120.0` |
| Player collision | Layer `1` only; Mask `1` only; leave other physics properties at defaults |
| Sprite2D | Position `(0, -16)`; Centered On; Visible On; default white Modulate and Self Modulate |
| Sprite2D Texture | New GradientTexture2D; Width `32`; Height `32` |
| Texture's Gradient | New Gradient with two stops: offset `0.0` color `#ffffff`, offset `1.0` color `#ffffff`; both alpha `1.0` |
| CollisionShape2D | Position `(0, -5)`; Disabled Off |
| CollisionShape2D Shape | New RectangleShape2D; Size `(20, 10)` |

Keep every node's Rotation `0` and Scale `(1, 1)`. In the Inspector use each resource property's dropdown to create the named resource, then click it to expand its settings. In the Gradient editor click each endpoint and edit its color. Identical opaque endpoint colors make a solid texture; other texture settings can stay at defaults.

The root marks the feet. The square extends 32 pixels upward from that origin, while the collision footprint extends only 10 pixels upward. Graphics and physics intentionally have different sizes. GradientTexture2D renders without an image file; the engine-fallback PlaceholderTexture2D is not needed.

## Complete Controller

Attach a GDScript to **Player's CharacterBody2D root**, disable the default platformer template, and save as `res://scenes/player.gd`. Its entire contents are:

```gdscript
extends CharacterBody2D

@export var speed: float = 120.0

func _physics_process(_delta: float) -> void:
	var movement := Input.get_vector("move_left", "move_right", "move_up", "move_down")
	velocity = movement * speed
	move_and_slide()
```

Indent the three function-body lines consistently. `velocity` is inherited, so do not redeclare it. The four arguments are negative X, positive X, negative Y, positive Y. Positive Y is down. Input.get_vector limits diagonal length; zero input replaces velocity with zero. Speed is in pixels per second, and move_and_slide handles the timestep, so do not multiply by `_delta`. Its underscore indicates the argument is intentionally unused. No `_ready` callback or printing is required.

## Home and Walls

Create a Node2D root named Home and save as `res://scenes/home.tscn`. Drag `player.tscn` from the FileSystem dock onto Home to instance it. Add Walls and its wall bodies as follows; add CollisionShape2D and Sprite2D directly under each wall body.

```text
Home (Node2D)
|-- Player (instance of res://scenes/player.tscn)
`-- Walls (Node2D)
    |-- BottomWall (StaticBody2D)
    |   |-- CollisionShape2D
    |   `-- Sprite2D
    |-- TopWall (StaticBody2D)
    |   |-- CollisionShape2D
    |   `-- Sprite2D
    |-- LeftWall (StaticBody2D)
    |   |-- CollisionShape2D
    |   `-- Sprite2D
    `-- RightWall (StaticBody2D)
        |-- CollisionShape2D
        `-- Sprite2D
```

Home and Walls have Position `(0, 0)`. The Player instance has Position `(320, 180)`, Motion Mode Floating, and Speed `120.0`; remove any conflicting instance overrides. Keep every node at Scale `(1, 1)` and Rotation `0`. No scripts belong on Home or the walls.

| StaticBody2D | Position relative to Walls | RectangleShape2D Size | GradientTexture2D Width / Height |
| --- | --- | --- | --- |
| BottomWall | `(320, 352)` | `(640, 16)` | `640` / `16` |
| TopWall | `(320, 8)` | `(640, 16)` | `640` / `16` |
| LeftWall | `(8, 180)` | `(16, 328)` | `16` / `328` |
| RightWall | `(632, 180)` | `(16, 328)` | `16` / `328` |

For **each** wall, assign a new RectangleShape2D to CollisionShape2D, use the table's size, keep the child at `(0, 0)`, and leave Disabled Off. Assign a new GradientTexture2D to Sprite2D with the table's dimensions, keep the sprite at `(0, 0)`, Centered On, Visible On, and default white Modulate/Self Modulate. Its Gradient has stops at `0.0` and `1.0`, **both `#4a90d9`, alpha `1.0`**. Other resource settings stay at defaults.

If duplicating walls instead, choose **Make Unique on BOTH Shape and Texture resources before resizing** each copy, or assign fresh resources. A shared inner Gradient is fine for the same blue color. Resize resources, not node scales. Leave all walls' collision Layer and Mask on `1` only. StaticBody2D blocks; Area2D would only detect overlap. The vertical walls join the horizontal walls without gaps.

## Verify the Result

1. Save both scenes and the script. Press F5 to run Home as the main scene, not F6 to run the current scene. Click the game view to give it keyboard focus.
2. Expect a white square near the center of a dark room framed by four blue walls. W/A/S/D move up/left/down/right; release stops immediately. Opposite keys cancel their axis, and diagonal travel is not faster than straight travel.
3. Walk into every wall. Each blocks passage. Hold a diagonal into a flat wall and check that the player slides along it without the Grounded-mode slowdown. Check all corners for escape or jitter.
4. Use Debug > Visible Collision Shapes before running to inspect the colliders. The player's head can overlap the top wall while its feet are blocked; this is expected and does not require a larger footprint.
5. If sliding is slow, verify Floating in player.tscn, inspect Home's Player instance for overrides, save, and restart. Use the Remote tree while running to confirm the actual value if needed. Do not remove diagonal normalization or compensate with extra velocity math.
6. Expect no recurring debugger errors and no continuous printed output. If a texture is invisible, check both gradient colors and alpha. If a wall fails to block, check its body type, assigned enabled shape, and layer/mask. Compare other symptoms with the lesson's troubleshooting table.

This completes milestone 1 only. Do not add farming, inventory, camera scrolling, or more town features here. Return to the [lesson](../LESSON.md) for reflection and the progress template, or the [overview](../../OVERVIEW.md) for the course sequence.
