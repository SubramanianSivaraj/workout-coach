# Workout Coach v5 — Real 3D Mannequin (weekend build plan)

**Target:** Sat 2026-07-11 → Sun 2026-07-12 · 5 focused sessions
**Branch:** `v5-3d` — ALL commits go here. NEVER push to `main` until the final
deploy step (GitHub Pages auto-deploys every push to `main`).

## Session protocol (read this first, every session)

1. `cd "Potfolio Dashboard/workout-coach-repo" && git checkout v5-3d && git pull`
2. Open this file, find the first unchecked task group below.
3. Do ONE session's scope only — do not run ahead even if budget remains.
4. Tick the boxes in this file, commit everything to `v5-3d`, push.
5. Update the "Status log" at the bottom with one line.

Preview locally: `.claude/launch.json` config `workout-coach` serves the project
folder on :5544 — copy repo `index.html` → project `workout-coach.html` to test,
or serve the repo folder directly.

## Locked architecture decisions (do not relitigate)

- **three.js vendored as a separate file** (`three.min.js`) in the repo, loaded
  via `<script>`. The HTML stays fully functional without it: if WebGL or the
  script fails, demos silently stay SVG. 3D is a progressive enhancement with a
  2D/3D toggle in the demo modal — the SVG renderer is NOT removed.
- **Mannequin from primitives** (capsule/sphere meshes per bone), not a loaded
  model file. Same skeleton proportions as the SVG engine (LEN table).
- **Reuse existing pose data.** Side-view exercises map angles directly onto the
  sagittal plane. Exercises that used 2D cheat views (front mode: lateral raise,
  upright row; flye mode) get a small `p3` override per pose adding real
  abduction angles. No exercise gets re-authored from scratch.
- **Camera:** simple custom drag-orbit + pinch zoom (no OrbitControls import),
  slow auto-rotate until first touch.
- **No audio, no tracking, no new storage keys** (except `wc-3d` toggle pref).

## Sessions

### Session 1 — Sat AM: scene foundation ✅ DONE
- [x] three.js r170 vendored as `three.module.min.js` (UMD builds discontinued
      upstream; loaded via a `<script type="module">` bridge that sets
      `window.THREE` and fires `three-ready`) + WebGL check + SVG fallback
- [x] 3D stage: renderer (pixelRatio capped at 2), hemi+directional lights,
      floor disc with PCF soft shadow, transparent canvas over stage gradient
- [x] Custom drag-orbit + pinch zoom (pointer events) + slow idle auto-rotate
- [x] Static standing mannequin from primitives (torso, hips, shoulders, neck,
      head, 2×arm, 2×leg, feet) using LEN proportions, skin/accent materials
- [x] "3D" toggle in demo modal stage (persists in `wc-3d`), Overhead Press
      only via `can3D()` gate; full dispose on toggle-off/close (verified)
- [x] Commit: "v5 s1: three.js scene, mannequin, orbit, modal toggle"

### Session 2 — Sat PM: rig + pose mapping ✅ DONE
- [x] Joint hierarchy in buildRig(): root(pelvis)→torsoG→neckG/head +
      shoulder/elbow/hand groups (children of torsoG), hip/knee/foot groups
      (children of root). +z limbs = near/base pose fields, −z = *2 far fields.
      Angle convention documented in code comment (bones rest along −Y,
      rotation.z=+angle; torso rests +Y, rotation.z=−angle; children subtract
      parent angle since 2D angles are absolute)
- [x] Pose interpolation: same lerpPose/ease/dur/hold ping-pong as SVG engine,
      stepped inside the 3D render loop
- [x] Verified animating in 3D: Flat Bench Press, Squats, Deadlift, EZ Bar
      Curl, Plank Hold (+ Overhead Press) — rig mounts, animates, disposes
- [x] Lying orientation correct (bench press horizontal on back, plank prone
      in a straight line). NOTE: lying exercises float at bench height — the
      bench MESH is session 3 scope (equipment)
- [x] Commit: "v5 s2: rigged mannequin animates side-plane exercises"

### Session 3 — Sun AM: all 28 exercises + equipment ✅ DONE
- [x] pose3() adapter (instead of per-pose `p3` overrides — cleaner): front
      mode → standing body + abd/fabd from the 2D ua/fa; flye → lying body,
      arms vertical + abd from `a`; Russian twists get real torso Y-twist.
      rig.apply() gained abd/fabd (rotation.x, ±s) and twist (torso rotation.y)
- [x] Equipment: dumbbells (handle + accent plates) attached to rig hand
      groups (both hands unless *2 arm fields exist, e.g. one-arm row);
      barbell/EZ bar spanning both hands with sized plates (15/8 units);
      lying bench derived from torso angle (mirror of SVG drawBenchAuto) —
      handles flat/incline/decline automatically; support props (row bench,
      Bulgarian bench, curl seat) as boxes from the 2D rect coords
- [x] Swept all 28: every one mounts, animates, disposes; console clean.
      Visually rough (acceptable, note for s4/s5 polish pass): one-arm row
      far-hand-to-bench alignment approximate; Russian twist dumbbells overlap
      at center; lateral-raise dumbbells rotate with the arm (real ones stay
      level) — all cosmetic, none blocking
- [x] Commit: "v5 s3: all exercises in 3D with equipment"

### Session 4 — Sun PM: parity polish + integration ✅ DONE
- [x] Working-muscle glow: MUS3 map (ex.mus[0] → torso/hips/ua/fa/th/sh rig
      meshes), materials cloned per-target (un-shared), emissive pulse in loop
- [x] Movement trace: near hand (ankle when grip none) sampled ×21 via
      getWorldPosition, dashed THREE.Line + arrow cone, skipped under .56
      world units (= the 2D 28-px rule)
- [x] Phase labels: .t3-label HTML overlay on the stage, PH table + si%2,
      accent ↑ / muted ↓, set immediately at mount and on segment change
- [x] Workout mode honors 3D pref: show3D(stage) parametrized, wo exercise
      phase mounts 3D; verified dispose on rest phase, remount on next set,
      dispose on end
- [x] Perf: visibilitychange pauses/resumes the 3D loop; DPR cap and full
      dispose (geo+materials+renderer) already in place, re-verified
- [x] Commit: "v5 s4: glow, traces, labels, workout mode, perf"

### Session 5 — Sun eve: QA + DEPLOY ✅ DONE
- [x] Full pass green: fresh profile flow, 28 exercises × 2D + 28 × 3D in the
      modal (no leaks), workout mode in both modes with dispose/remount across
      phases, adapted rest timer, console clean. Payload: 24.6 KB app +
      171.5 KB three.js gzipped ≈ 196 KB total — well under 1 MB
- [x] Footer bumped to v5; project `workout-coach.html`, `three.module.min.js`
      and `deploy/` synced from repo; `workout-coach-v4.html` archived
- [x] Deployed: `v5-3d` merged to `main`, live URL verified serving v5
- [x] Memory updated; WhatsApp release note delivered to user
- [x] Merge: "v5: real 3D mannequin demos"

## Status log
- 2026-07-07: Plan created, branch `v5-3d` pushed. Nothing built yet.
- 2026-07-07 (s1): Session 1 complete. 3D scene + static mannequin + orbit +
  modal toggle all verified in preview (load, toggle both ways, dispose on
  close, toggle hidden on non-3D exercises, console clean). Note for s2: the
  mannequin in buildMannequin() is positioned directly (no joint hierarchy) —
  s2 should restructure it into pivot groups (shoulder/elbow/hip/knee) before
  posing. can3D() gate currently allows Overhead Press only — widen in s3.
- 2026-07-07 (s2): Session 2 complete. buildRig() replaces the static mannequin;
  all six gated exercises animate with correct mechanics (squat depth, plank
  horizontal, bench lying) and clean console. Handoff to s3: (1) widen V5_3D
  gate to all exercises incl. front/flye modes — those two modes need a pose
  adapter (front poses only have ua/fa; flye only has `a`), plan is `p3`
  overrides per pose; (2) equipment meshes — rig exposes `hand` groups inside
  each elbow for attaching dumbbells/bars; bench mesh needed under lying
  exercises (derive from torso angle like SVG drawBenchAuto); (3) far-side
  material is dimmed accent (×.55) for side-readability, matches 2D.
- 2026-07-07 (s3): Session 3 complete. All 28 exercises 3D-capable (can3D gate
  fully open), equipment + benches render, front/flye adapters produce real
  abduction (flye & lateral raise verified visually — the marquee shots).
  Handoff to s4: glow → suggest emissive pulse on nearest body capsule mapped
  from ex.mus[0]; traces → sample near-hand world position via
  hand.getWorldPosition across pose t=0..1; labels → absolutely-positioned div
  over the stage reusing PH + si from the 3D loop (expose si on T3); workout
  mode → woRender exercise phase needs the same toggle/canvas plumbing as the
  modal (share applyStageMode-like helper); perf → pause loop on
  visibilitychange, cap DPR already done, dispose verified.
- 2026-07-07 (s4): Session 4 complete. Glow/trace/labels/wo-mode/perf all in;
  28-exercise regression sweep passed (mount+dispose per exercise), workout
  mode lifecycle verified (mount→rest dispose→remount→end dispose), console
  clean. Ready for s5: QA sweep both 2D/3D, payload audit, footer v5, sync
  project files + archive v4, MERGE TO MAIN (the only deploy step), memory +
  release note. s3 cosmetic notes stand (row far hand, twist db overlap,
  raise db orientation) — fix in s5 only if trivial, else ship and note.
- 2026-07-07 (s5): SHIPPED. Full QA matrix green (56 demo checks + workout
  mode both modes + fresh profile + console). ~196 KB gzipped total. Merged
  to main and verified live. The s3 cosmetic items ship as known-minor —
  candidates for a v5.1 polish alongside tester feedback. PLAN COMPLETE.
