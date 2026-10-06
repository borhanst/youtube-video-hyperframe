---
name: hyperframes-youtube-long
description: "Use this skill (together with hyperframes-video-common, read first) when turning a topic, a reference YouTube transcript, or a blog link into a long-form (7-8+ minute) YouTube video built with HyperFrames. This is the DEFAULT video-generation skill — use it whenever the user asks to make a video and does not explicitly ask for a Short/reel/vertical/under-60-second video (that goes to hyperframes-youtube-shorts instead). Covers what's specific to long-form: research depth, the 7-8+ minute / 12-20 scene target, standard long-form script beats (hook, dedicated subscribe-ask scene, setup, main sections, recap, CTA), the passing subscribe/notification engagement-overlay popups, 16:9 framing, and long-form pattern-interrupt/pacing intervals. Read hyperframes-video-common for the shared multi-video project structure, voiceover mechanics, build-loop mechanics, and realism/engagement techniques — this file only covers what differs for long-form."
---

# HyperFrames YouTube Video Generation — Long-form

**Read `hyperframes-video-common` first.** This file only covers what's specific to
long-form videos — duration/scene targets, script beats, the subscribe-ask scene, the
passing engagement-overlay popups, framing, and pacing intervals. Everything else
(project structure, voiceover generation, the scene-by-scene build loop, general
realism/engagement techniques, cross-session resumption) lives in
`hyperframes-video-common` and applies here unchanged.

**Format**: 16:9, 1920x1080, `format: "long"` in `progress.json`.

## Pipeline at a glance

| Stage | What happens | Output file(s) |
|---|---|---|
| 0. Setup | Per `hyperframes-video-common` | `projects/<slug>/progress.json` |
| 1. Intake & research | Read/search the topic, transcript, or blog | `projects/<slug>/research.md` |
| 2. Element sourcing | Per `hyperframes-video-common` | `projects/<slug>/assets/manifest.json` |
| 3. Script & scene plan | Full narration, 7-8+ min, 12-20 scenes | `projects/<slug>/script.md`, `scenes.json` |
| 4. Visual design plan | Palette, per-scene visuals, subscribe-ask + overlay placement | `projects/<slug>/design.md` |
| 5. Voiceover | Per `hyperframes-video-common` | `projects/<slug>/audio/scene-N.mp3` + `.srt` |
| 6. Build | Per `hyperframes-video-common`'s build loop | `compositions/<slug>/scenes/scene-N.html` |
| 7. Assembly & render | Mount scene strips, check, render final video | `out/<slug>.mp4` |
| 8. Thumbnail & metadata | Per `hyperframes-video-common` — titles, description, tags, thumbnail | `projects/<slug>/metadata/` |

---

## Stage 1 — Intake & research

Input is one of:
- **A topic** the user names directly.
- **A reference YouTube transcript** the user pastes or links — if a link, fetch the
  transcript (or ask the user to paste it if it can't be fetched).
- **A blog link** — fetch and read the full article, don't summarize from the URL alone.

Do real research, don't rely on memory alone:
- Pull the core narrative, key facts, figures, and any claims that need verifying.
- Note the natural sub-topics/beats — these become candidate scene boundaries.
- Note anything visually rich in the source (named products, places, diagrams, data,
  people, steps in a process) — these drive what visual elements Stage 2 needs.
- If it's a reference video, also note its pacing/structure choices worth reusing or
  improving on (hook style, section order, CTA placement) — but do not reuse its wording.

Write findings to `projects/<slug>/research.md`: a topic summary, the beat list, and a
running list of "visual needs".

---

## Stage 3 — Script & scene planning

Target: **minimum 7–8 minutes of narration** (roughly 1,000–1,200+ words at a natural
narration pace of ~140–150 wpm — check the actual draft's word count against runtime,
don't assume). Break the script into multiple scenes; a scene is a self-contained beat
with one dominant visual idea, typically 20–45 seconds of narration each (so a 7–8 min
video is usually 12–20 scenes).

Write two artifacts:

**`projects/<slug>/script.md`** — full narration in order, with scene headers, meant for
the user to read/approve like a normal script.

**`projects/<slug>/scenes.json`** — the same content structured for the build stages:

```json
[
  {
    "scene": 1,
    "title": "Hook",
    "narration": "...",
    "est_duration_sec": 28,
    "visual_elements": ["icon-lightbulb", "bg-gradient-1"],
    "purpose": "hook the viewer / state the question",
    "engagement_overlay": null
  }
]
```

`engagement_overlay` is `null` on most scenes and only set on the 2–3 scenes chosen in
Stage 4 to show the subscribe/notification popup, e.g.
`{ "type": "subscribe", "appear_at_sec": 2.0, "hold_sec": 3.5 }`.

**Standard long-form beats**, in order: hook (first 5–10s) → dedicated subscribe-ask
scene → setup/context → 3–5 main content sections, each its own scene cluster → recap →
CTA/outro. Confirm the script with the user before moving to design — scripts are
expensive to re-record.

**Early subscribe-ask scene.** Right after the hook lands (before setup/context
begins), insert one short dedicated scene, roughly 5–10 seconds, where the narration
directly asks the viewer to subscribe. Frame it around the honest observation that most
people watching will never come back:

- Acknowledge it plainly: most people watching this haven't subscribed, and most of
  them are about to keep scrolling past channels they actually like.
- Make the ask concrete and quick: one sentence telling them to hit subscribe/the bell
  now, before they get pulled into the rest of the video and forget.
- Keep it under ~2 short sentences of narration — don't slow the video's momentum
  right after the hook. Write the actual line to fit this video's voice/topic rather
  than reusing the same wording video to video.
- Mark this scene with `"purpose": "subscribe_ask"` so Stage 4/6 know to give it the
  dedicated subscribe visual described below.

---

## Stage 4 — Visual design planning

Write `projects/<slug>/design.md` covering:

**Palette & gradient** — chosen to fit the topic, not a generic default (one dominant
color, 1–2 supporting tones, one accent; pick a palette that would look wrong swapped
onto an unrelated topic). Record it in `progress.json.palette` too.

**Per-scene visual plan** — for every scene in `scenes.json`, decide background,
foreground elements (rule-of-thirds, not always centered), camera/motion, on-screen
text and its entrance timing, and the transition in/out. This file is what Stage 6
translates directly into composition HTML, so be concrete (exact hex codes, named
animation types, named layout positions) rather than vague mood words.

### Dedicated subscribe-ask scene vs. passing engagement overlay

Two different subscribe-related moments — keep them visually distinct so the second
doesn't feel like a repeat of the first:

- **The dedicated subscribe-ask scene** (right after the hook) is the one moment the
  narration itself talks about subscribing, so its visual should be the main focus of
  that scene: subscribe button/bell centered or large, held for the scene's full
  ~5–10 seconds, matched to what's being said.
- **The passing `engagement_overlay` popups** (below) are silent, small, corner
  notifications appearing later, *without* narration calling attention to them — a
  background reinforcement, not a repeat of the direct ask.

### Engagement overlays (subscribe / bell-notification popup)

- **How many, and where**: pick 2–3 scenes total (not every scene), clearly after and
  separate from the subscribe-ask scene — e.g. one around the 2–3 minute mark, one more
  around the midpoint or right before a strong section. Never in the hook, the
  subscribe-ask scene, or the outro/CTA scene.
- **Timing per appearance**: pop in, hold, pop out — roughly a 3–4 second total window,
  never longer; it should read as a passing notification. Trigger it when no other
  on-screen text is competing for attention.
- **Placement**: a corner (bottom-right or top-right), small relative to the frame
  (roughly 12–18% of frame width) — never centered, never covering the scene's focus.
- **Motion**: spring-like pop-in (slight overshoot), an optional subtle bell-wiggle or
  button-pulse while held, then a quick fade+slide-out.
- **Style**: pull colors from the Stage 4 palette so it looks like part of the video's
  own design system.
- **Vary it across videos**: don't hardcode the same appear-time/scene pattern for
  every video — pick placements per-video based on where that video's own hook and
  section breaks land.

### Long-form pacing

- **Pattern interrupt roughly every 10–20 seconds** (see `hyperframes-video-common`'s
  realism/engagement section for the technique) — new visual element, camera move,
  sound cue, or on-screen question. If a scene runs longer than ~20s with nothing
  changing on screen, split it or add a visual beat.
- **Open loops** at section boundaries — tease a payoff a beat before delivering it, to
  keep the retention curve up over the full 7–8 minutes.
- **Bias toward more, shorter scenes with more visual events** over fewer long static
  ones.

---

## Stage 7 — Assembly & render

Once every scene composition for this video exists and is approved:
1. Inside `compositions/<slug>/index.html`, lay out the scene sub-compositions
   back-to-back with `data-start` offsets and durations
   (`data-composition-src="scenes/scene-N.html"`), applying the Stage 4 transition
   between each pair.
2. Point the root `index.html` at this video (one `data-composition-src` line).
3. Sanity-check total duration against the 7–8 minute minimum; if short, that's a
   signal to revisit Stage 3 (more scenes/content), not to pad existing scenes.
4. Run `npm run check`, then render:
   ```bash
   npm run render   # or: npx hyperframes render -o out/<slug>.mp4
   ```
5. Update `projects/<slug>/progress.json.stage` to `"rendered"`, then move straight to
   **Stage 8 — Thumbnail & metadata generation**, which is fully specified in
   `hyperframes-video-common` (chapter timestamps for the description come from this
   video's `scenes.json` boundaries). Only mark the video `"done"` once metadata is
   also generated — a rendered file with no title/description/tags/thumbnail isn't
   upload-ready.
