# Spidey Testimonials 🕷️

An interactive 3D testimonial clothesline: Spider-Man hauls a web line strung with quote cards,
and reads the centred testimony aloud into a microphone he catches out of the air.

## Features

- **Spider-Man 3D character** (`assets/spiderman.glb`) posed entirely at runtime by `poseHuman()`
  — analytic two-bone IK for the limbs, per-finger grip, breathing, blinks, weight shift and
  recovery steps. No baked animation clips.
- **Real rope & cloth simulation** — helically wound strand geometry with soft-shadow cloth
  deformation on the hanging cards.
- **Mic catch & "mic drop" sequence** — pressing *Play this testimony* flies a studio microphone
  in from off-stage; he reaches out, catches it, reads the quote into it, then opens his fist and
  drops it. See the state machine below.
- **Speech synthesis** with sound-wave rings that bloom from his head while he speaks.
- **Drag, arrow keys and indicator dots** for navigating the line.
- **Fully offline** — every script, model and image is served from this folder. No CDN, no network.

## Mic sequence

`MIC_STATE.phase` runs `IDLE → FLY_IN → CATCH → HELD → DROP → SLIDE → IDLE`:

| Phase | What happens |
|---|---|
| `FLY_IN` | Mic arcs in from off-stage right, shedding a wave-arc trail. The right hand opens and reaches out to meet it. |
| `CATCH` | Mic dead-stops with a small recoil; the fist snaps shut around the handle. |
| `HELD` | Mic is drawn in to his mask and held there while the utterance plays. |
| `DROP` | Fist splays open. The mic falls under gravity (1800 px/s²) while the arm holds an extended open-palm pose. |
| `SLIDE` | Mic hits the floor, throws an impact ripple, bounces and skids off the left edge. |

Stopping playback mid-flight does not drop the mic in mid-air — it arms `dropRequested` so the
catch still reads, and the drop follows from his hand.

## Project structure

```
Spidey Testimony/
├── index.html                  # The entire component: markup, styles, and all WebGL logic
├── server.js                   # Static dev server on :8888 (run with `node server.js`)
├── README.md
├── assets/
│   ├── spiderman.glb           # Character model (~98MB)
│   ├── Mic.glb                 # Shure SM58 studio microphone (~5MB)
│   ├── puller.glb.js           # Base64 fallback character, fetched ONLY if spiderman.glb fails
│   └── golden-gate.jpg         # Stage backdrop
└── vendor/
    ├── three.bundle.js         # Three.js
    └── GLTFLoader.bundle.js    # GLTF loader
```

`index.html` is the single source of truth — there are no partial or backup copies of it.

## Quick start

```bash
node server.js
```

Then open `http://localhost:8888/`. The character model is ~98MB, so the stage renders well
before he does; he drops in once loaded.

## Notes for future work

- `assets/spiderman.glb` is ~98MB and dominates load time — it is 90% of the project by size.
  Running it through [gltfpack](https://github.com/zeux/meshoptimizer) or Draco compression is by
  far the highest-impact optimisation available.
- `puller.glb.js` is deliberately *not* in a `<script>` tag. It is lazy-loaded by
  `loadPullerFallback()` so its 2.2MB never touches the normal load path.
- Speech relies on the browser's `speechSynthesis`, which needs a real user gesture — a synthetic
  `.click()` will start an utterance and immediately cancel it.
