# Chapter 3: Placement And Moving Teams

[Course overview](../OVERVIEW.md) | [Chapter 2](../chapter_2/LESSON.md) | [Complete solution](cheat_sheet/SOLUTION.md)

Turn the stationary duel into a small team battle. Before Start, click an ally and choose its starting cell. During the fight, units choose opponents, walk around occupied cells, and attack when close enough. Add Hero, an ally with more health but a smaller attack, and see how placement changes who absorbs damage.

This chapter records the completed third milestone. Its mechanics and the exact reference scripts have automated validation; the original learner waived further manual tests. The checkpoints are learning and reconstruction aids, not a demand to manually repeat verified work. Headless tests do not observe editor clicks, readability, or visual feel.

## Before You Begin

Use your completed Chapter 2 project in **Godot 4.7.2**, with Compatibility rendering. Leave Home and its movement script unchanged. **F5 / Run Project** still opens Home; select Battle's scene tab and use **F6 / Run Current Scene** here. The equivalent toolbar buttons work too. `res://scenes/` means the `scenes` folder inside your own project, visible in Godot's FileSystem dock.

Preserve Project Settings: viewport `640 / 360`, window overrides `1280 / 720`, stretch mode `viewport`, aspect `keep`, scale mode `integer`, and Default Texture Filter `Nearest`. Search settings or enable Advanced Settings if necessary. Board stays at `(176, 24)`, six columns by six rows, Cell Size `48.0`. At 48, it ends at Y 312, above the controls. The original game's saved board uses 40; that difference changes display size, not cells, range, path lengths, or combat results. Use 48 throughout this course.

### What Changes

| Part | Chapter 3 responsibility |
| --- | --- |
| BattleUnit | Its team, cell, health, damage, range, tint, and health display |
| Board | Draw cells and selection; convert mouse positions; report living occupancy; find grid routes |
| Battle | Permit placement by phase; choose targets and movement; resolve a shared beat; finish and restore a fight |

The timer is still shared, repeating every **one second**. There are no per-unit timers, physics collisions, random choices, or simultaneous attacks. Movement is one immediate cell step, not a smooth animation. This uses Godot's built-in `AStarGrid2D`; you are not writing a pathfinding algorithm from scratch.

### Work In Small Pieces

Stop the running scene before changing saved settings. Save scripts and scenes with **Ctrl+S**. Use one indentation style, such as four spaces; Godot's script editor has **Edit > Indentation** conversion options. Replace old functions rather than adding duplicate names. `return` exits a function, whereas `pass` only does nothing.

The exercises describe functions to write and give small hints, not complete files. The [solution](cheat_sheet/SOLUTION.md) contains the exact final three scripts, scene settings, and connections for targeted comparison. You can consult it without having failed the lesson. For conversational help, use the [optional ChatGPT prompt](#optional-chatgpt-tutor), including its plain-text fallback.

In Exercises 1-3, keep the old timer handler but **do not press Start**: it still implements Chapter 2's stationary duel. In Exercises 4-6, add helpers without calling unfinished helpers from the timer. Exercise 7 replaces the old combat flow as one connected change; save all of that change before running combat. Godot must be able to parse every function even when it is not called, so finish a function or use a temporary valid stub while working, not half-written syntax.

## Exercise 1: Add Hero And Teams

**Goal:** the board contains three independent units in a deliberate order.

1. Open `res://scenes/battle_unit.tscn` from the FileSystem dock. Open its attached `battle_unit.gd` by clicking the script icon beside the root.
2. Add `class_name BattleUnit` above `extends Node2D`. This gives your script a type name that other scripts can use. Save it before using `BattleUnit` elsewhere.
3. Add an enum named `Team` with choices `ALLY` and `ENEMY`. Export `team: Team` defaulting to `Team.ALLY`. Export `attack_range` as an integer defaulting to 1; use the hint below to limit its Inspector field.
4. Keep Chapter 2's tint, maximum-health initialization, label formatting, and signed `take_damage` function. In `update_health_display()`, also set the root's `visible` property from whether health is greater than zero. Defeat now hides the entire marker and label. **Do not free the node**: Restart needs it.
5. Select HealthLabel in the source scene. Under **Mouse > Filter**, choose **Ignore** so the label does not consume board clicks. Keep its top-left anchors, position `(-24, -34)`, size `(48, 18)`, Center alignment, and font size override `12`. Save the scene.
6. Return to Battle. Select Board and use **Instantiate Child Scene**, choosing `battle_unit.tscn`. Rename the new instance `Hero` with F2. Instancing from the source avoids accidentally inheriting Friendly's overrides through duplication.
7. Set the three instance roots from the table. These exported properties do not require Editable Children. Save Battle.

| Instance | Team | Cell | Max Health | Attack Damage | Attack Range | Display Color |
| --- | --- | --- | --- | --- | --- | --- |
| Friendly | ALLY | `(0, 1)` | 20 | 6 | 1 | `#4a90d9` |
| Enemy | ENEMY | `(3, 0)` | 20 | 6 | 1 | `#e59b45` |
| Hero | ALLY | `(2, 0)` | 32 | 4 | 1 | `#00bd7c` |

Keep full tint alpha, unit root positions `(0, 0)` in the saved scene, rotation zero, and scale one. Keep the shared Sprite2D texture white: GradientTexture2D `32 / 32`, both Gradient endpoints white and opaque. Change instance tints, not the shared texture.

The final tree is:

```text
Battle (Node2D) [battle.gd]
+-- Board (Node2D) [board.gd]
|   +-- Friendly (battle_unit.tscn)
|   +-- Enemy (battle_unit.tscn)
|   `-- Hero (battle_unit.tscn)
+-- AttackTimer (Timer)
+-- StartButton (Button)
+-- RestartButton (Button)
`-- ResultLabel (Label)

Each unit instance contains:
+-- Sprite2D (Sprite2D)
`-- HealthLabel (Label)
```

Board must have **only unit direct children**, in exactly **Friendly, Enemy, Hero** order. Drag nodes within the Scene dock to fix order without changing their parent. This order controls turns and breaks target ties. It is not alphabetical sorting. Put no decoration or grouping nodes directly under Board.

### Type And List Hints

An enum is a set of named choices, as with Chapter 2's Phase. Team membership is data: a blue unit is not automatically an ally, and a node named Enemy is not automatically an enemy.

```gdscript
@export_range(1, 5) var attack_range: int = 1
```

Write Board's `get_units() -> Array[BattleUnit]`: create an empty typed array, loop through `get_children()`, append each child, then return the array. An **array** is an ordered list; `Array[BattleUnit]` says each item must be a unit reference. A reference points to the existing node, not a new copy of its health.

```gdscript
var units: Array[BattleUnit] = []
# Inside your child loop, append that child with units.append(unit).
```

Replace Board's two hard-coded `_ready()` position assignments with a loop over `get_units()`. Set each unit's `position` using the existing `cell_to_local(unit.cell)`. Keep drawing unchanged for now. The child-only contract makes this simple; putting a Label under Board is a setup error, not a new unit to silently skip.

**Checkpoint:** without Start, F6 initializes health to `20 / 20`, `20 / 20`, and `32 / 32`. The three local centers at size 48 are `(24, 72)`, `(168, 24)`, and `(120, 24)`. Explain why Hero needs no separate health script. Signed damage still harms with positive amounts, ignores zero, heals living units with negative amounts up to maximum, and cannot revive a defeated unit.

## Exercise 2: Turn Clicks Into Cells

**Goal:** Board can report which valid cell was clicked, without deciding placement policy.

1. Open **Project > Project Settings > Input Map**. Add action `select`, spelled exactly. Click its plus button, add a **Mouse Button** event, choose **Left Mouse Button**, and confirm. Keep Home's movement actions.
2. Confirm Chapter 2's temporary debug-damage handler is gone. Add a new Board `_unhandled_input(event: InputEvent) -> void` for `select`, not for damage.
3. Write Board's `local_to_cell(point: Vector2) -> Vector2i`: divide by Cell Size, floor the coordinates, then convert to `Vector2i`.
4. Write `is_inside_board(cell: Vector2i) -> bool` using `Rect2i(0, 0, columns, rows).has_point(cell)`.
5. Declare Board's custom signal `select_cell(cell: Vector2i)`. In the input handler, react only when `event.is_action_pressed("select")`. Convert `get_local_mouse_position()` and emit the signal only if the resulting cell is inside the board.
6. Declare `selected_cell: Vector2i` initially `Vector2i(-1, -1)`. Write `highlight_cell(cell: Vector2i) -> void` to store the cell and call `queue_redraw()`.
7. After the existing grid loops in `_draw()`, draw a gold unfilled rectangle, width `3.0`, around selected_cell only when it is inside the board. Its top-left is `Vector2(selected_cell) * cell_size`, its size is `Vector2(cell_size, cell_size)`.

### Coordinate And Signal Hints

Mouse screen coordinates include Board's offset; **local** coordinates already remove the parent's transform. Do not subtract `(176, 24)` again. Unlike the center helper, converting a click does not add half a cell.

```gdscript
Vector2i((point / cell_size).floor())
```

This is an expression to return from your helper, not a complete function. `floor()` rounds down: a click at local X `-1` becomes column `-1`, not `0`. Simply converting a small negative fraction to an integer would incorrectly accept a click just outside the left edge. Valid addresses are 0 through 5 on both axes; the right/bottom edge at 288 is already outside.

```gdscript
signal select_cell(cell: Vector2i)
```

That declaration belongs at the script's top level, outside functions. Later, inside the input handler's action-and-bounds checks, use `select_cell.emit(cell)` to send the notification. Do not put the emit call at the top level beside the declaration.

A custom signal is your own notification. Its cell parameter travels to the receiving function. Board says "this cell was clicked"; Battle will decide whether that means selection, movement, or nothing. `_unhandled_input` also lets Buttons handle their clicks before board input. `queue_redraw()` requests drawing again; do not call `_draw()` yourself or add an every-frame redraw loop.

**Checkpoint:** explain why `(47, 47)` becomes `(0, 0)`, `(48, 0)` becomes `(1, 0)`, and `(-1, 0)` must be rejected. Selection has no visible effect yet because Battle has not connected the signal. That connection is the next step, not a missing timer feature.

## Exercise 3: Place Allies Before Start

**Goal:** click an ally to select it, then an empty cell in columns 0, 1, or 2 to place it.

1. Add Board's `get_unit_at(cell: Vector2i) -> BattleUnit`. Search `get_units()` for a unit whose health is above zero and whose cell matches. Return it immediately; if the loop finds none, return `null` after the loop.
2. Add Battle's `selected_unit: BattleUnit = null`. Write `clear_selection()` to assign null and ask Board to highlight `Vector2i(-1, -1)`.
3. Write Battle's `_on_board_select_cell(cell: Vector2i) -> void` using the rules below. Match this exact name for the connection.
4. Save both scripts. Select **Board**, open **Signals** (or **Node > Signals**), double-click `select_cell`, and choose **Battle** as receiver. Enter `_on_board_select_cell`; expand Advanced if needed. Connect once and save Battle's **scene**. If Godot adds a `pass` stub, replace it rather than defining the method twice.
5. Select ResultLabel and set **Mouse > Filter** to **Ignore**. Keep StartButton and RestartButton at **Stop**. HealthLabel should already be Ignore in the source unit scene.

### Placement Rules

| Situation, evaluated in order | Action |
| --- | --- |
| Not PREPARATION, or outside Board | Return without changing anything |
| Clicked a living ALLY | Store its reference, highlight its cell, return |
| Clicked another occupied cell, no ally selected, or column is 3 or greater | Return without moving |
| Otherwise | Move the selected ally's cell and local position; move the highlight too |

An ignored click keeps the previous selection; it does not deselect. Clicking the other ally changes which ally is selected, not a swap. Enemy cannot be selected or moved. Empty cells outside the left half are forbidden **for preparation placement**; combat movement later can use the whole board.

`null` means "no node found/selected." Check for it **before** reading `.team` or `.cell`. A compound condition using `or` stops as soon as a true part is found, so early null guards protect later property access. For six columns, `cell.x >= $Board.columns / 2` rejects the right half. Bounds checking already rejects negative columns and rows.

```gdscript
selected_unit.cell = cell
selected_unit.position = $Board.cell_to_local(cell)
```

These are the two movement assignments inside the permitted branch, not a complete handler. Updating only position makes a marker look moved while combat still uses its old cell; updating only cell leaves the marker behind.

**Checkpoint:** select Hero and place it at empty `(2, 5)`, then return it to `(2, 0)`, if checking your reconstruction. Enemy, occupied destinations, right-half empty cells, and outside clicks must not cause illegal movement. The outline follows selection. Explain why the Board signal alone cannot enforce the PREPARATION rule.

## Exercise 4: Choose An Opponent In Range

**Goal:** combat can ask "can I attack now, and whom?" without moving yet.

In Battle, add these helpers in order:

1. `distance(start: Vector2i, end: Vector2i) -> int`: return Manhattan distance.
2. `is_in_range(attacker: BattleUnit, target: BattleUnit) -> bool`: reject null units, defeated units, same-team units, and units outside Board. Otherwise compare distance to the attacker's range.
3. `find_target(attacker: BattleUnit) -> BattleUnit`: reject a null or defeated attacker; loop over Board's units; consider only those accepted by `is_in_range`; keep the closest and return it, or null if none qualify.
4. In Board, add `count_alive_team() -> Dictionary`. Start both team counts at zero. For each living unit, increment the count under its team key. Return both counts even when one or both are zero.

### Distance And Search Hints

**Manhattan distance** counts horizontal plus vertical cell steps. `absi()` removes an integer's sign, so left and right distances count equally:

```gdscript
absi(end.x - start.x) + absi(end.y - start.y)
```

Range 1 reaches the four orthogonal neighbors. A diagonal is distance 2. Use `<= attacker.attack_range`, not just equality, so a larger range includes nearer cells too.

For a search, start `closest: BattleUnit = null` and `closest_distance: int = 2147483647`, a number larger than any distance here. `continue` skips the rest of **this loop iteration**, unlike `return`, which exits the whole function. Skip invalid candidates; update both remembered values when `current_distance < closest_distance`. Strict `<` keeps the first Board child on an equal-distance tie. Using `<=` would replace it with the last tied child.

A **dictionary** stores values under keys rather than list positions. Here its keys are teams and its values are counts:

```gdscript
var counts := {BattleUnit.Team.ALLY: 0, BattleUnit.Team.ENEMY: 0}
```

That local variable belongs inside `count_alive_team()`, before its loop. Inside the loop's living-unit branch, use `counts[unit.team] += 1`. `+= 1` means add one and store the result. Never count a hidden, zero-health unit as alive. Retaining that node is for reset, not for occupancy or combat.

**Checkpoint:** on paper, put Enemy at `(2, 2)`, Friendly at `(1, 2)`, and Hero at `(2, 1)`. Enemy chooses Friendly on the tie because Friendly occurs earlier. Move Friendly on paper to `(0, 2)` and Enemy instead chooses Hero. A distant opponent must not be returned as an attack target just because it is the only opponent.

## Exercise 5: Ask Godot For A Route

**Goal:** Board returns a complete route to an empty cell, avoiding all current living occupants.

1. In Board, declare `pathfinder: AStarGrid2D = AStarGrid2D.new()`. This creates a route-finding object, not another scene-tree node.
2. Extend Board's `_ready()` after its unit-placement loop. Set `pathfinder.region` to `Rect2i(0, 0, columns, rows)`; set diagonal mode to `AStarGrid2D.DIAGONAL_MODE_NEVER`; set both `default_compute_heuristic` and `default_estimate_heuristic` to `AStarGrid2D.HEURISTIC_MANHATTAN`. Finally call `pathfinder.update()` to build the grid.
3. Write `find_path(start: Vector2i, end: Vector2i) -> Array[Vector2i]`. Reject either endpoint outside Board and any living occupant at the destination, returning `[]`.
4. For each valid query, clear old solids with `pathfinder.fill_solid_region(pathfinder.region, false)`. Then loop through current units: mark a cell solid only if its unit is living, the cell is not start, and it is inside Board. Use `set_point_solid(unit.cell, true)`.
5. Request `get_id_path(start, end, false)` and return its `.slice(1)`.

### Path Hints

`Array[Vector2i]` is an ordered list of cell addresses. An empty list `[]` means no movement route. A **solid** cell is blocked for this search; it is not a physics collision shape. The actor stands on start, so that cell must stay open for its own query. A defeated unit's cell is open even though its node still exists.

The two heuristics tell Godot how to measure grid costs and estimate distance. Use the specified built-in settings rather than implementing A* or looking for a navigation plugin. The grid uses **cell IDs**, not screen pixels; it does not need Cell Size 48 to measure these paths.

The engine's full route includes start and goal. Array indices count from zero. `.slice(1)` returns everything from index 1 onward, removing start; now `path[0]` will be the next step, not the cell the actor already occupies. Never index an empty path. The third argument `false` disables partial routes: a route that only gets closer but never reaches the goal is not accepted.

Clear and rebuild solids on **every query**, not just every timer beat. Earlier units can move or die before a later actor asks. `update()` belongs to initial grid setup before solids, not after marking them: rebuilding can clear the blocked-cell data. Dimensions are configured before `_ready()` and remain 6 by 6 at runtime in this chapter.

**Checkpoint:** explain why asking for a path to Enemy's occupied cell should return empty, and why an old living occupant must stop blocking after moving away or dying. A legal returned path excludes start, includes goal, and advances only horizontally or vertically. A completely sealed route is empty, not a shorter success.

## Exercise 6: Find Somewhere To Attack From

**Goal:** choose the shortest reachable approach, not simply the Manhattan-nearest opponent.

Write two Battle helpers. Keep the candidate scan inside the first function; a separate pathfinding framework is unnecessary.

### One Opponent

`find_path_to_target(attacker: BattleUnit, target: BattleUnit) -> Array[Vector2i]` must:

1. Begin with an empty typed `shortest` array. Return it for null, dead, same-team, or out-of-bounds units, or if the attacker is already in range.
2. Scan Board **rows first, then columns**. For each cell, skip it if outside the attacker's range of the target or if occupied by a living unit.
3. Ask Board for a path from attacker.cell to that candidate cell. Give the variable an explicit type, `Array[Vector2i]`, when calling through `$Board` if Godot cannot infer it.
4. Skip empty paths. Keep a path if no route has been chosen yet or its number of steps is strictly smaller. Return the shortest route after both loops.

The opponent's cell is never your destination: an **empty attack-position cell** is. Scanning all 36 cells is manageable here and works for different attack ranges without a hand-written neighbor list.

```gdscript
if shortest.is_empty() or path.size() < shortest.size():
    shortest = path
```

Use this comparison **after** rejecting empty paths. Otherwise a failed route of length zero would appear "better" than every successful route. Equal lengths keep the first candidate, so preserve row-then-column scan order. Godot chooses the geometry of equally short routes to a given cell; we do not promise a custom left-first or up-first direction rule.

### Across Opponents

`closing_path(attacker: BattleUnit) -> Array[Vector2i]` must return empty for a null/dead attacker or whenever `find_target(attacker)` finds any attack now. Otherwise loop through living opponents, call `find_path_to_target` for each, skip empty routes, and retain the shortest actual route with the same strict comparison.

Do **not** first pick the Manhattan-nearest opponent and then route only to it. Walls made of living units may force a detour, or that opponent may have no reachable attack-position cell. Another, geometrically farther opponent can require fewer actual steps. Manhattan distance determines attack range and ranks available attack targets; actual path length ranks approaches.

**Checkpoint:** imagine one opponent at distance 3 behind blockers needing 8 steps to reach an attack cell, and another at distance 5 needing 4 steps. Choose the 4-step route. If any opponent is already attackable, choose no movement route at all. Explain the difference between null (no target) and an empty array (no route).

## Exercise 7: Resolve Turns And Restore Placement

**Goal:** replace the old hard-coded duel with team turns, a finite limit, and a reset of the player's chosen layout.

This is the largest integration step. Work on one function at a time, but save all the replacements before pressing Start. Retain the enum Phase and the three existing editor signal connections. Remove the old Friendly/Enemy attack sequence and the old `finish_battle(winner: String)` definition and calls; the new `finish_battle()` takes **no argument**.

### Remember The Layout

In Battle, export `max_ticks: int = 60` with `@export_range(1, 3600)`. Declare `elapsed_ticks: int = 0` and an empty `starting_cells: Dictionary[BattleUnit, Vector2i] = {}`. This dictionary maps each existing unit node to its chosen starting cell. It is a **snapshot**, a remembered layout from the moment Start was pressed, not a save file.

Update `_on_start_button_pressed()` in this order:

1. Reject calls unless PREPARATION, even if a disabled button normally prevents them.
2. Clear any old snapshot and record **every** unit's current cell, including Enemy. Hint: `starting_cells[unit] = unit.cell` inside the loop.
3. Zero elapsed_ticks and clear selection. Set phase FIGHTING, disable both buttons, and set ResultLabel to the String `Fighting`.
4. Start the shared AttackTimer. The first beat still waits a full interval.

Replace `reset_battle()` with this sequence:

1. Stop AttackTimer before changing state.
2. Loop over Board units. If the snapshot `.has(unit)`, restore `unit.cell` from it. Regardless, center that unit, assign health from its maximum, and call its display update. On initial `_ready()` the snapshot is empty, so the saved scene cells remain.
3. Zero elapsed_ticks, set PREPARATION, enable Start, disable Restart, set ResultLabel to `Ready`, and clear selection.
4. Only after restoration, clear the snapshot. A later Start must capture the next chosen layout rather than reuse the old one.

Keep Battle's `_ready()` calling reset. Keep Restart's handler guarded to RESULTS, then calling reset. Do not reload the scene, call `_ready()` manually, heal through `take_damage`, or recreate defeated units. Reset restores the three retained nodes, including their visibility.

### Finish Without Draws

Write `check_battle_end() -> bool` using Board's living team counts. It returns true if **either team has zero survivors**. This helper checks elimination only, not the timer limit.

Write the new `finish_battle() -> void`: count survivors again. Allies win only when at least one ally lives and **no enemy lives**. Show `Allies win` for that case, otherwise `Player loss`. Stop AttackTimer, set RESULTS, leave Start disabled, enable Restart, and clear selection. Both teams alive at timeout is a loss. Neither team alive is also a loss. There is **no draw result**.

### One Beat

Replace `_on_attack_timer_timeout()` using this outline. It is pseudocode, not GDScript to paste:

```text
Reject unless FIGHTING.
If a team is already eliminated or the tick limit was already reached:
    finish and return.
Count this beat by adding one to elapsed_ticks.
For each Board unit, in child order:
    Skip dead units.
    Find an opponent already in range.
    If found: damage that opponent.
    Otherwise: find a closing route.
        If not empty and its first step is still empty:
            move cell AND local position by that one step.
    If a team is now eliminated: finish and return immediately.
After ALL unit turns:
    If elapsed_ticks reached max_ticks: finish and return.
    Otherwise display how many ticks remain.
```

The `if`/`else` gives each living unit exactly **one action**: attack **or** move one cell **or** wait. Do not follow movement with another attack check. Later actors see the changed cells and health. Friendly acts before Enemy, and Enemy before Hero; when the last enemy dies, no later action is allowed.

For the countdown use a String, not a bare integer:

```gdscript
$ResultLabel.text = "Ticks left: %d" % (max_ticks - elapsed_ticks)
```

With a one-second timer, 60 beats normally mean roughly 60 seconds of fighting, not a precise wall-clock deadline. Incrementing before the loop makes the displayed count easy to explain. **Do not check timeout inside the unit loop.** Hero acts last and must still be able to win on beat 60. Elimination is checked immediately after each turn; timeout waits until every allowed turn has occurred. A lethal allied hit on that final beat wins rather than becoming a timeout loss.

### Confirm Editor Wiring

| Sending node | Signal | Receiver | Method |
| --- | --- | --- | --- |
| Board | `select_cell` | Battle | `_on_board_select_cell` |
| StartButton | `pressed` | Battle | `_on_start_button_pressed` |
| RestartButton | `pressed` | Battle | `_on_restart_button_pressed` |
| AttackTimer | `timeout` | Battle | `_on_attack_timer_timeout` |

Inspect each sender's Signals dock. Keep exactly one of each connection, saved in Battle's scene; do not also connect in code. Timer: Wait Time `1.0`, One Shot off, Autostart off, Paused off, Process Callback **Physics**. Battle's Max Ticks must be `60` for the normal encounter.

Keep the Chapter 2 controls directly under Battle, with all anchors zero: Start at `(176, 324)` size `(72, 32)`, Restart at `(256, 324)` size `(88, 32)`, ResultLabel at `(352, 324)` size `(220, 32)`. ResultLabel uses left/vertical-center alignment, fallback font, and size override `16`. Initial text is `Ready`; initial Start is enabled and Restart disabled.

**Checkpoint:** Start clears the outline and freezes placement. Combat moves and attacks on the same shared beat, but never both for one actor. Defeated markers hide and stop blocking. Results stop the timer; Restart restores all chosen cells, health, visibility, Ready UI, and empty selection. The automated edge cases include no-survivor loss, both-teams-alive timeout loss, and a winning Hero hit on the last allowed tick. You need not construct those special states by hand.

## Exercise 8: Compare Two Layouts

**Goal:** explain a measurable placement effect without claiming one arrangement is always better.

Keep Enemy at `(3, 0)` and keep all stats, teams, ranges, and child order unchanged. Only exchange the allies' starting cells:

| Layout | Friendly | Hero | Enemy's first target | Final Friendly / Hero / Enemy HP | Result |
| --- | --- | --- | --- | --- | --- |
| A | `(0, 1)` | `(2, 0)` | Hero | `8 / 20 / 0` | Allies win on tick 4 |
| B | `(2, 0)` | `(0, 1)` | Friendly | `2 / 32 / 0` | Allies win on tick 4 |

Here the three numbers are separate units' **current health**, not one label's current/maximum format. The exact reference scripts have been tested twice per layout with identical per-tick cells, health, and phases on each repeat. The original saved game was also checked separately. You may use that evidence instead of manually repeating the runs.

If you want to explore your reconstruction, begin with A. Press Start, observe the result, then Restart. To create B legally, select Hero and move it to temporary empty `(2, 5)`, move Friendly to `(2, 0)`, then move Hero to `(0, 1)`. Start again. Restart should now restore B, not A. Do not change stats to force the expected winner or reorder Board's children to fix a discrepancy.

**Small exercise:** calculate the total allied health remaining in each layout. Hint: add Friendly and Hero, excluding Enemy. A retains **28**, B retains **34**. In A, Friendly keeps more health, but Hero absorbs damage; A is not "better overall" on total health. Both finish at the same tick. The demonstrated effect is **who receives damage**, not a general strategy recommendation.

For a disagreement, compare the first differing beat, not only the final label. Use Godot's Remote scene tree to inspect cells and health if useful, or ask a tutor to help trace one beat from pasted code. Full trace logging is optional and is not a required new gameplay system.

## Troubleshooting

| Symptom | Check |
| --- | --- |
| `BattleUnit` type is unknown | Save battle_unit.gd with class_name at top; resolve its first parse error. Do not name another script class BattleUnit. |
| Typed array rejects a child | Board must contain only BattleUnit instances, directly, with no grouping/decoration nodes. |
| Hero stays at the corner | Board's ready loop must include all get_units(), not only Friendly and Enemy. |
| Click just outside Board selects column 0 | Floor divided local coordinates before converting to Vector2i; check bounds before emitting. |
| Health text blocks clicks | HealthLabel Mouse Filter Ignore in the source scene; ResultLabel Ignore; buttons Stop. |
| No selection outline | Check Input Map select/Left Mouse Button, the Board custom signal connection, preparation phase, and queue_redraw after highlighting. |
| Enemy can move or allies enter the right half before Start | Check team guard and empty-cell/half-board guard; color and node name do not enforce team rules. |
| Nothing moves | AStarGrid2D region/settings/update must exist; ask for an empty attack-position cell, not the occupied target. |
| Units follow stale blockers | Clear/repopulate solids for each query using current living occupancy, excluding acting start. |
| Units walk diagonally or attack diagonally at range 1 | Set no-diagonal mode and both Manhattan heuristics; range uses abs(dx) + abs(dy). |
| A route never takes the first step | Remove start from the engine path with slice(1); reject empty arrays before reading path[0]. |
| A reachable opponent is ignored | Skip empty paths before comparing lengths; closing_path compares actual routes across opponents, not Manhattan target preference. |
| A moving unit also attacks | Use one if/else for attack versus movement, with no second attack after the move. |
| A dead unit blocks or cannot return on Restart | Living-only lookup/solids/counts; hide, do not queue_free; reset assigns health and refreshes visibility. |
| Hero cannot win on the final tick | Elimination checks inside the loop must not include timeout; check timeout after all unit turns. |
| Wrong argument count for finish_battle | Replace the old winner-argument function and all calls with this chapter's no-argument version. |
| Reset restores the old layout | Snapshot all units on every accepted Start; restore before clearing; reset health, position, display, counter, phase, UI, and selection. |
| Combat runs too fast or starts again | Only one shared timer and one connection per signal; remove debug damage/per-frame combat; keep phase guards. |

Start with the first red message in Godot's Debugger, its filename and line number. Do not add a second timer or rewrite Home to address a Board error. For the complete settings and exact functions, use the [solution](cheat_sheet/SOLUTION.md).

## End Checkpoint

The completed scope is preparation placement plus a deterministic three-unit moving team battle: living occupancy, Manhattan attacks, built-in orthogonal routes, one action per turn, immediate elimination, a 60-beat limit with no draws, and full reset to the latest chosen layout. Home is still separate. Automated results are recorded in the [overview](../OVERVIEW.md#validation); checks you perform on your own reconstruction are separate evidence. Optional visual checks concern readable labels, cell alignment, outline visibility, and clickable controls. No new manual tests are required from the original learner who waived them.

Stop at this milestone. Do not add feeding, growth, tower runs, permanent death, saving, or more milestones to finish it. Later planned work is unchanged; course documentation is updated when a milestone is completed, not as a rolling expansion after every session.

Write a short note in your own words:

> I built ___. A team differs from a color because ___. A click becomes a cell by ___. Null differs from an empty route because ___. I compare Manhattan distance for ___ but actual path lengths for ___. A unit cannot move and attack on one turn because ___. Hero can win on the final beat because ___. Restart restores ___. My evidence is ___ (automated/reference, my own checks, or both). My next question is ___.

### Optional ChatGPT Tutor

Ordinary browser ChatGPT is enough. No agent, local assistant, terminal, subscription, or editor access is assumed. Replace the URL placeholder only if you have a repository URL; browsing is optional, not guaranteed.

```text
Be my beginner Godot tutor for Garden Below the Sky, Chapter 3.
Course URL, if available: <REPOSITORY_URL>
Read README.md, OVERVIEW.md, and chapter_3/LESSON.md. Say which files you
actually accessed. If you cannot browse them, ask me to paste their text;
do not pretend to have read them. Do not open the solution unless I ask
or we need a targeted comparison after hints.

Godot version / renderer: ___. Current exercise: ___.
What I understand so far: ___. My question or first exact error: ___.
Tests already reported or run, and their source: ___.
Give me one small editor/code task at a time and explain new syntax.
I operate Godot myself. Accept pasted scripts, a typed scene tree and
Inspector settings if screenshots/uploads are unavailable. Do not assume
you can see or edit my project. Do not ask me to manually repeat already
verified mechanics or treat headless tests as proof of visual feel.
Keep later milestones out of this task.
```

If browsing fails, open the course documents' **Raw** view or your local Markdown viewer and paste the overview plus this lesson, or the current exercise with its prerequisite settings. Uploads are optional. For code review, paste each relevant script with its filename, the exact first error, and what you expected. Include the unit child order, instance stats/cells, and four saved signal connections; script text cannot prove scene wiring.

At the chapter boundary you can ask:

> Review my pasted Chapter 3 code and setup before suggesting changes. Explain the first concrete issue in beginner language and suggest the smallest fix. Check legal preparation placement, living-only occupancy, range/target ties, actual reachable approach ranking, one action per actor, immediate elimination, final-tick allied victory, timeout/no-survivor loss, and restoration of the latest Start snapshot. Preserve signed damage with no revival and one shared timer. My evidence and remaining question are ___. Do not invent access to my editor or extend the project scope.

Finish with a learner-owned [resume note](../OVERVIEW.md#resume-in-a-new-chat). Neither a repository URL nor ChatGPT memory guarantees that a future chat knows your progress.

[Course overview](../OVERVIEW.md) | [Previous chapter](../chapter_2/LESSON.md) | [Complete solution](cheat_sheet/SOLUTION.md)
