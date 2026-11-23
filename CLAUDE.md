# Rooftop Drift

## Rule for every session

**After every new feature or fix, update this file before replying:** add it to "What's done", tick or remove it from "To do", and record any new decision under "What we decided". Then republish the game and give Parth the live link at the bottom of the reply.

Parth prefers short, to-the-point replies.

## How changes are shipped (agreed 19 Nov 2026)

The game is live for the public at https://parthysarkar.github.io/Rooftop-Drift/, so nothing reaches it untested.

1. Fix one thing. Run the headless tests in `/home/claude/t` first.
2. Publish to the **claude.ai artifact only** (it is private to Parth) and give him that link.
3. He tests on his iPhone and his computer and answers yes or no.
4. Only on a yes, push to GitHub. On a no, fix and republish the artifact. The live game never moves.
5. If a bad change ever does reach the live site, revert that commit and push; Pages rebuilds in about a minute. Local snapshots are at `/home/claude/index.backup*.html`.

**One change in flight at a time.** Do not start a second fix while the first is still waiting for Parth's verdict, and do not push an untested change. If he raises a new problem while one is open, say what is still open and ask whether to park the new one or switch. He decides; the default is to finish what is open first. The exception is something breaking the live game, which jumps the queue.

**Check at the start of every session** whether the artifact, this repo and `/home/claude/drift/index.html` match, and say so in one line.

---

## 1. What the project is

A 3D car drifting game that runs in the browser, inspired by the Tokyo Drift parking-garage race.

- You race one rival up a multi-floor parking garage. First to the roof wins.
- There are three tracks so far (see "Tracks" in section 2), chosen on the menu.
- Main target is iPhone Safari (touch controls), and it also works on desktop with a keyboard.
- Live link: https://claude.ai/artifact/Dg8iRgqowvB5tzS3dzWDoG
- Artifact slug: `66a46ccd-dc0b-4a2e-8a5c-4e45fbbedbc3` (republish with `publish_files`, mode `patch`, using the latest version as `base_version`).

### Files

| File | What it is |
|---|---|
| `index.html` | The whole game: HTML, CSS and JavaScript in one file |
| `cars/<key>.js` (12 files) | Car models as base64 GLB wrapped in JS (the host refuses `.glb` and `.bin` files) |
| `cars/*.glb` | The same optimised models as real GLB files (local only, not published) |
| `CLAUDE.md` | This file |

Libraries: three.js r128 from cdnjs, plus `GLTFLoader.js` from `cdn.jsdelivr.net/npm/three@0.128.0`.

### Working files (outside this folder)

- `/home/claude/m/bake.py`: rebuilds each Sketchfab model into one body plus four separate wheels (`wheel_fl`, `wheel_fr`, `wheel_rl`, `wheel_rr`), at real scale, with the pivot on the ground. Usage: `python3 bake.py <src.glb> <out.glb> <length_m>`.
- `/home/claude/m/glb.py`: GLB reader used by `bake.py`.
- After baking, shrink with `npx gltf-transform optimize ... --simplify-ratio 0.34 --texture-compress webp --texture-size 1024`, then re-wrap each model as `cars/<name>.js`.
- `/home/claude/t/test*.js`: headless Playwright tests (Chrome with SwiftShader), with the page served on `localhost:8765` and three.js loaded from local `node_modules`.

---

## 2. What we decided

### Look
- Cel-shaded anime style, matching Parth's reference screenshots: widebody cars, teal garage, big tyre smoke.
- Speed feel comes from motion blur (frame accumulation plus radial blur), camera shake and the field of view widening with speed.
- **Night run (dark mode) is the default:** the garage lights are off, and the scene is lit by headlights, tail lights, the city glow and the neon arrows. A "Garage lights: On" option keeps the original lit look.
- **Neon arrows:** blue outlined chevrons on the outside wall of every turn, with the light running in the direction of travel. They start before the big hairpins as a warning. Parth supplied a blue neon chevron reference.
- **Headlights:** three pieces, after the November 19 rework. (1) A real `SpotLight` on the nose that lights the road, walls and the other car. (2) Small bright sprites inside each lamp housing, plus a small halo. No big glow around the lamp (Parth said it broke the illusion). (3) The pool on the road: a 14x6 grid laid onto the road surface every frame (`beamFit`), which follows slopes and corners and fades out over `BEAM_FADE` metres as it reaches the edge of the road, dying about where the wall or barrier stands.
- **Do not go back to flat cards for light on the road.** There used to be two "streak" quads glued to the car body. Body roll in a drift reaches 5.2 degrees, which drops a 0.8 m wide card's outer edge 3.6 cm while it only floats 2.2 cm above the ground, so the road clipped it along a straight line. That was the jagged cut Parth reported, and the cards were removed on 19 Nov 2026. The pool had the same symptom from a different cause: its vertices used to be dragged back onto the road at the edge while their UVs stayed put, squashing the gradient into a hard line. The fade replaced that. `streakM` is still defined because the garage-lights code reads it; nothing draws with it.
- Tail lights are a small red core with a soft glow, brighter when braking or drifting.
- Smoke comes only from the two rear tyres, and only while sliding. It is soft and grows as it trails.
- No orange cones.
- **Boost flames:** yellow-orange (not blue), coming from each model's real exhaust tip: Evo IX one on the right, RX-7 one on the right, R34 one on the left, 350Z two. The boost screen tint is warm to match.

### Cars
- **12 cars**, all supplied by Parth from Sketchfab: Evo IX, RX-7, Skyline R34, 350Z, Supra, AE86 Trueno, 911 RWB, AMG GT3, AMG GT R, GT-R R35, M3 GTR, Charger R/T. Each is reduced to roughly 50k to 90k triangles (engine bays removed), keeps its own textures, and uses the game's toon shading.
- **Loading on demand:** only the player's and rival's cars are loaded (`loadCar`, `useCars`). At most four models stay in memory (`trimCars`); older ones are freed. This keeps 12 cars as light as 4 were.
- **Car list (Parth's request):** the menu shows two buttons, "You: ..." and "Rival: ...". Each opens a list; choosing a car closes it. The car the other side is using is greyed out. No long row of car buttons.
- **Lights on every car:** small headlight points and tail-light glow placed per model (`head`, `tail` in `CARS`), exhaust flames at the real exhaust tips (`ex`), and no light patch on the bonnet (the headlight light source sits ahead of the front bumper).
- **Fan-project notice:** the menu credits open with "A free fan project, not affiliated with or endorsed by Universal Pictures. Non-commercial, and it stays that way." The repo has `CREDITS.md` with the same notice, the per-model licence table, the sound credits and the trademark line, linked from `README.md`. Keep all of this in step if models or sounds change.
- **Licences:** ten cars by Ddiaz Design (CC BY-NC-SA 4.0), the R34 by vecarz (CC BY 4.0), the 911 RWB by autoNgraphic (CC BY 4.0). Credits are in the menu, and the game must stay non-commercial.
- **Not added:** `gporsche_turbo_2.glb` by SWIZ. Its licence is "Copyright to SWIZ" (all rights reserved, not Creative Commons), and it is 138 MB with about 3 million triangles. It needs the author's permission before it can go in.
- RX-7: its front "swatch" decal is removed because of a broken texture mask.
- **Wheels:** each wheel spins about its own axle. Camber is kept in a `<wheel>_tilt` node above the spin group (`untilt.py`), so cambered wheels (the R35 has 4.4 degrees) do not wobble. On the AE86 and M3, rims are the `SM_Hub` / `SM_Disk` parts.
- Converting a model: `/home/claude/m/bake2.py <src.glb> <out.glb> <length_m> [flip]` (wheels found by node and material names, ground set at the tyre contact, `flip` when the model faces the other way), then `untilt.py <in> <out>`, then `gltf-transform optimize`, then wrap as `cars/<key>.js`. `lamps.py` measures lamp positions and plots the rear for exhausts.

### Tracks
- Parth wants several tracks (levels) to switch between, all with the same lighting and atmosphere. New tracks are added as new entries; existing tracks are not changed.
- **Spiral** (the original): 5 floors, mostly right turns, about 300 m per floor.
- **Switchback** (added after Parth's in-car tandem drift video, where the turns go left and right): 3 floors, about 586 m per floor. Each floor has a left-right-left chicane, a right hairpin, a left hairpin, a right-left-right chicane, another right hairpin, a second chicane and a long fast right sweeper.
- How tracks work in code: `TRACKS` in `index.html` is a list of `{id, name, loops, fin, segs}`. `segs` is one closed loop of straights `['s', metres]` and arcs `['a', radius, degrees, +1 right / -1 left]`; segment 0 must be a 30 m straight (start grid), and the arcs must add up to +360 degrees. `layoutTrack()` computes the path and `dressTrack()` rebuilds all the garage meshes, so a new track only needs a new `TRACKS` entry. Parallel sections need about 30 m or more between centre lines so walls and walkways don't overlap.
- **Neon City** (built for the cinematic mode): a 1.3 km point-to-point route on an elevated city expressway at night, with pink and blue neon, towers, a wet road and both left and right turns. It is a different theme (`theme:'city'`, `open:true`, `base:30`), dressed by `dressCity()` instead of `dressGarage()`. The wet look is faked: the road is slightly see-through and each car has an upside-down copy drawn under it as its reflection, plus coloured glow patches on the road under signs and lamps.
- The menu has a Track picker, and the results screen has a "Next track" button. Best times are saved per track.
- **The game is a night game, and it stays one.** A fourth track, "Sunset Drive" (a Vice City style dusk sky over the Neon City expressway), was built and tried on 20 Nov 2026 and then dropped: Parth played it and it broke the illusion the rest of the game is built on. The whole look depends on darkness — headlights, tail lights, neon, smoke and flames all read because it is night, and the game is about Japanese nightlife. Do not propose a daylight or dusk track again, and do not chase whatever game is in the news; this one is its own thing.

### Cinematic mode (Parth's idea, from 19 reference frames)
- Goal: during a race the view cuts to film-style angles with slow motion, then returns to the normal game camera on straights and quiet parts. The player keeps control, with the illusion of driving through the shot.
- The in-race and finish shots run **only on Neon City** (`cineActive()`). The opening runs on **every track** (`cineOpen()`), menu toggle "Cinematic shots", default on.
- **Opening (about 7.6 s, hold Drift to skip):** front three-quarter close-up of your car, front wheel close-up, rival rear three-quarter, rival front low, top-down of both cars on the line (3), front wide of both (2), low from behind (1 and GO, with launch smoke), then a nitro view (close on the exhaust with flames, pulling back and up) that eases into the game camera. Parth rejected the earlier low side tracking shot here because the jump from side view to rear view was not smooth.
- **Garage opening (Spiral and Switchback, Parth 23 Nov):** the same first four shots, then one shot per number: front wide on 3, low from behind on 2, and on 1 the camera glides (1 s blend) from that low shot into the usual wide start camera, which eases in behind the car after GO as before. No top-down (the garage ceiling is too low, and Parth cut it), no nitro view, no launch smoke.
- **During the race:** a drift in a corner of 90 degrees or more triggers one of: low rear-wheel close-up, high aerial, low front shot the cars drive towards, front three-quarter close-up. A boost triggers an exhaust flame close-up. Running side by side with the rival on a straight triggers a high long-lens (near orthographic) view or the side tracking shot. At least 3.2 s between shots, and side-by-side shots at most every 16 s.
- **Finish:** slow-motion low shot at the line, then an orbit around the car, then the results.
- **How it works in code:** `SHOTS` is the shot library (`dur`, `ts` time scale, `fov`, `dof`, `focus`, `cam(u)`), and `cineUpdate()` is the director. `step(dt)` separates real time from game time (`gdt = dt * timeScale`), so slow motion slows the rival and the race clock too and the race stays fair. While a shot is on screen a steering assist (70%) keeps the car on the road. Shots use depth of field (needs WebGL2), letterbox bars, and hide the HUD. The blur buffer is reset on every cut.
- **Depth of field (Parth's rule):** each shot focuses on one feature and the rest falls off. Opening shots: near headlight of your car, front wheel, rival's tail light, rival's headlight. Drift shots: rear wheel, near headlight, exhaust for the flame shot. Two-car shots keep both cars sharp. Each shot has a `foc()` function returning the focus point and the half-depth that stays sharp; depth is measured along the lens axis.
- Not attempted: drivers visible inside the cars, rain and spray, reflections of the scenery (only cars are reflected).

### Winner sequence (Spiral and Switchback only, from 5 Tokyo Drift reference frames)
- Plays only when the player wins on a garage track. Neon City has its own sequence (next section). Losing shows the result card as before.
- The set is built as first drawn and then mirrored (`g.scale.z=-1`, and `winCar`/`winCam` flip z) so the ramp climbs to the right like the film; the WINNER letters are turned back so they still read correctly. No marks are painted on the ramp: drift marks come from the normal skid-mark system.
- It runs on a separate set (`winBuild()`, built on first use, parked 1500 m from the track): garage level, a 2.5-turn spiral ramp with globe lamps, a bridge, a rooftop with a neon WINNER sign. No crowd (Parth's choice).
- The car is scripted, not driven (`winUpdate`, `winPath`, `winCar`). Shots and times are in `WT`: 1 garage pan, 2 low behind at the ramp entry, 3 high close front on the ramp, 4 straight down on the tower, 5 rooftop (slide in, burnout, one full donut, WINNER letters light one by one). Cameras are in `winCam`.
- The result card appears after the donut (about 16 s); the camera then moves the sign and car clear of the card. Tap or Space skips to the end.
- **The car does not lean in either winner sequence** (`state==='win'` forces the roll target to 0). The winner shots are close-ups, and a real donut lean has nowhere to go: the body rolls about its centreline at ground level, so 5.2 degrees puts the outer tyre 8 cm into the road. Lifting each wheel back up instead (`plantWheels`, tried on 19 Nov) pushed the tyre into the arch on the widebody cars. Flat is what reads naturally; do not reintroduce the lean. Normal racing still rolls as before.
- State is `'win'`; `winEnd()` restores the track, lights and rival and is called from startRace, setTrack and the Garage button.

### Winner sequence, Neon City (from Parth's two Shibuya crossing photos with the route marked in green)
- Plays when the player wins on Neon City, after the slow-motion finish shot (cut at 2 s; 1.15 s with cinematic shots off).
- Own set (`winBuildCity()`, built on first use, parked at -1500/+1500): a proper square crossing where two 26 m roads meet, zebra crossings on all four sides plus both diagonals, towers lined up with the streets on the four corner blocks, and a round glass tower with a curved pink screen on the north-east corner. No crowd, no real brand names.
- Route (`WCP`, `wcPath`): down the north road, long left sweep round the tower's corner, short hook, stop under the screen.
- **One unbroken run, and it must stay that way.** `wcRun(t)` gives the distance along the path and the speed at any moment: the car cruises in at `WCV0` (about 50 km/h) until `WCTB`, then the slide scrubs speed off evenly until it comes to rest under the screen at `WCTT`. The flick into the drift happens at s=56, before the first cut, so the car is already sideways when the camera changes. Each shot used to re-run its own stretch of the path, which threw the car 42 m backwards at the first cut and read as the car stopping and restarting (Parth, 19 Nov). Cameras read `win.s` and never compute their own position along the path. If the timing is ever retuned, check the sequence for backward jumps in `win.s`.
- Three shots, phases in `WCT`: 1 high over the crossing, tracking the car in, 2 low ahead of the car as it slides past, 3 on the ground behind the car while it is still drifting, looking up past it at the lit WINNER screen (Parth's correction: the sign must be in the drifting shot, not shown after the car stops). Code: `winUpdateCity`, `winCamCity`. `win.kind` is `'roof'` or `'city'`.
- **The WINNER screen** is one curved panel per letter, each a 288x576 canvas drawn as an LED board (`LP` = cell size) with the glyph fitted inside a margin so it cannot touch the panel edge. It was half that resolution with 16 px cells until 19 Nov, when Parth said it was too pixelated to read.
- Result card after about 9.5 s; same skip as the rooftop sequence.
- Result card left untouched for 60 s (win or lose, any track) returns to the garage menu (`resIdle` in `step`). Any tap or key restarts the minute.

### Controls and rules
- **No zoom on touch screens:** iOS zooms on a double tap even with `user-scalable=no`, which left the game stuck zoomed in when Parth tapped the Nitro button twice. The game surfaces (canvas, `.ctl`, `.btn`, `.hud`) are `touch-action:none`, the HUD and menu buttons are `touch-action:manipulation`, a `touchend` guard swallows a second tap within 400 ms anywhere except the menus and the Sound/Music/Pause buttons (so their clicks still fire), and `dblclick` and the Safari `gesture*` (pinch) events are blocked.
- Gas is automatic. The left pad steers, hold Drift to slide, and Brake slows the car.
- **Nitro (player-controlled):** drifting fills a nitro tank (3 seconds when full, 1 second at the start). W on the keyboard, or the Nitro button on touch screens, switches it on; pressing again switches it off; it stops by itself when empty. A meter under the speed shows the level. A hard wall hit costs half a second of nitro. The rival's boost stays automatic.
- Keyboard: arrows or A/D to steer, W for nitro, Space to drift, S or Down to brake, Esc to pause or resume, R to restart.
- Start: both cars line up side by side, and a wide start camera (pulled back further on phones) shows both. After GO, the rival launches harder and the camera eases in behind the player over about 2.6 seconds. The rival holds its lane for the first 2.5 seconds, and cars cannot collide before GO.
- Three rival difficulties: Easy, Normal and Hard.
- **Rival behaviour (Parth's request):** the rival must not drift or weave on the gentle bends in the straights, and must not block or crash into the player. It only drifts in turns of 90 degrees or more, holds its lane through chicanes, takes the side of the track the player isn't using when they are close, and slows down rather than ramming the player from behind.
- **Rival boost (fairness):** the rival charges a boost while drifting in big corners and fires it on exit, by the same rule as the player (0.45 s minimum drift, up to 2.4 s of boost), with exhaust flames. During a boost its top speed rises from 35 to 42 m/s and it accelerates harder. Finish times barely change because the rival's pace is already tied to the gap to the player.
- **Rival lane changes:** the rival must move across the track on a diagonal with its nose turned, never slide sideways. Its sideways speed builds up and eases off gradually (capped at about 3.4 m/s), and the car yaws into the move (capped at about 11 degrees).
- **Car contact:** each car is two circles (nose and tail). Side contact pushes both cars apart sideways with no speed loss; if the rival reaches the player's tail it backs off; if the player hits the rival's tail it is a soft nudge.

### Sound
- No engine, beeps or wind (Parth found them annoying). The only game sound is a tyre screech built from Parth's reference mp3, embedded and looped while a car is sliding. The rival's screech fades with distance.
- **Nitro sound:** Parth's freesound "nitro activation" clip, trimmed to 2.36 s (the hit plus the whoosh, faded at the end), mono 24 kHz 56 kbps, embedded as base64 in `NITRO_SND`. It starts when the nitro lights up and is faded out in 0.16 s if the nitro is switched off or runs dry (`nitroSound`, watched once per frame in `draw`). It goes through the audio master, so the Sound button silences it. The rival's boost makes no sound.
- **Two buttons during a race, down the top right corner:** "Sound on/off" switches the tyre screech (the audio master gain), "Music on/off" silences the song without unloading it, and "Pause" below them. Both choices are kept in `localStorage` (`rd_sfx`, `rd_mus`) so they survive a reload. The Music button only appears when a song is loaded, so it is never a dead control. All three are the same width (`min-width:108px`) with 10 px gaps, and the race timer moved to `top:134px` to stay clear of them.
- **Music:** the Tokyo Drift song is not embedded, because hosting a commercial recording isn't allowed. Instead the player picks a song from their own device ("Choose a song from this device"). It is saved in the browser (IndexedDB) and loops during races. The file picker lists audio types explicitly so iPhone opens Files rather than Photos.

### Multiplayer (built; two players per room)
- PeerJS (`unpkg.com/peerjs@1.5.4`), loaded only when a room is created or joined, so single player never pays for it.
- The host's peer id is `rooftopdrift-room-<6 digits>`, so the code IS the address: no server of our own. Invite link is `?room=<code>`, shared by Copy, WhatsApp or the system Share sheet.
- Each side drives its own car and sends it 20 times a second (`netTick`); the friend's car is drawn from those messages about 100 ms in the past, interpolated between the two either side of that moment (`netCar`). Ghost position error measured at roughly 1 m.
- The host owns the track, the garage lights and the start; the guest's picks for those are locked (`#menu.guest`). Each player picks their own car and the two cannot be the same (`netCarClash`).
- Finishing: each side tells the other its time, the host compares them and announces the result (`netLocalFinish`, `netDecide`, `netResult`); the guest falls back to its own judgement after 2.5 s if the host goes quiet.
- A friend who closes the tab is noticed by a 7-second silence watchdog in `netTick`; the host keeps the room open on the same code, the guest can rejoin.
- Off during a two-player race: the cinematic shots and the winner sequences (`cineActive` and `win.pending` both check `net.on`), because slow motion would pull the two games apart and the celebration would leave the other player waiting. Pause is off too.
- Needs a normal web address. The claude.ai artifact blocks peer-to-peer connections, so there the game says so and stays single-player.
- **GitHub Pages:** the repo is `parthysarkar/Rooftop-Drift` (the clone's remote spells it lower case; GitHub redirects). Pushed on 2026-11-17: `git -C /home/claude/rooftop-drift push origin main`. Pages itself cannot be switched on from here (the API proxy blocks it), so Parth does it once in Settings -> Pages -> Deploy from a branch -> `main` / root. `.nojekyll` is in the repo. The game is then at https://parthysarkar.github.io/Rooftop-Drift/.

## 3. What's done

1. First version: 3D garage, a 5-loop spiral track, AI rival, drift physics, boost, motion blur, camera shake, touch and keyboard controls, simple placeholder cars.
2. Parth's four car models added, with spinning and steering wheels, a car picker, and a music-from-device button.
3. Game sounds replaced by the tyre screech only, Esc pause plus an on-screen Pause button, rear-tyre-only smoke, iPhone audio picker fix, and the camera pulled back during drifts.
4. Headlights and tail lights added (first attempt was too bright).
5. Headlights reworked to small points with faked floor light, cars start side by side, and the intro camera added.
6. Fixed the rival shoving the player before GO, removed the dotted glow under the cars and sharpened the depth calculation, and fixed the lamps on all four cars (they now draw on top of the lens glass).
7. Night run dark mode with animated blue neon chevrons, cones removed, and the Sound and Pause buttons moved to the top right.
8. RX-7 texture fix: front plate patches, windshield surround drawn as dark trim, and interior cut-outs fixed on all cars.
9. Track system refactored so tracks can be swapped, the second track "Switchback" added with left and right turns, a Track picker on the menu, "Next track" on the results screen, and best times saved per track.
10. Rival driving and car contact reworked: no drifting on gentle bends, the rival gives the player room and doesn't ram, and contact is soft (see "Rival behaviour" in section 2).
11. Rival lane changes now steer across diagonally instead of sliding. Fixed a bright patch on the centre of the bonnet: the headlight light source sat inside the car's nose, and now sits just ahead of the front bumper. Lens glass now reacts to scene lighting.
12. Rival car choice now sticks (tap your own car again to change it; shown on the menu). Boost flames are orange and placed on each car's real exhaust.
13. Neon City track added (open city route, wet road with car reflections, neon signs, towers). Cinematic mode added on that track: opening sequence, in-race drift, boost and side-by-side shots with slow motion and steering assist, finish sequence, depth of field and letterbox.
14. Launch shot 8 changed from side tracking to a nitro view that pulls back into the game camera. Depth of field reworked so every shot has its own focus point (from Parth's marked-up screenshots). Close shots now sit on the side of the car with more road.
15. The rival now earns and uses drift boosts, with flames.
16. Eight more cars added (12 total) with per-car lights and exhausts, on-demand loading, and an open-select-close car list for both player and rival. Fixed wheels looking sunk into the road: floor effects (shadow, light patches, decals) used a slope-scaled depth offset that pulled them in front of the tyres in low shots; they now sit 1 to 2 cm above the road with a constant offset. Menus scroll properly on small screens and the Sound button hides while a menu is open.
17. Nitro is now controlled by the player (W key or Nitro button, on/off toggle, tank filled by drifting, meter under the speed). Fixed the M3 GTR's rims not spinning and the GT-R R35's wheel wobble; all 12 cars had their wheels re-aligned to their spin axis.

---

18. Winner sequence for Spiral and Switchback: after a win the game cuts to a spiral ramp and rooftop set, five shots matched to Parth's reference frames, ending with a burnout, a donut and a neon WINNER sign.

19. Winner sequence for Neon City: scramble crossing set, three shots, glass tower with a pink WINNER screen. Result card now returns to the garage after 60 s without input.

20. Fixes after Parth's review: far-skyline towers can no longer stand on a track (they were landing on the Neon City start); Neon winner set rebuilt as a real square crossing with the WINNER screen in the drifting shot; rooftop sequence mirrored; painted ramp marks replaced by real drift marks (skid marks now follow the road slope).

21. Headlight pool on the road is no longer a rigid plane fixed to the car (it cut into sloped and curving road, giving jagged edges and dark patches in corners). It is now a 14x6 grid laid on the road surface every frame (`beamFit`, using `locate` for road height, `winGround` on the winner sets) and it stops at the walls.

22. Two-player rooms over PeerJS: Create room gives a 6-digit code, Join room or the invite link joins it, both cars race live, the host decides the result. Repo parthysarkar/rooftop-drift for GitHub Pages.

23. Sound and Music are now separate buttons during a race (the old single Sound button also muted the music, and with no song loaded it seemed to do nothing). Both settings are remembered between sessions.

24. Nitro sound added (Parth's clip) and double-tap zoom blocked on touch screens.

25. Fan-project and licence notice added to the menu credits, plus CREDITS.md in the repo (for Parth's LinkedIn post about the game).

26. Headlight rework after Parth spotted a jagged cut while drifting: the road pool now fades at the edge of the road instead of being dragged onto it, and the two flat streak cards glued to the car were removed outright (body roll pushed them under the road, where the road clipped them into a straight line). Confirmed by Parth on 19 Nov: "more natural".

27. Neon City winner sequence reworked: the car now makes one continuous drift from the approach to the sign (it used to jump backwards at every camera cut), and the WINNER screen is five times the resolution with finer LED cells and letters that fit their panels. Both confirmed by Parth on 19 Nov.

28. Wheels no longer sink into the road during the rooftop burnout and donut: the car stays flat through both winner sequences instead of leaning. Confirmed by Parth on 19 Nov.

29. Opening cinematic added to Spiral and Switchback: four close shots, then front wide on 3, low behind on 2, a glide into the usual start camera on 1, then GO. Confirmed by Parth on 23 Nov.

## 4. To do

### Check on Parth's iPhone (not yet confirmed)
- [ ] Song picker opens Files and plays the mp3, and the Sound / Music buttons both respond to a tap
- [ ] Tyre screech and nitro sound: volume balance and feel
- [ ] Double tap during a race no longer zooms the page
- [ ] Dotted speckle under the cars is gone
- [ ] RX-7 windshield rim is fixed
- [ ] Neon chevron size and brightness, and whether the track is readable in night mode
- [ ] Zoom-in after the start feels smooth
- [ ] Rival no longer blocks on the bends, and corner contact feels fair (in tests some light touches remain on Hard when racing side by side)
- [ ] Rival lane changes look natural, and the bonnet light patch is gone
- [ ] Neon City and cinematic mode: frame rate on the iPhone (reflections double the car geometry; depth of field runs during shots), whether shots are too frequent or too long, and how the framing looks in portrait
- [ ] Switchback track: how the left hairpin and the chicanes feel with touch controls, and race length (about a minute)

- Winner sequence: timing and feel of the five shots, smoke amount in the burnout, WINNER sign fitting the screen in portrait, result card not touching the sign.

- Neon City winner sequence: the cruise speed of the approach (slower than before, because one continuous slide has to end under the sign) and whether the car reads big enough in the overhead shot.

- Multiplayer on a real connection between two devices (only tested locally so far): feel of the friend's car, behaviour on a phone that gets backgrounded, two players on slow mobile data.

### Known issues
- [ ] RX-7 tail lights are weak from straight behind, because the lamps sit on the corners.

### Visual polish (ideas offered; Parth hasn't chosen yet)
- [ ] More garage detail: signs, parked cars, wall markings, pipes, a richer skyline
- [ ] Car rendering: anime outlines, paint highlights
- [ ] Effects: light shafts, better sparks, speed lines
- [ ] Menu and HUD: a garage showroom lobby, a nicer speedometer and results screen

### Car roster
- [ ] Lighter versions of each model for other players' cars in multiplayer.
- [ ] The SWIZ Porsche, only if Parth gets permission from the author.
- [ ] Exhaust positions on the AMG GT3 and M3 GTR are estimates (no clear tailpipes on the models).

### Cinematic follow-ups
- [ ] Parth's feedback on each shot against his reference frames
- [ ] In-race and finish shots on Spiral and Switchback (opening done; ceilings and pillars need camera checks)
- [ ] The starter character on the line (postponed; needs a model from Parth or Mixamo)

### Track ideas
- [ ] More tracks if Parth wants them (same atmosphere, different layouts)

### Last
- [ ] Multiplayer: rooms, room codes and WhatsApp invites (see section 2)
