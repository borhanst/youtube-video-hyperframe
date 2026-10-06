---
name: hyperframes-video-common
description: "Shared foundation for HyperFrames YouTube video generation, referenced by hyperframes-youtube-long and hyperframes-youtube-shorts — not meant to be triggered on its own. Covers: the multi-video project/folder structure (one HyperFrames repo hosting many videos, each self-contained and safely deletable), the always-check-before-building rule (dedicated HyperFrames skills + hyperframes docs), the component-reuse rule, the edge-tts male-voice voiceover mechanics, the scene-by-scene HyperFrames build loop mechanics, general realism/engagement defaults, post-render thumbnail and SEO metadata generation (multiple title options, description, tags/hashtags, thumbnail), and cross-session resumption. When a video-generation request comes in, first decide long-form vs shorts (see 'Format router' below), read this file for the shared mechanics, then read the chosen format-specific skill for what's unique to that format (duration/scene targets, script beats, pacing intervals, subscribe-ask handling)."
---

# HyperFrames Video Generation — Common Foundation

This file holds everything that's identical whether the video being built is a
long-form YouTube video or a Short. It does not describe a full pipeline on its own —
read it alongside whichever format-specific skill applies.

## Format router

Decide the format **before** doing anything else, then read that skill in addition to
this one:

- **Default: `hyperframes-youtube-long`.** If the user just says "make a video" / "create
  a HyperFrames video" / gives a topic-transcript-blog with no format specified, assume
  long-form.
- **Use `hyperframes-youtube-shorts` only when the user explicitly asks for one** —
  wording like "short", "Shorts", "reel", "vertical video", "under 60 seconds/a
  minute", or similar.
- If genuinely ambiguous (e.g. they mention both a long topic and "also make a short
  version"), ask which one, or confirm both are wanted as two separate outputs under
  two separate video-slugs.

---

**Golden rule: build ONE scene at a time.** Finish, preview, and get sign-off on a
scene before starting the next one. Never batch-generate all scenes' voiceover or all
scenes' composition HTML in a single pass, in either format.

**Always check before you build, never guess from memory.** Two checks are mandatory
before writing any composition HTML, every scene, every time:
1. **Load the HyperFrames domain skills.** This environment has dedicated HyperFrames
   skills (`/hyperframes`, `/hyperframes-core`, `/hyperframes-animation`,
   `/hyperframes-creative`, `/hyperframes-cli`, `/media-use`, etc.) — load the
   relevant one and follow it for anything it covers: composition structure,
   `data-*` attribute semantics, GSAP setup, deterministic-render rules, media
   treatments. The skills installed in `.opencode/skills/` govern the *pipeline*; the
   HyperFrames domain skills govern the *composition code itself* and take precedence
   on API-level details.
2. **Consult the local HyperFrames docs for anything you're not 100% certain of** —
   `npx hyperframes docs <topic>` (topics: `data-attributes`, `gsap`, `compositions`,
   `rendering`, `examples`, `troubleshooting`). For anything beyond that, consult
   `https://hyperframes.heygen.com/llms.txt` rather than training-data memory.

**Reuse existing components — never duplicate silently.** Before creating any new
sub-composition (a scene, a lower-third, an icon badge, a transition, anything), search
this video's own `compositions/<video-slug>/` folder for something that already does
the job, or something close enough to extend with a `data-variable-values` override.
Only write a new sub-composition when nothing existing covers it. Reuse is *encouraged
within one video's own folder*, and only *disallowed across different videos' folders*
(see "Multi-video project structure").

---

## Multi-video project structure

One HyperFrames repo holds many videos over time — long-form and Shorts alike. The
rule that makes them safe to delete independently: **everything about video X lives
under one folder named for X, and the root `index.html` only ever touches that video
through one `data-composition-src` line.**

```
index.html                       # root — one <div data-composition-src> for the active video
compositions/
  <video-slug>/                  # e.g. "2026-10-ai-agents-explained" or "2026-10-ai-agents-short"
    index.html                   # this video's own root sub-composition (scene strip)
    scenes/
      scene-1.html, scene-2.html, ...
    components/                  # pieces used only by this video (lower-thirds, badges, etc.)
projects/
  <video-slug>/
    progress.json                # this video's own progress tracker
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
video's own `compositions/<video-slug>/index.html` directly).

**To retire a video:** delete `projects/<video-slug>/`, delete
`compositions/<video-slug>/`, and (if it's the active one) repoint the one
`data-composition-src` line in `index.html`. That's it — no other video's files are
touched, so nothing else can break.

**Rules that keep this safe:**
- Never reference one video's `scenes/` or `components/` HTML files from another
  video's folder. If two videos want the same visual idea, duplicate the small block
  into both folders rather than share it — a shared file is exactly what makes
  deleting one video risk breaking another. This applies across a long-form video
  and its own Shorts cut too — separate slugs, separate folders.
- Each video's `progress.json`, assets, and audio live inside its own
  `projects/<video-slug>/` folder — never in a shared top-level `assets/` or `audio/`.
- Slug once at setup and reuse that exact slug everywhere
  (`projects/<slug>/`, `compositions/<slug>/`, `out/<slug>.mp4`). Suffix a Shorts cut
  of an existing video distinctly (e.g. `...-short`).

---

## Setup (first message of a new video)

Every video — long-form or Shorts — adds its own `projects/<video-slug>/` and
`compositions/<video-slug>/` folders. On the **first message of a new video**, pick
its slug and create its skeleton — don't touch any other video's folders:

```
projects/<video-slug>/
  progress.json       # { "stage": "...", "current_scene": N, "total_scenes": N, "notes": "" }
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

compositions/<video-slug>/
  index.html
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

## Element sourcing

Build `projects/<video-slug>/assets/manifest.json`, one entry per element:

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
- Prefer SVG or CSS-drawn icons over raster — they scale and recolor to the palette.
- Download chosen assets into `projects/<video-slug>/assets/raw/` and copy what each
  scene needs into the project `assets/` folder the composition actually references.
- Flag any gap where no good element exists yet — the design stage may need a
  generated/abstract visual (shape, gradient blob, typographic treatment) instead.

---

## Voiceover generation (edge-tts, male voice)

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
  --text "$(cat projects/<video-slug>/audio/scene-1-narration.txt)" \
  --write-media projects/<video-slug>/audio/scene-1.mp3 \
  --write-subtitles projects/<video-slug>/audio/scene-1.srt
```

After generating, measure the real duration of each mp3 (e.g. with `ffprobe`) and write
it back into `scenes.json` as `actual_duration_sec` — scene length must be driven by
the real audio length, not an earlier estimate.

Only generate the voiceover for the scene currently being built unless the user
explicitly asks to batch-generate all narration ahead of time.

---

## HyperFrames build loop (mechanics common to every scene, either format)

For **each scene of the current video, in order**:
1. Confirm `projects/<video-slug>/scenes.json` and `design.md` entries for this scene
   number are final.
2. Confirm `projects/<video-slug>/audio/scene-N.mp3` exists and `actual_duration_sec`
   is recorded.
3. **Check for reusable composition blocks first** — list the video's `components/`
   folder and skim any existing scene HTML for a piece that already does what this
   scene needs.
4. **Check anything HyperFrames-API-specific this scene needs** against
   `/hyperframes-core` + `npx hyperframes docs` before writing it from memory.
   **If the scene is a motion graphic** (kinetic type, logo reveal, shape/UI morph,
   count-up, no narration), also read
   `.opencode/skills/hyperframes-motion-graphics/SKILL.md` and follow its prompt
   patterns (structured `<inputs>/<direction>/<structure>/<build>/<gotchas>` template,
   beat-grid placement, loop closure).
5. Write `compositions/<video-slug>/scenes/scene-N.html`:
   - A root element with `data-composition-id`, `data-width`, `data-height`, and
     driven by the audio's real duration.
   - `<audio src="...">` (or `<video data-has-audio="true">`) for the scene's
     narration mp3 — never a guessed duration.
   - Background, foreground elements, motion, and text exactly per `design.md` —
     GSAP from the shared CDN script tag, registered as
     `window.__timelines["<video-slug>"] = gsap.timeline({ paused: true })` with
     scene timelines added non-paused.
   - Every timed visual element gets `data-start`, a duration, and `class="clip"`.
   - Deterministic logic only — no `Date.now()`, no `Math.random()`, no fetches.
   - Reuse pieces from this video's own `components/` only — never reach into another
     video's folder. If an existing block almost fits, extend it with a
     `data-variable-values` override rather than forking a near-duplicate.
6. Mount the scene in `compositions/<video-slug>/index.html` via
   `data-composition-src="scenes/scene-N.html"` (optionally as its own previewable
   root). Preview with `npx hyperframes preview --background`.
7. Fix anything off, run `npm run check`, and only then mark it complete: bump
   `projects/<video-slug>/progress.json.current_scene` and move on.

Do not write `scene-(N+1).html` before scene N has been previewed and approved.

### Component reuse & docs lookup

Run through this every time, not just on the first scene:

- **Reuse check** — before writing any new `.html` file (scene or component), look at
  what already exists in this video's `components/` and `scenes/` folders. A title
  card, an icon badge, a lower-third, a background-gradient wrapper, a Ken-Burns
  image — if one exists and does roughly the right thing, reuse it or extend it with
  a variable override instead of writing a near-duplicate.
- **Skill check** — load `/hyperframes-core` (and `/hyperframes-animation` /
  `/hyperframes-creative` as needed) before writing composition code; follow their
  conventions over inventing new ones.
- **Docs check** — `npx hyperframes docs <topic>` for anything not verified against
  the installed pinned version (`package.json`). HyperFrames APIs change between
  versions; don't assume an older or newer pattern from memory still applies.

**Media asset note:** scenes reference media through paths relative to the project
(`assets/...`, `projects/<video-slug>/audio/...` copied into `assets/` if needed).
Keep the per-video layout mirrored in `assets/<video-slug>/` so deleting the video
means deleting exactly its own `projects/` folder, `compositions/` folder, and
`assets/<video-slug>/` folder.

---

## Realism & engagement — shared defaults

Apply these as standing defaults on every scene, in either format. Record the
video-wide choices (grade, default spring config, SFX bed) once in `design.md` so every
scene builds on the same foundation. Exact pacing numbers (how often a pattern
interrupt happens, how long an overlay holds) differ by format — see the
format-specific skill — but the techniques themselves are shared:

- **Depth via parallax, always.** Every scene has at minimum a background layer and a
  foreground layer, moving at different speeds — e.g. background drifts/zooms at
  ~0.3–0.5% scale-per-second, foreground elements at ~1–2%, text/UI layer stays static
  or moves fastest. Never one flat plane.
- **Spring easing by default, linear only for utility motion.** Default entrance
  config: `gsap.to` with `ease: "back.out(1.4)"` or an explicit elastic/spring-like
  ease (slight overshoot-and-settle). Reserve linear for things that shouldn't feel
  alive (e.g. a progress bar).
- **Nothing is ever fully static.** Even a "still" background gets a slow continuous
  drift: a gradient angle animating a few degrees over the scene, or a slow Ken-Burns
  zoom (~102%→106% over the scene). True stillness reads as a slide, not video.
- **Soft, offset shadows on overlaid elements**, not flat/hard ones — e.g. 8–20px blur
  with a slight downward offset, so icons and cutout images look placed into the scene.
- **A touch of imperfection, applied uniformly across the whole video**: subtle film
  grain, a faint vignette, and one consistent color grade — pick it once per video in
  `design.md` and reuse it as a shared overlay/filter block, not redone per scene.
- **Rotate the layout**, scene to scene — centered icon + title below every time is the
  fastest way to look templated.
- **Caption/keyword sync to the actual spoken word, not scene boundaries.** Use the
  word-level timestamps from edge-tts's `--write-subtitles` to trigger keyword
  pop/highlight animations at the exact moment the word is spoken.
- **Sound design under the voiceover**: a low, consistent music bed (ducked under
  narration, e.g. -18 to -22dB), plus small whoosh/pop/click SFX timed to element
  entrances — sync the SFX hit to the same moment the spring animation settles.

---

## Thumbnail & metadata generation (after render)

Once `out/<video-slug>.mp4` exists and is approved, generate the upload package before
marking the video fully done. This is a required final stage for both formats — a
finished video with no metadata isn't ready to upload. Everything goes in
`projects/<video-slug>/metadata/`:

```
projects/<video-slug>/metadata/
  titles.md          # 5-8 title options, ranked, with the recommended pick marked
  description.md      # full YouTube description, ready to paste
  tags.json            # tag list, ready to paste (long-form) / hashtag list (shorts)
  thumbnail/
    thumb-1.png, thumb-2.png, ...   # long-form only, see below
```

### Keyword research first

Don't write titles/tags from assumption alone — search the web for how people actually
phrase searches around this topic (autocomplete-style phrasing, related/"people also
search" terms, what similar existing videos on the topic are titled) and let that
inform word choice. Favor the phrasing real searches use over a cleverer synonym.

### Titles — generate multiple, always

Write **5–8 distinct title options** to `titles.md`, not one. Vary the *angle*, not
just the wording, using proven patterns — mix several of: a direct benefit/outcome, a
curiosity gap ("why X actually..."), a number/list format, a how-to, a question, a bold
or contrarian claim. For each option note which pattern it uses, so the set is
genuinely varied rather than eight rewordings of the same idea. Mark one as the
recommended primary pick, with a one-line reason.

Rules that apply to every option:
- Put the primary keyword/topic phrase as early in the title as it can naturally sit.
- Front-load what matters: YouTube truncates long titles, so the first ~40–50
  characters should stand alone; stay under ~60–70 characters total where possible.
- Write for a human reading it in a feed — it still has to make someone want to click.
- **Shorts**: title can be shorter and punchier since the hook is also delivered
  on-screen in the first second.

### Description — SEO-structured, ready to paste

Write `description.md` as the actual text to paste into YouTube, in this order:
1. **First 1–2 lines** (shown before "...more") — the strongest, keyword-natural
   summary of the video.
2. A short expanded summary paragraph, naturally including the primary keyword and
   2–3 related phrases from the keyword research — written for a human, not stuffed.
3. **Long-form only**: chapter timestamps (`0:00 Hook`, `0:45 ...`, etc.) matching
   the actual scene boundaries from `scenes.json`.
4. Any links/CTA the user wants included.
5. A line of 3–8 relevant hashtags at the very bottom.

**Shorts**: keep the whole description brief (a couple of sentences plus hashtags);
always include `#Shorts`.

### Tags / hashtags

```json
{
  "tags": ["primary keyword phrase", "broader topic tag", "specific long-tail phrase"],
  "hashtags": ["#TopicTag", "#Shorts"]
}
```

Mix broad category tags, the specific topic phrase, and a few longer-tail phrases —
don't just repeat the title's words. Roughly 10–15 tags is typical.

### Thumbnail (long-form: required — shorts: optional/skip)

**Long-form**: build the thumbnail as its own small composition (e.g.
`compositions/<video-slug>/thumbnail.html`), reusing the same palette, fonts, and
block style from `design.md` so it looks like it belongs to the video. Guidelines:
- 1280x720, big and legible at small sizes — 3–6 words of bold text maximum, high
  contrast.
- Reuse an existing icon/image component from this video's own `components/` rather
  than building new thumbnail-only assets.
- Generate 2–3 distinct variants. Render stills by pointing the render at the
  thumbnail composition with a short duration.

**Shorts**: YouTube generates a cover from the video itself by default — skip a
dedicated thumbnail unless the user asks.

Once titles, description, tags, and (for long-form) thumbnails are written, update
`projects/<video-slug>/progress.json.stage` to `"metadata_done"`.

---

## Cross-session resumption checklist

When picking a video back up in a new session:
1. Identify the video's slug (ask the user if ambiguous, or list `projects/*/progress.json`).
2. Read `projects/<video-slug>/progress.json` for `format`, `stage`, and
   `current_scene` — `format` tells you which of `hyperframes-youtube-long` /
   `hyperframes-youtube-shorts` to read alongside this file.
3. Read the file(s) that stage produces (`research.md`, `scenes.json`, `design.md`, …)
   inside that same folder for full context.
4. Continue at that exact point: e.g. `stage: "build", current_scene: 6` means scenes
   1–5 are done and approved — go straight to
   `compositions/<video-slug>/scenes/scene-6.html`.
