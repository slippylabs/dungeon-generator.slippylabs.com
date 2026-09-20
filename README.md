# Dungeon Generator

Four procedural dungeon algorithms — BSP rooms, cellular caves, drunkard's walk and room accretion — seeded, guaranteed connected, and exportable as JSON, CSV or ASCII. Runs entirely in your browser.

**Live:** <https://dungeon-generator.slippylabs.com/>

## What it does

- **BSP rooms and corridors** — architecture; **cellular caves** — erosion; **drunkard's walk** — wandering; **room accretion** — sprawl.
- Every map is seeded, so a level you like is a number you can keep.
- Every map is **checked for connectivity after generation**, and the region count is printed on screen.
- Spawn and exit placed at the two ends of the longest path, a distance-from-spawn heatmap, and a "regions before repair" view.
- Export as JSON (with rooms, spawn, exit and a legend), a CSV grid, or ASCII.

## How it works

Three of the four generators can produce a disconnected map by nature — caves island themselves constantly, accretion can reject every bridge, and a badly seeded BSP can seal a corridor. So generation is only half the job.

Every map goes through a repair pass. Caves keep only the largest region and fill the rest back in, because an orphaned cave pocket is just rock; everything else is tunnelled together, because an orphaned *room* is a level-design mistake. The connectivity flood fill is **4-connected**: a diagonal-only link is not a step a tile game can take, and counting it as connected produces a map that passes the check and traps the player.

The exit is the floor tile furthest from the spawn *by walking distance*, found the standard two-BFS way — which is why it feels like the end of the level rather than a corner of the map.

A door is a **corridor** tile that touches a room, not the room tile beside it. Marking the room tile puts the door inside the room, which looks fine on a map and breaks every "am I in a room" test downstream.

## Verification

`verify_dungeon.py` uses `scipy.ndimage.label` as an independent connected-components oracle — code with nothing in common with the page's own flood fill — over **336 maps** across all four algorithms and four awkward aspect ratios:

- Exactly **one** 4-connected walkable region in every map.
- Rooms inside bounds and non-overlapping; every door adjacent to a room; a solid border.
- Spawn and exit both walkable, and the exit is the farthest reachable tile under an independent BFS — **zero** disagreement.
- Dead ends counted independently; the ASCII, CSV and JSON exports all reproduce the grid exactly.

A control sweeps the cave settings: at 58% rock with no smoothing, the raw maps come out in up to **417 regions**, every one repaired to exactly 1 — so the repair pass is doing real work rather than decoration. **42,648 checks.**

The BSP splitter refuses to split a partition below twice its minimum leaf size. Without that, a long thin map produced leaves too small to hold a room, the room got clamped up to its minimum and spilled across the partition line into its neighbour.
