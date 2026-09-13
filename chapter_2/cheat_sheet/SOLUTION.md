# Chapter 2 Solution: An Automatic Duel

[Course overview](../../OVERVIEW.md) | [Back to the lesson](../LESSON.md)

This is the complete reference for the completed Chapter 2 milestone in Godot 4.7.2. It includes the editor setup as well as the scripts: copying scripts alone does not create nodes or connect signals. The learner reported all milestone checks passing. Separate checks of these exact snippets are described in the [validation notes](../../OVERVIEW.md#validation); the tables below are the hands-on checks to reproduce in your own project.

Work in the Chapter 1 project using the Godot editor. No terminal or assistant project access is needed. Keep Home and the town movement unchanged.

## Project Settings

Open **Project > Project Settings**. Use its search field or enable Advanced Settings to locate hidden fields. Preserve these Chapter 1 settings:

| Setting | Value |
| --- | --- |
| Application > Run > Main Scene | `res://scenes/home.tscn` |
| Display > Window > Size > Viewport Width | `640` |
| Display > Window > Size > Viewport Height | `360` |
| Display > Window > Size > Window Width Override | `1280` |
| Display > Window > Size > Window Height Override | `720` |
| Display > Window > Stretch > Mode | `viewport` |
| Display > Window > Stretch > Aspect | `keep` |
| Display > Window > Stretch > Scale Mode | `integer` |
| Rendering > Textures > Canvas Textures > Default Texture Filter | `Nearest` |

`res://` is Godot's project folder. All positions below use the 640 by 360 logical viewport, not the larger displayed window.

**F5 / Run Project** must still run Home. Select the Battle scene tab and use **F6 / Run Current Scene** for the duel. You can use the equivalent run toolbar buttons if function keys are unavailable.

The final duel needs **no custom input action**. Start and Restart are ordinary Buttons. If you created `debug_damage` bound to Space during the lesson, remove Board's temporary `_unhandled_input()` function. You may delete the action under **Project > Project Settings > Input Map**, but leaving an unused action is harmless. Keep Chapter 1's existing inputs. Normal UI keyboard activation of a focused Button can still use Space; that must not directly damage a unit.

## Create The Scenes

Use **Scene > New Scene > 2D Scene** for each Node2D root. Rename nodes with F2; add children with **Add Child Node** (Ctrl+A). Save with Ctrl+S. Node names are case-sensitive.

Create the reusable BattleUnit first. In the Scene dock, select its root, choose **Attach Script**, and use `res://scenes/battle_unit.gd`. Replace any generated template with the complete BattleUnit script below. Save before expecting exported properties to appear in the Inspector.

### Reusable Unit

Save this scene as `res://scenes/battle_unit.tscn`:

```text
BattleUnit (Node2D) [battle_unit.gd]
+-- Sprite2D (Sprite2D)
`-- HealthLabel (Label)
```

The tree characters illustrate parent/child relationships. Sprite2D and HealthLabel are both direct children of BattleUnit.

| Node | Property | Value |
| --- | --- | --- |
| BattleUnit | Position / rotation / scale | `(0, 0)` / `0` / `(1, 1)` |
| BattleUnit | Cell | `(0, 0)` in the reusable source scene |
| BattleUnit | Display Color | White in the reusable source scene |
| BattleUnit | Max Health | `20` |
| BattleUnit | Attack Damage | `6` |
| Sprite2D | Position / rotation / scale | `(0, 0)` / `0` / `(1, 1)` |
| Sprite2D | Offset | `(0, 0)` |
| Sprite2D | Offset > Centered | On |
| Sprite2D | Texture | New `GradientTexture2D`, Width `32`, Height `32` |
| Sprite2D | Texture > Gradient | New `Gradient`, both endpoint colors white, full alpha |
| Sprite2D | Modulate / Self Modulate in editor | White; script applies the instance tint at runtime |
| HealthLabel | Anchors | Top-left; all four anchor values `0` |
| HealthLabel | Layout > Transform > Position | `(-24, -34)` |
| HealthLabel | Layout > Transform > Size | `(48, 18)` |
| HealthLabel | Text | `20 / 20` as an editor placeholder |
| HealthLabel | Horizontal Alignment | Center |
| HealthLabel | Theme Overrides > Font Sizes > Font Size | Enabled, `12` |
| HealthLabel | Theme Overrides > Fonts | Leave unset; use fallback font |

For the texture, select Sprite2D and open the Texture property's dropdown. Choose **New GradientTexture2D**, then click that resource to expand it. Set Width and Height to 32. Create/expand its Gradient; select each of the two endpoint markers and set its color to white with full opacity. A default black-to-white gradient is not the intended marker.

For Label layout, select the top-left anchor preset before entering Position and Size under **Layout > Transform**. Do not put this Label in a Container. The marker's origin is the cell center, not a town character's feet. It needs no CharacterBody2D, collision shape, font asset, or external image.

### Battle Scene

Create a new 2D scene, name its root `Battle`, and save as `res://scenes/battle.tscn`. Add Board and the four sibling nodes shown here:

```text
Battle (Node2D) [battle.gd]
+-- Board (Node2D) [board.gd]
|   +-- Friendly (instance of battle_unit.tscn)
|   |   +-- Sprite2D
|   |   `-- HealthLabel
|   `-- Enemy (instance of battle_unit.tscn)
|       +-- Sprite2D
|       `-- HealthLabel
+-- AttackTimer (Timer)
+-- StartButton (Button)
+-- RestartButton (Button)
`-- ResultLabel (Label)
```

To add the two instances, select Board and use **Instantiate Child Scene**, choosing `battle_unit.tscn`. Name the first Friendly and repeat for Enemy. Do not manually rebuild or separately edit their internal children; the source scene supplies them. Both instance roots must be directly beneath Board.

Set the following in the Inspector. Save scripts before editing their exported settings. Require **positive** Max Health and Attack Damage on both instances. Plain `@export` does not enforce that requirement automatically.

| Node | Property | Value |
| --- | --- | --- |
| Battle | Position / rotation / scale | `(0, 0)` / `0` / `(1, 1)` |
| Board | Position | `(176, 24)` |
| Board | Rotation / scale | `0` / `(1, 1)` |
| Board | Columns / Rows / Cell Size | `6` / `6` / `48.0` |
| Friendly | Cell | `(2, 2)` |
| Friendly | Display Color | `#4a90d9`, alpha 1 |
| Friendly | Max Health / Attack Damage | `20` / `6` |
| Enemy | Cell | `(3, 2)` |
| Enemy | Display Color | `#e59b45`, alpha 1 |
| Enemy | Max Health / Attack Damage | `20` / `6` |
| Both unit instances | Position / rotation / scale in editor | `(0, 0)` / `0` / `(1, 1)`; Board sets position at runtime |
| AttackTimer | Wait Time | `1.0` seconds |
| AttackTimer | One Shot | Off |
| AttackTimer | Autostart | Off |
| AttackTimer | Process Callback | Physics |
| AttackTimer | Paused | Off |
| StartButton | Text / Disabled | `Start` / Off |
| StartButton | Position / Size | `(176, 324)` / `(72, 32)` |
| RestartButton | Text / Disabled | `Restart` / On |
| RestartButton | Position / Size | `(256, 324)` / `(88, 32)` |
| ResultLabel | Text | `Ready` |
| ResultLabel | Position / Size | `(352, 324)` / `(220, 32)` |
| ResultLabel | Horizontal / Vertical Alignment | Left / Center |
| ResultLabel | Theme Overrides > Font Sizes > Font Size | Enabled, `16` |

Use top-left anchors, all anchor values zero, for StartButton, RestartButton, and ResultLabel. Enter positions and sizes under **Layout > Transform**. Leave button fonts at their default and ResultLabel's font resource unset for fallback font. Keep all nodes visible and leave other properties at defaults.

Attach `res://scenes/board.gd` to Board and `res://scenes/battle.gd` to Battle. Use the complete scripts below, with no extra template methods. **Do not run the completed Board or Battle script until all of their referenced children exist.** If reconstructing the lesson incrementally, Board's child placement `_ready()` belongs after unit instancing, not in the initial board-only step.

At runtime, Board places Friendly at local `(120, 120)` and Enemy at `(168, 120)`. Board's own position is applied by the scene tree. Do not add it to the helper's result or change these assignments to `global_position`.

## Complete Scripts

There are exactly three scripts in this solution. They use four spaces per indentation level. If combining snippets with tab-indented code, use the script editor's Edit > Indentation menu to make each file consistent. The editor-created signal connections are a separate required step after them.

### scenes/board.gd

```gdscript
extends Node2D

@export var columns: int = 6
@export var rows: int = 6
@export var cell_size: float = 48.0


func _ready() -> void:
    $Friendly.position = cell_to_local($Friendly.cell)
    $Enemy.position = cell_to_local($Enemy.cell)


func _draw() -> void:
    for column in columns:
        for row in rows:
            var top_left := Vector2(column * cell_size, row * cell_size)
            var rectangle := Rect2(top_left, Vector2(cell_size, cell_size))
            draw_rect(rectangle, Color.BLACK, true)
            draw_rect(rectangle, Color.SLATE_GRAY, false, 1.0)


func cell_to_local(cell: Vector2i) -> Vector2:
    return (Vector2(cell) + Vector2(0.5, 0.5)) * cell_size
```

### scenes/battle_unit.gd

```gdscript
extends Node2D

@export var cell: Vector2i
@export var display_color: Color = Color.WHITE
@export var max_health: int = 20
@export var attack_damage: int = 6

var health: int


func _ready() -> void:
    $Sprite2D.self_modulate = display_color
    health = max_health
    update_health_display()


func update_health_display() -> void:
    $HealthLabel.text = "%d / %d" % [health, max_health]


func take_damage(amount: int) -> void:
    if amount == 0 or health == 0:
        return

    health = clampi(health - amount, 0, max_health)
    update_health_display()
```

### scenes/battle.gd

```gdscript
extends Node2D

enum Phase { PREPARATION, FIGHTING, RESULTS }

var curr_phase: Phase = Phase.PREPARATION


func _ready() -> void:
    reset_battle()


func reset_battle() -> void:
    $AttackTimer.stop()
    curr_phase = Phase.PREPARATION

    $Board/Friendly.health = $Board/Friendly.max_health
    $Board/Friendly.update_health_display()
    $Board/Enemy.health = $Board/Enemy.max_health
    $Board/Enemy.update_health_display()

    $StartButton.disabled = false
    $RestartButton.disabled = true
    $ResultLabel.text = "Ready"


func _on_start_button_pressed() -> void:
    if curr_phase != Phase.PREPARATION:
        return

    curr_phase = Phase.FIGHTING
    $StartButton.disabled = true
    $ResultLabel.text = "Fighting"
    $AttackTimer.start()


func finish_battle(winner: String) -> void:
    $ResultLabel.text = "Winner: %s" % winner
    $AttackTimer.stop()
    curr_phase = Phase.RESULTS
    $RestartButton.disabled = false


func _on_attack_timer_timeout() -> void:
    if curr_phase != Phase.FIGHTING:
        return

    var enemy = $Board/Enemy
    var friendly = $Board/Friendly

    enemy.take_damage(friendly.attack_damage)
    if enemy.health == 0:
        finish_battle("Friendly")
        return

    friendly.take_damage(enemy.attack_damage)
    if friendly.health == 0:
        finish_battle("Enemy")
        return


func _on_restart_button_pressed() -> void:
    if curr_phase != Phase.RESULTS:
        return

    reset_battle()
```

## Connect The Signals

Save the scripts first. Signal connections belong to `battle.tscn`, not to the handler names in `battle.gd`. Use the editor **only**, with exactly one connection for each row:

| Sending node | Signal | Receiver | Exact receiver method |
| --- | --- | --- | --- |
| StartButton | `pressed` | Battle | `_on_start_button_pressed` |
| RestartButton | `pressed` | Battle | `_on_restart_button_pressed` |
| AttackTimer | `timeout` | Battle | `_on_attack_timer_timeout` |

1. Open Battle and select StartButton in the Scene dock.
2. Open the **Signals** dock beside the Inspector; in layouts with a Node dock, choose **Node > Signals**.
3. Double-click `pressed()` to open the connection dialog.
4. Select **Battle**, the root with `battle.gd`, as the receiving node. Set Receiver Method to `_on_start_button_pressed`. Expand Advanced if necessary to edit the method name.
5. Click Connect. The complete script already contains this function; do not paste another copy or append a second stub.
6. Repeat for RestartButton's `pressed()` and AttackTimer's `timeout()`, using the exact methods in the table.
7. Save Battle with Ctrl+S. Inspect each signal's listed connections to confirm its receiver and method.

Do not add code-based `.connect(...)` calls. If you previously connected a signal to a different method, disconnect the incorrect connection in the Signals dock rather than leaving it alongside the correct one. A matching handler name without a connection does nothing.

## Why This Version Works

- `_draw()` records static drawing commands; Godot retains them. There is no `@tool`, `_process()`, continuous redraw, or font drawing. A future runtime drawing change would request `queue_redraw()`.
- `for column in columns` iterates integers from zero through columns minus one. The nested row loop draws 36 cells, each with a black fill before its slate-gray outline.
- `cell_to_local()` converts a whole-number cell address to its center in Board's coordinate space. Direct children use that `position` unchanged; the parent offset is not added twice.
- Each Sprite2D has its own tint and each BattleUnit has its own health variable. Sharing a white GradientTexture2D is safe because the code does not mutate that resource. Color is presentation, not team identity.
- `_ready()` initializes health after Inspector overrides are applied. Child callbacks run before parent callbacks, so Battle's initial reset can use both units and their labels safely.
- `take_damage` has a provisional signed contract: positive harms, zero is ignored, negative heals an active unit up to maximum, and zero health cannot revive. This keeps the chosen lesson behavior without deciding the final game's healing design. Automatic attacks only pass positive damage.
- `return` exits a rejected handler or a resolved attack sequence. `pass` would merely do nothing and let execution continue, allowing unwanted actions afterward.
- A single repeating one-second Timer supplies the shared attack beat. Friendly attacks first on every tick. A just-defeated Enemy cannot retaliate. This is an explicit first-strike advantage, not simultaneous combat or individual attack speeds.
- Health-label changes and the winner label are the basic visible feedback. Units remain visible at zero; there is no `queue_free()` or animation dependency.
- Battle resets the two existing units directly. Stopping the timer is centralized in `reset_battle()`, which is called at initial readiness and on Restart. Do not call `_ready()` manually or add a per-unit reset helper to match this reference.
- Reset retains max health, attack damage, cells, and colors. It restores health, display text, phase, buttons, and an inactive timer. The next Start waits a fresh full interval. No movement means there are no moved positions to restore.

## Reproduction Checks

Perform these checks in your own editor. The [automated validation](../../OVERVIEW.md#validation) of the reference code does not verify your local node setup, signal connections, or display.

### Board And Unit Setup

1. Run Battle with F6. The board has six rows and six columns. The markers are blue and orange with readable `20 / 20` labels. F5 still opens Home.
2. Stop, change Board's Cell Size to 40, and rerun. Friendly's local center becomes `(100, 100)` and Enemy's becomes `(140, 100)`. View them under the Scene dock's Remote tree if desired. Restore 48, whose centers are `(120, 120)` and `(168, 120)`.
3. Move Board temporarily and rerun. Its grid and children should move together. Restore `(176, 24)` afterward.
4. Change only Friendly's Display Color and rerun. Enemy should not change. Restore Friendly to `#4a90d9`.
5. Override Enemy's Max Health to 30 in the saved scene and rerun. Its initial label is `30 / 30`. Restore 20.

### Health Contract

The lesson's temporary Space handler tests these cases before automatic combat. The final scripts intentionally do not contain it. The [health exercise](../LESSON.md#exercise-3-health-you-can-test) explains the Input Map setup and short temporary call sequences for checking healing and non-revival without adding a permanent test UI.

| Starting health | Maximum | Amount | Expected health |
| --- | --- | --- | --- |
| 20 | 20 | 6 | 14 |
| 14 | 20 | 6 | 8 |
| 8 | 20 | 6 | 2 |
| 2 | 20 | 6 | 0 |
| 0 | 20 | 6 | 0 |
| 14 | 20 | 0 | 14 |
| 14 | 20 | -6 | 20 |
| 20 | 20 | -6 | 20 |
| 0 | 20 | -6 | 0 |

Only the receiving unit changes. The other unit remains untouched. Encounter reset can restore a zero-health unit because it assigns a fresh starting health directly; that is not revival through `take_damage`.

### Both Winner Paths

Leave both Attack Damage values at 6. Run once with Enemy Max Health 20 and again with Enemy Max Health 25. Friendly Max Health stays 20. Stop the scene before changing saved Inspector values.

| Timer tick | Friendly / Enemy, Enemy max 20 | Friendly / Enemy, Enemy max 25 |
| --- | --- | --- |
| Before Start | `20 / 20` | `20 / 25` |
| 1 | `14 / 14` | `14 / 19` |
| 2 | `8 / 8` | `8 / 13` |
| 3 | `2 / 2` | `2 / 7` |
| 4 | `2 / 0` | `0 / 1` |
| Result text | `Winner: Friendly` | `Winner: Enemy` |

In this table, the slash separates the **two units' current health**, not one label's current/maximum format. Each on-screen label still displays its own current and maximum values.

### Restart And Timing

1. Run Battle and wait several seconds without Start. ResultLabel stays `Ready` and neither health value changes. Start is enabled and Restart disabled.
2. Press Start once. The label becomes `Fighting`; both buttons are disabled. First damage arrives after about one second, then on each subsequent one-second beat. Repeated Start attempts must not restart or accelerate the timer.
3. At defeat, the winner appears once, Start remains disabled, and Restart becomes enabled. Wait several seconds: there are no further attacks and no retaliation from a just-defeated unit.
4. Press Restart. Both health labels refill to their own maximums, the label becomes `Ready`, Start is enabled, and Restart is disabled. Nothing attacks until Start.
5. Complete at least three Start -> result -> Restart cycles. Every new Start waits the full first interval; unchanged stats produce the same health sequence and result. There must be no duplicate units or extra attacks.
6. Check restarting after both winner paths, then restore Enemy Max Health to 20 and save. Restart is results-only, not a mid-fight reset.

## Common Setup Problems

| Problem | Fix |
| --- | --- |
| Blank board in the 2D editor | Expected without `@tool`. Run the scene to see custom drawing. If blank while running, check the attached script, positive exported dimensions, visible state, and Board position. |
| Missing-node or null-reference error | Compare the tree and names exactly. The source script's root determines lookup paths. Board needs direct children Friendly and Enemy; Battle needs Board and all four siblings. Create children before running final `_ready()` methods. |
| Units not centered | Keep Sprite2D centered, offset and position zero, scale one. Use the helper's local result without another Board offset. |
| Tint looks wrong or both instances recolor | Both Gradient endpoints must be white with full alpha. Change instance Display Color, not the shared Gradient resource. |
| Exported settings absent | Save the script, fix parsing errors, and select the correct scripted instance root. Root exports do not require Editable Children. |
| Wrong initial health | Use `health = max_health` in `_ready()`, not a dependent variable initializer. Confirm the intended instance override is saved. |
| Missing or clipped health text | Match `HealthLabel`, its position `(-24, -34)`, size `(48, 18)`, font size 12, and center alignment. Leave its fallback font available. |
| Button does nothing | Verify the signal is actually connected to Battle and its scene is saved. Naming the handler alone is insufficient. Check the phase and Disabled state. |
| Timer is inert or fires only once | Connect timeout, call start on Start, keep One Shot off, Wait Time 1.0, Process Callback Physics, and Paused off. Autostart remains off. |
| Extra damage or accelerated fights | Remove all debug damage handlers and any per-frame damage code. Inspect for extra signal connections or code connections alongside editor wiring. |
| Fighting continues after defeat | Use return after finish_battle, not pass; keep the timeout phase guard and timer stop. |
| Reset keeps old damage or timing | Compare reset_battle, both label refresh calls, phase and button restoration, and the centralized timer stop. Do not manually call `_ready()`. |

Stop at this milestone: a predictable, stationary automatic duel with results and repeatable reset. No additional systems are required to reproduce the completed chapter.

[Course overview](../../OVERVIEW.md) | [Back to the lesson](../LESSON.md)
