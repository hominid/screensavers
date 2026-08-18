# RAVEN ANIMATION — CLAUDE CODE BRIEFING

## CONTEXT

This is a browser-based "screensaver" animation (`7.html`) — a single self-contained HTML/JS/Canvas file. The animation depicts a stylized 3D bird flying in a Lissajous path against a dark background, rendered as a constellation/particle mesh with glowing faces.

The goal is to transform it from a generic ethereal bird into something that evokes **ravens playing on thermals at the Desert View Watchtower, Grand Canyon** — social, unpredictable, iridescent-black, canyon-sky atmosphere.

There is no build system. No dependencies. Open the file directly in a browser to evaluate.

---

## ARCHITECTURE SUMMARY

The core engine is sound — do not refactor it. Key systems worth understanding:

- **`nodes[]`** — 29 named 3D points defining the skeleton. Wing nodes carry `flap` (amplitude) and `phase` (offset) properties.
- **`faces[]`** — polygon panels defined as arrays of node indices. Rendered back-to-front (painter's algorithm on projZ).
- **`NodeParticle` class** — one per node. `update()` applies flap sine, then 3D rotation (bank → pitch → yaw), then perspective projection. Stores `projX`, `projY`, `projZ`, `scale`.
- **`getFlightPath(t)`** — returns `{x, y, z}` world position. Currently a Lissajous figure. Yaw/pitch/bank are derived from the path derivative.
- **`drawEtherealBird()`** — draws faces with additive blending, then draws node particles with shadow blur + twinkling.
- **`animate()`** — main loop. Increments `globalTime += 0.015`, computes orientation, updates all particles, draws.

---

## WORK LIST — DO IN ORDER, STOP FOR VISUAL SIGN-OFF BETWEEN BATCHES

### BATCH 1 — Behavior & feel (no geometry changes)

**Step 1 — Glide/flap state machine**

Add a state to each bird: `gliding` (boolean) and `glideTimer`. When gliding, set `flapSpeed` to near-zero and reduce flap amplitude to ~10% (subtle wrist adjustments only). Alternate between flapping (~2–4s) and gliding (~3–8s) with some randomness per bird. Ravens on thermals glide far more than they flap.

**Step 2 — Flight path: thermal circling + noise**

Replace the Lissajous in `getFlightPath()` with a path that:
- Slowly circles/drifts (loose ellipse or Perlin-noise-driven offset)
- Includes occasional "thermal climb" (slow upward spiral) and "drop" (quick dip, then correction)
- Varies speed — slow at the top of a thermal, faster in a dive
- Keeps the bird roughly centered on screen but with genuine unpredictability

Simple approach: base ellipse + layered sine offsets at incommensurate frequencies + a slow drift component.

**Step 3 — Color: iridescent black**

Ravens are not blue-white. They are black with blue-purple-green iridescent sheen. Changes needed:
- Node particle color: `'40, 20, 60'` (deep purple-black base) with occasional shimmer toward `'80, 120, 200'`
- Face fill: `rgba(10, 5, 20, ...)` — nearly black, very subtle
- Face stroke: `rgba(100, 80, 180, ...)` — iridescent edge highlight
- Keep additive blending — it will naturally produce the sheen effect on overlapping faces
- Background: shift from `#001524` toward `#000008` or a very dark warm-grey (`#0a0806`) for desert night sky

---

### BATCH 2 — Environment & multiplicity

**Step 4 — Background: canyon sky + rim silhouette**

Draw a background scene on a second canvas layer (or just draw it first each frame before clearing the bird layer). Elements:
- Sky gradient: deep blue-black at top → warm dark ochre near horizon (desert twilight)
- Simple canyon rim silhouette: a rough polygon at the bottom 20–25% of the screen, dark reddish-brown (`#1a0a04`), jagged but not complex
- Optional: 3–5 faint static stars in the upper sky region
- The watchtower doesn't need to be drawn — just the sense of being at altitude above a canyon

**Step 5 — Multiple birds (2–3 ravens)**

Refactor to support multiple birds. Each bird needs:
- Its own `NodeParticle[]` array
- Its own `globalTime` offset (stagger by 1.5–3s so they're not synchronized)
- Its own flight path offset (shift the center point so they don't stack)
- Its own glide/flap state

3 birds is the sweet spot. They should feel like they're loosely aware of each other — similar altitude, nearby paths — without explicit flocking math.

---

### BATCH 3 — Geometry (do last, after behavior feels right)

**Step 6 — Raven-specific geometry**

Current issues:
- Tail is a wide symmetrical fan (turkey/peacock shape). Raven tail is a **wedge/diamond** — narrow, pointed at center, slightly forked. Reduce tail spread (nodes 13/14 z-values) and bring tail center (node 12) further back.
- Wing primaries should be longer relative to body, with spread "fingers" at the tip. Add 1–2 additional tip nodes to suggest primary feather separation.
- Head/beak: raven beak is heavy and curved. The current beak (node 0) is just a forward point — acceptable for now.

---

## IMPLEMENTATION NOTES

- `globalTime += 0.015` runs at ~60fps = roughly 1 time unit per second. So `flapSpeed = 5` = 5 cycles/second (fast). Glide should reduce this to ~0.3–0.5.
- The camera is fixed at `camZ = -1000`, `focalLength = 1000`. The bird's Z range in `getFlightPath` is currently `cos(t*0.45)*300 - 200` — center around -200, range ±300. Widening this (e.g. ±500) gives more dramatic near/far size variation.
- `bank = yawDelta * -15` — this is already physically reasonable. Don't increase the multiplier.
- Face winding order isn't enforced — the painter's algorithm handles depth correctly without backface culling.
- Shadow blur is the main perf cost. If 3 birds feel sluggish, reduce `shadowBlur` from `6 * p.scale * twinkle` to a fixed `3` or skip it for the 2 background birds.

---

## EVALUATION CRITERIA (what to look for at each sign-off)

After Batch 1: Does it feel like a soaring bird rather than a flying robot? Is the glide convincing?  
After Batch 2: Does it feel like a place? Can you feel the altitude and the canyon?  
After Batch 3: Is the silhouette unmistakably a raven?

---

## FILES

- `7.html` — the only file. All code is inline.
- Serve locally with `python3 -m http.server` or open directly as `file://`.
