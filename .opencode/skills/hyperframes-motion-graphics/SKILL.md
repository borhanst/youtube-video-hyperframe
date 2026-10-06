---
name: hyperframes-motion-graphics
description: "Prompt patterns for generating short design-led motion graphics (kinetic type, logo reveals, shape morphs, UI-state morphs, showreel-style pieces, 3D/text effects) — distilled from prompt-motion.com examples — adapted to HyperFrames compositions. Load this alongside /hyperframes-animation and /hyperframes-core whenever a scene or whole video is a motion graphic (motion-is-the-message, no narration or a short one), or when writing the prompt layer that drives a one-shot motion-graphic composition."
---

# Motion Graphics Prompt Patterns (from prompt-motion.com)

People ship one-shot motion graphics with Claude Opus 5.5 using two styles of prompt:
a **one-liner brief** and a **structured prompt template**. Both work; the structured
template is markedly more reliable. Adapt either to HyperFrames.

## Pattern A — one-liner brief

Short, opinionated, names the vibe and the deliverable. Good for quick loops and
showreel-style pieces.

> Make a dynamic 15-second motion graphics video that shows what an incredible motion
> designer you are, like it's your showreel for a resume. Go all out.

> Create an impressive motion design video of a slow-reveal transition that assembles
> and eventually reveals the [Brand] logo.

## Pattern B — structured template (preferred)

```md
<inputs> Ask me for: <state list / colors / music> </inputs>
<direction> Dribbble-level <style>. One <element>, never cut: every state is the same
  element morphing ... Springs everywhere, a tiny overshoot at most. Banned: bouncy
  easing, particle bursts, glows, gradients on UI chrome, mismatched icon strokes, dead
  time, anything that looks like a template. </direction>
<structure> <BPM/seconds>, something happens on every beat. Hook → delivery →
  payoff → loop close. </structure>
<build> 1. One composition, every style computed from time — no CSS transitions, no
  timers, no state carried between frames. 2. Springs are closed-form step responses.
  3. <drag/byte-accurate behaviors>. 4. Analyze the song for the beat grid; start on a
  downbeat. 5. Render one frame per beat before full render; fix anything off-grid. </build>
<gotchas> Never will-change on anything the camera scales. Text swapping inside a
  morphing container needs its own enter/exit timing. Make the last frame identical to
  the first — cursor position and speed included — or the loop stutters. </gotchas>
<start> Ask me for the inputs, then show the plan on the beat grid before writing code. </start>
```

## HyperFrames adaptation rules

- "Every style computed from time" maps to: one paused
  `window.__timelines["id"] = gsap.timeline({ paused: true })`, all motion as
  `tl.set/to(..., t)` calls — no setInterval, no CSS `transition`, no state carried
  between frames. Same determinism rule as everywhere in HyperFrames.
- Springs → GSAP eases with overshoot (`back.out(1.4)`, `elastic.out(1, 0.6)`) sized
  like a spring; never a CSS spring curve.
- "Render one frame per beat" → `npx hyperframes preview --background` + scrub through
  the beat grid; fix before `npm run render`.
- Loop pieces → first and last frame must match including cursor/camera position.
- Beat-synced pieces → measure peaks of the track first, place every visual hit on a
  measured beat, nothing loose.
- Keep UI-morph pieces clean: one canvas background, black/white or one accent, one
  clean sans (e.g. Inter/Geist-style), banned-list aesthetics above.

## Realistic icons & logos from the web

- Never draw generic placeholder icons/logos — for a professional piece, use the
  real asset. Search the web for the official SVG (brand sites, Simple Icons,
  Brandfetch, seeklogo), download into `assets/<slug>/`, and reference the local
  path in the composition. Record the source in the video's
  `projects/<slug>/assets/manifest.json` like any other element.
- Prefer SVG (scales, recolors); rasterize only when no SVG exists. Keep one
  consistent icon stroke/weight set across the piece.

## The "showreel for your résumé" piece

When asked for a professional motion-graphics showpiece, treat the whole video as a
resume showreel — motion is the content, and every beat should demonstrate craft:

> Make a dynamic 15-second motion graphics video that shows what an incredible
> motion designer you are, like it's your showreel for a resume. Go all out.

Structure it like a showreel: 3–5 short self-contained segments (kinetic title,
logo reveal, shape/UI morph, stat count-up, UI animation), each beat-synced, hard
cuts on the beat, one consistent palette + grain/vignette throughout, and a loop
back to the opening frame. One-shot brief or structured template — either passes;
the structured template (Pattern B) is more reliable for segments that must hit a
beat grid.

## Ready-to-use scene prompts (drop into `design.md`)

1. **Kinetic showreel** — 15s, 3–4 typographic beats, hard cuts on the beat, one
   accent color, camera zoom on every 2nd beat, loop back to frame 1.
2. **Logo reveal** — slow assemble from scattered parts → lockup → tagline → hold.
3. **Shape/UI morph** — one shape morphing size/radius/content across 8–12 UI states,
   cursor-driven, 120 BPM grid, last state = first state (loop).
4. **Stat count-up** — big number springs to its target, label enters on settle,
   3-beat structure, nothing else moves.

Always: springs for entrances, linear only for progress bars, subtle grain + vignette
uniform across the piece, and run `npm run check` before render.
