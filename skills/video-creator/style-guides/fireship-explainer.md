# Style Guide: Fireship-Style Explainer Video

**Format:** 1920×1080 (16:9 widescreen)
**Target Duration:** Match the title exactly — "60 Seconds" = 59–61s, "100 Seconds" = 98–102s
**Frame Rate:** 60fps — kinetic text and code animations demand the extra smoothness
**Use Case:** Fast-paced tech concept intros, language/tool/framework explainers, developer-facing product overviews

---

## Core Philosophy

> **Every second earns its place. Cut everything else.**

Fireship's style is **image-first, text-last**. The voice tells the story. The screen shows *proof* — images, charts, code, diagrams. Kinetic text is a last resort, not a default.

**Five unbreakable rules:**

1. **No dead air.** A frame without new information is a cut.
2. **Narration drives everything.** Visuals *illustrate* what the voice says — they don't transcribe it.
3. **Respect the viewer's intelligence.** Don't put on screen what they can already hear.
4. **Images before words. Always.** If a real image, screenshot, chart, or diagram can replace kinetic text, it must.
5. **One visual anchor per VO beat.** Every sentence must have a corresponding visual. Kinetic text alone is not a visual.

---

## Visual Hierarchy — What Goes On Screen

Before writing a single word of kinetic text, work through this list top-to-bottom. Use the **first option that applies** — stop there.

| Priority | Visual Type | When to use |
|:---:|:---|:---|
| 1 | **Code block** | Any command, API, syntax, or file is mentioned |
| 2 | **Real image / screenshot** | Any person, product, UI, website, or tool that has a real visual |
| 3 | **Chart / diagram / graph** | Any comparison, benchmark, architecture, or flow |
| 4 | **Meme / generated image** | Humor beats, absurd facts, cultural references |
| 5 | **Key term / phrase** (1–5 words max) | Named concepts, proper nouns, numbers — paired with a visual |
| 6 | **Full kinetic sentence** | Only when literally nothing visual exists — rare |

**In practice:**
- VO says *"Tanner Linsley built it"* → web-search for Tanner's profile photo or GitHub avatar, show it
- VO says *"3× faster than Node"* → bar chart (React bars, not words)
- VO says *"The homepage at tanstack.com"* → screenshot of the actual site
- VO says *"Run bun install"* → code block, not a description
- VO says *"He built it over Christmas"* → meme, not kinetic text
- VO says *"Works with React, Vue, Solid"* → framework logos side by side, not words

**The kinetic text test:** Before adding a `<KineticLine>`, ask: *"Does a real image, screenshot, or chart exist for this?"* If yes — find it, download it, show it instead. Kinetic text for a full sentence should appear fewer than 3 times in a 60-second video outside of the title hook and outro.

### Visual Budget Per Scene

| Scene | Max full kinetic sentences | Required visuals |
|:---|:---:|:---|
| Title Hook | 1 (subtitle) | Topic logo or icon — web search it |
| Origin | 0 | Timeline + person photo + meme |
| Core Concepts | 0 | Diagrams, badges, charts, screenshots |
| Code | 0 | Code block is the visual |
| Outro | 2 | Topic name only |

### Sourcing Real Images

When the VO mentions a real person, product, website, framework, or tool:

1. **Web-search** for an official image, avatar, or screenshot
2. Download it to `public/images/[topic-name]/`
3. Use `<Img src={staticFile(...)} />` — never hot-link external URLs in Remotion
4. Crop/resize to fit the scene layout (person photos: square crop, UIs: 16:9 or native ratio)

Common sources:
- **Person avatars:** GitHub profile photo, Twitter/X profile, company blog headshot
- **Framework logos:** Official GitHub repo, official docs site, Simple Icons (SVG)
- **Product screenshots:** Official marketing site, docs, or app itself
- **UI screenshots:** Record at 2× Retina, crop to relevant region only

### Stacking Images with Text

When a visual AND a label both belong on screen at the same time:

- **Split layout:** visual takes 40–50% of frame width; text/label takes the rest
- **Overlay label:** small Fira Code label below or beside the image, not on top of it
- **Never cover a visual with full-sentence kinetic text** — if the image is there, let it breathe

---

## Visual Style

### Tone & Aesthetic
- **Dark and technical:** Atom One Dark as the universal ground truth
- **Dry wit over polish:** A well-timed meme beats a slick transition every time
- **Code is king:** If you can show a snippet, show it — don't describe it
- **Visuals illustrate, not narrate:** The voice carries the story; the screen provides proof

### Layout Principles
- **Safe zones:** Keep content within inner 1760×960 (40px margins)
- **Full-dark canvas:** Near-black background is the default; elements float on top
- **Split-screen:** Left code / right diagram for concept-plus-example moments
- **Center-stage for hooks:** Opening definition gets maximum real estate — large, centered, white
- **No lower thirds.** Topic name appears as a kinetic title card only

### Color Palette (Atom One Dark)

| Element | Hex | Usage |
|:---|:---|:---|
| Canvas background | `#1E2127` | Base layer for all scenes |
| Code editor bg | `#282C34` | Code block panels |
| Primary text | `#ABB2BF` | Body narration text |
| White emphasis | `#FFFFFF` | Keyword callouts, topic title |
| Syntax: keywords | `#C678DD` | Purple — `function`, `const`, `class` |
| Syntax: strings | `#98C379` | Green — string literals |
| Syntax: numbers | `#D19A66` | Orange — numeric values |
| Syntax: functions | `#61AFEF` | Blue — function/method names |
| Syntax: comments | `#5C6370` | Muted gray — `// like this` |
| Brand accent | `#FF6B35` | Fire orange — callouts, highlights, title |
| Accent warm | `#FFB347` | Amber — secondary accent |
| Cool gray | `#545E74` | Diagram labels, timestamps |
| Error / Warning | `#E06C75` | Red — bugs, caveats |
| Success | `#98C379` | Green — confirms, results |

### Typography

All sizes are in CSS pixels on a 1920×1080 canvas (safe zone 1760×960). These are not suggestions — undersized text wastes the frame.

| Role                 | Font      | Size         | Weight | Color                  | Notes |
|:---------------------|:----------|:-------------|:-------|:-----------------------|:------|
| Opening topic title  | Rubik     | 140–160px    | 900    | `#FFFFFF`              | Tight tracking `−0.02em`, centered, max 2 lines |
| Kinetic body text    | Rubik     | 56–68px      | 400    | `#ABB2BF`              | Max line width 1400px; hard-wrap at ~30 chars/line |
| Keyword callouts     | Rubik     | 72–88px      | 700    | accent/syntax color    | Inline with body or standalone; pulse to 110% |
| Section label / term | Rubik     | 48–56px      | 600    | `#FFFFFF`              | Short identifiers (1–4 words) accompanying a visual |
| Code block body      | Fira Code | 30–36px      | 400    | syntax-highlighted     | Left-aligned; ~50–60 chars per line at 32px |
| Diagram labels       | Rubik     | 28–34px      | 400    | `#545E74`              | Node labels, axis ticks, timeline years |
| Outro CTA            | Rubik     | 48–56px      | 400    | `#545E74`              | Topic name returns here at 52px |

**Font ligatures must be enabled for code blocks.**  
`=>`, `!=`, `>=` render as ligatures — non-negotiable for authenticity.

**Line height:** 1.1× for titles, 1.2× for body kinetic text, 1.5× for code blocks.  
**Letter spacing:** `−0.02em` on titles (tighter = more cinematic), `0` on body and code.  
**Never let kinetic body text span the full 1760px safe-zone width** — cap at 1400px max-width and center it.

### Kinetic Text Engine

Text appears **word-by-word**, synchronized to within ~2 frames of the audio. Reserved for: topic title, key terms/numbers, and the outro — not for narrating every sentence.

- **Entrance:** Instant pop-in (0 frames) or 3-frame opacity snap — no dissolves
- **Exit:** Hard cut to next line — never fade out
- **Keyword pulse:** Keyword scales 110–115% for 4 frames then settles
- **Color shifts:** Keywords render in syntax/accent colors inline; rest stays `#ABB2BF`
- **Sentence limit:** Max 2 full sentences of kinetic text per scene with no other visual

### Code Animation

- **Reveal speed:** 3 characters per frame at 60fps (~180 chars/sec) — readable as it types, not a blink
- **Syntax highlighting:** Active from character 1 — never render plain then colorize
- **Line-by-line timing:** Each line appears as the VO describes it — tight sync
- **Active line highlight:** `rgba(255,255,255,0.05)` background bar tracks current line
- **Cursor:** Blinking block cursor visible while typing, disappears when line completes
- **Panel style:** Rounded rect `8px`, background `#282C34`, no title bar chrome

### Meme / Image Inserts

Use the **meme-factory skill** to generate memes via memegen.link. Always web-search for the most contextually relevant reference first.

**Minimum 2 images per video.** Not a suggestion.

- **Frequency:** 2–4 per video — humor and visual proof points, not just punchlines
- **Timing:** Images don't have to wait for a joke beat. Drop one on a surprising fact, a chaotic situation, a named person, or anywhere the content benefits from a visual anchor
- **Duration:** 20–120 frames — a quick smash-cut stays 20–33f; a contextual panel can persist for seconds
- **Never use a generic meme** — the image must directly reinforce the specific moment in the VO

**Layout freedom — place images with personality:**

Images do not need to sit in a clean box. Think of them as visual objects dropped onto the canvas:

| Style | CSS | When to use |
|:---|:---|:---|
| **Full frame** | `width:100%, height:100%, objectFit:cover` | Pure humor smash-cut |
| **Split panel** | `flex: 0 0 45%` alongside text | Image reinforces ongoing content |
| **Tilted card** | `transform: rotate(±3–6deg)` | Casual, energetic — good for memes |
| **Floating inset** | `position:absolute`, corner or edge, 30–45% width | Doesn't interrupt main visual |
| **Oversized bleed** | Extends past safe zone intentionally | Dramatic emphasis, rare |
| **Small accent** | 20–25% width, beside a keyword | Logo/avatar next to a named concept |

**Rules for free placement:**
- The image must not obscure the code block, active kinetic text, or the key diagram element
- A slight rotation (±3–6°) adds energy — use on memes, casual context photos; not on logos or UI screenshots
- Drop shadows (`box-shadow: 0 8px 32px rgba(0,0,0,0.6)`) ground floating images on the dark canvas
- `border-radius: 8–12px` on all non-full-frame images

### Real Images & Screenshots

This is the most underused visual type. Use it aggressively:

- **People:** Any named person in the VO gets their photo shown. Web-search GitHub avatar, Twitter profile, or company headshot. Crop square. Show alongside their name.
- **Products/tools:** Any named tool, framework, or product gets its logo or screenshot. Source from official site or Simple Icons.
- **UI/website:** When the VO references a site or app, show a cropped screenshot. Record at 2× Retina. Use `ffmpeg` or macOS screenshot if needed.
- **Download all images** to `public/images/[topic-name]/` before building. Never hot-link.

### Diagrams & Graphics

Diagrams are **mandatory** for any comparison, architecture, benchmark, or multi-step flow. If you're writing kinetic text for something that could be a diagram, it's a diagram.

- **Style:** Flat, minimal. Nodes draw in left-to-right or top-to-bottom as the VO names them
- **Timelines:** Year labels in `#D19A66` amber, connector in `#545E74` gray, dot at event — always shown alongside real photos of the era/person if available
- **Bar charts:** Fill from bottom, bars labeled; topic/success → `#98C379`, others → `#545E74` — use for any speed, size, or adoption comparison
- **Pipelines:** Boxes connected by `→`, each box draws in with the VO beat that describes it
- **Library/feature grids:** Colored badge per item, highlights the active one as the VO names it
- **Icon rows:** Framework logos side by side for compatibility lists — use actual SVG logos, not text

### Visual Polish (Always On)

Both effects applied globally as `<AbsoluteFill>` compositor layers on top of all scenes:

- **Vignette:** Radial gradient from transparent center to `rgba(0,0,0,0.55)` at edges. `<VignetteOverlay strength={0.55} />` from `src/fireship-shared/`
- **Film grain:** SVG `feTurbulence` seeded by `useCurrentFrame()` for per-frame variation, opacity 0.04. `<FilmGrain opacity={0.04} />` from `src/fireship-shared/`

### Transitions

| Type | Usage | Timing |
|:---|:---|:---|
| **Hard cut** | Default between every sentence/beat | 0 frames |
| **Smash cut** | Into humor beat or unexpected fact | 0 frames |
| **Instant pop** | New text block arriving | 0–2 frames |
| **Cross-fade** | Never | — |
| **Slide wipe** | Never | — |
| **Zoom** | Rare — push into a key code line only | 8–12 frame lerp |

---

## Audio Design

### Voice
- **Tone:** Deadpan, matter-of-fact, slightly sardonic — like a professor who's seen too much
- **Pace:** 180–220 WPM — significantly faster than a standard tutorial
- **Pauses:** 3-frame gaps between sentences only. No dramatic pauses.
- **Humor delivery:** Flat. The joke is funnier when the voice doesn't acknowledge it.
- **Energy:** Consistent and even throughout — no ramp-up, no big finish

### TTS Voice Personality

The voice must sound like a professor who has seen too much: deadpan, dry, never warm.

| Provider | Recommended Voice | Notes |
|:---|:---|:---|
| ElevenLabs | George | Authoritative, dry — first choice |
| ElevenLabs (alt) | Daniel | British, wry — good second option |
| macOS | `Daniel -r 270` | Calibrated for ~57-58s per 130-word script |
| OpenAI TTS | `onyx` at 1.10× | Deep and measured |

> **Avoid warm, energetic, or "podcast host" voices.** The joke is funnier when the voice doesn't acknowledge it's a joke.

For provider setup, API keys, recording commands, and duration calibration see `tts-providers.md`.

### Duration (CRITICAL)

**The title says N seconds. The video must be N seconds.**

Word count budget — count before recording. If over, cut sentences:

| Target | Max VO words | Notes |
|:---|---:|:---|
| 60 seconds | ~130 words | Leaves ~3s for meme beats + gaps |
| 100 seconds | ~220 words | Leaves ~5s for meme beats + gaps |
| 120 seconds | ~265 words | Leaves ~5s for meme beats + gaps |

### Recording Workflow
Record **sentence-by-sentence** — one WAV file per sentence. Sequence with 3-frame gaps. This replicates Jeff's actual Premiere Pro workflow of recording phrase-by-phrase and razor-cutting to near-zero gaps.

### Export Settings
- Format: WAV
- Sample Rate: 48kHz
- Bit Depth: 24-bit
- Channels: Stereo

### Audio Levels

| Source | Level | Notes |
|:---|:---|:---|
| Narration | 1.0 | The only thing that matters |
| Background music | 0.05–0.07 | Subtle ambient synth only — often absent entirely |
| UI/code SFX | 0 | Fireship uses none |
| Humor beat SFX | Optional, 0.25 | One single sound effect maximum |

**No marimba. No upbeat corporate music. No SFX on every click.**

---

## Script Format

```
[00:00-00:05] TITLE HOOK
Scene: Dark canvas. Topic name slams in at frame 0.
VO:    "[Topic] — [one punchy descriptor]."
VISUAL: Topic name in Rubik 140–160px Black (900) white, centered, tracking −0.02em
KINETIC: Each word pops in 2 frames after the previous
[Words: 12–18 | ~5s]

[00:05-00:20] ORIGIN / HISTORY (1–3 sentences MAX)
Scene: Horizontal timeline graphic or dark canvas + year callout
VO:    "It was [created] in [year] by [person] — [ironic backstory]."
VISUAL: Year in D19A66 amber; name in 61AFEF blue
KINETIC: Phrase-by-phrase, keywords colored
OPTIONAL: Meme insert on absurd origin detail (20 frames)
[Words: 25–40 | ~10s]

[00:20-01:00] CORE CONCEPTS (3 beats)
Scene: Diagram / chart / code block for each beat — kinetic text only for labels
VO:    "[Concept 1]. [Concept 2]. [Concept 3]." — 1–2 sentences each
VISUAL: New diagram draws in with each VO beat; screen changes every 4–8s
KINETIC: Keywords and numbers only — no full sentence transcription
RHYTHM: If screen hasn't changed in 6 seconds, cut something
[Words: 80–120 | ~30s]

[01:00-01:40] CODE WALKTHROUGH
Scene: SyntaxBlock panel, dark bg, code types in at 3 chars/frame
VO:    "To get started, [command]. [Key API] does [thing]."
CODE:  Lines reveal in sync with VO — one concept per line
HIGHLIGHT: Active line tinted; named elements flash accent color
ZOOM:  Optional single push-in to key line (10-frame lerp)
[Words: 60–100 | ~25s]

[01:40-END] OUTRO
Scene: Dark canvas, topic name returns at 52px
VO:    "This has been [Topic] in [N] seconds.
        [One dry opinion/joke.]
        Thanks for watching and I will see you in the next one."
VISUAL: Text dims; end card region fades in
[Words: 25–35 | ~10s]
```

### Humor Injection Rules

Fireship's humor is **dry, not performed:**

- **The absurd detail:** Historical fact stated flatly. *"He built it over Christmas."*
- **The understatement:** Wild thing described as mundane. *"This adds a 280MB Chrome."*
- **The meme frame:** 20–33 frames on a surprising fact. Immediately back to content.
- **One joke per video.** Two is pushing it. Three is a comedy channel.

---

## Remotion Configuration

```typescript
// Root.tsx
<Composition
  id="FireshipExplainer"
  component={FireshipExplainer}
  durationInFrames={AUDIO_CONFIG.totalFrames}  // from audioTimings.ts
  fps={60}
  width={1920}
  height={1080}
/>
```

### Project Structure

```
public/
  audio/[topic-name]/
    vo_01.wav … vo_N.wav   ← one file per sentence
    AUDIO_TIMING.md        ← frame-accurate timings
  memes/                   ← downloaded meme images (staticFile)
src/
  [topic-name]/
    audioTimings.ts        ← AUDIO_CONFIG, SECTION_TIMINGS, VO_TIMINGS, THEME
    [Topic]Explainer.tsx   ← main composition
    scenes/
      Scene1_TitleHook.tsx
      Scene2_Origin.tsx
      Scene3_CoreConcept.tsx  ← one per major concept beat
      SceneN_Code.tsx
      SceneN_Outro.tsx
    components/            ← video-specific components
  fireship-shared/
    VignetteOverlay.tsx    ← always import, always render
    FilmGrain.tsx          ← always import, always render
    KineticLine.tsx        ← word-by-word text (shared across all Fireship videos)
    SyntaxBlock.tsx        ← animated syntax-highlighted code
```

### Key Components

**KineticLine** — words pop in sequentially, keywords colored:
```tsx
<KineticLine
  words={[
    { text: "Built" },
    { text: "in", },
    { text: "Zig", color: "#C678DD", bold: true, pulse: true },
  ]}
  startFrame={40}
  intervalFrames={18}  // frames between each word appearing
  fontSize={60}        // 56–68px for body; 72–88px for keyword-heavy callout lines
/>
```

**SyntaxBlock** — code types in with live syntax highlighting:
```tsx
<SyntaxBlock
  lines={[
    {
      code: "const server = Bun.serve({ port: 3000 });",
      startFrame: 120,
      tokens: [
        { text: "const ", color: "#C678DD" },
        { text: "server ", color: "#ABB2BF" },
        { text: "= ", color: "#ABB2BF" },
        { text: "Bun", color: "#61AFEF" },
        { text: ".serve({ port: ", color: "#ABB2BF" },
        { text: "3000", color: "#D19A66" },
        { text: " });", color: "#ABB2BF" },
      ],
    },
  ]}
  charsPerFrame={3}
  fontSize={32}        // 30–36px for code panels on 1920×1080
  showCursor
/>
```

**VignetteOverlay + FilmGrain** — always on, in every composition:
```tsx
// In [Topic]Explainer.tsx — after all <Series> content
<VignetteOverlay strength={0.55} />
<FilmGrain opacity={0.04} />
```

**VO audio placement** — individual files, each at their global start frame:
```tsx
{VO_TIMINGS.map(({ file, startFrame }) => (
  <Sequence key={file} from={startFrame}>
    <Audio src={staticFile(`audio/[topic]/${file}`)} volume={1.0} />
  </Sequence>
))}
```

### `audioTimings.ts` structure

```typescript
export const AUDIO_CONFIG = {
  fps: 60,
  totalDuration: 59.07,   // measured from actual WAV files
  totalFrames: 3544,      // totalDuration × fps
} as const;

// Scene boundaries — used for Series.Sequence durations
export const SECTION_TIMINGS = {
  titleHook: { startFrame: 0, endFrame: 363, duration: 364 },
  // ...
} as const;

// Individual VO file placements — used for <Audio> tracks
export const VO_TIMINGS = [
  { file: "vo_01.wav", startFrame: 0   },
  { file: "vo_02.wav", startFrame: 364 },
  // ...
] as const;

// Atom One Dark design tokens
export const THEME = { bg: {...}, text: {...}, syntax: {...}, accent: {...} } as const;
```

---

## Tailwind Library Reference

Tailwind is loaded globally via `src/index.css`. Use `className` for all static visual
properties. Use `style={{}}` only for values driven by `useCurrentFrame()`.

### Color Tokens

| Element | className | Hex |
|:---|:---|:---|
| Canvas background | `bg-fire-canvas` | `#1E2127` |
| Code editor bg | `bg-fire-code` | `#282C34` |
| Primary text | `text-fire-text` | `#ABB2BF` |
| White emphasis | `text-fire-white` | `#FFFFFF` |
| Brand accent (fire) | `text-fire-accent` / `bg-fire-accent` | `#FF6B35` |
| Amber accent | `text-fire-amber` | `#FFB347` |
| Dim / diagram | `text-fire-dim` | `#545E74` |
| Error / warning | `text-fire-error` | `#E06C75` |
| Syntax: keyword | `text-fire-keyword` | `#C678DD` |
| Syntax: string | `text-fire-string` | `#98C379` |
| Syntax: number | `text-fire-number` | `#D19A66` |
| Syntax: function | `text-fire-fn` | `#61AFEF` |
| Syntax: comment | `text-fire-comment` | `#5C6370` |

### Typography Tokens

| Role | className |
|:---|:---|
| Opening topic title (140–160px) | `text-slammer font-black tracking-[-0.02em]` |
| Keyword callout (72–88px) | `text-headline font-bold` |
| Kinetic body text (56–68px) | `text-display font-normal` |
| Section label / term (48–56px) | `text-title font-semibold` |
| Code block body (30–36px) | `text-code font-fira` |
| Diagram labels (28–34px) | `text-diagram text-fire-dim` |
| Outro CTA (48–56px) | `text-heading text-fire-dim` |

### Scene Layout Patterns

```tsx
// ── Standard scene: full dark canvas, centered content
<AbsoluteFill className="bg-fire-canvas scene-center">
  <div className="scene-safe scene-center gap-8">
    <h1 className="text-slammer font-black text-fire-white tracking-[-0.02em]"
        style={{ opacity: interpolate(frame, [0, 3], [0, 1]) }}>
      Bun.
    </h1>
  </div>
</AbsoluteFill>

// ── Split screen: code left, diagram right
<AbsoluteFill className="bg-fire-canvas">
  <div className="scene-safe split-screen h-full items-center">
    <div className="split-screen-left">
      <div className="code-panel">
        <SyntaxBlock ... />
      </div>
    </div>
    <div className="split-screen-right scene-center gap-6">
      {/* diagram content */}
    </div>
  </div>
</AbsoluteFill>

// ── Floating image inset (meme/photo)
<div className="image-card absolute right-16 top-1/2 w-[420px]"
     style={{ transform: `rotate(${rotation}deg) translateY(${y}px)` }}>
  <Img src={staticFile("images/meme.jpg")} />
</div>
```

### `audioTimings.ts` THEME is still the source of truth for JS

The `THEME` constant in `audioTimings.ts` remains the canonical reference when
color values are needed as **JavaScript strings** (e.g. inline `style={}` for
animated values, or as props to `<KineticLine color={...} />`). Tailwind classes
cover static `className` usage; THEME covers dynamic/animated usage.

```typescript
// Use THEME values when passing colors as JS props or animated style values
<KineticLine words={[{ text: "Zig", color: THEME.syntax.keyword }]} />
// Use Tailwind when applying static layout/color to a container
<div className="bg-fire-canvas text-fire-text text-code font-fira">
```

---

## Render Instructions

```bash
# Preview (hot reload)
npm start

# Test render — hook only (first scene)
npx remotion render [TopicExplainer] out/[topic]-test.mp4 --frames=0-363

# Full render
npx remotion render [TopicExplainer] out/[topic].mp4 \
  --codec=h264 \
  --fps=60 \
  --crf=14 \
  --audio-codec=aac \
  --audio-bitrate=320k
```

**Output:** 1920×1080, H.264 CRF 14, AAC 320kbps, 60fps

> CRF 14 (not 16) is required — kinetic text motion reveals compression artifacts at CRF 16+.
