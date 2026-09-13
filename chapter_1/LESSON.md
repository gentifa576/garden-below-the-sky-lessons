# Chapter 1: Walk Around Home

[Course overview](../OVERVIEW.md) | [Solution and exact setup](cheat_sheet/SOLUTION.md) | [Next chapter](../chapter_2/LESSON.md)

Build the first playable room in **Garden Below the Sky**: a white square that moves with WASD inside four blue walls. You will learn how scenes, graphics, collision, input, and a small script work together.

This chapter uses Godot **4.7.2 stable**, Godot 4 GDScript, and no downloaded assets. The chapter's movement and wall-sliding setup has been user-tested. You need only Godot and this lesson; programming experience and an AI tutor are not required.

Work through one checkpoint at a time. Try each task before opening the solution. A checkpoint is a small observable result, not a requirement to understand everything immediately.

## 1. Meet the Editor

Open your empty project named **Garden Below the Sky**. If you have not created it yet, use the Project Manager's Create option, choose an empty project folder, select Compatibility, and create the project. If it already exists, do not create another copy. Compatibility is a provisional renderer choice for this simple 2D prototype, not a final lighting decision.

- The **Scene dock** shows the nodes in the scene you are editing, arranged as a tree.
- The **FileSystem dock** shows project files, including saved scenes and scripts.
- The **Inspector** shows properties of the selected node or resource. Click a node before changing its properties.
- The **2D workspace** lets you arrange objects visually; the **Script workspace** edits code.
- The bottom **Output** panel shows printed messages. **Debugger** shows errors and warnings from a running game.
- A **project** contains settings and files for the whole game. A **scene** is one saved tree of nodes, not necessarily an entire level.
- A **node** is a building block. The **root** is the top node in a scene; children sit beneath it and move with it.
- `res://` means the project's own folder. `res://scenes/home.tscn` means the file `home.tscn` inside its `scenes` folder, not a web address.

Use the Inspector's search box when a property is hard to find. Expand sections by clicking their headings. Save after each checkpoint with Ctrl+S, and stop a running game with F8 before editing.

**Checkpoint:** identify the Scene dock, FileSystem dock, and Inspector. An empty project does not have to show a playable scene yet.

## 2. Make a Canvas and Home

Open Project > Project Settings. Enable Advanced Settings if fields are hidden. Set the following, using the settings search when needed:

| Setting | Value |
| --- | --- |
| Display > Window > Size > Viewport Width / Height | `640` / `360` |
| Display > Window > Size > Window Width / Height Override | `1280` / `720` |
| Display > Window > Stretch > Mode | `viewport` |
| Display > Window > Stretch > Aspect | `keep` |
| Display > Window > Stretch > Scale Mode | `integer` |
| Rendering > Textures > Canvas Textures > Default Texture Filter | `Nearest` |
| Rendering > Environment > Defaults > Default Clear Color | `#202020`, fully opaque |

The viewport is the game's drawing area, measured in pixels. The window override requests a desktop window twice as wide and tall. `keep` preserves proportions, possibly adding bars; `integer` favors whole-number enlargement. Nearest keeps pixel edges crisp. These presentation settings are provisional.

1. Close Project Settings and choose Scene > New Scene, then **2D Scene**. Its root is a Node2D, a node with a 2D position.
2. Rename the root `Home` using the Scene dock's rename action. Leave its position `(0, 0)` and scale `(1, 1)`.
3. Save the scene. Create a `scenes` folder in the project's save dialog, then save as `res://scenes/home.tscn`.
4. Press F5, which runs the project. When asked for a main scene, choose the current Home scene. The main scene is the scene Godot starts when you run the project.

**Checkpoint:** the game shows an empty dark canvas. The embedded preview may fit the editor panel rather than display a literal 1280 x 720 window. That does not change your 640 x 360 game coordinates. No camera is needed for this room. Stop the game before continuing.

## 3. Build a Visible Player

Make Player a separate scene so you can reuse it. A scene **instance** is a placed copy of that saved scene, retaining a connection to its original setup.

1. Choose Scene > New Scene > Other Node. Search for `CharacterBody2D`, create it as the root, and name it `Player`.
2. In the Inspector set **Motion Mode to Floating**. This is essential: Grounded is for floor/ceiling movement, whereas Floating suits this top-down room.
3. Leave Player's position `(0, 0)` and scale `(1, 1)`. Save as `res://scenes/player.tscn`.
4. Select Player and use Add Child Node to add `Sprite2D`. This node displays an image, but does not provide collision.
5. Select Sprite2D. In the Texture property's dropdown choose **New GradientTexture2D**, then click the assigned resource to expand its properties. Set Width `32` and Height `32`.
6. In that texture's Gradient property create a **New Gradient** if none exists. Expand it, select the left endpoint in the gradient bar, and set its color to `#ffffff` with alpha fully opaque. Select the right endpoint and give it the same color. Both ends must match to get a solid white square rather than a fade.
7. On Sprite2D itself leave Centered enabled and scale `(1, 1)`. Under Transform, set Position to `(0, -16)`.
8. Select Player again and add `CollisionShape2D` as its other child, not a child of Sprite2D.
9. Select CollisionShape2D. In Shape choose **New RectangleShape2D**, expand it, and set Size to `(20, 10)`. Set the node's Position to `(0, -5)` and leave scale `(1, 1)`. Leave Disabled unchecked.

A **resource** holds data used by a node: the texture describes pixels, and the shape describes collision geometry. The node's position and the resource's size are different settings. Resize these resources, not the physics nodes' scales.

The Player root is at the character's feet. A centered 32-pixel-high sprite at Y `-16` extends from Y `-32` to `0`. The collision rectangle at Y `-5` extends from `-10` to `0`. It represents the small ground footprint, not the whole character's visible body.

In Godot 2D, X increases to the right and **Y increases downward**. Child positions are relative to their parent. Negative Y places these children above Player's feet.

Use GradientTexture2D here because it actually renders an image without an asset file. `PlaceholderTexture2D` is an engine fallback, not needed for this exercise.

Your Player scene should now look like this:

```text
Player (CharacterBody2D; Floating)
|-- Sprite2D
`-- CollisionShape2D
```

1. Save Player and switch back to Home using its scene tab.
2. Drag `player.tscn` from the FileSystem dock onto Home in the Scene dock. This creates the Player instance under Home.
3. Select that instance and set its Position to `(320, 180)`. Save Home.
4. Press F5. Do not use F6 here: F6 runs only the current scene, which might be the isolated Player instead of Home.

**Checkpoint:** a stationary white square appears near the center of the dark canvas. CollisionShape2D should no longer show a missing-shape warning. The feet, rather than the square's center, are at `(320, 180)`. Movement does not exist yet.

## 4. Name the Controls

Open Project > Project Settings > Input Map. An **action** names an intention such as moving left; a key binding connects that action to a keyboard key.

1. Enter `move_left` in Add New Action and click Add. Spelling, capitalization, and underscores matter.
2. Find its row and click the plus button to add an event. In the event dialog choose a keyboard event, use the listening/detection control, and press A.
3. Enable the **Physical** key option so the event uses the physical key position, then confirm. Check the resulting event is shown as a physical key. Physical WASD follows these key positions even when a keyboard layout labels them differently.
4. Repeat for the remaining rows below. Leave each action's default deadzone at `0.5`.

| Action name | Physical key | Direction |
| --- | --- | --- |
| `move_left` | A | Negative X |
| `move_right` | D | Positive X |
| `move_up` | W | Negative Y |
| `move_down` | S | Positive Y |

**Checkpoint:** Input Map contains all four actions, each with the intended physical key. They will not move anything until a script reads them.

## 5. Write the Movement

Open `player.tscn`, select its CharacterBody2D root, and choose Attach Script. Select GDScript, disable the default template, and set the path to `res://scenes/player.gd`. Create the script. Keep `extends CharacterBody2D` at the top; remove any generated gravity, jumping, or platformer movement code.

`extends` says this script builds on CharacterBody2D. That node already provides a `velocity` property and a collision-aware movement method, so you do not need to create them yourself.

### Small Syntax Examples

These fragments explain the language; they are not a second controller to paste into your script.

```gdscript
var distance: float = 12.5
@export var practice_number: float = 3.0
var direction := Vector2(1.0, 0.0)
```

- `var` declares a variable: a named place to store a value.
- `=` assigns a value. `*` multiplies numbers, or scales a vector by a number.
- `: float` specifies a type that can hold decimal numbers. `120.0` is a float literal, a value written directly in code.
- `@export` exposes a script variable in the Inspector. Use this for a setting you want to tune.
- `Vector2` holds two numbers, X and Y. Here `(1.0, 0.0)` points right.
- `:=` declares a variable and infers its type from the assigned value. It is not the same syntax as `: float`.
- Quotation marks form a string, such as `"move_left"`. Input action names are strings, not variable names.

A function is a named block of instructions. Godot calls some functions automatically; these are **callbacks**. For example, this optional temporary probe runs when a node enters the scene and is ready:

```gdscript
func _ready() -> void:
	print("Player is ready")
```

`func` declares the function. Parentheses hold its arguments, if any; `-> void` says it returns no value. The final colon starts its body. Indentation puts the next line inside that body; use the editor's Tab indentation consistently. `print(...)` writes to Output. This probe should print once per Player instance when you press F5. Remove the whole probe afterward, not just its body: an empty callback is not valid code.

### Your Controller Task

Build your controller from this recipe, using the syntax above and the method names below. The separate solution is available if you get stuck, but try one line at a time first.

1. Under `extends`, declare an exported variable named `speed`, typed as `float`, with the initial value `120.0`. This is movement speed in game pixels per second.
2. Declare the callback with the header `func _physics_process(_delta: float) -> void:`. Indent all its instructions beneath it. Godot calls it at a regular physics rate, normally 60 times per second.
3. Inside that callback, declare a local variable named `movement` using `:=`. Set it to the result of `Input.get_vector(...)`.
4. Replace `...` with four comma-separated action strings, in this exact order: negative X, positive X, negative Y, positive Y. Use the names from Input Map. The dots are an explanation placeholder, not valid controller code.
5. Assign the existing `velocity` property to `movement` multiplied by `speed`. Do this every tick, even if no keys are pressed. Do not write `var velocity`: the body already owns this property.
6. Call `move_and_slide()` on the next line, with no arguments. This asks CharacterBody2D to move and respond to solid obstacles. Do not change `position` directly for this controller.

The algorithm in plain-language pseudocode is:

```text
Each physics tick:
    read a direction from the four actions
    replace velocity with direction times speed
    move the body while checking for collisions
```

`Input.get_vector` returns a Vector2. No input produces `(0, 0)`, so replacing velocity also stops the player. D alone produces `(1, 0)`; W alone produces `(0, -1)`. W+D produces approximately `(0.707, -0.707)`, not `(1, -1)`: diagonal length is limited to one, preventing faster diagonal travel. Opposite keys cancel their axis.

`_delta` is an argument containing the time since the previous physics tick, in seconds. Its leading underscore signals intentional non-use. **Do not multiply velocity by `_delta` here**: `move_and_slide()` already uses the timestep internally. Keep velocity in pixels per second.

**Checkpoint:** save, press F5, click the game view to focus it, and test these before adding walls:

- W, A, S, and D move in the expected directions, with no gravity drift.
- Releasing all keys stops the player immediately.
- Two neighboring direction keys move diagonally without increasing total speed.
- A+D together cancel horizontal movement; W+S cancel vertical movement.
- Changing Speed in the Player scene's Inspector changes movement after saving and restarting. Restore it to `120.0` afterward; check Home's instance has no conflicting Speed override.
- Debugger has no recurring errors. With the optional probe removed, Output need not print anything.

There are no boundaries yet: walking offscreen is expected. Stop and restart to return to the starting position. For input diagnosis you may temporarily put `print(movement)` after reading input; remove it after testing because it prints every physics tick.

## 6. Build Four Solid Walls

A Sprite2D can look like a wall without blocking anything. A StaticBody2D provides a stationary physics body, and its CollisionShape2D defines the solid area. `Area2D` detects overlap but does not block this movement; do not use it for these walls.

1. Stop the game and open Home. Add a Node2D child named `Walls`, at position `(0, 0)` and scale `(1, 1)`.
2. Under Walls add a StaticBody2D named `BottomWall`. Set its Position to `(320, 352)` and keep scale `(1, 1)`.
3. Under BottomWall add CollisionShape2D with a new RectangleShape2D. Set the resource's Size to `(640, 16)`. Leave the child node at `(0, 0)`, scale `(1, 1)`, and Disabled unchecked.
4. Also under BottomWall add Sprite2D. Give it a new GradientTexture2D of Width `640` and Height `16`. Create/expand its Gradient and set **both endpoint colors to `#4a90d9`**, fully opaque, using the same procedure as for the player.
5. Keep this sprite Centered, at Position `(0, 0)` and scale `(1, 1)`. Save and run Home. Walk downward: the blue strip should stop your player.
6. Stop the game. Duplicate BottomWall three times using the Scene dock's Duplicate action, and rename the copies using the table. Set each body's position.
7. Before resizing a duplicate, open the resource dropdown beside its Shape and choose **Make Unique**. Do the same for its Sprite2D Texture. Then change shape Size and texture Width/Height to the table's values. Alternatively assign fresh RectangleShape2D and GradientTexture2D resources and configure them from scratch.

| Wall body | Position relative to Walls | Rectangle size and texture dimensions |
| --- | --- | --- |
| BottomWall | `(320, 352)` | `640 x 16` |
| TopWall | `(320, 8)` | `640 x 16` |
| LeftWall | `(8, 180)` | `16 x 328` |
| RightWall | `(632, 180)` | `16 x 328` |

Duplicating a node can leave both copies referring to the same resource. Editing a shared shape or texture can resize another wall unexpectedly. Make **both** resources unique before resizing; making just the node a duplicate is not enough. Sharing the inner Gradient is fine while all wall colors stay identical.

The horizontal walls span the viewport width. The vertical walls fill the space between them, from Y `16` to `344`. Keep Home, Walls, bodies, and children at unit scale. Leave Player and every wall's collision layer and mask on the default **layer 1 only**.

**Checkpoint:** save and run Home. All four blue walls are visible and block passage. Hold a diagonal into a flat wall: the player should continue sliding along it. Test every corner for escape or jitter. Enable Debug > Visible Collision Shapes before running to see the actual collision rectangles; turn it off afterward if desired.

The white square's head may overlap the top wall while its feet remain blocked. This is expected for a ground footprint. Do not enlarge the collider just to hide this visual overlap; scenery and draw order come later.

## 7. Troubleshoot One Symptom

| Symptom | First checks |
| --- | --- |
| Blank game or wrong starting position | Use F5 with Home as main scene; confirm Home contains Player at `(320, 180)`. F6 may run Player alone. |
| Invisible or fading square/wall | Assign GradientTexture2D, set its dimensions, and set both Gradient endpoints to the same opaque color. Check Sprite2D is visible and at unit scale. |
| Missing-shape warning or walking through walls | Assign enabled RectangleShape2D resources; check StaticBody2D parents and default layer/mask 1. A sprite alone cannot collide. |
| Keys do nothing | Focus the game; compare Input Map names with the four quoted strings; check physical events and that the script is attached to Player's root. |
| Red script error | Read the first error and its line number. Check spelling, straight quotes, parentheses, the callback's final colon, and consistent indentation. Fix one error, then rerun. |
| Continued movement after release | Assign velocity every physics tick, including when movement is zero. |
| Falling, jumping, or unexpected drift | Remove platformer template code; keep only the top-down controller and use Floating. |
| Extremely slow movement everywhere | Restore Speed to `120.0` and remove any multiplication by delta. Check instance overrides. |
| Diagonal sliding is slow against a wall | Set Player's Motion Mode to Floating. Save, inspect Home's Player instance for a Grounded override, and restart. Compare D with W+D against the flat top wall, away from corners. Do not remove input normalization or add a speed-compensation hack. |
| Resizing one wall changes another | Make both its Shape and Texture resources unique, then restore the table's dimensions. |
| Appearance and collision disagree | Use Visible Collision Shapes. Check child positions, resource sizes, and unit scales; the player's smaller footprint is intentional. |

If Floating is already set but sliding remains wrong, inspect the running Player through the Scene dock's Remote tree and confirm its actual Motion Mode. Compare the script and wall geometry with the solution before changing movement math.

## 8. Finish and Reflect

Milestone 1 is complete when F5 opens Home, WASD and key release work, diagonals have consistent speed, all four walls block and allow sliding, corners hold, and there are no recurring debugger errors. Stop here: no farming, inventory, animation, camera scrolling, or large map yet. [Chapter 2](../chapter_2/LESSON.md) is a separate automatic-duel scene.

Answer in your own words: What is the difference between a scene and its instance? Why does a wall need both a sprite and a collider? Why assign velocity when no key is held? Why is positive Y downward relevant to W? Why use Floating? Why make resources unique before resizing?

Record your progress, even if a checkpoint is not finished:

```text
Chapter: 1 - Walk Around Home
Godot version / renderer:
My project folder:
Last checkpoint passed:
Main scene:
Movement / release / diagonal / opposite-key results:
Four-wall / sliding / corner results:
Debugger errors (exact text, or none):
One concept I can explain:
Current problem and what I tried:
Next single checkpoint:
```

Optional: ordinary ChatGPT can act as a tutor using text, code, errors, or screenshots you paste or attach. It cannot be assumed to see your editor or files. No special integration is required. Try this prompt:

> I am an absolute beginner following Chapter 1 of Garden Below the Sky. Here is the relevant lesson text and my current code, exact error, or screenshot. Help me with one checkpoint at a time. Give a hint first, not the complete solution. Do not assume you can access my editor or project files; ask for the specific evidence you need. My expected result is __; my actual result is __.

[Compare with the solution](cheat_sheet/SOLUTION.md) | [Return to overview](../OVERVIEW.md)
