---
name: hyperframes-youtube-video
description: "Use this skill when the user wants to turn a topic, a reference YouTube transcript, or a blog link into a long-form (7-8+ minute) YouTube video built with HyperFrames, inside a single HyperFrames repo that hosts many such videos over time. Covers: analyzing the source material, sourcing visual elements (icons/images/video), planning the script and scene breakdown, planning per-scene visuals (color/gradient, camera movement, element placement), generating male-voice narration with edge-tts, and building the video one scene at a time inside a self-contained per-video folder that plugs into the shared root via a single data-composition-src reference. Trigger whenever the user says something like 'make a video', 'create a HyperFrames video', 'generate a YouTube video', or gives a transcript/blog/topic to turn into a video. Always work scene-by-scene — never generate all scenes before the first one is reviewed and approved — never let one video's files depend on another's, so any video's folder can be deleted without touching the rest — always load the HyperFrames domain skills and check `npx hyperframes docs` before writing composition code rather than relying on memory — and always reuse an existing component within the current video's own folder instead of creating a near-duplicate."
---

# HyperFrames YouTube Video Generation

A repeatable pipeline for turning a topic, reference transcript, or blog post into a
long-form, narrated YouTube video. The pipeline is deliberately staged and file-backed
so that work can pause and resume across different chat sessions — every stage writes
its output to a file in `projects/`, and `projects/<slug>/progress.json` records
exactly where things stand.

For the split variant (common foundation + separate long/shorts skills), see
`hyperframes-video-common`, `hyperframes-youtube-long`, and `hyperframes-youtube-shorts`.
This file is the self-contained equivalent covering the long-form pipeline end to end.

**Golden rule: build ONE scene at a time.** Finish, preview, and get sign-off on a scene
before starting the next one. Never batch-generate all scenes' voiceover or all scenes'
composition HTML in a single pass.

**One HyperFrames project hosts many videos.** Every video gets its own self-contained
folder (scenes, components, audio, assets, progress file all live inside it) and
exposes exactly one `data-composition-src` reference that the shared root plugs in.
Deleting a video's folders — plus its one line in the root `index.html` — must never
break any other video. See "Multi-video project structure" below before Stage 0 on a
repo that already has other videos in it.

**Always check before you build, never guess from memory.** Two checks are mandatory
before writing any composition HTML, in every scene, every time:
1. **Load the HyperFrames domain skills.** This environment has dedicated HyperFrames
   skills (`/hyperframes`, `/hyperframes-core`, `/hyperframes-animation`,
   `/hyperframes-creative`, `/hyperframes-cli`, `/media-use`, etc.) — load the
   relevant one and follow it for composition structure, `data-*` attribute semantics,
   GSAP setup, determinism, media treatments. This skill governs the *pipeline*
   (research → script → voiceover → scene-by-scene build); the domain skills govern
   the *composition code itself* and take precedence on API-level details.
2. **Consult the local HyperFrames docs** (`npx hyperframes docs <topic>` —
   `data-attributes`, `gsap`, `compositions`, `rendering`, `examples`,
   `troubleshooting`) and `https://hyperframes.heygen.com/llms.txt` for anything you're
   not 100% certain of. HyperFrames APIs change across versions; don't rely on
   training-data memory beyond the basics.

**Reuse existing components — never duplicate silently.** Before creating any new
sub-composition (a scene, a lower-third, an icon badge, a transition, anything), search
this video's own `components/` folder — and this video's `scenes/` folder for anything
already built — for something that already does the job, or something close enough to
extend with a `data-variable-values` override. Only write a new sub-composition when
nothing existing covers it. Reuse is *encouraged within one video's own folder*, and
only *disallowed across different videos' folders*.

## Pipeline at a glance

| Stage | What happens | Output file(s) |
|---|---|---|
| 0. Setup | Scaffold project folders + progress tracker | `projects/<slug>/progress.json` |
| 1. Intake & research | Read/search the topic, transcript, or blog | `projects/<slug>/research.md` |
| 2. Element sourcing | Collect icons, images, video clips (user-supplied or web) | `projects/<slug>/assets/manifest.json` |
| 3. Script & scene plan | Full narration script broken into scenes | `projects/<slug>/script.md`, `scenes.json` |
| 4. Visual design plan | Palette/gradient, camera moves, element layout per scene | `projects/<slug>/design.md` |
| 5. Voiceover | edge-tts male voice, one file per scene, with timing | `projects/<slug>/audio/scene-N.mp3` + `.srt` |
| 6. Build | One scene HTML at a time, reviewed before next | `compositions/<slug>/scenes/scene-N.html` |
| 7. Assembly & render | Mount scene strip, check, render final video | `out/<slug>.mp4` |
| 8. Thumbnail & metadata | Titles, description, tags, thumbnail | `projects/<slug>/metadata/` |

Re-open a session by reading `projects/<slug>/progress.json` first — it says which
stage and which scene number is currently in flight, so work resumes there instead of
restarting.

---

## Multi-video project structure

One repo holds many videos over time. The rule that makes them safe to delete
independently: **everything about video X lives under one folder named for X, and the
root `index.html` only ever touches that video through one `data-composition-src`
reference.**

```
index.html                       # root — thin entry pointing at the active video
projects/
  <video-slug>/                  # e.g. "2026-10-ai-agents-explained"
    progress.json                 # this video's own progress tracker
    research.md
    script.md
    scenes.json
    design.md
    assets/
      manifest.json
      raw/
    audio/
      scene-1.mp3, scene-1.srt, scene-2.mp3, ...
    metadata/

compositions/
  <video-slug>/
    index.html                    # <-- this video's scene strip (one source of truth)
    scenes/
      scene-1.html, scene-2.html, ...
    components/                   # pieces used only by this video (lower-thirds, badges, etc.)
assets/
  <video-slug>/                   # staged media the composition actually references

out/
  <video-slug>.mp4
```

**The root `index.html` stays a thin entry** that embeds the active video:

```html
<div data-composition-id="main" data-width="1920" data-height="1080"
     data-composition-src="compositions/<video-slug>/index.html"
     data-start="0" data-duration="/* total */" data-track-index="0"></div>
```

To switch which video renders, change that one `data-composition-src` (or render the
video's own `compositions/<video-slug>/index.html` target directly).

**To retire a video:** delete `projects/<video-slug>/`, delete
`compositions/<video-slug>/`, delete `assets/<video-slug>/`, and repoint the one
`data-composition-src` line in `index.html`. That's it — no other video's files are
touched, so nothing else can break.

**Rules that keep this safe:**
- Never reference one video's `scenes/` or `components/` HTML files from another
  video's folder. If two videos want the same visual idea, duplicate the small block
  into both folders rather than share it — a shared file is exactly what makes
  deleting one video risk breaking another.
- Each video's `progress.json`, assets, and audio live inside its own
  `projects/<video-slug>/` folder — never in a shared top-level `assets/` or `audio/`.
- Slug once at Stage 0 (e.g. `YYYY-MM-short-topic-slug`) and reuse that exact slug
  everywhere (`projects/<slug>/`, `compositions/<slug>/`, `assets/<slug>/`,
  `out/<slug>.mp4`). Suffix a Shorts cut of an existing video distinctly
  (e.g. `...-short`).

---

## Stage 0 — Setup

On the **first message of a new video**, pick its slug and create its skeleton — don't
touch any other video's folders:

```
projects/<video-slug>/
  progress.json       # { "stage": "...", "current_scene": N, "total_scenes": N, "notes": "" }
  research.md          # Stage 1 output
  script.md             # Stage 3 output (full narration, human-readable)
  scenes.json           # Stage 3 output (machine-readable scene breakdown)
  design.md             # Stage 4 output
  assets/
    manifest.json        # Stage 2 output — every icon/image/video with source + local path
    raw/                  # downloaded/user-supplied source files
  audio/
    scene-1.mp3, scene-1.srt, scene-2.mp3, ...
  metadata/

compositions/<video-slug>/
  index.html             # this video's scene strip (created in Stage 6)
  scenes/
  components/
```

Initialize `projects/<video-slug>/progress.json`:

```json
{
  "slug": "",
  "format": "long | short",
  "topic": "",
  "source_type": "topic | transcript | blog",
  "source_ref": "",
  "stage": "intake",
  "total_scenes": null,
  "current_scene": 0,
  "voice": "",
  "palette": {},
  "notes": ""
}
```

Update this file after every stage and every scene — it is the single source of truth
for "what's done" on *this* video when the user switches sessions. If the repo already
has other videos, confirm the new slug doesn't collide and leave every existing
`projects/<other-slug>/` and `compositions/<other-slug>/` folder untouched.

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
- Note anything visually rich in the source (named products, places, diagrams,
  data, people, steps in a process) — these drive what visual elements Stage 2 needs.
- If it's a reference video, also note its pacing/structure choices worth reusing or
  improving on (hook style, section order, CTA placement) — but do not reuse its wording.

Write findings to `projects/<slug>/research.md`: a topic summary, the beat list, and a
running list of "visual needs" (things that will want an icon, image, chart, or clip).

---

## Stage 2 — Visual element sourcing

Build `projects/<slug>/assets/manifest.json`, one entry per element:

```json
{
  "id": "icon-lightbulb",
  "type": "icon | image | video",
  "description": "idea / insight moment",
  "source": "user-provided | web-search | generated",
  "url_or_path": "",
  "license_note": "",
  "used_in_scenes": [2, 5]
}
```

- If the user supplies elements, catalog and reuse those first.
- Otherwise search the web for royalty-free/CC-licensed icons, images, or clips —
  record the license/source for each.
- Prefer SVG/CSS-drawn icons or inline SVG over raster images — they scale and
  recolor cleanly to match the palette.
- Download assets to `projects/<slug>/assets/raw/`, then copy what each scene needs
  into `assets/<slug>/` (the path composition HTML actually references).
- Flag any gap where no good element exists yet — Stage 4 may need a generated/abstract
  visual (shape, gradient blob, typographic treatment) instead.

---

## Stage 3 — Script & scene planning

Target: **minimum 7–8 minutes of narration** (roughly 1,000–1,200+ words at a natural
narration pace of ~140–150 wpm — check the actual draft's word count against runtime,
don't assume). Break the script into multiple scenes; a scene is a self-contained beat
with one dominant visual idea, typically 20–45 seconds of narration each (so a 7–8 min
video is usually 12–20 scenes).

Write two artifacts:

**`projects/<slug>/script.md`** — full narration in order, with scene headers, meant for
the user to read/approve like a normal script:

```markdown
## Scene 1 — Hook
[narration text for scene 1]

## Scene 2 — ...
[narration text]
```

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

Standard YouTube long-form beats: hook (first 5–10s), **an early subscribe ask right
after the hook** (its own short scene — see below), setup/context, 3–5 main content
sections each with its own scene cluster, a recap, and a CTA/outro. Confirm the script
with the user before moving to design — scripts are expensive to re-record.

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
- Mark this scene in `scenes.json` with `"purpose": "subscribe_ask"` so Stage 4/6 know
  to give it the dedicated subscribe visual described below (distinct from the brief
  passing `engagement_overlay` popups used later in the video).

---

## Stage 4 — Visual design planning

Write `projects/<slug>/design.md` covering:

**Palette & gradient** — chosen to fit the topic, not a generic default (one dominant
color, 1–2 supporting tones, one accent; pick a palette that would look wrong swapped
onto an unrelated topic). Record it in `progress.json.palette` too:

```json
{ "primary": "#0B1D3A", "secondary": "#1F6FEB", "accent": "#FFB020", "bg_gradient": ["#0B1D3A", "#12294F"] }
```

**Per-scene visual plan** — for every scene in `scenes.json`, decide:
- **Background**: solid, gradient, image, or video, and how it shifts scene to scene
  (keep a consistent system — e.g. gradient direction/angle changes but the palette
  doesn't, so the video feels like one piece).
- **Foreground elements**: which icons/images/clips from the manifest appear, and where
  (rule of thirds placement, not just centered every time).
- **Camera/motion**: e.g. slow Ken Burns zoom-in on a background image, parallax
  between background and foreground layers, spring-like scale/slide-in for icons
  (`ease: "back.out(1.4)"`-style), subtle continuous drift on gradients. Vary the
  motion language across scenes so the video doesn't feel like one template repeated.
- **Text on screen**: any titles, callouts, or captions, their entrance timing.
- **Transition in/out**: cut, cross-fade, slide, or wipe into the next scene.

This file is what Stage 6 translates directly into composition HTML, so be concrete
(exact hex codes, named animation types, named layout positions) rather than vague mood
words.

### Dedicated subscribe-ask scene vs. passing engagement overlay

The video now has two different subscribe-related moments — keep them visually
distinct so the second one doesn't feel like a repeat of the first:

- **The dedicated subscribe-ask scene** (right after the hook, per Stage 3) is the one
  moment the narration itself talks about subscribing, so its visual should be the main
  focus of that scene, not a small corner popup: e.g. the subscribe button/bell
  centered or large or near-full-height, held for the scene's full ~5–10 seconds,
  matched to what's being said.
- **The passing `engagement_overlay` popups** (below) are silent, small, corner
  notifications that appear later in the video *without* narration calling attention to
  them — a background reinforcement, not a repeat of the direct ask.

### Engagement overlays (subscribe / bell-notification popup)

A short subscribe-button or bell-notification popup, the kind viewers are used to
seeing pop in and then disappear again, adds normal YouTube polish — but only if it
behaves like a brief interruption, not a persistent UI element. Plan it explicitly
rather than leaving it as an afterthought in Stage 6:

- **How many, and where**: pick 2–3 scenes total for a 7–8 minute video (not every
  scene), and keep them clearly after and separate from the dedicated subscribe-ask
  scene — e.g. one around the 2–3 minute mark and one more around the midpoint or
  right before a strong section. Never in the hook, in the subscribe-ask scene itself,
  or in the final outro/CTA scene. Record the chosen scenes in `scenes.json` via
  `engagement_overlay` (schema above).
- **Timing per appearance**: pop in, hold, pop out — roughly a 3–4 second total window
  (`hold_sec` in the schema), never longer; it should read as a passing notification,
  not a banner that sits on screen. Trigger it at a moment with no other on-screen
  text competing for attention.
- **Placement**: a corner (bottom-right or top-right is most natural for a
  notification-style popup), sized small relative to the frame (roughly 12–18% of
  frame width) — never centered, never covering a foreground element or icon that's
  currently the focus of the scene.
- **Motion**: pop-in with a spring-like ease (slight overshoot reads as
  "notification"), an optional subtle bell-wiggle or button-pulse while held, then a
  quick fade+slide-out — don't just cut it in and out.
- **Style**: pull colors from the same Stage 4 palette (e.g. accent color for the
  bell/button, primary/secondary for the background chip) so it looks like part of the
  video's own design system, not a generic sticker pasted on top.
- **Vary it across videos**: don't hardcode the exact same appear-time/scene pattern
  for every video — pick placements per-video based on where that video's own hook and
  section breaks land.

### Long-form pacing

- **Pattern interrupt roughly every 10–20 seconds** — new visual element, camera move,
  sound cue, or on-screen question. If a scene runs longer than ~20s with nothing
  changing on screen, split it or add a visual beat.
- **Open loops** at section boundaries — tease a payoff a beat before delivering it, to
  keep the retention curve up over the full 7–8 minutes.
- **Bias toward more, shorter scenes with more visual events** over fewer long static
  ones.

---

## Stage 5 — Voiceover generation (edge-tts, male voice)

Generate narration **per scene**, not as one giant file — per-scene audio files make it
possible to resync/regenerate a single scene later without re-cutting the whole track.

```bash
pip install edge-tts --break-system-packages   # once per environment
edge-tts --list-voices | grep -i male           # pick a male voice, e.g. en-US-GuyNeural, en-GB-RyanNeural
```

For each scene, generate audio plus word-level timing (the timing file lets you
sync on-screen text/element entrances to the actual spoken words instead of a guess):

```bash
edge-tts \
  --voice en-US-GuyNeural \
  --text "$(cat projects/<slug>/audio/scene-1-narration.txt)" \
  --write-media projects/<slug>/audio/scene-1.mp3 \
  --write-subtitles projects/<slug>/audio/scene-1.srt
```

After generating, measure the real duration of each mp3 (e.g. with `ffprobe`) and write
it back into `scenes.json` as `actual_duration_sec` — scene length must be driven by
the real audio length, not the earlier estimate.

Only generate the voiceover for the scene currently being built (see Stage 6) unless
the user explicitly asks to batch-generate all narration ahead of time.

---

## Stage 6 — HyperFrames build (one scene at a time)

For **each scene of the current video, in order**:
1. Confirm `projects/<slug>/scenes.json` and `design.md` entries for this scene
   number are final.
2. Confirm `projects/<slug>/audio/scene-N.mp3` exists and `actual_duration_sec`
   is recorded.
3. **Check for reusable composition blocks first** — list the video's `components/`
   folder and skim any existing scene HTML for a piece that already does what this
   scene needs.
4. **Check for anything HyperFrames-API-specific this scene needs** (see below) before
   writing it from memory.
5. Write `compositions/<slug>/scenes/scene-N.html`:
   - A root element with `data-composition-id`, `data-width`, `data-height`, and a
     duration matching `actual_duration_sec * 1` (plus a small pad).
   - The narration: `<audio src="...">` pointing at the scene mp3 (copy into
     `assets/<slug>/audio/` so the composition can reference a project-relative path).
   - Background, foreground elements, motion, and text exactly per `design.md` — GSAP
     from the shared CDN script tag, registered as
     `window.__timelines["<video-slug>"] = gsap.timeline({ paused: true })` with
     scene timelines added non-paused.
   - Every timed visual element gets `data-start`, a duration, and `class="clip"`.
   - Deterministic logic only — no `Date.now()`, no `Math.random()`, no network fetches.
   - Reuse pieces from this video's own `components/` only — never reach into another
     video's folder. If an existing block almost fits, extend it with a
     `data-variable-values` override rather than forking a near-duplicate.
6. Mount the scene in `compositions/<slug>/index.html` via
   `data-composition-src="scenes/scene-N.html"`, with `data-start` and `data-duration`.
7. Preview with `npx hyperframes preview --background`, fix anything off, run
   `npm run check`, and only then mark it complete: bump
   `projects/<slug>/progress.json.current_scene` and move on.

Do not write `scene-(N+1).html` before scene N has been previewed and approved.

**When a scene's `purpose` is `"subscribe_ask"`:** build its subscribe/bell visual as
its own reusable block too, e.g. `components/subscribe-ask-hero.html` in this video's
folder — a large/centered treatment sized for that scene's full duration, distinct from
`engagement-overlay.html` below. It can share the same underlying asset and palette
(reuse the visual language) while differing in size, placement, and hold-duration so
the two moments don't look identical to a viewer.

**When a scene has `engagement_overlay` set:** build the popup as its own reusable
block, `compositions/<slug>/components/engagement-overlay.html`, driven by
`data-composition-variables` (e.g. `type`, `appearAtSec`, `holdSec`, palette colors
from `design.md`), not hardcoded per scene. Mount it inside the scene with
`data-variable-values` carrying that scene's `engagement_overlay` entry from
`scenes.json`. Building it once as a shared, props-driven block (rather than
copy-pasting bespoke markup into each scene) is exactly the kind of reuse the rules
ask for, and it's still fully self-contained inside this video's own folder, so the
delete-safety rule holds.

### Component reuse & docs lookup

Run through this every time, not just on the first scene:

- **Reuse check** — before writing any new `.html` file (scene or component), look at
  what already exists in this video's `components/` and `scenes/` folders. A title
  card, an icon badge, a lower-third, a background-gradient wrapper, a Ken-Burns
  image — if one exists and does roughly the right thing, reuse it or extend it with a
  variable override (e.g. a `color` or `icon` variable) instead of writing a
  near-duplicate. Only create a new block when nothing existing is close.
- **Skill check** — load `/hyperframes-core` (and `/hyperframes-animation` /
  `/hyperframes-creative` as needed) before writing composition code; follow their
  conventions and helper patterns over inventing new ones.
- **Docs check** — `npx hyperframes docs <topic>` for anything not verified against
  the installed pinned version (`package.json`): sub-composition mounting, media
  attributes, `window.__timelines` registration, render flags. HyperFrames APIs change
  between major versions — don't assume an older or newer pattern from memory still
  applies.

**Media asset note:** scenes reference media through project-relative paths. Keep the
per-video layout mirrored in `assets/<video-slug>/` so deleting the video means
deleting exactly its own `projects/<slug>/`, `compositions/<slug>/`, and
`assets/<slug>/` folders plus its one line in root `index.html`.

---

## Stage 7 — Assembly & render

Once every scene composition for this video exists and is approved:
1. Inside `compositions/<slug>/index.html`, lay out the scene sub-compositions
   back-to-back with `data-start` offsets and durations, applying the Stage 4
   transition between each pair.
2. Point the root `index.html` at this video (one `data-composition-src` line).
3. Sanity-check total duration against the 7–8 minute minimum; if short, that's a
   signal to revisit Stage 3 (more scenes/content), not to pad existing scenes.
4. Run `npm run check`, then render:
   ```bash
   npx hyperframes render -o out/<slug>.mp4
   ```
5. Update `projects/<slug>/progress.json.stage` to `"rendered"`, then move to
   **Stage 8 — Thumbnail & metadata generation**, fully specified by the common
   foundation (chapter timestamps for the description come from this video's
   `scenes.json` boundaries). Only mark the video `"done"` once metadata is also
   generated — a rendered file with no title/description/tags/thumbnail isn't
   upload-ready.

---

## Cross-session resumption checklist

When picking a video back up in a new session:
1. Identify the video's slug (ask the user if ambiguous, or list `projects/*/progress.json`).
2. Read `projects/<slug>/progress.json` for `format`, `stage`, and `current_scene`.
3. Read the file(s) that stage produces (`research.md`, `scenes.json`, `design.md`, …)
   inside that same folder for full context — don't re-derive from scratch, and don't
   read other videos' folders unless asked.
4. Continue at that exact point: e.g. `stage: "build", current_scene: 6` means scenes
   1–5 are done and approved — go straight to
   `compositions/<slug>/scenes/scene-6.html`.
