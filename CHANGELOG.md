# Changelog: Simple Conveyor

All notable changes to this package.

---

## [1.0.0]

First release.

### Puzzles

- **The rules:** the player pushes the piece off a stop tile, every belt it enters carries it on its way, and it rests on
  the first stop tile a ride reaches. It wins by coming to rest on the goal. Ice keeps the piece going the way it was
  moving.
- **The belt rule:** no belt ever points into a wall, a hole or off the board, in any switch state, and no ride goes round
  for ever. Nothing the piece can reach is a dead end.
- **Boards:** 4 to 32 cells each way, with stop tiles on up to 15 to 50% of the board, walls on up to 15%, and ice patches
  if you want them.
- **Gimmicks:** up to 2 switches that reverse the switched belts or turn them a quarter turn, 3 gems to take before the
  goal counts, and 6 holes on ice boards.
- **Chest spots:** ask for up to 40 resting places off every best route, each with the extra moves going by it costs, for
  a game to put chests, keys or side rooms on.
- **Every gimmick matters:** each switch and gem is needed by the best solution, the belts are climbed again around them,
  and each hole is one a ride of two or more cells really ends in.
- The data names cells by what they do (Arrow, Normal, Obstacle, Hazard), so the same puzzle can take any look.
- Every kind of tile has its own shape as well as its colour, so colour-blind players can tell them apart.

### Solving and rating

- An exact solver over every reachable position, in every switch state and with every set of gems taken. It gives the
  **proven par**, one best solution, the number of best solutions with a completeness flag, and whether a player **can get
  stuck**, plus how the solution feels: choices, decoys, wrong turns and long detours, and the chest spots.
- A search the budget cuts short is marked as not proven, and the generator never accepts one.
- **Difficulty** (1 to 5 stars) from the square root of the par times the places the piece can come to rest, times
  1 + gems / 4, with bands measured against 1,500 puzzles played by a simulated player (the same yardstick as Simple Ice
  Slide's stars).

### Generation

- `ConveyorGenerator.Generate`, `GenerateVerified` (keeps trying seeds until a puzzle is proven solvable with no dead
  ends, inside the par window, keeps the belt rule, has every gimmick and chest spot it asked for, and is on its star
  target) and `GenerateBatch`, each with a background-thread `Async` version that can be cancelled. `GenerateVerified` can
  also report what each board is doing and stop at a time limit.
- Deterministic: the same settings and seed always give the same puzzle, and every puzzle stores the seed that made it.
- Every miss is written to the puzzle's notes. Nothing is quietly downgraded.
- 21 presets, each built 24 times in testing with every build getting everything its preset asked for, and **Auto**,
  which fixes settings that fight each other and says what it changed. Every fix comes from a measurement.

### Playing

- `ConveyorRules`: the play rules your game uses, the same ones the solver counted with. It covers moves (with every cell
  passed, for animating), why a move can't start, gems, switches and where each belt points now, falls into holes, hints
  from any position (`Hint`), the win check and a data validator.

### Editor

- The **Conveyor Generator** window: every setting with tooltips, a live warning of what Auto would change, presets, a
  seed you can keep, batches of up to 50 built on a background thread with progress, Cancel and a time limit, Only on
  target, and Save as a puzzle or a pack.
- The **analysis pane**, in the window and in both inspectors: the verdict, a picture of the puzzle with a plain-words
  legend, a slider that steps through the best solution with the switched belts turning, a Chest spots toggle, hover to
  read a cell, zoom, the move list with Copy solution, Copy seed, the facts and the notes.
- **Export** as PNG or SVG: the colour view, a printable black on white sheet that draws every tile by its shape alone,
  or an answer key with the best solution drawn on, with the chest spots marked if you like. A pack exports every puzzle
  with its answer key in one go.
- **Inspectors** with Verify (solve again and restamp the verdict) and Re-roll (a new puzzle from the same settings),
  including re-rolling every puzzle in a pack that missed its request.

### Demo

- One scene and one component, `ConveyorDemo`, which builds everything in code with no prefabs, models, textures or audio
  files.
- A welcome screen, then four modes: Belts, Gems, Switches, and Ice and Holes. Each has Endless puzzles made to your
  choice of belts only or with ice and a difficulty, and a level pack that climbs from 1 star to 5. The demo ships with 42
  levels.
- The belts run all the time, the puck rides the path the rules give, gems spin away as it passes, the switched belts
  turn when it rests on a switch, and it drops into a hole on a fall. Play has undo, restart, hints, a solver that can
  play the rest, a camera you can turn, tilt and zoom, and moves against par with a rating.
- Keyboard, swipe and tap controls that follow the camera, so they stay right however it is turned.
- Sounds and effects made in code, the same in every render pipeline.
- Works in URP, HDRP and the Built-in pipeline. On HDRP a small volume gives it the same look, and a build step keeps the
  demo's shader in builds so it never draws pink.

### Documentation

- Offline HTML documentation (Home, Quick Start, Generating, API Reference, Demos, Troubleshooting and About, plus a
  single-page version), opened from **Window > Living Failure > Simple Conveyor > Documentation**.
