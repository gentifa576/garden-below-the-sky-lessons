# Chapter 2: An Automatic Duel

[Course overview](../OVERVIEW.md) | [Chapter 1](../chapter_1/LESSON.md) | [Complete solution](cheat_sheet/SOLUTION.md)

Build a small battle you can start, watch, and restart. Two colored squares stand on a board, exchange damage once a second, and display a winner. You do not control their attacks.

This chapter records the completed second milestone. The learner reported that all milestone checks passed. The exercises below let you reconstruct it and understand the decisions, rather than merely copy a finished script.

## Before You Begin

Use Godot 4.7.2 and your Chapter 1 project. You only need the Godot editor. No terminal, programming assistant, or assistant access to your project is required.

Keep your Home scene and town movement unchanged. Home remains the project's main scene: **F5 / Run Project** opens Home. Select the Battle scene tab and use **F6 / Run Current Scene** for these exercises. On keyboards that reserve function keys, use the editor's run buttons and their tooltips instead.

The project uses a logical viewport of **640 by 360**. In **Project > Project Settings > Display > Window**, preserve Chapter 1's viewport width 640, viewport height 360, window width override 1280, window height override 720, stretch mode `viewport`, stretch aspect `keep`, and stretch scale mode `integer`. Use the settings search or Advanced Settings if a property is hidden. Keep **Rendering > Textures > Canvas Textures > Default Texture Filter** set to **Nearest**. The larger window displays the same logical coordinates; do not double the positions in this chapter.

If Home is not already the main scene, set **Application > Run > Main Scene** to `res://scenes/home.tscn`. `res://` means the project folder shown in Godot's FileSystem dock, not a folder you need to locate with a terminal.

The finished responsibilities are deliberately small:

| Part | Responsibility |
| --- | --- |
| Battle | Starts, times, finishes, and resets the duel |
| Board | Draws the grid and converts cells to local positions |
| BattleUnit | Stores one unit's health and damage; updates its appearance |

There is no movement, pathfinding, physics body, individual attack speed, simultaneous resolution, team system, or random damage. Blue and orange distinguish the markers visually; color is not a team identifier.

## How To Work

Do one exercise at a time. Save scenes and scripts with **Ctrl+S**, run Battle, then stop the running scene before changing editor settings. When a step asks you to write a function, try its described behavior first. The hints teach the syntax you need; the linked solution contains the three complete scripts and all editor setup.

To attach a script, select its node in the Scene dock, use **Attach Script**, set the requested `res://scenes/...gd` path, and create it. If Godot offers a default template, turn the template off or delete its unused `_ready()` and `_process()` placeholders. Each function name should appear only once in a script.

Godot scripts use **GDScript**. Indentation defines which lines belong to a function, loop, or condition. This chapter's code examples use **four spaces per indentation level**. If you paste them into a tab-indented script, use the script editor's **Edit > Indentation** menu to convert that file to one consistent style. Do not mix tabs and spaces in one file. A line ending in `:` usually opens an indented block. Lines at the same indentation are peers; moving a line changes when it runs.

## Exercise 1: Draw A Board

**Goal:** run a separate scene containing exactly six rows and six columns, with no units or combat code yet.

1. Choose **Scene > New Scene**, then **2D Scene**. Rename the Node2D root `Battle` by selecting it and pressing F2. Save as `res://scenes/battle.tscn`.
2. Select Battle, use **Add Child Node** (Ctrl+A), choose `Node2D`, and name it `Board`. Names and capitalization matter.
3. Select Board. In **Transform > Position**, set X to `176` and Y to `24`. Leave rotation zero and scale `(1, 1)`.
4. Attach `res://scenes/board.gd` to Board. Its first line is `extends Node2D`.
5. Declare exported integer settings named `columns` and `rows`, each defaulting to `6`, and a float setting named `cell_size`, defaulting to `48.0`.
6. Write `_draw()` to draw each cell, first as a black filled rectangle and then as a slate-gray outline with width `1.0`.
7. Save and run Battle with F6. Count the rows and columns. Set Board's Cell Size to `40` in the Inspector, rerun, then restore `48`.

### Syntax And Drawing Hints

`extends Node2D` tells Godot this script adds behavior to a 2D node. `@export` exposes a setting in the Inspector; `var` declares a variable; `: int` means a whole number; `: float` means a number that can have a fractional part; `=` assigns a value.

Use this declaration as a pattern for the other settings:

```gdscript
@export var columns: int = 6
```

`func` declares a function, a named operation. Godot calls the special `_draw()` function when it needs drawing commands. `-> void` says the function does not return a value to its caller.

Start with one cell. A `Vector2` holds two numeric coordinates, and a `Rect2` holds a rectangle's top-left position and size:

```gdscript
var top_left := Vector2(0, 0)
var rectangle := Rect2(top_left, Vector2(cell_size, cell_size))
draw_rect(rectangle, Color.BLACK, true)
draw_rect(rectangle, Color.SLATE_GRAY, false, 1.0)
```

Here `:=` declares a variable and lets Godot infer its type from the value. The dots in `Color.BLACK` select a named constant. `true` and `false` are Boolean values, meaning yes and no. The third argument to `draw_rect` decides whether the rectangle is filled. Draw the black fill **before** its outline so the fill does not cover that cell's outline.

Once that works, wrap the drawing in these loops inside `_draw()`:

```gdscript
for column in columns:
    for row in rows:
        # Calculate this cell's rectangle and draw it here.
        pass
```

`for` repeats a block. In GDScript, iterating over an integer `6` produces `0, 1, 2, 3, 4, 5`, not six itself. The inner row loop runs six times for each of the six columns: 36 rectangles. Replace `pass` with your drawing lines; `pass` is only a do-nothing placeholder. A line starting with `#` is a comment, not an instruction to Godot.

Replace the one-cell top-left with `Vector2(column * cell_size, row * cell_size)`. `*` multiplies. Keep size `Vector2(cell_size, cell_size)`.

Drawing uses **Board-local coordinates**: `(0, 0)` is Board's origin, already moved to `(176, 24)` in Battle. Do not add that offset inside the drawing loop. At size 48 the board is `6 * 48 = 288` pixels wide and tall.

Godot retains, or caches, these drawing commands. `_draw()` is not an every-frame simulation loop. A later runtime change to drawing data would need `queue_redraw()` to request an update. You do not need that for this static board, and you should not call `_draw()` yourself or add `_process()` to redraw continuously.

Do not add `@tool`, font drawing, unit lookups, or a Board `_ready()` yet. Without `@tool`, the custom rectangles need not appear in the editor's 2D view; check the running scene.

**Checkpoint:** F6 shows the board, F5 still opens Home, and changing one Cell Size setting changes every rectangle.

## Exercise 2: Place Two Units

**Goal:** make one reusable marker scene and place two separate instances by logical cell coordinates.

A **scene instance** is a copy of a saved scene used inside another scene. Both copies share a design, but each has its own nodes and runtime variables. This saves building the same marker twice.

1. Make another new 2D scene. Name its Node2D root `BattleUnit` and save it as `res://scenes/battle_unit.tscn`.
2. Add a `Sprite2D` child named exactly `Sprite2D`. This node displays a texture. Do not add the town's Player, CharacterBody2D, or collision nodes.
3. Select Sprite2D. In its **Texture** dropdown choose **New GradientTexture2D**. Click the resource to expand it and set Width and Height to `32`.
4. Within that texture, create a **New Gradient** if needed. Expand the Gradient and select each of its two endpoint markers in turn. Set both endpoint colors to white, including full alpha. Alpha is opacity; full alpha means visible, not transparent.
5. Keep Sprite2D's **Offset > Centered** enabled, position `(0, 0)`, offset `(0, 0)`, and scale `(1, 1)`. This marker's root is its center, unlike a town character whose origin represents its feet.
6. Attach `res://scenes/battle_unit.gd` to BattleUnit. Export `cell: Vector2i` and `display_color: Color = Color.WHITE`. In `_ready()`, assign `$Sprite2D.self_modulate = display_color`.
7. Save the script and scene. Return to Battle. Select Board, then use **Instantiate Child Scene** in the Scene dock and choose `battle_unit.tscn`. Name the instance `Friendly`. Repeat under Board and name the second instance `Enemy`.
8. Select Friendly's instance root and use its exported Inspector fields to set Cell to `(2, 2)` and Display Color to `#4a90d9`. Set Enemy to cell `(3, 2)` and color `#e59b45`. Keep full alpha for both. Do not recolor the shared Gradient.
9. Now that both child nodes exist, add `cell_to_local()` and `_ready()` to Board. Use the helper to set each unit's position from its own cell.

### Placement Hints

`Vector2i` holds two **integers**, appropriate for a cell address. Cell `(2, 2)` means column 2, row 2, counting from zero at the top-left. It is not pixel `(2, 2)`.

`_ready()` is called when a node enters the running scene and its children are ready. `$Sprite2D` is shorthand for `get_node("Sprite2D")`, a lookup relative to the node that owns the script. `self_modulate` tints only that Sprite2D. A white texture makes the selected color visible without mixing it with a preexisting texture color.

Use this helper in Board:

```gdscript
func cell_to_local(cell: Vector2i) -> Vector2:
    return (Vector2(cell) + Vector2(0.5, 0.5)) * cell_size
```

The parentheses after the function name describe its input, called a **parameter**. Here `cell` is a Vector2i supplied by whoever calls the function. `-> Vector2` promises a Vector2 result. `return` sends that result back and leaves the function. `Vector2(cell)` converts the integer address to a vector that can include fractions. Adding half a cell on each axis reaches the cell's center, then multiplying turns cells into pixels.

In Board's `_ready()`, this is one of the two assignments you need:

```gdscript
$Friendly.position = cell_to_local($Friendly.cell)
```

Write the equivalent for Enemy. Keep placement out of `_draw()`. Both units must be **direct children of Board**, because their `position` is local to their parent. Do not use `global_position` for this result and do not add `Board.position` again. Godot already includes the parent's transform when displaying its children.

Saved script exports appear on each instance root in the Inspector. You do not need **Editable Children** to change these root properties. If the fields are absent, save the script, fix any parser errors, and verify the script is attached to BattleUnit, not Sprite2D.

### Check Placement

| Board Cell Size | Friendly local center | Enemy local center |
| --- | --- | --- |
| 48 | `(120, 120)` | `(168, 120)` |
| 40 | `(100, 100)` | `(140, 100)` |

Run at both sizes without changing the cell addresses. To inspect runtime positions, use the Scene dock's **Remote** tree while the scene runs and select each unit. Return to **Local** and stop the scene before editing saved values. Restore Cell Size to 48. Temporarily move Board and rerun: grid and markers should move together. Restore Board to `(176, 24)`.

**Checkpoint:** two differently colored squares are centered in adjacent cells. Changing one instance's Display Color does not change the other.

## Exercise 3: Health You Can Test

**Goal:** each unit starts with its own maximum health, displays it, and handles a temporary damage input safely.

1. Open `battle_unit.tscn`. Add a `Label` child of BattleUnit named `HealthLabel`, alongside Sprite2D.
2. Use the Label's top-left anchors. Under **Layout > Transform**, set Position to `(-24, -34)` and Size to `(48, 18)`. Set Horizontal Alignment to **Center**. Under **Theme Overrides > Font Sizes**, enable Font Size and set it to `12`. Leave the font resource unset to use Godot's fallback font. Set placeholder Text to `20 / 20`.
3. In `battle_unit.gd`, export `max_health: int = 20` and `attack_damage: int = 6`. Declare a separate, non-exported `var health: int` for the current value. Require both exported numbers to be positive in the Inspector; this chapter does not support zero or negative starting stats.
4. Extend the existing `_ready()`: keep the tint assignment, set `health = max_health`, then call a new `update_health_display()` function.
5. Write `update_health_display()` to set the label from the two numbers. Do not read health back out of label text.
6. Write `take_damage(amount: int) -> void` using the lesson's signed-amount rules below.
7. Save BattleUnit. In Battle, verify both instances have Max Health `20` and Attack Damage `6`.

### State And Text Hints

`max_health` and `attack_damage` are **configuration**: values chosen before a fight. `health` is **runtime state**: a value that changes during a fight. Initialize health in `_ready()`, not with `var health = max_health`. Inspector overrides are available in `_ready()`, whereas an earlier variable initializer can use the script's default instead of the instance's intended maximum.

Each unit's script variables belong to its own node. A shared texture resource does not make health shared. Godot readies children before their parent: each BattleUnit can initialize its Sprite2D and Label before Board's `_ready()`, and Board is ready before Battle's `_ready()`.

For the label assignment, use:

```gdscript
$HealthLabel.text = "%d / %d" % [health, max_health]
```

Quoted text is a **String**. Each `%d` is a whole-number placeholder. The `%` after the string inserts the ordered values in the square-bracketed list: first current health, then maximum health. A function call such as `update_health_display()` runs that function; empty parentheses mean you supply no inputs.

### A Provisional Damage Contract

For this lesson, keep the name `take_damage` and give its signed input the following explicit meaning:

| Input or condition | Behavior |
| --- | --- |
| Positive amount | Subtract health |
| Zero amount | Ignore the call |
| Negative amount on a living unit | Heal, up to maximum health |
| Current health is zero | Ignore every amount; do not revive |

This is a bounded lesson rule, not a commitment to healing mechanics in the final game. Automatic attacks use only positive Attack Damage. Resetting the encounter later is a separate operation, not an attempt to heal a defeated unit through `take_damage`.

Start the function with a **guard**, an early rejection of a call that should do nothing:

```gdscript
if amount == 0 or health == 0:
    return
```

`if` runs its block only when its condition is true. `==` compares values; it is different from assignment `=`. `or` accepts either condition. A bare `return` immediately leaves this function. **`pass` does not leave**: it does nothing and then execution continues, so it is not a substitute for this guard.

After the guard, assign `clampi(health - amount, 0, max_health)` to health, then refresh the display. `clampi(value, minimum, maximum)` bounds an integer to the inclusive range. Subtracting a negative adds: `14 - (-6)` becomes `20`. Never delete the unit at zero with `queue_free()`; its marker and zero label stay visible.

### Add Temporary Space Damage

This is a test tool, not the final combat controls.

1. Open **Project > Project Settings > Input Map**. Type `debug_damage` into Add New Action and add it.
2. Use the action's plus button to add an event. Choose a keyboard event, use the key-listening option, press **Space**, and confirm. Close Project Settings.
3. Add the following temporary function to `board.gd`, at top-level indentation, alongside `_draw()` and `_ready()`.

```gdscript
func _unhandled_input(event: InputEvent) -> void:
    if event.is_action_pressed("debug_damage"):
        $Enemy.take_damage(6)
```

Godot passes an input event to `_unhandled_input` if earlier input handling has not consumed it. `event: InputEvent` names that supplied event and its type. `is_action_pressed` checks this event, not whether the key is being held on every frame. Its default `allow_echo = false` rejects keyboard-repeat echo events. Press and release Space separately for each test hit.

### Check Health

1. Run Battle. Both labels start at `20 / 20`. Four separate Space presses make Enemy show `14`, `8`, `2`, then `0`. Extra presses leave it at zero. Friendly stays at 20.
2. Stop. For a healing check, keep `$Enemy.take_damage(6)` and temporarily add `$Enemy.take_damage(-3)` on the next line at the **same indentation inside the input condition**. Save, rerun, and press Space once. The two calls run in order: 20 becomes 14, then 17. The final label should show `17 / 20`.
3. Stop and change that second call to `$Enemy.take_damage(-99)`. Save, rerun, and press Space once. Damage first leaves 14, then healing is capped at 20, not 113.
4. Stop. Change the first call to `$Enemy.take_damage(99)` and the second to `$Enemy.take_damage(-6)`. Save, rerun, and press Space once. The first call reduces health to zero; the second cannot revive it. The label stays `0 / 20`.
5. Stop. Change the first call back to `$Enemy.take_damage(6)` and the second to `$Enemy.take_damage(0)`. Save, rerun, and press Space once. Health stays at 14 after the zero call. Then remove the second call entirely so the handler again contains just its original one damage call.
6. Stop. Override Enemy's Max Health to `30` in the saved scene and rerun. Its initial label must read `30 / 30`, not `20 / 30`. Restore Max Health to 20.

These short temporary sequences test the function without adding a permanent test interface. Godot may only display the final health value after both calls; the intermediate arithmetic explains the expected result.

**Checkpoint:** independent health, readable numbers, bounded subtraction, and initialization that respects instance settings.

## Exercise 4: A Timed Duel

**Goal:** Start begins automatic attacks, and defeat stops them immediately with a named winner.

Remove Board's **entire temporary `_unhandled_input()` function now**. You may remove `debug_damage` from Input Map too; an unused action is harmless. Do not leave Space secretly dealing damage during the automatic duel. Space may still activate a focused Button through Godot's normal UI controls, which is different from debug damage.

### Build The Controls

Add these as **direct children of Battle**, not Board. Select Battle before adding each node. Button and Label are Control nodes: use top-left anchors and **Layout > Transform** for their positions and sizes, not a Container. Keep Battle itself at `(0, 0)`.

| Node name | Type | Position | Size | Other settings |
| --- | --- | --- | --- | --- |
| AttackTimer | Timer | Not applicable | Not applicable | Wait Time `1.0`, One Shot off, Autostart off, Process Callback Physics |
| StartButton | Button | `(176, 324)` | `(72, 32)` | Text `Start`, Disabled off |
| RestartButton | Button | `(256, 324)` | `(88, 32)` | Text `Restart`, Disabled on |
| ResultLabel | Label | `(352, 324)` | `(220, 32)` | Text `Ready`, Font Size override `16`, fallback font |

Leave button fonts at their defaults. Use left alignment for ResultLabel; vertical centering is useful for lining up with the buttons. The board ends at Y 312 and the controls end at Y 356, inside the 360-high viewport.

A **Timer** waits and emits a `timeout` event. With One Shot off it repeats until stopped. Physics processing gives this shared timer a regular physics update basis; you still do not need to write `_physics_process()`. Its one-second beat is not one attack every rendered frame and is not an individual speed per unit.

### Write The Battle Flow

Attach `res://scenes/battle.gd` to Battle. Use `extends Node2D`, then declare:

```gdscript
enum Phase { PREPARATION, FIGHTING, RESULTS }
var curr_phase: Phase = Phase.PREPARATION
```

An `enum` defines a named set of choices. This variable holds one choice at a time. `Phase.FIGHTING` is clearer than remembering an unexplained number. The phase describes what the battle may do:

| Phase | Allowed behavior |
| --- | --- |
| PREPARATION | Start is allowed; no attacks; Restart unavailable |
| FIGHTING | Timer resolves attacks; both buttons unavailable |
| RESULTS | No attacks; Start unavailable; Restart allowed |

Create `_on_start_button_pressed()` and `_on_attack_timer_timeout()` as void functions. You can let the editor create their initial stubs when connecting signals below, or write them first. Do not create two copies of a function.

Start's function must reject calls outside preparation. Here is the guard pattern:

```gdscript
if curr_phase != Phase.PREPARATION:
    return
```

`!=` means not equal. After the guard, set the phase to FIGHTING, disable StartButton, set ResultLabel's text to `Fighting`, and call `$AttackTimer.start()`. The first attack waits a full interval. Disabling a button helps the player; the guard also protects the function's rule.

For timeout, write the following sequence in order:

1. Return unless the current phase is FIGHTING.
2. Make local references with `var enemy = $Board/Enemy` and `var friendly = $Board/Friendly`. A node path with `/` steps down the tree; these paths start at Battle, which owns this script.
3. Call `enemy.take_damage(friendly.attack_damage)`. The attacker supplies its damage; the receiving unit owns the health change.
4. If Enemy's health is zero, call `finish_battle("Friendly")`, then return immediately from the timeout function.
5. Otherwise call `friendly.take_damage(enemy.attack_damage)`.
6. If Friendly's health is zero, call `finish_battle("Enemy")`, then return.

Create `finish_battle(winner: String) -> void`. Set the result text using `"Winner: %s" % winner`, stop the timer, set the phase to RESULTS, and enable RestartButton. `%s` inserts text, unlike `%d` for an integer. Start remains disabled from when the fight began.

This order deliberately gives Friendly **first-strike advantage**. A defeated Enemy does not retaliate. Attacks are not simultaneous, and no individual cooldowns run behind the scenes. Positive health and positive attack damage ensure this tiny duel eventually has a winner; unsupported zero-damage configurations could otherwise stall it.

### Connect Signals In The Editor

A **signal** is a notification a node sends. A **handler** is the function connected to receive it. Naming a function `_on_...` does not connect anything by itself. Connections are saved in the scene, not automatically inferred from the script.

1. Save `battle.gd`. Select StartButton in the Scene dock.
2. Open the **Signals** dock beside the Inspector. In editor layouts that group it under **Node**, open **Node > Signals**.
3. Double-click `pressed()` (or select it and click Connect).
4. In the connection dialog, choose **Battle** as the receiving node. Set Receiver Method to exactly `_on_start_button_pressed`. If necessary, expand Advanced to edit the receiver method.
5. Click Connect. If Godot created a method containing `pass`, replace that placeholder with the intended logic. If your method already existed, inspect it rather than pasting a second definition.
6. Select AttackTimer and repeat for `timeout()`, receiving on Battle at `_on_attack_timer_timeout`.
7. Save **Battle's scene** as well as its script. Implement and connect RestartButton in the next exercise.

| Sending node | Signal | Receiving node | Method |
| --- | --- | --- | --- |
| StartButton | `pressed` | Battle | `_on_start_button_pressed` |
| AttackTimer | `timeout` | Battle | `_on_attack_timer_timeout` |

Connect each signal exactly once through the editor. Do not also write `.connect(...)` calls in code. Select a signal and inspect its listed connections if unsure.

### Check The Attack Order

Before pressing Start, neither unit should lose health, even after several seconds. After Start, you should see one pair of health updates per second until the last tick.

| Tick | Friendly health, both max 20 | Enemy health, both max 20 |
| --- | --- | --- |
| Before Start | 20 | 20 |
| 1 | 14 | 14 |
| 2 | 8 | 8 |
| 3 | 2 | 2 |
| 4 | 2 | 0 |

The result is `Winner: Friendly`. Wait several seconds afterward: no further attacks occur. Try clicking Start again: it is disabled, and the handler also rejects the wrong phase.

Stop, set Enemy's Max Health to `25`, save, and rerun. Leave both damage values at 6:

| Tick | Friendly health | Enemy health |
| --- | --- | --- |
| Before Start | 20 | 25 |
| 1 | 14 | 19 |
| 2 | 8 | 13 |
| 3 | 2 | 7 |
| 4 | 0 | 1 |

This result is `Winner: Enemy`. Restore Enemy's maximum to 20 after testing both outcomes. The visible numerical changes count as basic attack feedback here. No shader, animation, or imported artwork is required.

## Exercise 5: Restart Without Reloading

**Goal:** Results lets you prepare the same encounter again without closing the running scene.

This completed version resets the two existing units directly from Battle. It does not instantiate replacements, delete defeated nodes, reload Home, or introduce a per-unit reset helper. A helper could be a later cleanup if repeated code grows, but is not needed for this milestone.

1. Write `reset_battle() -> void` in `battle.gd`. First stop AttackTimer, then set the phase to PREPARATION.
2. Assign Friendly's health from Friendly's max_health and call Friendly's `update_health_display()`. Do the same for Enemy. For example, use `$Board/Friendly.health = $Board/Friendly.max_health` followed by a call on that same node.
3. Enable StartButton, disable RestartButton, and set ResultLabel's text to `Ready`.
4. Add Battle's `_ready() -> void` and have it call `reset_battle()`. Both units already exist and have readied before this parent callback runs.
5. Write `_on_restart_button_pressed() -> void`. Return unless the phase is RESULTS, then call `reset_battle()`.
6. Select RestartButton and connect its `pressed` signal **once**, using the same Signals dock steps, to Battle's `_on_restart_button_pressed`. Save the scene.
7. Confirm `finish_battle()` enables RestartButton. Do not enable it during preparation or fighting.

Keep the timer stop in `reset_battle()` so initial setup and Restart use the same reset path. The restart handler need not stop it a second time. Do not call `_ready()` manually: it is a lifecycle callback, not a reset button API.

Reset changes current health, timer activity, phase, buttons, and result text. It retains max health, attack damage, cell, and display color. Positions need no reset because units never moved. Restart prepares the duel; it does not start the timer. The next Start begins a new full one-second interval.

**Checkpoint:** complete Start -> result -> Restart -> Start at least three times. Verify full health on each restart, no attacks while Ready, the same first interval and results with unchanged stats, and no attacks after a result. Also test restarting after the Enemy-wins configuration. Restore the default stats afterward. Restart must be unavailable before Start and during combat.

## Troubleshooting

| Symptom | Check |
| --- | --- |
| F5 opens Home rather than Battle | Correct. Select Battle and use F6 for this chapter. |
| No grid in the editor | Expected without `@tool`; run Battle. If still absent, check Board's script, visible state, positive dimensions, and position. |
| Node not found or a null-instance error | Match names and capitalization, check direct parenting, and check which node owns the script. Board uses `$Enemy`; Battle uses `$Board/Enemy`. Do not run the placement `_ready()` before units exist. |
| Units are offset by the board position twice | Use Board-local `position` from the helper; do not add `(176, 24)` or assign it to `global_position`. |
| Markers have the wrong color or both change | Use white Gradient endpoints and each Sprite2D's `self_modulate`; do not change a shared Gradient to tint an instance. Check alpha too. |
| Inspector exports are missing | Save the script, resolve parser errors, and select the instance root with the correct attached script. |
| Health starts at 20 despite a different maximum | Initialize health from max_health in `_ready()`. Check saved instance overrides and whether you are inspecting Local or Remote. |
| Label is unreadable or clipped | Check exact name, position, width 48, height 18, font size 12, center alignment, and fallback font. Leave Sprite2D centered at its origin. |
| Start or Restart does nothing | The method name alone is insufficient. Inspect its signal connection and receiver, and save the scene. Confirm its phase guard permits that action. |
| Start changes the label, but no attacks occur | Check Timer timeout connection, Wait Time 1.0, and the call to `start()`. Autostart should remain off. |
| Only one attack happens | One Shot must be off. |
| Damage occurs too often or before Start | Remove the debug input handler; inspect duplicate or extra signal connections; keep Autostart off. Do not add another timer or a per-frame damage call. |
| A defeated unit still attacks | Use `return` immediately after finishing the battle. `pass` does not exit a function. |
| Restart behaves differently each time | Stop the timer in reset, refill both health values and labels, restore the phase and button states, and keep only one connection per signal. |

For an error, start with the first red message in Godot's bottom Debugger panel. It usually names a script and line. Fix that first rather than adding unrelated code. The [solution](cheat_sheet/SOLUTION.md) includes the exact final scene tree, settings, connections, and scripts for comparison.

## End Checkpoint

The milestone is complete when Home still opens with F5, Battle runs with F6, the board scales from one setting, the two instances have independent health, Start produces timed automatic attacks, both winner cases match the tables, and three restarts repeat cleanly. Health numbers and the winner label provide the required basic feedback.

Stop here. Do not add future systems just to finish this chapter. Course updates happen at milestone boundaries, not after every small edit.

Write a short learner note in your own words:

> I built ___. A cell address differs from a pixel position because ___. Board-local positions work because ___. The guard uses return rather than pass because ___. Friendly wins equal-stat fights because ___. Reset restores ___ but retains ___. My checked outcomes were ___. I still want to understand ___.

### Optional ChatGPT Review

You can use ordinary ChatGPT in a browser by pasting text or uploading files and screenshots. It does not need project access, a terminal, or a special coding product. This is optional; the editor checks and solution are enough to continue independently.

Paste this request with the relevant materials at the milestone boundary:

> I am an absolute beginner using Godot 4.7.2. I have completed Chapter 2: a stationary two-unit automatic duel. Please review the attached/pasted scripts and scene-tree screenshot before suggesting changes. Home stays the main scene; Battle runs separately. Board places two direct child units by cell center. One shared one-second timer attacks Friendly-first. take_damage accepts positive damage, ignores zero, allows negative amounts to heal living units up to maximum, and never revives a zero-health unit. Automatic attacks are positive. Battle resets both units directly. Signals are connected once in the editor, not code. My test results are: ___. My first error or question is: ___. Explain any issue in beginner language and suggest the smallest fix. Do not add future systems or assume you can open my project.

Attach `board.gd`, `battle_unit.gd`, and `battle.gd`, or paste each with its filename. Include a scene-tree screenshot, relevant Inspector values, and the three signal connections; scripts alone do not show editor connections. If something failed, include the exact error text and what you did immediately before it.

[Course overview](../OVERVIEW.md) | [Previous chapter](../chapter_1/LESSON.md) | [Next: Placement And Moving Teams](../chapter_3/LESSON.md) | [Complete solution](cheat_sheet/SOLUTION.md)
