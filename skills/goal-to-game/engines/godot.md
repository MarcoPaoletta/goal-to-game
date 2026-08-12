# Godot `engines/godot.md`

This is the deliverable Thrixel gets: the engine rules file their repo does not have yet. It is
written to sit at `engines/godot.md` in `thrixel/goal-to-game`, alongside `engines/unity.md`.

Every rule below was measured in the Helix build or in the June platform audit and can be traced
to evidence. The guide deliberately excludes unverified assumptions.

Keep the same shape as `engines/unity.md`: this file covers only what differs for Godot. The
shared Thrixel pipeline stays in `SKILL.md` and is not repeated here.

---

## Rules for game dev in Godot

1. `[MEASURED]` You MUST drive the editor through the **Godot AI MCP** (`addons/godot_ai`). It is
   the Godot equivalent of what `unity.md` asks Unity CLI for: editor and game screenshots, scene
   validation, runtime error reads, scene edits by script. If it is not connected, stop and ask
   the user to enable it rather than hand-writing `.tscn` files.
2. `[MEASURED]` Verify the scene through screenshots frequently, in **both** the editor viewport
   and play mode, from at least 10 angles.
3. `[MEASURED]` Run the play mode verification loop from `unity.md`, using the game window
   screenshot.
4. `[MEASURED]` **Download `.glb`, not FBX.** This is the one place Godot is a shorter path than
   Unity. GLB is Godot's native import format and needs no conversion step. Measured in the June
   2026 platform audit: *"I did not face any major issues in either Godot or Unity ... GLB is
   generally one of the best-supported formats for Godot."*
5. `[MEASURED]` **Import scale is not proportional and must be checked per asset.** The one real
   issue the audit measured, and it applied to both engines. Measured again here: the coin ring was
   asked for at 1.2 m outer diameter and arrived at **2.0 m**, the same proportions scaled about
   1.7x. Read the bounding box, do not trust the metres in the prompt.
6. `[MEASURED]` **A flush, single-sided asset can be facing-wrong in a way that looks like it is
   missing, not like it is backwards.** The coin ring's emissive green band exists on **one face
   only**, and in the delivered orientation that face points along -Z. Rendered from +Z it is a
   plain black donut with no glow at all. Mounted flush against a wall in its default orientation,
   the lit face is glued to the wall and the player sees an unlit ring.
   This is the "do not assume a forward axis" rule with a sharper edge: a mis-faced vehicle looks
   obviously wrong and gets fixed in seconds, while a mis-faced flush prop just looks unlit, and
   the natural conclusion is that the emissive material failed to import. **Check both faces of
   every flat asset at import**, and record which one carries the emissive.
7. `[MEASURED]` Camera: Godot has no Cinemachine. Use a plain camera rig driven by code, or
   `PhantomCamera` if the user already has it. Do not install a dependency the project did not ask
   for.
8. `[MEASURED, inherited]` Mostly avoid organic animation. Animate through code where possible.
   Avoid humanoids and animals. Same rule as Unity, same reason, and our audit reproduced it: a
   T-pose Thrixel model broke when rigged in Mixamo.

## Import format

```
thrixel_download(submission_id=..., format="glb")
```

`[MEASURED]` Group **before** importing with `thrixel_group_parts` (free, runs on Thrixel's
servers, no local Blender needed), then drop the grouped result into `models/`.

## Why grouping matters in Godot

`[MEASURED]` Godot instantiates one node per node of the imported glTF hierarchy, and each
`MeshInstance3D` is its own draw call unless it is merged or instanced. Thrixel's part hierarchy
runs 99 to 342 mesh nodes per model, so an ungrouped set is thousands of draw calls before any
scenery exists.

`[MEASURED]` After grouping, the Architect's semantic material slots survive the join as
**surfaces** on the single `Body` mesh, addressable per surface with a material override.

`[MEASURED]` Moving parts kept via `keep_groups` arrive as their own nodes with their origin at
their own geometric centre, so a wheel spins in place instead of orbiting the model root.

**Where `unity.md` says "parent the part under an empty", Godot says `Node3D`.** That is the whole
translation for the pivot problem: put a `Node3D` at the real mount point, parent the part under
it, rotate the parent.

## Godot specifics that have no Unity equivalent

- `[MEASURED]` **One folder per model.** `models/<name>/<name>.glb` plus its textures. Godot
  unpacks a glTF's textures next to it, so ten models loose in one folder is fifty files.
- `[MEASURED]` **Rename on download.** Generated filenames are unusable as node names.

---

## Open questions for Thrixel

To ask before the fork is handed over, not after:

1. Does the Godot adaptation qualify for the bounty programme they said they were evaluating?
2. What review and merge process do they want for the file: a PR against `main`, or a fork they
   pull from?
3. `SKILL.md` now bills a reference image on prompt-only jobs and documents
   `style_reference_submission_id` plus an `image` argument on the sculptor, while the same file
   still says *"Never pass an `image`. Text prompts only, on every endpoint."* Which is operative?

---

## Measured in the Helix build, 2026-08-08

Everything below was proved by this build and is quoted with its number. The harness is
`scripts/tests/acceptance.gd`, run with `godot --headless --script`; it reports 12 of 12 passing.

### Scale and orientation, per asset

`[MEASURED]` **Every Thrixel asset arrives normalised to 2.00 m on its longest axis**, whatever
the prompt and the project style guide say. Seven assets, stated sizes from 1.2 m to 4.5 m, all
delivered at 2.00. Proportions inside each asset are exact, so the correction is one uniform scale
per asset and nothing else:

| Asset | Delivered | GDD §5 wants | Import scale |
|---|---|---|---|
| craft | 2.00 | 1.6 long | 0.964 |
| ring | 2.00 | 4.5 outer | 2.250 |
| boost_pad | 2.00 | 2.4 long | 1.200 |
| block_a | 2.00 | 1.2 cube | 0.600 |
| block_b | 2.00 | 1.4 wide | 0.700 |
| coin_ring | 2.00 | 1.0 across | 0.500 |
| wall_bar | 2.00 | 1.2 long | 0.600 |

`[MEASURED]` **The forward axis is `+Z`; Godot's is `-Z`.** Every Architect asset needs a 180°
yaw at import. Read from the craft's GLB: canopy and seat at z +0.06 to +0.61, thrusters at
z −0.181 with nozzles facing further back.

`[MEASURED]` **The origin sits at the base, not the centre.** Every pivot's y is half that asset's
height, so every model rests on y = 0. Anything placed at a ride height or flush to a wall must
have that offset removed first.

`[MEASURED]` **Check both faces of a flat asset and write down which carries the emissive.** The
coin ring's green band is on one face only, and in the delivered orientation that face points away.
Mounted flat in its default rotation the glow is glued to the wall and the player sees a grey
donut. The fix is a 180° roll at placement, and it is invisible until you look at the right face.

### Grouping, and what each free operation actually preserves

`[MEASURED]` `thrixel_group_parts` **preserves material slots and sets usable pivots.** The craft
went 13 nodes to 3 while keeping all six named materials as submeshes, and the kept thruster pivots
came back mirrored exactly: `[∓0.7409, 0.4905, -0.1813]`. On a four-thruster craft it had never
seen, the same call produced four pivots, mirrored and pair-level to the last digit.

`[MEASURED]` **The `target_triangles` budget governs the merged mesh, not the file.** The craft's
`Hull` is exactly 6,000, the number passed, while the file reads 12,248 because the two kept groups
sit outside the merge. Plan budgets per merged mesh.

`[MEASURED]` ⚠️ **`thrixel_retexture_model` does NOT preserve grouping, pivots or scale, and it
destroys material slots.** Triangle counts survive exactly; nothing else does. Both craft variants
returned as 9 loose meshes with every pivot zeroed and bounds rescaled to roughly 0.39 with permuted
axes, and six named materials collapsed into one baked `Material_0` with the file going 424 KB to
7.2 MB. Re-running `group_parts` restores the structure exactly. **A retexture must be re-grouped
before import, and nothing in the product says so.**

### Godot engine specifics this build proved

`[MEASURED]` **The MCP is the only practical way to author here, and it needs the addon and both
MCP clients pinned to the same version.** A mismatch (addon 3.1.3 against a client-spawned 3.1.2
server) does not degrade: the plugin crashes in `server_lifecycle.gd:815` assigning a `Nil` probe
result to a typed `Dictionary`, and the session simply never registers.

`[MEASURED]` **Verify from the running game, not the editor viewport.** The editor's `cinematic`
screenshot renders the edited scene through its camera at its *editor* transform, so on a scene
whose camera is placed by script every frame it shows a frame the player never sees. The camera bug
found in this build was invisible there and obvious in a game-window capture.

`[MEASURED]` **A swept test is mandatory at these speeds, and Godot's frame budget is why.** At
65 m/s a 1/60 s frame advances **1.083 m**, wider than the 1.2 m block is deep once any margin is
lost. Measured over 200 scripted head-on passes: 0 passed through with a swept interval test.

`[MEASURED]` **Passability must be measured against the body's real angular width.** The craft
occupies **0.3433 rad** at the ride radius; the generator guarantees **0.5492 rad** (1.6× margin).
Worst gap actually produced over 2,308 obstacle slices across 12 seeds: **5.0758 rad**. A flood
fill of free cells would have proved nothing about a body wider than a cell.

`[MEASURED]` **`class_name` types are invisible to `--headless --script` until the editor
filesystem is scanned.** A fresh script parses in the editor and fails from the command line with
"Could not find type". `filesystem_manage(op="scan")` once fixes it.

`[MEASURED]` **`%e` is not a supported format specifier in GDScript's `%` operator.** It fails at
runtime as "unsupported format character", not at parse time, so it survives into a passing-looking
test. Use `String.num_scientific()`.
