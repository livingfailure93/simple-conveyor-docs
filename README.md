# Simple Conveyor

**Conveyor belt puzzles for Unity that are generated, solved exactly and rated.**

Simple Conveyor builds a conveyor belt puzzle from your settings, then proves it can be solved and works out its **par**,
the fewest moves it takes. The player pushes the piece off a stop tile, and the belts carry it their way until it comes to
rest on another one, so the puzzle is reading where each ride ends. It can add ice, switches that turn belts, gems to
collect and holes, and it can keep resting places off the best route for chests or keys. Every puzzle is rated 1 to 5
stars and can be saved as a ScriptableObject, exported as a PNG or SVG picture, or played in the included 3D demo.

*Documentation:* `Window > Living Failure > Simple Conveyor > Documentation` (offline HTML, in `Documentation/`).

---

## The rules

The piece rests on a stop tile, and the player sends it north, east, south or west. It moves onto the next cell, and every
**belt** it enters carries it on the way that belt points until it reaches a **stop tile** and rests. Stop tiles are never
side by side, so every move rides. **Ice** keeps the piece going the way it was moving until a stop tile stops it, a belt
turns it, or a wall leaves it resting in front. It wins by coming to rest on the goal.

Every generated board keeps one hard rule: **no belt ever points into a wall, a hole or off the board**, in any switch
state, and no ride goes round for ever. Boards go from 4 to 32 cells each way. Every kind of tile has a shape as well as
its colour, in the pictures and in the demo, so colour-blind players can tell them apart.

| Gimmick | What it does |
|---|---|
| **Ice** | plain slippery floor that keeps the piece going (on or off). |
| **Switches** | stop tiles that turn the switched belts each time the piece rests on one: Reverse or Rotate (up to 2). |
| **Gems** | taken the moment a ride enters their cell; all of them before the goal counts (up to 3). |
| **Holes** | a ride that ends in one loses the attempt; ice boards only, since no belt points into one (up to 6). |
| **Chest spots** | resting places off every best route, for a chest, a key or anything a detour should pay for (up to 40). |

The data names cells by what they do (a belt is an Arrow, a stop tile is Normal floor, a wall is an Obstacle, a hole is a
Hazard), because Simple Conveyor shares its rules engine with Simple Ice Slide.

---

## What makes it "verified"

- **The par is real.** The solver searches every position a puzzle can reach, in every switch state and with every set of
  gems taken, so a 20-move puzzle really takes 20 moves. A search cut short is marked as not proven, and the generator
  never accepts one.
- **No dead ends.** Nothing the piece can reach is a place it can never win from.
- **The belt rule holds**, whichever way the switches have turned the belts.
- **Every gimmick matters.** Each switch and gem is needed by the best solution, and each hole is one a ride of two or
  more cells really ends in.
- **Stars from measurement.** The 1 to 5 star rating is the square root of the par times the places the piece can come to
  rest (times 1 + gems / 4), measured against 1,500 puzzles played by a simulated player. Ask for a star count and the
  generator keeps trying until it gets it.
- **Misses are reported.** When a star count, a par window or a gimmick that doesn't fit can't be met, the puzzle's notes
  say what happened and what you got instead.

---

## Quick start

1. Open **Window > Living Failure > Simple Conveyor > Conveyor Generator**.
2. Press **Presets**, pick one (for example *Switches/Reverse*) and press **Generate**. Puzzles are built and solved on a
   background thread, with a progress bar, Cancel and a time limit.
3. Read the analysis pane: the par, the stars, a picture of the puzzle, a plain-words list of what is on it, and a slider
   that steps through the best solution.
4. **Save** the selected puzzle as an asset, or the whole set as a pack, and **Export** PNG or SVG pictures.
5. Open `Scenes/SimpleConveyorDemo.unity` and press Play.

### From code

```csharp
using SimpleConveyor;

var settings = new ConveyorSettings
{
    Width = 8, Height = 8,
    Pickups = 1,                 // a gem to take before the goal counts
    TargetDifficulty = 3,        // keep trying seeds until it rates 3 stars
};
ConveyorData puzzle = ConveyorGenerator.GenerateVerified(settings);
Debug.Log($"Par {puzzle.Par} moves, {puzzle.Difficulty} stars, solvable: {puzzle.Solvable}");
```

Run your own game on it with the same rules the solver used, so your game always agrees with the par:

```csharp
ConveyorPlay play = ConveyorRules.NewGame(puzzle);
if (ConveyorRules.TryMove(puzzle, play, 1, out string reason))   // 0 north, 1 east, 2 south, 3 west
{
    if (play.Lost) Debug.Log("Into a hole!");
    else if (ConveyorRules.IsSolved(puzzle, play)) Debug.Log("Solved!");
}
else Debug.Log(reason);                                           // why not: a wall or the edge right next to the piece

if (ConveyorRules.Hint(puzzle, play, out int dir))
    Debug.Log($"A best move from here: {dir}. {ConveyorRules.PickupsLeft(puzzle, play)} gems left.");
```

`ConveyorGenerator` also has `Generate`, `GenerateBatch` and an `Async` version of each, which run on a background thread
and can be cancelled. Another overload of `TryMove` lists every cell a ride passes through, for animating it, and
`ConveyorRules.BeltWay` says where a belt points with the switches as they stand.

---

## The demo

One scene and one component, `ConveyorDemo`, which builds everything in code: no prefabs, models, textures or audio files.
It opens on a welcome screen, then four modes:

| Mode | What you play |
|---|---|
| **Belts** | Belts and stop tiles: plan the rides that reach the goal. |
| **Gems** | Ride over every gem to take it, then reach the goal. |
| **Switches** | Rest on a switch and every belt with yellow corners turns. |
| **Ice and Holes** | Ice keeps the puck sliding, and a slide can end in a hole. |

Each mode has **Endless** puzzles made to your choice of belts only or with ice and a difficulty, and a **Level Pack** that
climbs from 1 star to 5. The demo ships with 42 levels. The belts run all the time, the switched belts turn when a switch
acts, and play has undo, restart, hints, a solver that can play the rest for you, a camera you can turn, tilt and zoom,
moves against par, and sounds and effects made in code.

Controls: arrow keys, WASD, a swipe or a tap beside the puck to move. U undo, R restart, H hint, F solve, [ ] turn,
Home / End tilt, wheel zoom, V reset the view, M mute, Esc for the menu.

---

## Layout

```
SimpleConveyor/
├─ Runtime/
│  ├─ Api/               ConveyorGenerator, ConveyorPresets (and Auto), ConveyorAudit
│  ├─ Data/              ConveyorSettings, ConveyorData
│  ├─ Generation/        the belt builder, switches, holes, gems, Rng
│  ├─ Solving/           solver and star rating
│  ├─ Rules/             ConveyorRules and ConveyorPlay: the play rules
│  ├─ Presentation/      ConveyorExport (PNG and SVG), ConveyorSketch
│  ├─ ScriptableObjects/ puzzle asset, pack, IPuzzleProvider
│  └─ Demo/              3D board, game, HUD, hub, sounds, effects, HDRP look
├─ Editor/               Conveyor Generator window, analysis pane, inspectors, documentation menu, build look keeper
├─ Documentation/        offline HTML docs
├─ Generated/            the demo's four level packs
└─ Scenes/               the demo scene
```

Three assemblies: `SimpleConveyor` (the core, with no package dependencies), `SimpleConveyor.Demo` (Input System and
Unity UI, plus HDRP when it is installed) and `SimpleConveyor.Editor`.

---

## Requirements

- Unity **6000.6** or newer.
- **URP** (recommended), **HDRP** and the **Built-in** pipeline. The demo uses the active pipeline's default lit shader
  and keeps it in builds, and on HDRP it adds a small volume so it looks the same.
- Demo only: the Input System package (set Active Input Handling to *Input System Package* or *Both*) and Unity UI.

## Limits, honestly

- Stars follow the board's size: up to 6 x 6 gives 1 star, 7 x 7 2, 8 x 8 3, 10 x 10 4, and 12 x 12 and up 5. A star
  target above the size's most comes back as the closest puzzle, with a note.
- Gems lengthen puzzles fast (3 gems take a 10 x 10 board from a par of about 26 to about 69), so cap gem puzzles with a
  maximum par or a star target.
- Holes need ice, because no belt points into one, and they don't change the par or the stars.
- A belts-only board has few chest spots of its own (about 3 to 7 on 8 x 8), and each one asked for past that costs about
  a move of par. Ice boards have many more.
- Measured on a desktop PC, every preset builds a verified puzzle in under a second (most in 5 to 300 ms).

## Support

**livingfailuregames@gmail.com**. Please include your Unity version and, for anything about a particular puzzle, its seed
and settings, since a seed rebuilds the puzzle exactly.

Licensed under the Unity Asset Store EULA. © Living Failure.
