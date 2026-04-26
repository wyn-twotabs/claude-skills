# Style Guide: Instructional Software Video

**Format:** 1920×1080 (16:9 widescreen)
**Target Duration:** 2–5 minutes
**Frame Rate:** 30fps (UI animations match real screen recordings)
**Use Case:** Showcasing software built for clients; onboarding, walkthroughs,
feature demos

---

## Visual Style

### Tone & Aesthetic
- **Professional but approachable:** Clean, modern, trustworthy
- **Brand-forward:** Use client's or your studio's brand palette
- **Screen-accurate:** UI recordings should be crisp, not stylized
- **Minimal decoration:** Let the software speak — don't compete with the UI

### Layout Principles
- **Safe Zones:** Keep key content within inner 1760×960 (40px margin each side)
- **Lower Thirds:** Name/title cards at bottom 20% of frame
- **Side-by-side Layouts:** Allowed for before/after or comparison views
- **Full-screen App:** Preferred for core demo segments

### Typography & Text
- **Font:** Inter or your brand typeface
- **Heading:** Bold, 48–56px, white or dark depending on background
- **Body/Captions:** Regular, 24–28px
- **Lower Third Name:** 32px bold, accent color
- **Lower Third Title:** 22px regular, white/light gray

### Color Palette (Defaults — override with client brand)

| Element | Value |
|---|---|
| Background (slides/titles) | `#0F1117` near-black |
| Primary Text | `#FFFFFF` white |
| Secondary Text | `#A0A8B8` cool gray |
| Accent / CTA | `#4F8EF7` brand blue |
| Success / Positive | `#3ECF8E` green |
| Warning / Note | `#F5A623` amber |
| Screen Overlay Tint | `rgba(15,17,23,0.72)` |

### On-Screen Annotations
- **Click Indicators:** Animated circular pulse (2px border, accent color)
- **Highlight Boxes:** Rounded rect, 2px accent border, subtle fill
- **Arrows:** Simple, flat, accent color — no drop shadows
- **Zoom Regions:** Smooth lerp zoom to 200% on key UI elements
- **Tooltips:** Dark background, white text, 8px border radius

### Transitions
- **Cut:** Default between segments
- **Cross-fade (12 frames):** Between major sections
- **Slide Wipe:** Avoid — too informal
- **Zoom In/Out:** Use for focus moments only

---

## Audio Design

### Voice
- Tone: Clear, authoritative, professional — like a senior colleague explaining
- Pace: 140–160 WPM (slower than Shorts — viewers need time to follow UI)
- Energy: Calm and confident, not sales-y

### TTS Voice Personality

Professional and measured — like a senior colleague walking you through something. Never sales-y.

| Provider | Recommended Voice | Notes |
|:---|:---|:---|
| ElevenLabs | Adam | Professional, neutral, trustworthy |
| ElevenLabs (alt) | Rachel | Clear, composed, corporate-friendly |
| macOS | `Reed (English (US)) -r 150` | Measured and authoritative |
| OpenAI TTS | `onyx` or `shimmer` at 0.90× | Calm, clear delivery |

For provider setup, API keys, recording commands, and calibration see `tts-providers.md`.

### Export Settings
- Format: WAV
- Sample Rate: 48kHz
- Bit Depth: 24-bit
- Channels: Stereo (for widescreen playback)

### Audio Levels
- Narration: 100% (1.0)
- Background Music: 8% (0.08) — subtle ambient/lo-fi, NEVER marimba
- UI Sound Effects: 20% (0.20) — soft clicks matching on-screen interactions
- Silence between steps: 0.5–1.0s natural pauses for viewer to process

---

## Script Format

```
[00:00-00:15] TITLE CARD / INTRO
Scene: Studio logo + project title on dark background
Voiceover: "In this video, we'll walk through [feature/product name]
            and how it solves [client problem]."
[VISUAL: Animated title reveal, logo fade-in]
[NO SFX]
[Words: ~28 | 150 WPM = ~11s]

[00:15-00:40] CONTEXT / PROBLEM STATEMENT
Scene: Slide or simple graphic explaining the problem
Voiceover: "Before [product], the team had to [pain point]..."
[VISUAL: Simple diagram or before-state screenshot]
[HIGHLIGHT: Key pain point phrase]
[Words: ~50 | 150 WPM = ~20s]

[00:40-XX:XX] FEATURE WALKTHROUGH (repeating block per feature)
Scene: Full-screen app recording
Voiceover: "Here, you can see [feature]. Let's [action]."
[ZOOM: Smooth zoom to relevant UI element]
[ANNOTATE: Click pulse + highlight box on interactive element]
[PAUSE: 0.75s after each action for viewer to follow]
[Words: ~30 per feature block | 150 WPM = ~12s per block]

[XX:XX-END] SUMMARY + CTA
Scene: Title card or final app overview
Voiceover: "That's [product]. It gives [client] the ability to [benefit]."
[VISUAL: Feature list animates in one-by-one]
[CTA: "Learn more at [URL]" lower third]
[Words: ~35 | 150 WPM = ~14s]
```

---

## Screen Recording Guidelines

- **Resolution:** Record at 1920×1080 or 2× Retina then downscale
- **Cursor:** Use a large, visible cursor (size 2× in macOS Accessibility)
- **Browser Zoom:** Set app UI to 100% zoom before recording
- **Clean Environment:** Hide bookmarks bar, close unrelated tabs, use
  a clean desktop/wallpaper
- **Pacing:** Move deliberately — pause 0.5–1s before and after each click
- **No Typing Errors:** Script interactions in advance, record clean takes
- **Tool:** OBS Studio or macOS Screenshot (`⌘+Shift+5`)

---

## Remotion Configuration

```typescript
// Root.tsx
<Composition
  id="InstructionalVideo"
  component={InstructionalVideo}
  durationInFrames={totalFrames}   // from AUDIO_TIMING.md
  fps={30}
  width={1920}
  height={1080}
/>
```

### Project Structure

```
/instructional-video
├── public/
│   ├── audio/
│   │   ├── full_narration.wav
│   │   ├── bg_ambient.mp3
│   │   └── sfx/ (click.wav, whoosh.wav)
│   ├── recordings/           ← screen recording segments (.mp4)
│   └── slides/               ← static slide assets (.png)
├── src/
│   ├── Root.tsx
│   ├── InstructionalVideo.tsx
│   ├── audioTimings.ts
│   ├── scenes/
│   │   ├── Scene_TitleCard.tsx
│   │   ├── Scene_ProblemStatement.tsx
│   │   ├── Scene_FeatureDemo.tsx     ← accepts recording segment + annotations
│   │   └── Scene_Summary.tsx
│   └── components/
│       ├── LowerThird.tsx
│       ├── ClickIndicator.tsx
│       ├── HighlightBox.tsx
│       ├── ZoomRegion.tsx
│       └── AnnotationArrow.tsx
```

### Key Components

**ClickIndicator** — animated pulse ring on click events:
```tsx
// Ripple effect at click position, synced to audio timestamp
<ClickIndicator x={960} y={540} triggerFrame={450} color="#4F8EF7" />
```

**ZoomRegion** — smooth lerp zoom to a UI area:
```tsx
// Zooms from full-frame to a 400×300 region centered on x,y
<ZoomRegion
  targetX={800} targetY={400}
  zoomWidth={400} zoomHeight={300}
  startFrame={300} endFrame={420}
/>
```

**LowerThird** — name/title card:
```tsx
<LowerThird
  name="Invoice Module"
  subtitle="Client: Acme Corp"
  accentColor="#4F8EF7"
  triggerFrame={60}
/>
```

---

## Tailwind Library Reference

Tailwind is loaded globally via `src/index.css`. Use `className` for all static visual
properties. Use `style={{}}` only for values driven by `useCurrentFrame()`.

### Color Tokens

| Element | className | Hex |
|:---|:---|:---|
| Background (slides/titles) | `bg-instr-canvas` | `#0F1117` |
| Primary text | `text-white` | `#FFFFFF` |
| Secondary text | `text-instr-secondary` | `#A0A8B8` |
| Accent / CTA | `text-instr-accent` / `bg-instr-accent` | `#4F8EF7` |
| Success / positive | `text-instr-success` | `#3ECF8E` |
| Warning / note | `text-instr-warning` | `#F5A623` |
| Screen overlay tint | use `style={{ background: 'rgba(15,17,23,0.72)' }}` | — |

> Screen overlay tint requires `rgba()` — no Tailwind token, always inline.

### Typography Tokens

| Role | className |
|:---|:---|
| Heading (48–56px) | `text-title font-bold` |
| Body / captions (24–28px) | `text-body font-normal` |
| Lower third — name (32px) | `text-subhead font-bold text-instr-accent` |
| Lower third — title (22px) | `text-caption font-normal text-white` |

### Scene Layout Patterns

```tsx
// ── Title card scene
<AbsoluteFill className="bg-instr-canvas scene-center">
  <div className="scene-safe scene-center gap-6">
    <h1 className="text-title font-bold text-white text-center"
        style={{ opacity: interpolate(frame, [0, 12], [0, 1]) }}>
      Feature Title
    </h1>
    <p className="text-body text-instr-secondary text-center">
      Subtitle or context line
    </p>
  </div>
</AbsoluteFill>

// ── Lower third overlay — anchored to bottom 20% of frame
<div className="lower-third">
  <div className="flex flex-col gap-1">
    <span className="text-subhead font-bold text-instr-accent"
          style={{ opacity: interpolate(frame, [0, 8], [0, 1]) }}>
      Invoice Module
    </span>
    <span className="text-caption font-normal text-white">
      Client: Acme Corp
    </span>
  </div>
</div>

// ── Annotation highlight box — static border, animated opacity/scale
<div className="absolute rounded-lg border-2 border-instr-accent"
     style={{
       left: x, top: y, width: w, height: h,
       opacity: interpolate(frame, [triggerFrame, triggerFrame + 6], [0, 1]),
     }} />

// ── Side-by-side comparison layout
<AbsoluteFill className="bg-instr-canvas">
  <div className="scene-safe split-half h-full items-center gap-8">
    <div className="split-half rounded-lg overflow-hidden">
      {/* before state */}
    </div>
    <div className="split-half rounded-lg overflow-hidden">
      {/* after state */}
    </div>
  </div>
</AbsoluteFill>
```

---

## Render Instructions

```bash
# Preview
npm start

# Full render
npx remotion render InstructionalVideo out/final.mp4 \
  --codec=h264 \
  --crf=16 \
  --audio-codec=aac \
  --audio-bitrate=320k

# Test intro
npx remotion render InstructionalVideo out/test.mp4 --frames=0-150
```

**Output:** 1920×1080 MP4, H.264 CRF 16 (higher quality for client delivery),
AAC 320kbps
