# Style Guide: YouTube Shorts

**Format:** 1080×1920 (9:16 vertical)
**Target Duration:** 45–60 seconds
**Frame Rate:** 30fps (animations at 6–12fps)
**Use Case:** General topics, viral/educational content, broad audiences

---

## Visual Style

### Animation & Motion
- **Low Frame Rate Animations:** Animate at 6–12fps for deliberate, intentional feel
- **Eased Movements:** Spring animations with ease curves (no linear motion)
- **Purposeful Transitions:** Every cut serves the narrative
- **Layered Textures:** Subtle noise/grain overlays for a tangible "real document" feel

### Typography & Text
- **Bold Sans-Serif Headlines:** Large, confident type for key statements
- **Highlighted Text Effect:** Amber/yellow bars wipe across text as narrator reads
- **Low Frame Rate Text Animations:** Text appears with snappy, deliberate timing
- **High Contrast:** White/black text with breathing room

### Color & Visual Design
- **Minimalist Palette:** 2–3 colors max per video
- **Flat Design:** Clean, flat graphics over 3D/glossy
- **Paper-Toned Backgrounds:** Warm or scene-tinted paper colors with crumpled texture
- **Per-Scene Background Tints:** Distinct tone per section (see palette below)
- **Highlight Color:** Amber `#D4900A` — warmer than pure yellow, reads better on texture

### Per-Scene Color Palette

| Scene | Background | Text | Accent |
|---|---|---|---|
| Hook / Conclusion | `#1A1612` dark espresso | `#F0E6D0` cream | `#D4900A` amber |
| Setup | `#C4AF92` tan kraft | `#1A1208` dark brown | `#3A6E9E` steel blue |
| Section A | `#9AAFC4` dusty blue | `#0C1820` dark navy | `#1A4F7A` deep navy |
| Section B | `#C4A092` terracotta | `#1E0C0A` dark red | `#7A1A1A` burgundy |
| Section C | `#92B49A` sage green | `#08140C` dark forest | `#1A5C2C` forest green |

### Paper Texture Implementation (Remotion)

Use `PaperBackground` component with two SVG `feTurbulence` filters:

```tsx
<PaperBackground
  color="#C4AF92"       // scene-specific paper color
  sceneId="setup"       // unique ID avoids SVG filter collisions
  grainOpacity={0.12}   // reduce to 0.08 for dark backgrounds
  crumpleOpacity={0.22}
/>
```

### Information Visualization
- **Charts & Graphs:** Animated builds with sequential data point reveals
- **Infographics:** Icons, arrows, shapes building piece-by-piece
- **Photo Integration:** Real photos with graphic overlays, zoom/pan effects

### Effects
- **Chromatic Aberration:** Slight RGB split on transitions
- **Vignette:** Subtle edge darkening
- **Fast Box Blur:** Brief blur on transitions
- **Posterize Time:** Choppy frame rate for deliberate motion

---

## Audio Design

### Voice
- Tone: Conversational, informative, clear
- Pace: 160–180 WPM
- Energy: Medium-high

### TTS Voice Personality

Conversational and engaging — like a knowledgeable friend explaining something cool.

| Provider | Recommended Voice | Notes |
|:---|:---|:---|
| ElevenLabs | Josh | Engaging, clear, energetic |
| ElevenLabs (alt) | Bella | Warm, conversational |
| macOS | `Samantha (Enhanced) -r 165` | Clear and natural |
| OpenAI TTS | `alloy` or `nova` at 0.95× | Friendly and approachable |

For provider setup, API keys, recording commands, and calibration see `tts-providers.md`.

### Export Settings
- Format: WAV
- Sample Rate: 48kHz
- Bit Depth: 24-bit
- Channels: Mono

### Audio Levels
- Narration: 100% (1.0)
- Background Music: 12% (0.12) — marimba/minimal instrumental
- SFX: 25–30% (0.25–0.3)

---

## Script Format

```
[00:00-00:03] HOOK
Scene: Clean background with bold text
Voiceover: "Ever wondered why [intriguing question]?"
[VISUAL: Photo with slow zoom]
[HIGHLIGHT: "key phrase"]
[SFX: soft pop on text entry]
[Words: ~9 | 180 WPM = ~3s]

[00:03-00:11] SETUP
Scene: Introduce main visual
Voiceover: "Here's the thing..."
[ANIMATE: 8fps bar chart build]
[SFX: click per bar]
[Words: ~21 | 180 WPM = ~7s]

[00:11-00:45] EXPLANATION
Scene: Sequence of 3–4 visuals
Voiceover: "First... then... finally..."
[VISUAL: Sequential infographic]
[ZOOM: Slow push on key detail]
[HIGHLIGHT: Data points]
[Words: ~90 | 180 WPM = ~30s]

[00:45-01:00] INSIGHT + CTA
Scene: Return to opening or satisfying conclusion
Voiceover: "So that's why [payoff]"
[ANIMATE: 6fps final graphic, snappy end]
[SFX: conclusive whoosh]
[Words: ~27 | 180 WPM = ~9s]
```

---

## Remotion Configuration

```typescript
// Root.tsx
<Composition
  id="YouTubeShort"
  component={YouTubeShort}
  durationInFrames={totalFrames}   // from AUDIO_TIMING.md
  fps={30}
  width={1080}
  height={1920}
/>
```

### Project Structure

```
/youtube-short
├── public/audio/
│   ├── full_narration.wav
│   ├── bg_music.mp3
│   └── sfx/ (pop.wav, whoosh.wav, click.wav)
├── src/
│   ├── Root.tsx
│   ├── YouTubeShort.tsx
│   ├── audioTimings.ts
│   ├── scenes/
│   └── components/
│       ├── PaperBackground.tsx
│       ├── AudioSyncedHighlight.tsx
│       ├── TextureOverlay.tsx
│       └── ChromaticShift.tsx
```

### Key Utilities
- `lowFpsFrame(frame, 8)` — posterize to 8fps for deliberate animation
- `spring({ damping: 200, stiffness: 100 })` — signature easing
- `AudioSyncedHighlight` — word-level highlight bars tied to audio timings

---

## Tailwind Library Reference

Tailwind is loaded globally via `src/index.css`. Use `className` for all static visual
properties. Use `style={{}}` only for values driven by `useCurrentFrame()`.

### Color Tokens

Five scene-specific bg/text/accent triples. PaperBackground takes a `color=""` prop —
use `var(--color-shorts-*)` to reference the token value without duplicating the hex.

| Scene | bg className | text className | accent className |
|:---|:---|:---|:---|
| Hook / Conclusion | `bg-shorts-espresso` | `text-shorts-cream` | `text-shorts-amber` |
| Setup | `bg-shorts-tan` | `text-shorts-brown` | `text-shorts-steel` |
| Section A | `bg-shorts-dusty` | `text-shorts-navy` | `text-shorts-deep` |
| Section B | `bg-shorts-terra` | `text-shorts-darkred` | `text-shorts-burgundy` |
| Section C | `bg-shorts-sage` | `text-shorts-forest` | `text-shorts-fern` |

### Typography Tokens

| Role | className |
|:---|:---|
| Bold headline | `text-hero font-black` |
| Highlighted / callout text | `text-title font-bold` |
| Body / setup text | `text-body font-normal` |
| Source / label | `text-label` |

### Scene Layout Patterns

```tsx
// ── Hook scene: dark espresso canvas, centered text
<AbsoluteFill className="bg-shorts-espresso">
  <PaperBackground
    color="var(--color-shorts-espresso)"
    sceneId="hook"
    grainOpacity={0.08}
  />
  <div className="scene-center scene-safe h-full gap-8">
    <h1 className="text-hero font-black text-shorts-cream text-center"
        style={{ opacity: interpolate(frame, [0, 6], [0, 1]) }}>
      Your headline here.
    </h1>
    <span className="text-heading font-bold text-shorts-amber">
      Key phrase
    </span>
  </div>
</AbsoluteFill>

// ── Setup scene: warm kraft paper
<AbsoluteFill className="bg-shorts-tan">
  <PaperBackground
    color="var(--color-shorts-tan)"
    sceneId="setup"
    grainOpacity={0.12}
    crumpleOpacity={0.22}
  />
  <div className="scene-center scene-safe h-full gap-6">
    <p className="text-title font-bold text-shorts-brown text-center">
      Setup content here.
    </p>
  </div>
</AbsoluteFill>

// ── Highlight wipe: animated width via style, static color via className
<div className="relative inline-block">
  <div className="highlight-bar" style={{ width: `${pct}%` }} />
  <span className="relative text-heading font-bold text-shorts-espresso">
    Highlighted text
  </span>
</div>
```

---

## Render Instructions

```bash
# Preview
npm start

# Full render
npx remotion render YouTubeShort out/final.mp4 \
  --codec=h264 \
  --crf=18 \
  --audio-codec=aac \
  --audio-bitrate=320k

# Test first 100 frames
npx remotion render YouTubeShort out/test.mp4 --frames=0-100
```

**Output:** 1080×1920 MP4, H.264, AAC 320kbps
