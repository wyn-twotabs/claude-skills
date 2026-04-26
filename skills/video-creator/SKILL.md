---
name: video-creator
description: Multi-format video creation workflow using Remotion — from topic to rendered video with TTS narration. Supports YouTube Shorts, Instructional (widescreen), Instagram Square, and Fireship Explainer formats.
metadata:
  tags: video, remotion, tts, youtube-shorts, instagram, instructional, fireship, miro, storyboard
---

# Claude Code Skill: Video Creator (Multi-Format)

A sequential workflow for creating professional videos across multiple formats,
from topic to rendered video with synchronized TTS narration.

---

## Style Guides

Before executing any workflow, Claude must ask the user which video format
they are creating and load the corresponding style guide.

### Available Style Guides

| ID | File | Format | Use Case |
|---|---|---|---|
| `shorts` | `style-guides/youtube-shorts.md` | 1080×1920 (9:16) | YouTube Shorts – general topics |
| `instructional` | `style-guides/instructional-software.md` | 1920×1080 (16:9) | Client software showcase/tutorials |
| `instagram` | `style-guides/instagram-square.md` | 1080×1080 (1:1) | Instagram – software news snippets |
| `fireship` | `style-guides/fireship-explainer.md` | 1920×1080 (16:9) | Fast-paced tech explainers — "X in N Seconds" format |

---

## Activation Prompt

Use this to start the workflow:

```
🎬 Activate Video Creator

Topic: [YOUR TOPIC HERE]
```

> ⚠️ `🎬 Activate Video Creator` is the **only** trigger for this skill.
> Do not activate mid-conversation from unrelated messages.

---

## Session Context

Claude must initialize and maintain this context object at activation and
reference it at every step. Do not allow any field to drift or be forgotten
across a long session.

```
format:           <selected style guide ID>
topic:            <confirmed topic string>
tts_provider:     <selected provider + voice name>
asset_manifest:   public/images/asset-manifest.json
timing_file:      AUDIO_TIMING.md
render_output:    output/video-<topic-slug>-<YYYYMMDD>.mp4
```

---

## Step 0: Format Selection (ALWAYS FIRST)

When activated, Claude must ask:

```
Before we begin, which video format are you creating?

1. 🎬 YouTube Short       — vertical 1080×1920, general topics, fast-paced
2. 🖥️  Instructional      — widescreen 1920×1080, software showcase for clients
3. 📷  Instagram Square   — 1080×1080, software news snippets
4. 🔥  Fireship Explainer — widescreen 1920×1080, "X in N Seconds" tech explainer

Reply with the number or name of the format.
```

Once selected, Claude must:
- State which style guide file is being loaded
- Read and internalize ALL rules from that `.md` file
- Initialize the Session Context above
- Run the Preflight Check below before proceeding

---

## Step 0b: Preflight Check

Run all checks **before** any content work begins. Report every failure
upfront — do not silently skip or fail mid-workflow.

### Files
- [ ] `style-guides/<selected-id>.md` exists and is readable
- [ ] `asset-sourcing.md` exists in the skill's base directory
- [ ] `tts-providers.md` exists in the skill's base directory
- [ ] `public/images/` directory exists or can be created
- [ ] Remotion project is initialized (`package.json` contains `remotion` dependency)
- [ ] `remotion.config.ts` is present (warn if missing — will need to be created)
- [ ] Tailwind library is installed — `package.json` contains `@remotion/tailwind-v4` and `tailwindcss`
- [ ] `src/index.css` exists and starts with `@import "tailwindcss"`
- [ ] `src/Root.tsx` imports `./index.css`
- [ ] `remotion.config.ts` calls `enableTailwind` from `@remotion/tailwind-v4`

If Tailwind is not set up, run:
```bash
npm install -D @remotion/tailwind-v4 tailwindcss
```
Then add to `remotion.config.ts`:
```ts
import { enableTailwind } from "@remotion/tailwind-v4";
Config.overrideWebpackConfig((c) => enableTailwind(c));
```
And ensure `src/index.css` contains `@import "tailwindcss";` and is imported in `src/Root.tsx`.

### Environment Variables

| Variable | Required When | Action if Missing |
|---|---|---|
| `UNSPLASH_ACCESS_KEY` | Always (optional) | Warn, skip Unsplash sourcing |
| `GIPHY_API_KEY` | Always (optional) | Warn, skip Giphy sourcing |
| `ELEVENLABS_API_KEY` | ElevenLabs selected | Block — prompt user to set before continuing |
| `OPENAI_API_KEY` | OpenAI TTS selected | Block — prompt user to set before continuing |
| `MIRO_BOARD_URL` | Step 6 (Storyboard) | Warn, prompt user to provide board URL manually |

If all checks pass, confirm:
```
✅ Preflight passed. Proceeding with [FORMAT NAME] workflow.
```

If any blockers exist, list them and halt until resolved.

---

## Autonomy Rules

### Claude decides autonomously
- Which SimpleIcons slug to use for a given brand
- Frame-accurate timing calculations from measured audio durations
- Whether an asset needs a license check
- File naming and directory structure

### Claude must ask the user
- Format selection (Step 0)
- TTS provider and voice — suggest top 2 options with reasoning, let user decide
- Any script revision direction
- Whether to proceed past each Approval Gate

---

## Complete Workflow

All visual, technical, and audio decisions are driven by the loaded style guide
unless explicitly overridden below.

---

### Step 1: Topic Input & Validation

Analyze the topic against the loaded style guide constraints:
- Does it fit the target duration?
- Can it be effectively visualized in the chosen format?
- Is it scoped correctly for the audience?

Ask 2–3 clarifying questions if needed before proceeding.

---

### Step 2: Script Creation

Follow the **Script Format** defined in the loaded style guide.

All timestamps, scene structure, word counts, and visual markers must match
the guide's conventions.

Regardless of style guide, every scene in the output script **must** conform
to the following minimum schema. Style guides may extend these fields but
cannot omit them.

#### Minimum Scene Schema

```
scene_id:      integer — sequential, starting at 1
start_frame:   integer — inclusive
end_frame:     integer — inclusive
narration:     string  — the exact TTS line for this scene
visual:        string  — description of what is on screen
transition_in: string  — e.g. "fade", "slide-up", "cut"
transition_out: string — e.g. "fade", "slide-down", "cut"
sfx:           string or null — sound effect name or null
```

---

### Step 3: Approval Gate A — Script

Present the script with:
- Estimated total duration
- Number of scenes
- Key visuals needed per scene
- Total word count

Await one of:
- ✅ **APPROVED** — Proceed to Step 4
- 🔄 **REVISE** — [specific changes requested]
- ❌ **START OVER** — [new direction]

---

### Step 4: Asset Sourcing

Read `asset-sourcing.md`, then walk through the approved script scene-by-scene
and identify every visual that is **not** a code block or kinetic text.

#### Deduplication
Before downloading any asset, check if a file with the same name or hash
already exists in `public/images/`. Reuse existing files — do not re-download.

#### Source Priority

| Priority | Source | Best For |
|:---:|---|---|
| 1 | **SimpleIcons** (`simpleicons.org`) | Brand/tech logos — no API key needed |
| 2 | **Unsplash API** | Photos, people, backgrounds, product shots |
| 3 | **Giphy API** | Reaction GIFs / humor beats |
| 4 | **meme-factory skill** | Custom text-on-template memes |
| 5 | **WebSearch fallback** | Anything not found above |

> WebSearch fallback: only use assets with **CC0, Unsplash License, or explicit
> commercial-use permission**. Do not use assets with ambiguous or restrictive licenses.

#### All-Sources-Fail Fallback
If no source yields a usable asset, generate a placeholder: a solid colored
rectangle with the asset label as white text (implementable directly in Remotion).
Mark the asset in the manifest as `"status": "placeholder"` so the user can
replace it before final render.

#### Asset Manifest
Generate `public/images/asset-manifest.json` as described in `asset-sourcing.md`.
Every asset entry must include at minimum:

```json
{
  "scene_id": 1,
  "asset_id": "scene-1-logo-react",
  "file": "public/images/scene-1-logo-react.svg",
  "source": "simpleicons",
  "license": "CC0",
  "status": "ready"
}
```

#### Approval Gate B — Asset List
Before executing any downloads, present:
- Full asset list (count, type, source, any API calls required)
- Any assets that will be skipped due to missing API keys
- Any assets falling back to placeholder

Await GO / ADJUST before downloading.

---

### Step 5: TTS Provider & Voice Selection

Read `tts-providers.md`, then present the options:

```
Which TTS provider do you want to use?

1. 🎙️  ElevenLabs  — highest quality, most natural (requires ELEVENLABS_API_KEY)
2. 🍎  macOS say   — free, offline, fast (good for drafts)
3. 🤖  OpenAI TTS  — solid quality (requires OPENAI_API_KEY)
```

Suggest the top 2 voices best suited to the chosen format with brief reasoning.
Let the user make the final selection.

#### Approval Gate C — TTS Cost & Voice
Before generating audio, show:
- Selected provider and voice
- Total character count across all narration lines
- Estimated cost (ElevenLabs / OpenAI — calculate from current per-character pricing)

Await GO before proceeding.

#### Recording Rules
- Split narration at sentence boundaries (`.`, `!`, `?`) — one audio file per line
- File naming: `audio/scene-{scene_id}-line-{line_id}.mp3`
- On API failure: retry once, then fall back to macOS `say` and flag the file for re-recording
- Normalize all files to **-16 LUFS, -1.5 dBTP** using `ffmpeg-normalize`
- After normalization, measure each file's duration
- If any file deviates more than **15%** from its expected duration, re-record that line
- Concatenate scene audio with `ffmpeg` and measure total duration before
  generating `AUDIO_TIMING.md`
- Verify total is within **±2s** of the style guide's target duration

#### AUDIO_TIMING.md Schema
Generate `AUDIO_TIMING.md` with the following exact format:

```
| scene_id | line_id | file                          | start_frame | end_frame | duration_s | words                        |
|----------|---------|-------------------------------|-------------|-----------|------------|------------------------------|
| 1        | 1       | audio/scene-1-line-1.mp3      | 0           | 89        | 2.97       | ["React","is","fast"]        |
| 1        | 2       | audio/scene-1-line-2.mp3      | 90          | 179       | 3.00       | ["and","easy","to","learn"]  |

FPS: 30
Total frames: 900
Total duration: 30.0s
Target duration: 30s
Delta: 0.0s ✅
```

Do not proceed to Step 6 until `AUDIO_TIMING.md` is generated and the delta
is within ±2s.

---

### Step 6: Update Miro Storyboard

Read the board URL from `MIRO_BOARD_URL` env var. If not set, ask the user
to provide it before continuing.

Follow the **Storyboard Layout** defined in the loaded style guide.

Each frame card must include:
- Scene ID and narration line
- Exact `start_frame` and `end_frame` from `AUDIO_TIMING.md`
- Word-level sync points for any text highlights
- Animation trigger points
- SFX placement
- Visual transition cues (in and out)
- Asset reference (filename from the asset manifest)

---

### Step 7: Generate Remotion Code

#### Approval Gate D — Storyboard Review
Before generating code, confirm the storyboard looks correct to the user.
Present a plain-text summary of each scene (visual + narration + timing).

Await GO / ADJUST before writing code.

#### Code Generation Rules
Follow the **Remotion Configuration** from the loaded style guide for:
- Composition dimensions and FPS
- Component structure and file layout
- Animation utilities (`interpolate`, `spring`, `useCurrentFrame`)
- Audio integration (one `<Audio>` tag per line file, offset by `start_frame`)
- Visual effects (texture overlays, chromatic aberration, etc.)

Validate that `remotion.config.ts` specifies the correct dimensions and FPS
for the selected format. Create or update the file if it does not match.

#### Tailwind Library Usage (REQUIRED)

The Tailwind design system in `src/index.css` defines all palette colors and
the video typography scale. Apply the following rules in every generated component:

**The core rule:**
```
className  →  ALL static visual properties (color, layout, font-size, spacing, border-radius)
style={{}} →  ONLY animated values driven by useCurrentFrame() / interpolate() / spring()
```

**What to put in `className`:**
- Background and text colors (`bg-fire-canvas`, `text-fire-text`, `text-shorts-cream`, etc.)
- Font sizes (`text-slammer`, `text-hero`, `text-headline`, `text-title`, etc.)
- Font weight, tracking, leading (`font-black`, `tracking-[-0.02em]`, `leading-none`)
- Layout structure (`scene-center`, `scene-safe`, `split-screen`, `lower-third`, `bottom-bar`)
- Spacing, border-radius, overflow (`p-8`, `gap-6`, `rounded-lg`, `overflow-hidden`)
- Static shadows and borders (`image-card`, `border-2`, `border-instr-accent`)

**What stays in `style={{}}`:**
- `opacity` — always animated, never static
- `transform` — translateY, scale, rotate driven by frame
- `width` / `height` — only when animated (e.g. progress bars, wipe effects)
- `color` / `background` when passed as JS props to shared components (KineticLine, PaperBackground)

**Color tokens as JS values** — when a color must be passed as a prop or used in
an animated `style`, reference the CSS custom property:
```tsx
// As a prop to a component that expects a string
<PaperBackground color="var(--color-shorts-tan)" sceneId="setup" />
// In an animated style (rare — prefer className for static color)
style={{ background: `rgba(${r},${g},${b}, ${interpolate(frame, ...)})` }}
```

**Token quick-reference by format:**

| Format | Scene bg | Body text | Accent |
|:---|:---|:---|:---|
| Fireship | `bg-fire-canvas` | `text-fire-text` | `text-fire-accent` |
| Shorts Hook | `bg-shorts-espresso` | `text-shorts-cream` | `text-shorts-amber` |
| Shorts Setup | `bg-shorts-tan` | `text-shorts-brown` | `text-shorts-steel` |
| Instructional | `bg-instr-canvas` | `text-white` | `text-instr-accent` |
| IG Breaking | `bg-ig-breaking-bg` | `text-white` | `text-ig-breaking` |
| IG Launch | `bg-ig-launch-bg` | `text-white` | `text-ig-launch` |
| IG Stats | `bg-ig-stats-bg` | `text-white` | `text-ig-stats` |
| IG Opinion | `bg-ig-opinion-bg` | `text-ig-opinion-text` | `text-ig-opinion` |
| IG Tool | `bg-ig-tool-bg` | `text-ig-tool-text` | `text-ig-tool` |

Full token tables and layout component examples are in the **Tailwind Library Reference**
section at the bottom of each style guide file.

**NEVER create new fonts, colors, or components (HARD RULES):**

- **No new colors** — never define a new color token, hex value, or `--color-*` CSS variable.
  Always use an existing token from `src/index.css`. If no token fits, pick the closest one.
- **No new fonts** — never import a new font family or define a new `--font-*` variable.
  Always use the font families, sizes, and weights already defined in `src/index.css`.
- **No new components** — do not create a new shared component (e.g. a new `KineticLine`,
  `PaperBackground`, `LowerThird`, etc.) unless the user explicitly grants permission.
  **Before creating any new component, stop and ask the user:**
  > "I need a component that doesn't exist yet: `<ComponentName>`. May I create it?"
  Only proceed after receiving explicit approval.

---

### Step 8: Render

#### Step 8a: Test Render
Run a test render of the first 100 frames before the full render:

```bash
npx remotion render <CompositionId> --frames=0-99 --output=test-render.mp4
```

Validate:
- [ ] No TypeScript or compilation errors
- [ ] Audio is present and syncs correctly at frame 0
- [ ] No visual overflow or clipping at format boundaries
- [ ] Transitions render without missing keyframes

**Do not proceed to full render if the test render fails.** Fix all issues first.

#### Step 8b: Full Render
Follow the **Render Instructions** from the loaded style guide.

Output file must be named:
```
output/video-<topic-slug>-<YYYYMMDD>.mp4
```

Where `<topic-slug>` is the topic lowercased with spaces replaced by hyphens,
and `<YYYYMMDD>` is today's date.

---

## Final Checklist

### Setup
- [ ] Format selected and style guide loaded in full
- [ ] Session context initialized
- [ ] Preflight checks passed (files, env vars, Remotion project)

### Content
- [ ] Topic validated against format constraints
- [ ] Script follows style guide structure and minimum scene schema
- [ ] Script approved (Gate A)

### Assets
- [ ] Asset list reviewed and approved (Gate B)
- [ ] All assets sourced, deduplicated, downloaded to `public/images/`
- [ ] Asset manifest (`public/images/asset-manifest.json`) generated
- [ ] All placeholder assets flagged for user review

### Audio
- [ ] TTS cost and voice approved (Gate C)
- [ ] Audio recorded per line with correct file naming
- [ ] All files normalized to -16 LUFS / -1.5 dBTP
- [ ] All durations verified within ±15% per line
- [ ] Total duration within ±2s of target
- [ ] `AUDIO_TIMING.md` generated with correct schema

### Storyboard
- [ ] Miro storyboard updated with frame-accurate timings and asset references

### Code & Render
- [ ] Storyboard reviewed and approved (Gate D)
- [ ] `remotion.config.ts` matches selected format dimensions and FPS
- [ ] Remotion code generated and compiles without errors
- [ ] Test render (frames 0–99) passes all validation checks
- [ ] Full render completed
- [ ] Output file named with topic slug and date
