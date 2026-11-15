# AnimForge Studio — User Manual

Nov 14, 2026

## What it does

AnimForge Studio fixes animation clips in Roblox Studio, whether they came from Mixamo, the Animation Editor, or AI video capture. One click (Fix All) levels slanted feet, removes the baked-in hip offset, turns a backwards-facing clip around, takes out joint pops and leg tremble, puts the feet on the ground, and cuts a walk down to one clean loop. It then tells you the WalkSpeed at which the feet stop sliding.

It also converts animations downloaded from Mixamo into clips that play on a Roblox R15 rig, straight from the FBX file, with no retargeting in Blender first.

It works on the `KeyframeSequence` clips the Animation (Clip) Editor saves. It changes a rig only when you click Drop to floor, and by default it never touches your original clip: every fix writes to a copy named `<clip>_fixed`, and every click is one Undo step.

Use it when an imported clip:

- floats above or sinks below the floor
- faces the wrong way (plays backwards relative to the character's movement)
- slides forward instead of staying in place
- pops, stalls or shuffles when it loops
- has feet that stay tilted, toe down, while they are planted
- has knees or legs that snap or tremble
- jitters from noisy mocap
- skates because the walk doesn't match the character's speed
- needs trimming, slowing, mirroring, or a limb held still

Every tool in this manual is in the plugin; there is one version.

## Install and first run

1. Install from the Creator Store (Plugins tab → Manage Plugins, or the Store page). Restart Studio: plugins load at startup.
2. Click **AnimForge** in the Plugins tab. The dock opens on the right. Drag it wider than 340 px for the full layout; narrower, it stacks.
3. Open the Clip Editor on your rig, import your clip (⋯ → Import → From File for FBX), and **Save** it. Studio stores it as a `KeyframeSequence` under `ServerStorage → RBX_ANIMSAVES → <rig name>`. That is the object the plugin works on.

Two things you select in the Explorer:

| What | Where | Why |
| --- | --- | --- |
| The clip | `ServerStorage → RBX_ANIMSAVES → <rig> → <clip>` | What gets fixed |
| The rig | `Workspace → <rig model>` | Lets the plugin measure where the feet are |

Click the rig once and press **Use this rig** so you don't have to re-select it each time. The status bar at the bottom shows what's active: `Rig: Rig (R15) · Clip: Walk`.

After any fix, **reload the clip in the Clip Editor** (⋯ → Load → `<clip>_fixed`). The editor keeps a stale copy until you do.

## Quick fix

Four buttons cover most imports.

**Use this rig.** Select the rig model in the Explorer and click. The plugin reads its joints (Motor6D or AnimationConstraint), finds the feet, and measures the ground line. Stays set until you pick another rig.

**Drop to floor.** Moves the rig so its feet stand on the floor beneath it. Do it once after you place a rig, so what you see in the viewport matches the ground the plugin measures.

**Check clip.** Read-only report on the selected clip: keyframe count, length, root offset, and three verdicts: `floating`, `drifting`, `flipped`. With a rig set it also prints foot clearance (`min / max`, 0 = touching the floor) and how many of the clip's poses match the rig. Run it before and after a fix.

Check clip also prints three lines that tell you what Fix All will do:

- `planted foot tilt` (needs a rig): how far the feet are off level while planted. 0 = sole flat; over 6° = slanted, and Flatten feet fixes it.
- `smoothing`: whether the legs read as trembling, and so whether Fix All will smooth the clip. A clip keyed by hand is reported as hand-keyed and left alone.
- `walk speed` (walks and runs): the speed the clip was made for. See *Walk speed report* below.

**Fix All (auto).** One click, one Undo step:

1. Flatten feet: levels planted feet that stay slanted.
2. Root Fix with *Frame 1* and *In place*; flips 180° automatically if Check clip says `flipped=true`.
3. De-pop: removes one-keyframe joint snaps.
4. Smooth, if the clip's legs read as trembling. Hand-keyed clips are always skipped. Planted feet keep the level Flatten gave them. If smoothing would flatten the steps of a fast run (the feet would cover over 25% less ground), Fix All undoes it and logs `Smooth: reverted`.
5. Feet on ground in *auto* mode: plants every frame for clips named walk, run, idle, climb or swim, otherwise lifts the whole clip so the lowest frame touches.
6. Find loop, for those same locomotion names: keeps the cleanest single cycle and seals it. If no clean cycle is found, it falls back to Loop Pose (tail 0.25).

The log shows each step as `1/6` to `6/6`, then `-> selected <clip>_fixed` so your next click works on the copy. For a walk or run, the log ends with the walk speed report.

Three checkboxes directly under the Fix All button switch steps off: *Flatten feet*, *De-pop* and *Smooth*. All three are on by default and remembered between sessions. A step that is off says so in the log, for example `1/6 Flatten feet: off`. If Flatten feet is off and the clip's planted feet are tilted, the log ends with a `WARN` line that says how far they are off level and how to fix it.

On a clip converted from Mixamo, Fix All skips Flatten feet, De-pop and Smooth whatever the checkboxes say, because that motion is already clean and its fast moves are real. The log says so on each skipped step. The Smooth button still works on those clips if you want it.

What Fix All does **not** do: trim, mirror, mask, or reduce keyframes. Those are in Advanced.

## Advanced: Mixamo to R15

Turns an animation downloaded from Mixamo into a clip that plays on a standard R15 rig. The two buttons are under Advanced > Mixamo. The plugin converts files you downloaded yourself; it contains no Mixamo animations and is not affiliated with or endorsed by Adobe.

**Download settings on mixamo.com**

| Setting | Use |
| --- | --- |
| Format | FBX Binary (.fbx) |
| Skin | With Skin or Without Skin; both work |
| Frames per second | 30 |
| Keyframe Reduction | none |

Keyframe Reduction must be none. A reduced file has lost the motion between its keys, and the plugin refuses it.

**Import Mixamo FBX files...** reads the files directly, with no Clip Editor import.

1. Select your R15 rig and click **Use this rig**.
2. Click **Import Mixamo FBX files...** and pick one or more files.
3. Each file becomes a clip named `<file name>_r15` under `ServerStorage > RBX_ANIMSAVES > <rig name>`.
4. Select the new clips and click **Fix All**.

One Undo removes every clip from that import, as long as you haven't loaded one of them in the Animation Editor first.

**Retarget Mixamo to R15** is for a clip you already imported through the Clip Editor (its poses are named `mixamorig:...`). Import the FBX there with Rig: Animation Editor Rig, Rig Type R15, Scale Unit Stud, Scale Factor 1.0. Then select the clip, several clips, or a Folder, and click the button. Each becomes `<name>_r15`.

Use the Import button when you have the FBX file. It reads the character's true rest pose from the file and refuses reduced files. On the Retarget path a character other than Mixamo's default bots can come out with the chest and head a few degrees off, and a reduced file imports with wandering feet that nothing can repair.

**What the conversion does**

- Maps Mixamo's skeleton onto the 15 R15 parts. Fingers and toes are not carried over.
- Takes the arms from Mixamo's T-pose to the Roblox arms-down rest pose.
- Scales how far the hips travel to your rig's leg length, so steps match the rig's size.
- Turns the clip to face forwards on a Roblox rig.
- Raises the hips where the head, torso or thighs would otherwise go under the floor.
- Turns Loop on for movement clips (walks, runs, idles and the like) that end on the pose they started on. The log says when it does.

The log prints one line per file with the keyframe count, the hip scale, and whether the hips were raised. A refused file gets a line starting `Import Mixamo FBX: skipped` with the reason.

## Advanced: Root position (Root Fix)

Fixes the hips. Imports often bake world movement into the hip pose, so the character floats, slides, or faces the wrong way. Root Fix rewrites only that one pose; the rest of the body is untouched.

**Root position** chooses what happens to the hip's position:

| Option | What it does | Use when |
| --- | --- | --- |
| Frame 1 (default) | Subtracts frame 1's position from every frame | Clip is offset by a constant amount (the usual Mixamo case) |
| Average | Subtracts the average position instead | Frame 1 is itself odd (a take that starts mid-step) |
| Zeroed | Removes all hip movement | The import put garbage motion on the root |
| Original | Leaves position alone | You only want the flip |

**In place (strip XZ travel).** On: forward/sideways travel is removed, vertical motion (bobbing, jumps) is kept. Off: travel is kept as root motion for clips that should move the character.

**Flip 180°.** Rotates the hip pose half a turn so the body faces the same way as the root. Fix All sets this automatically; here you tick it yourself. Check clip tells you if a clip needs it (`flipped=true`).

Why a flip matters in game: Roblox moves and turns the `HumanoidRootPart`; the animation is stored relative to it. A clip rotated 180° relative to the root plays backwards whenever the player moves, and turning the model or camera does not fix it.

## Advanced: Flatten feet

Levels feet that stay tilted while they are planted: the toe-down or rolled-ankle look common in video and AI captures. Needs a rig.

During each plant it turns the foot about the ankle until the sole is level, and keeps the toe-out angle. The ankle doesn't move, so nothing else in the pose changes. It eases in and out over 0.15 s at each end of a plant, so the foot doesn't snap as it lands or lifts.

It only changes a plant where the foot stays slanted the whole time, more than 6° off level. A foot that passes through level, as in a normal heel-to-toe step, is left alone. The log lists each foot's plants: how many were levelled and how many were already level.

Run it on the original clip, before Root Fix. It finds plants from the clip's own travel, so a clip that has already been made in-place gives it less to work with. Fix All runs it first for that reason.

## Advanced: Feet on ground

Puts the feet on the floor. Needs a rig (selected or set with Use this rig). The plugin runs forward kinematics through the rig's joints for every keyframe, finds the lowest foot point, and shifts the hips vertically so it sits on the ground line (the rest-pose foot level).

When the feet are not the lowest part of the body (a handstand, a crawl, a character lying down), the step grounds whichever of the head, torso or thighs is lowest on that keyframe. The log names the part, for example `the lowest part of the body is not a foot (Head)`.

**Mode:**

| Mode | What it does | Use for |
| --- | --- | --- |
| auto (default) | perFrame for walk/run/idle/climb/swim names, lowest for everything else | Most clips |
| perFrame | Every keyframe gets its own shift so a foot touches every frame | Walks, runs, idles |
| lowest | One shift for the whole clip so the lowest frame touches | Jumps, attacks: keeps relative height |
| clip | Only lifts frames where feet go below the floor; never lowers | Clips that are mostly right |
| median | One shift based on the middle frame | Locomotion where one foot is planted most of the time |

**Smoothing window (perFrame).** Averages the per-frame shift over this many keyframes (3 by default) so noisy mocap doesn't make the character bounce. Each keyframe is then pulled back to within 0.01 studs of the ground, so the averaging can't leave a foot sunk or hovering. 1 = no smoothing.

**Ground Y override.** Blank = the rig's rest-pose foot level (usually −3 studs under the hip for R15). Type a number to set it yourself.

Run it **after** Loop Pose and Smooth, which move the hips slightly, and **before** Find loop. Fix All already does this in the right order.

Check clip's `foot clearance` line is how you verify: after a perFrame snap on a walk it should read close to `min 0.000`.

## Advanced: Find loop, Loop Pose, Trim & Speed

**Find loop** cuts a walk or run down to its cleanest single cycle and seals it, so it loops with no stall or foot shuffle. Needs a rig. Run it after Feet on ground.

- It finds the stride by comparing poses, so it works on R15 and R6.
- It seals the cut by crossfading the last 0.25 s into the motion that led up to the cut. The loop reaches its first pose at full speed instead of slowing to a stop.
- A clip whose first and last keyframes already match is left as it is. A clip with no repeating step, such as a fight or a jump, is left as a one-shot.
- On a raw clip that still travels, it warns you to make it in place first (Root Fix with *In place*).
- The log reads like `kept one cycle 2.20-3.41 s (1.21 s of 4.37 s), match 0.10`. Match is how closely the two ends agree: 0 = identical, under 0.30 is clean.

Use Loop Pose instead when Find loop reports no cycle, or for a clip that should keep its full length.

**Loop Pose** makes a cycle wrap without a pop. It blends the last part of the clip back toward the first frame and sets the clip's Loop flag.

- *Tail fraction* (0.25 default): how much of the end gets blended. 0.25 = last quarter. Use 0.15 for short, snappy cycles; 0.3–0.4 if it still pops.
- *Strength* (1 default): 1 = the last frame exactly equals the first. Lower values leave a softer, partial match.

**Trim** cuts the clip to a time range and rebases it so the new start is 0 s. *Trim start* / *Trim end* in seconds; end = 0 means "to the end". Use it to drop the dead air at the start of a mocap take or to carve one action out of a longer capture.

**Speed** rescales time. Factor 2 = twice as fast (half the length); 0.5 = half speed. Keyframes are retimed, not dropped, so the motion stays intact.

The log reports the new length after each of these so you can confirm the numbers.

## Advanced: Mask / Lock at rest

Controls individual limbs. Two actions on the same part list:

- **Mask** deletes those parts' poses from the clip. The parts stop being animated by this clip, so another animation (a weapon hold, a wave) can drive them on top. This is the standard "upper-body / lower-body" split.
- **Lock at rest** keeps the poses but pins them at the rest pose. The parts hold still explicitly. Use it when you want a limb visibly frozen rather than handed off.

**Restore from original** puts the ticked parts back the way they were in the original clip. Use it to take back a Mask or Lock at rest on those parts.

**Picking parts.** Tick limbs in the checklist; the Parts field fills with the real part names. Each limb row has its sub-parts next to it (Upper / Lower / Hand or Foot), so you can lock just a hand or just a shin. A half-ticked limb shows a dash. **Whole body** ticks everything. You can also type names into the field; the checkboxes follow what you type.

**Rig type.** *Auto* reads the selected clip (then the rig) and shows what it found, e.g. `Auto (R15)`. Set *R15* or *R6* by hand if auto guesses wrong. Switching converts ticked limbs to the other rig's names. R6 has no sub-parts, so only whole limbs show.

R15 names per limb: Left Arm = `LeftUpperArm, LeftLowerArm, LeftHand`; Left Leg = `LeftUpperLeg, LeftLowerLeg, LeftFoot`; Torso = `UpperTorso, LowerTorso`; Head = `Head`. R6: `Left Arm`, `Left Leg`, `Torso`, `Head`.

**Common mistake:** locking only the hand freezes only the wrist; the arm still swings because the upper and lower arm are still animated. Tick the whole limb. If you type a joint name like `LeftShoulder`, the log tells you the part name you meant (`LeftUpperArm`).

The log prints how many poses were changed, e.g. `reset 204 poses` for 34 keyframes × 6 parts. 0 means the names didn't match the clip: run List parts to see what's in it.

## Advanced: Smooth, De-pop, Keyframe reduction, Mirror, Pack validator, Debug

**Smooth** averages every pose over a window of keyframes (5 by default, must be odd). Takes the shake out of video or phone mocap. The root is excluded so the clip doesn't drift. On a clip that loops, it wraps around the loop point so the loop stays sealed. Larger windows soften more but can blur fast strikes; try 3 first on combat clips.

*The Smooth checkbox under Fix All* lets Fix All run this step for you. It smooths a clip when the legs read as trembling, which it judges by how often the motion reverses from one keyframe to the next. Clips keyed by hand, with few or unevenly spaced keyframes, are never smoothed. The reading is a judgement: a clean clip with a key on every frame can occasionally be misread and come out slightly softer. If that happens, untick the box and run Fix All again. Your original clip is untouched.

**De-pop** removes one-keyframe joint snaps: a joint at rest that jumps 8° or more in a single keyframe, like the knee pop at each step in some generated walks. It replaces the jump with an even ramp, for that joint only. Real fast moves build up over several keyframes and are left alone. *The De-pop checkbox under Fix All* runs it as part of Fix All.

**Keyframe reduction** drops keyframes closer together than 1/fps (30 by default). A 60 fps import becomes half the size with no visible change. First and last keyframes are always kept. The log reads `removed X of N keyframes`.

**Mirror** flips the clip left↔right: a right-hand punch becomes a left-hand punch. It reflects every pose across the body's centre plane and swaps Left\*/Right\* names, including keyframe markers like `LeftFootstep`. Writes `<clip>_mirrored`, or changes the clip itself when Work on a copy is off. Works on R15 and R6; custom rigs need symmetric joints.

**Pack validator** checks a set of clips against the Avatar Animation Pack slots before you upload: Idle (up to 2), Walk, Run, Jump, Fall, Climb, Swim, SwimIdle. It reports missing slots, wrong Loop flags (Jump must not loop; everything else must), lengths outside the usual range, non-Core priority, and clips with no `HumanoidRootPart` pose. Select the rig's entry under RBX\_ANIMSAVES, a Folder, or the clips themselves, then click and read the log. Studio's own upload check is still the final word.

**Debug rig** prints what the plugin sees in the rig: root part, joint count and type (Motor6D / AnimationConstraint), the solve order, which parts it treats as feet, and the ground line. Run it once per rig if Feet on ground misbehaves.

**List parts** prints every part name the selected clip animates. Use it to check names before Mask / Lock at rest.

## Walk speed report

For a walk or run, Check clip and Fix All print the speed the clip was made for, in studs per second. The report only reads the clip; it never changes it.

| Log line | What it tells you |
| --- | --- |
| `walk speed` | The WalkSpeed at which the planted feet hold still. Play the clip as it is (on an NPC, or from your own script) at that WalkSpeed and the feet won't slide. |
| `on a player character` | Roblox's default Animate script rescales walk and run animations with the character's speed. This line gives the Speed factor to apply first (Advanced → Trim & Speed) so the feet match under that script. |
| `it looks right from … to …` | The WalkSpeed range where the clip plays close to the pace it was made at, and how many times faster it would play at the default 16. |
| `foot slide left` | How far each planted foot still slides per step at the matched speed: 0.15 studs or less = locked; 0.15 to 0.30 = slight, visible only up close; over 0.30 = sliding. |

Captured walks are usually slow. A real walking pace comes out at roughly 2 to 4 studs per second, and Roblox's default WalkSpeed is 16. A captured walk used at 16 plays several times too fast and looks frantic. Set the character's WalkSpeed near the reported figure, apply the Speed factor the report gives, or capture a jog or run for default-speed movement.

The plugin takes two readings of the speed. If they differ by more than 25%, the line reads `walk speed: can't be read reliably on this clip` and gives no figure, so you never get a number you can't trust. If a walk or run can't be measured at all, the line reads `walk speed: n/a` with the reason. On a raw clip that still travels, Check clip adds a `check:` line that compares the figure with the clip's own travel.

Fights, jumps, waves and idles get no walk speed lines.

## Workflows

**Mixamo download to R15**

1. On mixamo.com, download the animation with Format: FBX Binary and Keyframe Reduction: none.
2. In Studio, select your R15 rig and click **Use this rig**. Click **Drop to floor** if it is in the air.
3. Under Advanced > Mixamo, click **Import Mixamo FBX files...** and pick the files. You can pick many at once.
4. Select the new `_r15` clips and click **Fix All**.
5. Load `<name>_r15_fixed` in the Animation Editor and play it.
6. For a walk or run, read the walk speed line and set the WalkSpeed or Speed it suggests.
7. Publish to Roblox from the Animation Editor.

**Other imported clips (the common case)**

A file straight from Mixamo uses Mixamo's own bone names (`mixamorig:Hips` and so on) and won't move a Roblox rig until it is converted. Use the Mixamo workflow above for those. **List parts** shows the names a clip uses.

1. Clip Editor → ⋯ → Import → From File → your FBX. Save as a clear name (`Walk`, `JumpStart`). The name matters: Fix All treats walk/run/idle/climb/swim as cycles.
2. Select the rig → **Use this rig**.
3. Select the clip → **Check clip**. Note `floating`, `flipped`, foot clearance.
4. **Fix All**. Read the six log lines.
5. Check clip again: `flipped=false`, clearance near 0.
6. Clip Editor → ⋯ → Load → `<clip>_fixed`. Play. Feet on the grid, facing forward, smooth loop.
7. Clip Editor → ⋯ → Publish to Roblox as usual.

**Video or AI-captured walk**

1. Import the clip and save it with walk or run in its name.
2. Select the rig → **Use this rig**. Select the clip → **Check clip**. Note `planted foot tilt`, `smoothing` and `walk speed`.
3. **Fix All**, with all three *Fix All* checkboxes ticked.
4. Read the log: step 1 says how many plants were levelled, step 6 says which cycle was kept, and the last lines give the walk speed.
5. Clip Editor → ⋯ → Load → `<clip>_fixed`. Play it looping and watch the loop point and the planted feet.
6. Set your NPC's WalkSpeed to the reported figure. For a player character, apply the Speed factor from the report first.

**Building an Avatar Animation Pack**

1. Fix each clip as above. Name them exactly `Idle`, `Walk`, `Run`, `Jump`, `Fall`, `Climb`, `Swim`, `SwimIdle`.
2. Put the `_fixed` copies in one Folder.
3. Select the Folder → **Pack validator**. Fix anything flagged (usually a Loop flag or a length).
4. Upload through Studio's Avatar Animation Pack flow.

**Several clips at once**

Select many clips (Ctrl-click) or a Folder, then click any button. Each clip is processed on its own, gets its own `_fixed` copy, and the whole batch is one Undo step. Fix All on a Folder of twenty Mixamo clips is a single click.

**Keeping the original**

*Work on a copy* (on by default) writes to `<clip>_fixed` and leaves the original alone. Clicking again with the original selected reuses the same copy instead of making `_fixed_fixed`. Select the copy itself to keep stacking fixes on it. Untick the option to edit originals in place.

## Troubleshooting

| Symptom | Cause | Fix |
| --- | --- | --- |
| "Select one or more KeyframeSequences first" | Nothing selected, or you selected the rig only | Click the clip under `ServerStorage → RBX_ANIMSAVES → <rig>` |
| "needs a rig" | No rig selected or set | Click the rig model → Use this rig |
| "Clip doesn't match this rig (2 of 16 poses found)" | Clip is for a different rig type (R15 clip on an R6 rig) | Set the matching rig; the plugin stops rather than guess |
| Fix looks unchanged in the Clip Editor | The editor is showing its old copy | ⋯ → Load → `<clip>_fixed` again |
| Still faces backwards after Fix All | Check clip didn't flag it (rare rigs) | Tick Flip 180° → Root Fix on the `_fixed` copy |
| Feet hover in lowest mode | Expected: only the lowest frame touches | Use perFrame (or auto) for walks and runs |
| Feet pop between frames in perFrame | Noisy mocap | Raise Smoothing window to 5 or 7 |
| Lock at rest `reset 0 poses` | Part names don't exist in the clip | Use the checklist, or List parts to see real names |
| Arm still moves after locking the hand | Only the wrist was locked | Tick the whole limb (Upper + Lower + Hand) |
| Loop still pops | Tail too short | Raise Tail fraction to 0.3–0.4 |
| Undo greyed out after a fix | The Clip Editor took over the undo stack when you loaded the clip | Undo before loading, or delete the `_fixed` copy by hand |
| `foot clearance: n/a` | Rig has no Motor6D/AnimationConstraint joints, or feet not found | Run Debug rig; for custom rigs name feet `LeftFoot`/`RightFoot` |
| Nothing happens, no log line | Button errored | Look in Studio's Output window (View → Output) and report it |
| Feet still point toe-down after Fix All | The Flatten feet box under Fix All is unticked (log step 1 says off) | Tick it and run Fix All on the original clip again |
| Walk stalls or shuffles where it loops | The end was blended into the start, not cut to one cycle | Run Find loop, or Fix All with walk or run in the clip name |
| Walk looks frantic in game | A slow captured walk is playing at WalkSpeed 16 | Use the WalkSpeed or Speed factor from the walk speed report |
| Legs still tremble after Fix All | The Smooth box under Fix All is unticked, or the clip read as clean | Tick it, or click Smooth on the fixed copy |
| Fix All softened a clean clip | It read as trembling; a clean clip with a key on every frame can be misread | Untick Smooth under Fix All and run Fix All again |
| An imported Mixamo clip doesn't move the rig | The clip still has Mixamo bone names (List parts shows mixamorig: names) | Select it and click Retarget Mixamo to R15, or use Import Mixamo FBX files... on the original file |
| Import Mixamo FBX says a file was skipped | The file was downloaded with keyframe reduction, or it is not a binary FBX | Download it again with Format: FBX Binary and Keyframe Reduction: none |
| Hands go through the floor on a crawl, a fall or a floor pose | The plugin keeps the head, torso and thighs above the floor; it does not re-pose the arms | Known limit. Adjust the arm keys by hand in the Animation Editor if it shows |
| Two hands that should meet (a sword grip, a clap) have a gap | R15 arms are shorter than a Mixamo character's, and the conversion copies joint angles | Known limit. Adjust the arm keys by hand if it shows |

Saves aren't where you expect: since late 2023 the Clip Editor stores saves in `ServerStorage → RBX_ANIMSAVES → <rig name>`, not under the rig. Each rig has its own folder.

## Compatibility and limits

**Supported**

- R15 rigs, including the 2026 Rig Builder rigs that use `AnimationConstraint` joints
- R6 rigs
- Custom block rigs built from `Motor6D` or `AnimationConstraint` joints (Feet on ground needs parts it can recognise as feet: `LeftFoot`/`RightFoot`, anything named *foot*, or it falls back to the two lowest parts)
- Clips saved by the Clip Editor or imported from Roblox or FBX. Tested with Mixamo downloads, AI video capture clips and clips made in the Animation Editor
- Mixamo FBX downloads (binary FBX, Keyframe Reduction: none), converted to R15 clips
- Studio light and dark themes

**Not supported in this version**

- Skinned-mesh rigs animated through `Bone` instances. Their clips target bones, not parts, and the solver doesn't read them yet. This is the top item for the next release.
- Clips that animate the `HumanoidRootPart` itself. Roblox convention keeps the root fixed; the plugin assumes that.
- `CurveAnimation` clips (the newer curve format). Save as `KeyframeSequence` first.

**Known limits**

- The walk speed report assumes the character walks forwards. A clip of a character walking backwards is read the wrong way round.
- A clean clip with a key on every frame at 20 fps or more, whether keyed by hand or professionally cleaned, can read as trembling, so Fix All may smooth it. Untick *Smooth under Fix All* for those clips.
- Mixamo conversion needs a standard R15 rig. R6 and custom rigs are not supported for it.
- Mixamo conversion reads binary FBX files downloaded without keyframe reduction. Reduced files are refused.
- On converted Mixamo clips, hands and shins can pass through the floor in crawling, falling and lying poses, and two-handed poses leave a gap between the hands. Fingers are not carried over.

**What it never does**

- Changes any part of the place other than the selected clips, apart from moving the rig when you click Drop to floor
- Uploads anything; publishing stays in the Clip Editor
- Touches the original clip while *Work on a copy* is on

**Version.** v1.2 — built and tested against Roblox Studio, November 2026. The plugin is a plain `.rbxm`; you can open it in Studio and read every line.
