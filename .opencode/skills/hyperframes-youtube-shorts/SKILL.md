---
name: hyperframes-youtube-shorts
description: "Use this skill (together with hyperframes-video-common, read first) when turning a topic, a reference transcript, or a blog link into a vertical YouTube Short (or Reel/TikTok-style clip) built with HyperFrames, roughly 15-60 seconds long. Use ONLY when the user explicitly asks for a short/Short/reel/vertical video/under-60-second video — otherwise default to hyperframes-youtube-long. Covers what's specific to Shorts: the compressed single-beat structure, 9:16 framing and platform-UI safe zones, an immediate sub-2-second hook, fast pattern-interrupt pacing, big bold word-synced captions, a loop-friendly ending, and why the long-form subscribe-ask scene and passing engagement-overlay popups are dropped or replaced. Read hyperframes-video-common for the shared multi-video project structure, voiceover mechanics, build-loop mechanics, and realism/engagement techniques — this file only covers what differs for Shorts."
---

# HyperFrames YouTube Video Generation — Shorts

**Read `hyperframes-video-common` first.** This file only covers what's specific to
Shorts. Everything else (project structure, voiceover generation, the scene-by-scene
build loop, general realism/engagement techniques, cross-session resumption) lives in
`hyperframes-video-common` and applies here unchanged — the difference is pacing,
structure, and framing, not the underlying mechanics.

**Format**: 9:16, 1080x1920, `format: "short"` in `progress.json`. Target length
**15–60 seconds** (up to ~90s only if the topic genuinely needs it — shorter beats
retention better than longer).

## Pipeline at a glance

| Stage | What happens | Output file(s) |
|---|---|---|
| 0. Setup | Per `hyperframes-video-common` | `projects/<slug>/progress.json` |
| 1. Intake & research | Distill to ONE sharp idea, not a full topic survey | `projects/<slug>/research.md` |
| 2. Element sourcing | Per `hyperframes-video-common`, fewer elements needed | `projects/<slug>/assets/manifest.json` |
| 3. Script & beat plan | One continuous script, 3-6 quick-cut beats | `projects/<slug>/script.md`, `scenes.json` |
| 4. Visual design plan | Vertical framing, safe zones, loop ending | `projects/<slug>/design.md` |
| 5. Voiceover | Per `hyperframes-video-common`, single fast-paced take | `projects/<slug>/audio/scene-N.mp3` + `.srt` |
| 6. Build | Per `hyperframes-video-common`'s build loop | `compositions/<slug>/scenes/scene-N.html` |
| 7. Assembly & render | Mount beats, check, render final video | `out/<slug>.mp4` |
| 8. Metadata | Per `hyperframes-video-common` — title, description, hashtags (thumbnail optional) | `projects/<slug>/metadata/` |

---

## Stage 1 — Intake & research

A Short has room for **one idea**, not a topic survey. From the given topic/transcript/
blog, pick the single sharpest, most surprising, or most useful angle rather than
trying to compress the whole source into 30 seconds. If the source is a long-form
transcript/blog, this is often the one moment worth extracting — not a summary of
everything in it.

Write `projects/<slug>/research.md`: the one core idea, the one supporting fact/example
that proves it, and the visual it needs.

---

## Stage 3 — Script & beat plan

Target: **15–60 seconds** of continuous narration (roughly 35–150 words at ~140–150
wpm). This is not a multi-section video — it's one idea delivered fast, broken into
**3–6 quick-cut beats** (not "scenes" in the long-form sense; each beat is 3–8 seconds).
Standard structure:

1. **Hook — first 1–2 seconds, not 5–10.** The idea/question/claim, stated immediately.
   A Short loses most of its potential audience in the first second if it doesn't. No
   slow build, no channel intro.
2. **2–4 quick delivery beats** — the proof, the explanation, the twist, building
   directly on the hook with no filler.
3. **Payoff / punchline / key takeaway** — land the idea clearly.
4. **Loop-friendly last beat** (see Stage 4) instead of a long-form-style outro/CTA.

No recap beat, no dedicated subscribe-ask scene, no multi-section "setup → main content
→ conclusion" arc — that structure is what makes long-form work over 7+ minutes and is
exactly what kills pacing in something meant to be watched in 30 seconds.

`projects/<slug>/scenes.json` uses the same shape as long-form's, with beats instead of
scenes and much shorter `est_duration_sec` per beat:

```json
[
  { "scene": 1, "title": "Hook", "narration": "...", "est_duration_sec": 2, "visual_elements": ["..."], "purpose": "hook" },
  { "scene": 2, "title": "Beat 1", "narration": "...", "est_duration_sec": 5, "visual_elements": ["..."], "purpose": "delivery" }
]
```

`engagement_overlay` is generally left unused for Shorts (see below) — omit the field
or leave it `null` throughout.

---

## Stage 4 — Visual design planning

Write `projects/<slug>/design.md` covering the same palette/per-scene visual plan as
long-form (see `hyperframes-video-common`'s realism/engagement section for the
techniques), plus what's Short-specific:

**Vertical framing & platform safe zones.** 1080x1920. Keep all essential text/visuals
inside the safe zone — roughly the center 80% horizontally and clear of the bottom
~15–20% and the right-edge ~10–15% of the frame, since Shorts/Reels/TikTok overlay
their own UI (caption bar, like/comment/share rail, username, progress bar) in those
regions. Never place a key visual or the caption text where platform UI will sit on top
of it.

**Fast pattern-interrupt pacing.** A cut, camera move, or new visual element roughly
**every 1–3 seconds** — much tighter than long-form's 10–20s interval, because Shorts
are judged on a swipe-away decision continuously, not once. Every beat should feel like
it's moving.

**Captions are the dominant visual, not a supplement.** Big, bold, high-contrast
word-by-word (karaoke-style) captions, word-synced to the edge-tts timestamps
(`hyperframes-video-common`'s caption-sync technique) — for Shorts this typically
covers 1–2 words on screen at a time, large and centered, rather than long-form's
smaller lower-third captions. Most Shorts viewers watch muted at first; the video
needs to work from captions alone.

**No dedicated subscribe-ask scene, no passing `engagement_overlay` popups.** Both come
from `hyperframes-youtube-long` and don't fit here: a Short has no room to spend 5–10
seconds on a subscribe ask without gutting retention, and a corner notification popup
competes with the platform's own UI in exactly the region that's already reserved for
it. If a subscribe/follow prompt is wanted at all, keep it to a small, static text
element folded into the final loop-friendly beat (below), not an animated popup.

**Loop-friendly ending.** Design the last beat so it flows naturally back into the
first — matching color/composition, or a line that finishes the thought the hook opened
— so a viewer who reaches the end and the video loops doesn't feel a hard seam. Looped
rewatches are a meaningful engagement signal on Shorts; a jarring end-to-start cut
works against that.

---

## Stage 7 — Assembly & render

Once every beat composition for this video exists and is approved:
1. Inside `compositions/<slug>/index.html`, lay out the beats back-to-back — usually
   hard cuts or very fast cross-fades rather than long-form's slower transitions, to
   keep the pace up.
2. Point the root `index.html` at this video (one `data-composition-src` line).
3. Sanity-check total duration is inside the 15–60s target (up to ~90s only if
   necessary) — if it's run long, that's a signal to cut a beat, not to slow the pacing.
4. Run `npm run check`, then render:
   ```bash
   npm run render   # or: npx hyperframes render -o out/<slug>.mp4
   ```
5. Update `projects/<slug>/progress.json.stage` to `"rendered"`, then move to
   **Stage 8 — Thumbnail & metadata generation** in `hyperframes-video-common` — for
   Shorts this means title/description/hashtags (`#Shorts` included) and, unless asked,
   skipping a dedicated thumbnail. Only mark the video `"done"` once that's generated.
