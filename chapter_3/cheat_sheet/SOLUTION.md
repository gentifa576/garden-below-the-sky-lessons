# Chapter 3 Solution: Placement And Moving Teams

[Course overview](../../OVERVIEW.md) | [Back to the lesson](../LESSON.md)

This is the complete Godot 4.7.2 reference for milestone 3. It replaces the three Chapter 2 scripts, not Home or its movement script. Build the scenes and connections below as well as the code. No terminal, external assets, Codex, or assistant access to your machine is required.

## Editor Setup

Use your Chapter 2 project, or reconstruct its scenes using these settings. Keep Home as `res://scenes/home.tscn`, the main scene. **F5 / Run Project** opens Home; select Battle and use **F6 / Run Current Scene** here. `res://` is your own project folder. Create the `scenes` folder in Godot's FileSystem dock if needed. Use Compatibility rendering.

In Project > Project Settings, retain viewport width/height `640 / 360`, window width/height overrides `1280 / 720`, stretch mode `viewport`, stretch aspect `keep`, stretch scale mode `integer`, and Default Texture Filter `Nearest`. Search settings or enable Advanced Settings if needed. Do not double any positions to match the larger window.

### Reusable Unit

Create/open `res://scenes/battle_unit.tscn`. Use a Node2D root named BattleUnit with `res://scenes/battle_unit.gd` attached. Add children with Ctrl+A; rename with F2. Save scripts and scenes with Ctrl+S.

```text
BattleUnit (Node2D) [battle_unit.gd]
+-- Sprite2D (Sprite2D)
`-- HealthLabel (Label)
```

| Node | Settings |
| --- | --- |
| BattleUnit | Position `(0, 0)`, rotation `0`, scale `(1, 1)`; Team `ALLY`, Cell `(0, 0)`, Display Color white, Max Health `20`, Attack Damage `6`, Attack Range `1` |
| Sprite2D | Position/offset `(0, 0)`, scale `(1, 1)`, Centered on, Modulate and Self Modulate white |
| Sprite2D Texture | New GradientTexture2D, Width/Height `32 / 32`; its Gradient has two white endpoints, both alpha `1` |
| HealthLabel | Top-left anchors (all `0`), position `(-24, -34)`, size `(48, 18)`, text `20 / 20`, horizontal alignment Center, font size override `12`, fallback font |
| HealthLabel Mouse > Filter | `Ignore` (so health text does not swallow board clicks) |

Select Sprite2D, open Texture's dropdown, create GradientTexture2D and expand it. Create/expand its Gradient and set both endpoint colors to white, not the default black-to-white. Tints belong to unit instances, not that shared resource. Label position/size are under Layout > Transform; do not use a Container. Keep the root and its children visible in the saved scene. Defeat hides the root at runtime.

### Battle Scene

Create/open `res://scenes/battle.tscn` with this exact tree. Attach scripts via the Scene dock's Attach Script button. To add Hero, select Board, use Instantiate Child Scene, and choose `battle_unit.tscn`. Do not duplicate Friendly's instance overrides unintentionally.

```text
Battle (Node2D) [battle.gd]
+-- Board (Node2D) [board.gd]
|   +-- Friendly (instance of battle_unit.tscn)
|   +-- Enemy (instance of battle_unit.tscn)
|   `-- Hero (instance of battle_unit.tscn)
+-- AttackTimer (Timer)
+-- StartButton (Button)
+-- RestartButton (Button)
`-- ResultLabel (Label)
```

Each unit instance contains the Sprite2D and HealthLabel from its source scene. **Board has exactly three direct children, in order Friendly, Enemy, Hero.** Drag them in the Scene dock to fix order, keeping the same parent. No Labels, timers, decoration nodes, or grouping nodes go directly under Board. Child order is action order and breaks equal-distance target ties. Names identify nodes; the Team export, not the name or color, identifies a team.

| Node | Property / value |
| --- | --- |
| Battle | Position `(0, 0)`, rotation `0`, scale `(1, 1)`, Max Ticks `60` |
| Board | Position `(176, 24)`, rotation `0`, scale `(1, 1)`, Columns `6`, Rows `6`, Cell Size `48.0` |
| Friendly | Team `ALLY`, Cell `(0, 1)`, Max Health `20`, Attack Damage `6`, Attack Range `1`, Display Color `#4a90d9` |
| Enemy | Team `ENEMY`, Cell `(3, 0)`, Max Health `20`, Attack Damage `6`, Attack Range `1`, Display Color `#e59b45` |
| Hero | Team `ALLY`, Cell `(2, 0)`, Max Health `32`, Attack Damage `4`, Attack Range `1`, Display Color `#00bd7c` |
| All unit roots | Position `(0, 0)` in editor (Board sets it at runtime), rotation `0`, scale `(1, 1)`, tint alpha `1` |
| AttackTimer | Wait Time `1.0`, One Shot off, Autostart off, Paused off, Process Callback Physics |
| StartButton | Text `Start`, Disabled off, position `(176, 324)`, size `(72, 32)` |
| RestartButton | Text `Restart`, Disabled on, position `(256, 324)`, size `(88, 32)` |
| ResultLabel | Text `Ready`, position `(352, 324)`, size `(220, 32)`, horizontal Left, vertical Center, font size override `16`, fallback font, Mouse Filter Ignore |

Use top-left anchors, all zero, for the three controls directly under Battle. Leave button Mouse Filter at Stop and their fonts at defaults. Keep positive health/damage settings and nonoverlapping, in-bounds starting cells. Columns and Rows remain six for this chapter; Cell Size must be positive. Board geometry is configured before `_ready()`; runtime resizing is not implemented. We retain Chapter 2's **48**, although the original game's saved scene uses 40. At 48 the board is 288 pixels square, ending at Y 312, above the buttons. No original game files are needed.

### Mouse Input

In Project > Project Settings > Input Map, add an action named exactly `select`. Use its plus button to add a Mouse Button event, choose **Left Mouse Button**, and confirm. Preserve Home's movement actions. Remove any old debug-damage handler; `select` is the only new action. Input flows through Board's `_unhandled_input`, then a custom signal, then Battle's phase-guarded placement handler. Buttons handle their own clicks first.

## Complete Scripts

Replace the entire contents of each matching Chapter 2 script. There are exactly three complete GDScript blocks below. Save `battle_unit.gd` first so Godot recognizes `BattleUnit` as a type. Use four spaces per indentation level, or convert the whole file consistently using Edit > Indentation. Do not run until the tree exists and signals are connected.

### scenes/board.gd

```gdscript
extends Node2D

signal select_cell(cell: Vector2i)

@export var columns: int = 6
@export var rows: int = 6
@export var cell_size: float = 48.0

var selected_cell: Vector2i = Vector2i(-1, -1)
var pathfinder: AStarGrid2D = AStarGrid2D.new()


func _ready() -> void:
    for unit in get_units():
        unit.position = cell_to_local(unit.cell)
    pathfinder.region = Rect2i(0, 0, columns, rows)
    pathfinder.diagonal_mode = AStarGrid2D.DIAGONAL_MODE_NEVER
    pathfinder.default_compute_heuristic = AStarGrid2D.HEURISTIC_MANHATTAN
    pathfinder.default_estimate_heuristic = AStarGrid2D.HEURISTIC_MANHATTAN
    pathfinder.update()


func get_units() -> Array[BattleUnit]:
    var units: Array[BattleUnit] = []
    for unit in get_children():
        units.append(unit)
    return units


func cell_to_local(cell: Vector2i) -> Vector2:
    return (Vector2(cell) + Vector2(0.5, 0.5)) * cell_size


func local_to_cell(point: Vector2) -> Vector2i:
    return Vector2i((point / cell_size).floor())


func is_inside_board(cell: Vector2i) -> bool:
    return Rect2i(0, 0, columns, rows).has_point(cell)


func _unhandled_input(event: InputEvent) -> void:
    if event.is_action_pressed("select"):
        var cell := local_to_cell(get_local_mouse_position())
        if is_inside_board(cell):
            select_cell.emit(cell)


func highlight_cell(cell: Vector2i) -> void:
    selected_cell = cell
    queue_redraw()


func get_unit_at(cell: Vector2i) -> BattleUnit:
    for unit in get_units():
        if unit.health > 0 and unit.cell == cell:
            return unit
    return null


func count_alive_team() -> Dictionary:
    var counts := {BattleUnit.Team.ALLY: 0, BattleUnit.Team.ENEMY: 0}
    for unit in get_units():
        if unit.health > 0:
            counts[unit.team] += 1
    return counts


func find_path(start: Vector2i, end: Vector2i) -> Array[Vector2i]:
    if not is_inside_board(start) or not is_inside_board(end):
        return []
    if get_unit_at(end) != null:
        return []

    # Rebuild occupancy for this query, not just at the start of a tick.
    pathfinder.fill_solid_region(pathfinder.region, false)
    for unit in get_units():
        if unit.health > 0 and unit.cell != start and is_inside_board(unit.cell):
            pathfinder.set_point_solid(unit.cell, true)
    return pathfinder.get_id_path(start, end, false).slice(1)


func _draw() -> void:
    var size := Vector2(cell_size, cell_size)
    for column in columns:
        for row in rows:
            var top_left := Vector2(column * cell_size, row * cell_size)
            var rectangle := Rect2(top_left, size)
            draw_rect(rectangle, Color.BLACK, true)
            draw_rect(rectangle, Color.SLATE_GRAY, false, 1.0)
    if is_inside_board(selected_cell):
        draw_rect(Rect2(Vector2(selected_cell) * cell_size, size), Color.GOLD, false, 3.0)
```

### scenes/battle_unit.gd

```gdscript
class_name BattleUnit
extends Node2D

enum Team { ALLY, ENEMY }

@export var team: Team = Team.ALLY
@export var cell: Vector2i
@export var display_color: Color = Color.WHITE
@export var max_health: int = 20
@export var attack_damage: int = 6
@export_range(1, 5) var attack_range: int = 1

var health: int


func _ready() -> void:
    $Sprite2D.self_modulate = display_color
    health = max_health
    update_health_display()


func update_health_display() -> void:
    $HealthLabel.text = "%d / %d" % [health, max_health]
    visible = health > 0


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

@export_range(1, 3600) var max_ticks: int = 60

var elapsed_ticks: int = 0
var curr_phase: Phase = Phase.PREPARATION
var selected_unit: BattleUnit = null
var starting_cells: Dictionary[BattleUnit, Vector2i] = {}


func _ready() -> void:
    reset_battle()


func reset_battle() -> void:
    $AttackTimer.stop()
    for unit in $Board.get_units():
        if starting_cells.has(unit):
            unit.cell = starting_cells[unit]
        unit.position = $Board.cell_to_local(unit.cell)
        unit.health = unit.max_health
        unit.update_health_display()
    elapsed_ticks = 0
    curr_phase = Phase.PREPARATION
    $StartButton.disabled = false
    $RestartButton.disabled = true
    $ResultLabel.text = "Ready"
    clear_selection()
    starting_cells.clear()


func _on_start_button_pressed() -> void:
    if curr_phase != Phase.PREPARATION:
        return
    starting_cells.clear()
    for unit in $Board.get_units():
        starting_cells[unit] = unit.cell
    elapsed_ticks = 0
    clear_selection()
    curr_phase = Phase.FIGHTING
    $StartButton.disabled = true
    $RestartButton.disabled = true
    $ResultLabel.text = "Fighting"
    $AttackTimer.start()


func _on_restart_button_pressed() -> void:
    if curr_phase != Phase.RESULTS:
        return
    reset_battle()


func clear_selection() -> void:
    selected_unit = null
    $Board.highlight_cell(Vector2i(-1, -1))


func _on_board_select_cell(cell: Vector2i) -> void:
    if curr_phase != Phase.PREPARATION or not $Board.is_inside_board(cell):
        return
    var unit = $Board.get_unit_at(cell)
    if unit != null and unit.team == BattleUnit.Team.ALLY:
        selected_unit = unit
        $Board.highlight_cell(cell)
        return
    if unit != null or selected_unit == null or cell.x >= $Board.columns / 2:
        return
    selected_unit.cell = cell
    selected_unit.position = $Board.cell_to_local(cell)
    $Board.highlight_cell(cell)


func distance(start: Vector2i, end: Vector2i) -> int:
    return absi(end.x - start.x) + absi(end.y - start.y)


func is_in_range(attacker: BattleUnit, target: BattleUnit) -> bool:
    if attacker == null or target == null:
        return false
    if attacker.health <= 0 or target.health <= 0 or attacker.team == target.team:
        return false
    if not $Board.is_inside_board(attacker.cell) or not $Board.is_inside_board(target.cell):
        return false
    return distance(attacker.cell, target.cell) <= attacker.attack_range


func find_target(attacker: BattleUnit) -> BattleUnit:
    if attacker == null or attacker.health <= 0:
        return null
    var closest: BattleUnit = null
    var closest_distance: int = 2147483647
    for unit in $Board.get_units():
        if not is_in_range(attacker, unit):
            continue
        var current_distance := distance(attacker.cell, unit.cell)
        if current_distance < closest_distance:
            closest_distance = current_distance
            closest = unit
    return closest


func find_path_to_target(attacker: BattleUnit, target: BattleUnit) -> Array[Vector2i]:
    var shortest: Array[Vector2i] = []
    if attacker == null or target == null:
        return shortest
    if attacker.health <= 0 or target.health <= 0 or attacker.team == target.team:
        return shortest
    if not $Board.is_inside_board(attacker.cell) or not $Board.is_inside_board(target.cell):
        return shortest
    if is_in_range(attacker, target):
        return shortest
    for row in $Board.rows:
        for column in $Board.columns:
            var cell := Vector2i(column, row)
            if distance(cell, target.cell) > attacker.attack_range:
                continue
            if $Board.get_unit_at(cell) != null:
                continue
            var path: Array[Vector2i] = $Board.find_path(attacker.cell, cell)
            if path.is_empty():
                continue
            if shortest.is_empty() or path.size() < shortest.size():
                shortest = path
    return shortest


func closing_path(attacker: BattleUnit) -> Array[Vector2i]:
    var shortest: Array[Vector2i] = []
    if attacker == null or attacker.health <= 0 or find_target(attacker) != null:
        return shortest
    for unit in $Board.get_units():
        if unit.health <= 0 or unit.team == attacker.team:
            continue
        var path := find_path_to_target(attacker, unit)
        if path.is_empty():
            continue
        if shortest.is_empty() or path.size() < shortest.size():
            shortest = path
    return shortest


func check_battle_end() -> bool:
    var counts: Dictionary = $Board.count_alive_team()
    return counts[BattleUnit.Team.ALLY] == 0 or counts[BattleUnit.Team.ENEMY] == 0


func finish_battle() -> void:
    var counts: Dictionary = $Board.count_alive_team()
    var allies_win: bool = counts[BattleUnit.Team.ALLY] > 0 and counts[BattleUnit.Team.ENEMY] == 0
    $ResultLabel.text = "Allies win" if allies_win else "Player loss"
    $AttackTimer.stop()
    curr_phase = Phase.RESULTS
    $StartButton.disabled = true
    $RestartButton.disabled = false
    clear_selection()


func _on_attack_timer_timeout() -> void:
    if curr_phase != Phase.FIGHTING:
        return
    if check_battle_end() or elapsed_ticks >= max_ticks:
        finish_battle()
        return

    elapsed_ticks += 1
    for unit in $Board.get_units():
        if unit.health <= 0:
            continue
        var target := find_target(unit)
        if target != null:
            target.take_damage(unit.attack_damage)
        else:
            var path := closing_path(unit)
            if not path.is_empty() and $Board.get_unit_at(path[0]) == null:
                unit.cell = path[0]
                unit.position = $Board.cell_to_local(unit.cell)
        if check_battle_end():
            finish_battle()
            return

    # Timeout is checked after ALL actions on the last allowed beat.
    if elapsed_ticks >= max_ticks:
        finish_battle()
        return
    $ResultLabel.text = "Ticks left: %d" % (max_ticks - elapsed_ticks)
```

## Connect The Signals

Save all scripts. Retain Chapter 2's three connections and add Board's custom signal. Select the sending node, open Signals (or Node > Signals), double-click the signal, choose Battle as receiver, and enter the exact method name (expand Advanced if needed). Click Connect and save Battle's **scene**, not just its script. Existing methods need no second stub.

| Sender | Signal | Receiver | Method |
| --- | --- | --- | --- |
| Board | `select_cell` | Battle | `_on_board_select_cell` |
| StartButton | `pressed` | Battle | `_on_start_button_pressed` |
| RestartButton | `pressed` | Battle | `_on_restart_button_pressed` |
| AttackTimer | `timeout` | Battle | `_on_attack_timer_timeout` |

Each must be connected **exactly once**. Inspect the listed connections; disconnect obsolete ones rather than leaving duplicates. Do not also use `.connect()` in these scripts. A function's name alone does not wire a signal. If the custom signal is missing, save Board's script and resolve parser errors first.

## Design Contracts

- `class_name BattleUnit` makes the unit script usable as a type. `Array[BattleUnit]` is an ordered list of unit references. Every direct Board child must be a unit. Do not add unrelated children there.
- `null` means no unit. Empty typed arrays mean no movement path. Check both before accessing properties or `path[0]`.
- Manhattan distance is `abs(dx) + abs(dy)`. Range 1 permits orthogonal neighbors, not diagonals. `find_target()` considers only living opponents already in range and keeps the first child on ties by using strict `<`, not `<=`.
- AStarGrid2D uses a cell-ID region, no diagonals, and Manhattan compute/estimate heuristics. `update()` builds it before any solids are set. Each valid path query clears old solids and marks current living occupants, excluding start. Out-of-bounds occupants are skipped defensively. Invalid endpoints or an occupied destination return empty. The engine's path includes start and end; `.slice(1)` removes start so index 0 is the next step. The third argument `false` forbids partial paths: unreachable routes stay empty. Runtime board resizing is outside scope.
- An approach destination is an **empty cell within range**, not the opponent's occupied cell. Scan the whole grid for one opponent and keep the shortest nonempty actual route. Then `closing_path()` compares routes across all living opponents. A Manhattan-near opponent can require a longer detour than another opponent. Empty paths must not beat reachable paths; equal lengths keep the first candidate. If any attack is already possible, there is no closing move. Equal-route geometry is chosen by Godot, not by a promised hand-written direction order.
- Each timer beat visits Friendly, Enemy, Hero. Each living unit attacks **or** moves one orthogonal cell **or** waits. A move never grants an attack on that same beat. Later units see earlier moves and deaths. Team elimination stops immediately, preventing retaliation.
- The course API uses `max_ticks` and `elapsed_ticks`, not the original game's `max_time`/`curr_time`. A beat is counted before its action loop. `check_battle_end()` checks only elimination; timeout is checked after the loop. With a one-second timer, 60 beats normally take about 60 seconds, not a wall-clock deadline. Hero may land a winning hit on beat 60. `finish_battle()` has no winner argument now: only a surviving ally with no surviving enemy wins. Both teams alive at timeout or neither team alive means **Player loss**, never a draw. Label values are Strings, including the formatted countdown, not bare integers.
- Zero HP now hides the root and label, unlike Chapter 2, but does not free the node. Living units alone occupy cells and count toward results. Signed `take_damage` remains: positive harms, zero does nothing, negative heals living units up to maximum, and zero-health units cannot revive that way. Reset directly restores health and visibility; no healing gameplay is added.
- Start snapshots **all three** cells, including Enemy's, after placement. Reset stops the timer, restores that snapshot, centers units, refills health and visibility, resets phase/counter/buttons/selection, and leaves no old snapshot. The next Start records the newly chosen layout. Restart's callback remains results-only; timeout, selection, and Start also have phase guards.

## Reproduction Checks

Use the lesson's [end checkpoint](../LESSON.md#end-checkpoint) and [placement comparison](../LESSON.md#exercise-8-compare-two-layouts) as expected outcomes when reconstructing or debugging. The original milestone passed headless logic tests and its owner waived further manual tests; this reference has separate exact-code validation in the [overview](../../OVERVIEW.md#validation). You need not manually repeat already verified logic checks. Neither test record is a claim that your editor or desktop was observed. Optional visual-feel checks concern readability and the selection outline, not proof of combat arithmetic.

| Layout | Friendly start | Hero start | Enemy start | Enemy first attacks | End Friendly / Hero / Enemy | Result |
| --- | --- | --- | --- | --- | --- | --- |
| A | `(0, 1)` | `(2, 0)` | `(3, 0)` | Hero | `8 / 20 / 0` | Allies win, tick 4 |
| B | `(2, 0)` | `(0, 1)` | `(3, 0)` | Friendly | `2 / 32 / 0` | Allies win, tick 4 |

The automated comparison runs each layout twice and compares every beat's cells, health, and phase, not only its winner. If exploring this yourself, swap allies without overlap by moving Hero to empty `(2, 5)`, moving Friendly to its destination, then moving Hero to its destination. Restart restores the last Start layout. The comparison shows **different damage allocation**, not that Hero-front is universally better: A retains 28 total ally HP; B retains 34. No feeding, growth, permanent death, tower progression, saving, or milestone 4 work is required.

Additional expected cases covered by the exact-script fixture:

| Case | Expected result |
| --- | --- |
| Click outside the board, on Enemy, or on an empty right-half cell | No illegal placement; Enemy never becomes selected |
| Click an ally, then empty `(2, 5)` during preparation | That ally moves to the cell center; selection outline follows |
| Select or Start during fighting/results; Restart before results | Rejected by the relevant phase guard |
| Range 1, diagonal opponent | Not in attack range; orthogonal adjacency is required |
| Blocked destination or fully sealed route | Empty path, never a partial route or an occupied step |
| An earlier actor moves or defeats an occupant | The next path query uses the new living occupancy |
| A nearer opponent requires a longer detour | Prefer the shorter reachable approach to another opponent |
| Unit moves into range | It cannot also attack on that turn |
| Last enemy dies during any actor's turn | Immediate Allies win; no later retaliation |
| Hero kills the last enemy on the last permitted beat | Allies win, not timeout loss |
| Both teams still alive at 60 beats, or no survivors | Player loss, never a draw |
| Restart, choose a different layout, Start, Restart | Full health/visibility and the latest chosen cells, including Enemy's, return; timer, counter, selection, and UI reset |

[Course overview](../../OVERVIEW.md) | [Back to the lesson](../LESSON.md)
