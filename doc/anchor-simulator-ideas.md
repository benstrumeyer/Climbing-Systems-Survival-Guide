# Interactive Anchor & Systems Simulator (three.js)

A browser-based rigging sandbox that teaches climbing systems **without any real gear**. Drag bolts, slings, knots, and carabiners in 3D, pull on the master point, and watch the sim tell you whether your rig is safe and *why*.

The goal is to make the failure modes *visible*. Reading "a 120° V-angle doubles load on each piece" is forgettable. Dragging a master point outward and watching a red force vector triple in length while a bolt icon pops off the wall is not.

---

## Why this exists

- Climbing systems are high-consequence and low-feedback. You find out you rigged it wrong when something breaks.
- Textbooks (Long, *Freedom of the Hills*) are good but static. Diagrams don't let you *try the bad version*.
- This simulator lets the user **deliberately build unsafe systems** in a zero-consequence environment, then shows the exact failure.
- Aligns with the overall project vision: teach the systems that keep me alive on El Cap, hands-on.

---

## Core pedagogical principle

> **Every lesson ends with the user breaking something on purpose.**

Show the concept → let the user rig it → let them rig it *wrong* → visualize the failure → quiz them on why.

---

## MVP: The 2-Bolt Top Anchor

The atomic unit. Every trad anchor, rappel station, and multi-pitch belay is a variation on this.

### Scene
- Vertical rock face with two bolts drilled at the top.
- Sling, cordelette, or rope connecting the bolts to a master point.
- A draggable "load" (the climber) pulling down from the master point.
- Third-person orbit camera; optional first-person "belayer POV".

### Interactions
- **Drag the master point** up/down/left/right — V-angle changes in real time.
- **Drag the bolts** horizontally — change bolt spacing.
- **Swap rigging style** via a dropdown:
  - Sliding X (no limiter knots)
  - Sliding X *with* limiter knots
  - Cordelette with overhand
  - Quad
  - Equalette
  - Two-piece clove-hitch-in-rope (belay anchor)
- **Click a bolt to "fail" it** — watch what happens.
- **Pull harder** — scale the load from bodyweight to fall-factor-2 forces.

### Live readouts (HUD)
- **V-angle** at the master point, color-coded: green <60°, yellow 60–90°, red >90°, flashing red >120°.
- **Load per piece** as a percentage of applied load, using `F_piece = (F/2) / cos(θ/2)`.
- **Extension distance** if either piece fails (how far the master point drops).
- **Shock load estimate** when a piece fails (peak force from the drop).
- **SERENE/ERNEST checklist** auto-ticking as the rig changes:
  - Solid
  - Equalized
  - Redundant (no single point of failure)
  - Efficient
  - No Extension
  - Timely (not graded, just a reminder)

### Verdicts
- **SAFE** — green master point, thumbs up, unlock next lesson.
- **MARGINAL** — yellow, explains what's borderline.
- **UNSAFE** — red, animates the failure (bolt rips, master point drops, climber falls).

### The money shot
Click "Simulate piece failure" on a sliding-X *without* limiter knots → watch the master point shoot sideways and the climber drop a meter with a shock-load spike. Then toggle limiter knots on and repeat. The difference is felt, not memorized.

---

## Physics model (simple but honest)

Don't need a full rigid-body sim. A static force model plus scripted failure animations gets ~90% of the teaching value.

- **Static equalization**: `F_piece = (F_total / 2) / cos(angle/2)` for symmetric two-piece.
- **Asymmetric equalization**: weight by leg length for cordelette-style fixed master points.
- **Extension**: geometric — when a piece "fails", recompute master point as the intersection of the surviving leg with the load line. Distance dropped = delta.
- **Shock load**: rough approximation using `F = m * (1 + sqrt(1 + 2*k*h/(m*g)))` with a conservative rope/sling `k`. Numbers don't need to be perfect; the *shape* of the curve is what teaches.
- **Piece strength**: each bolt/cam has a rated load. If `F_piece > rating`, it fails. Ratings come from real gear specs (bolts ~25kN, good cam ~12kN, micro nut ~2kN).

---

## Progression of lessons

Each lesson is a scene preset. Completing one unlocks the next.

1. **Load multiplication** — two bolts, drag the master point, watch forces climb.
2. **Redundancy** — try a single-bolt anchor; sim refuses with explanation.
3. **Extension** — sliding X with/without limiter knots; fail a piece.
4. **Cordelette vs. equalette** — equalization quality under shifting load direction.
5. **Natural anchors** — slinging a tree, threading a horn, tying off a chockstone.
6. **Trad anchor** — you pick the placements yourself (next section).
7. **Multi-pitch belay station** — anchor + tie-in + belay device orientation + redirecting the second's belay.
8. **Rappel station** — rigging the rope through rings, extended device, prusik backup, pre-rig for two climbers.
9. **Self-rescue rigging** — escape the belay, transfer load to anchor, tie off the belay device, build a releasable mule-overhand.
10. **Hauling** — 2:1, 3:1 Z-drag, drop loop, progress capture.

---

## Trad-placement mini-game

Once the user groks fixed bolt anchors, graduate to **building the anchor from scratch**:

- A crack with slots, pods, flares, and constrictions rendered in 3D.
- A rack of cams (colored BD sizes), nuts, hexes, tricams.
- Drag gear into slots. Sim evaluates:
  - **Size match** — is the cam in its operating range? Tipped out? Over-cammed?
  - **Rock quality** — is the crack behind a hollow flake? (visual cue: tap to hear hollow)
  - **Direction of pull** — will it walk, invert, or lift out?
  - **Extension** — did you sling it long enough to avoid walking?
- Then equalize the placements into an anchor using the rigging tools from the MVP.
- Load test by rigging a belay and "catching" a fall.

---

## Knot-tying sub-simulator

Separate mini-tool, linked from the main sim. Rope rendered as a spline with a few dozen control points and collision/self-collision resolution.

- **Guided mode**: step-by-step animation of figure-8 follow-through, clove hitch, munter, bowline on a bight, alpine butterfly, prusik, autoblock, Munter-mule-overhand.
- **Free mode**: user drags the rope end through bights and around strands; sim recognizes when a valid knot has formed.
- **Dressing check**: knot is "tied" but sim flags if it's not dressed or not backed up.
- **Load test**: pull on it — does it hold, slip, or capsize (bowline under ring-loading)?

Hard to build, huge payoff. This alone could be the killer feature.

---

## Self-rescue rigging scenarios

Scripted scenes with a failure state and a correct rigging sequence:

- **Escape the belay** — partner unconscious on lead, you need to go to them.
- **Ascending a fixed rope** — rig two prusiks or micro-traxion + tibloc.
- **Lowering a second past an overhang** — reverso in guide mode, releasing under load.
- **Tandem rappel with injured partner** — extended device, backup, weight transfer.
- **Pickoff from a stuck rappel** — load transfer between two systems.

Each scenario scores on: correct sequence, no deadly omissions (backup? autoblock?), efficiency.

---

## Stretch ideas

- **VR mode** via WebXR — rig an anchor in room-scale, actually reaching for gear on a virtual rack.
- **Multiplayer belay** — one user leads, one belays, both see the same rope in real time.
- **Fall physics** — drop-test mode where you set fall factor and watch peak force on the anchor.
- **Rockfall / ice screws / snow anchors** — alpine systems, deadman placements, T-slots.
- **Big wall rigging** — haul system, portaledge suspension, fifi hook + daisy progression, docking the pig.
- **"Spot the error" mode** — sim generates a pre-built rig with one subtle mistake; user has to find it before loading it.
- **Integration with the coach** — the LLM coach picks the next scenario based on what the user got wrong last time.

---

## Tech sketch (non-binding)

- **three.js** for 3D, OrbitControls, DragControls.
- **Cannon-es** or **Rapier** only if/when rope dynamics need real physics. Otherwise keep it geometric.
- Single-page app, no backend. Scene presets as JSON. Lesson state in localStorage.
- Eventually embed inside the Climbing Systems coach app as the "lab" tab.

---

## Open questions

- How much rope physics is enough? Static geometry covers 80% of lessons; real ropes are needed for knot-tying and dynamic falls.
- How to represent friction (knots, belay device, redirects) without a full sim?
- Should the sim use real kN numbers (educational but intimidating) or bodyweight multiples (intuitive)?
- Gamify it (scores, unlocks) or keep it sober? Probably sober — the stakes should feel real.
- Should failures show gore (falling climber) or stay abstract (red X)? Abstract is probably better; this is a teaching tool, not a shock piece.
