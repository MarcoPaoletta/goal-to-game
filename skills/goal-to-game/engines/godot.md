# Godot

Engine-specific rules for the Godot path. The shared Thrixel asset pipeline is in
[../SKILL.md](../SKILL.md); this file covers only what differs for Godot.

Every rule below was measured while building two complete Godot 4.7 games with this skill and
Thrixel assets, twenty eight generated models between them. Anything that could not be verified in
those builds was left out rather than guessed.

# Rules for Game dev

When developing in Godot, you MUST set up the following checklist, and verifiably and rigorously
check each off your list:

1) You MUST drive the editor through the **Godot AI MCP** (`addons/godot_ai`). It is the Godot
   equivalent of what Unity does with the Unity CLI: editor and game screenshots, scene validation,
   runtime error reads, scene edits by script. If it is not connected, you MUST stop and ask the
   user to enable it. Never hand write `.tscn` or `.tres` files, and never scaffold nodes from
   runtime code to work around a missing connection.
2) You must FREQUENTLY verify scene setup through screenshots, in BOTH the editor viewport and play
   mode, from at least 10 angles.
3) You must follow EVERY step in the Thrixel asset import inspect loop below.
4) You MUST run the play mode verification loop below multiple times.
5) Download every Thrixel asset as **GLB**, not FBX. This is the one place Godot is a shorter path
   than Unity: GLB is Godot's native import format, so there is no conversion step.
6) Godot has no Cinemachine. Use a plain camera rig driven by code, or `PhantomCamera` if the user
   already has it. Do not install a dependency the project did not ask for.
7) Mostly avoid organic animations. Animate everything through code where possible. Avoid adding
   humanoids or animals to the game.
8) The addon and the MCP client must be pinned to the **same version**. A mismatch does not
   degrade, it fails silently: an addon on 3.1.3 against a client spawned 3.1.2 server crashes the
   plugin in `server_lifecycle.gd` assigning a `Nil` probe result to a typed `Dictionary`, and the
   session simply never registers.

# Thrixel asset import inspect loop

For EVERY Thrixel asset you download, follow the shared inspect loop in `unity.md`, at the same two
points: when the asset is downloaded, and again in play mode. Four things about the delivered files
are specific to this pipeline and cost a build each time they are missed.

**Every asset arrives normalised to 2.00 m on its longest axis**, whatever the prompt and the
project style guide asked for. Seven assets in one build, requested between 1.2 m and 4.5 m, all
delivered at 2.00. Proportions inside each asset are exact, so the correction is one uniform scale
per asset and nothing else. Read the bounding box; do not trust the metres in the prompt.

| Asset | Delivered | Asked for | Import scale |
|---|---|---|---|
| craft | 2.00 | 1.6 long | 0.964 |
| ring | 2.00 | 4.5 outer | 2.250 |
| boost_pad | 2.00 | 2.4 long | 1.200 |
| block_a | 2.00 | 1.2 cube | 0.600 |
| block_b | 2.00 | 1.4 wide | 0.700 |
| coin_ring | 2.00 | 1.0 across | 0.500 |
| wall_bar | 2.00 | 1.2 long | 0.600 |

**The forward axis is `+Z` and Godot's is `-Z`**, so every Architect asset needs a 180 degree yaw
at import. Read from the craft's GLB: canopy and seat at z +0.06 to +0.61, thrusters at z -0.181
with the nozzles facing further back.

**The origin sits at the base, not the centre.** Every pivot's y is half that asset's height, so
every model rests on y = 0. Anything placed at a ride height, or flush against a wall or ceiling,
has to have that offset removed first.

**Check both faces of a flat asset and record which one carries the emissive.** A single sided prop
can be facing wrong in a way that looks like it is missing rather than backwards: one build's coin
ring has its emissive band on one face only, pointing along -Z, so rendered from +Z it is a plain
black donut with no glow. Mounted flush in its delivered orientation the lit face is glued to the
wall and the player sees an unlit ring. A mis faced vehicle looks obviously wrong and gets fixed in
seconds; a mis faced flush prop just looks unlit, and the natural conclusion is that the emissive
material failed to import.

# Play mode verification loop

Run the loop described in `unity.md`, with one substitution that matters in Godot.

**Verify from the running game, not the editor viewport.** The editor's `cinematic` screenshot
renders the edited scene through its camera at that camera's *editor* transform, so on a scene
whose camera is placed by script every frame it shows a frame the player never sees. The camera bug
found in one of these builds was invisible in the editor capture and obvious in a game window
capture.

## Import format

```
thrixel_download(submission_id=..., format="glb")
```

Group **before** importing with `thrixel_group_parts` (free, runs on Thrixel's servers, no local
Blender needed), then drop the grouped result into `models/`.

## Why grouping matters in Godot

Godot instantiates one node per node of the imported glTF hierarchy, and each `MeshInstance3D` is
its own draw call unless it is merged or instanced. Thrixel's part hierarchy runs 99 to 342 mesh
nodes per model, so an ungrouped set is thousands of draw calls before any scenery exists.

After grouping, the Architect's semantic material slots survive the join as **surfaces** on the
single `Body` mesh, addressable per surface with a material override.

Moving parts kept via `keep_groups` arrive as their own nodes with their origin at their own
geometric centre, so a wheel spins in place instead of orbiting the model root.

**Where `unity.md` says "parent the part under an empty", Godot says `Node3D`.** That is the whole
translation for the pivot problem: put a `Node3D` at the real mount point, parent the part under it,
rotate the parent.

## What each free operation preserves

`thrixel_group_parts` **preserves material slots and sets usable pivots.** One craft went from 13
nodes to 3 while keeping all six named materials as submeshes, and the kept thruster pivots came
back mirrored exactly at `[±0.7409, 0.4905, -0.1813]`. On a four thruster craft it had never seen,
the same call produced four pivots, mirrored and pair level to the last digit.

**The `target_triangles` budget governs the merged mesh, not the file.** That craft's `Hull` is
exactly 6,000, the number passed, while the file reads 12,248 because the two kept groups sit
outside the merge. Plan budgets per merged mesh.

⚠️ **`thrixel_retexture_model` does NOT preserve grouping, pivots or scale, and it destroys
material slots.** Triangle counts survive exactly; nothing else does. Both craft variants returned
as 9 loose meshes with every pivot zeroed and bounds rescaled to roughly 0.39 with permuted axes,
and six named materials collapsed into one baked `Material_0`, with the file going from 424 KB to
7.2 MB. Re running `group_parts` restores the structure exactly. **A retexture must be re grouped
before import.**

## Keeping a generated set consistent

Generate the variants of a thing against the first one as `style_reference_submission_id`. That is
what keeps them one family visually, and it has a second effect worth planning around: in the
second build the three extra hulls came back with the **same part names** as the original
(`fin_L`, `fin_R`, `core_L`, `core_R`), so grouping, banking and thruster animation carried
straight over with no per model code.

**Resolve materials by the name of the Architect material slot, not by its baked colour.**
`hull_teal_plastic`, `core_yellow` and `band_cyan_emissive` say what a surface *is*; the colour
only says what it happened to look like. Matching on colour is what makes a teal housing and a cyan
light strip resolve to the same material, because they are one hue at two brightnesses, and what
makes a mid grey snap to coral, because coral's green and blue channels sit nearer to 0.5 than
charcoal's do.

## Godot specifics that have no Unity equivalent

- **One folder per model.** `models/<name>/<name>.glb` plus its textures. Godot unpacks a glTF's
  textures next to it, so ten models loose in one folder is fifty files.
- **Rename on download.** Generated filenames are unusable as node names.
- **`class_name` types are invisible to `--headless --script` until the editor filesystem is
  scanned.** A fresh script parses in the editor and fails from the command line with "Could not
  find type". One `filesystem_manage(op="scan")` fixes it.
- **`%e` is not a supported format specifier in GDScript's `%` operator.** It fails at runtime as
  "unsupported format character", not at parse time, so it survives into a test that looks like it
  passes. Use `String.num_scientific()`.
- **A `Label` with autowrap off reports the width of its longest line as its minimum size** and
  drags its container off the screen. That is how one build's title screen instructions ran off the
  right edge while every anchor was correct.

## Two things speed breaks, and they are worth checking in any fast game

**A swept collision test is mandatory, and Godot's frame budget is why.** At 65 m/s a 1/60 s frame
advances 1.083 m, wider than a 1.2 m obstacle is deep once any margin is lost, so a point test at
the frame boundary lets the player pass straight through solid geometry. Measured over 600 scripted
head on passes at the worst frame step: the swept interval test caught 600, a point test missed 274.

**Passability must be measured against the body's real width, not against free cells.** The craft
occupies 0.3433 rad at its ride radius while the generator guarantees 0.5492 rad, a 1.6x margin.
A flood fill of free cells would have proved nothing about a body wider than one cell.
