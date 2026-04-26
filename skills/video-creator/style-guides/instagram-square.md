# Style Guide: Instagram Square Video

**Format:** 1080×1080 (1:1 square)
**Target Duration:** 30–60 seconds
**Frame Rate:** 30fps (animations at 12fps)
**Use Case:** Software news snippets, quick tech updates, feed posts

---

## Visual Style

### Tone & Aesthetic
- **Bold and scroll-stopping:** Must grab attention in a busy feed
- **Opinionated typography:** The headline IS the visual
- **News-forward:** Feels like a punchy tech headline, not an ad
- **High information density:** Pack in the key fact fast

### Layout Principles
- **Safe Zone:** Inner 960×960 (60px margin all sides) — Instagram crops edges
- **Typography-led:** Lead with a large bold statement, visuals support it
- **Grid-based:** 3-column implied grid for icon/stat layouts
- **Bottom Bar:** Reserved for source attribution or CTA (last 80px)

### Typography & Text
- **Font:** Inter or Space Grotesk
- **Headline:** Black/ExtraBold weight, 72–88px, centered or left-aligned
- **Subhead / Context:** SemiBold, 32–36px, secondary color
- **Data Callout:** 96–120px for a single key stat (makes it shareable)
- **Source Tag:** 20px regular, muted color, bottom bar

### Color Palette

| Scene Type | Background | Headline | Accent |
|---|---|---|---|
| Breaking news / major update | `#0D0D0D` black | `#FFFFFF` white | `#FF3B30` red |
| Product launch / positive news | `#0A1628` deep navy | `#FFFFFF` white | `#4F8EF7` blue |
| Stats / data | `#1A1A2E` dark indigo | `#FFFFFF` white | `#F5A623` amber |
| Opinion / take | `#F5F0E8` off-white | `#1A1208` dark | `#D4900A` amber |
| Tool / resource | `#0D1F0D` dark green | `#EFFFEF` light | `#3ECF8E` green |

### Design Patterns
- **Stat Cards:** Single large number + short label, centered with color block
- **Headline Scroll:** 2–3 word chunks animate in with snappy timing
- **Logo Lockup:** Software/company logo + headline — clean, no clutter
- **Progress Bar:** Thin accent-color bar fills at bottom as video plays
- **Tag Clouds:** Keywords appear and arrange themselves (12fps, springy)

### Effects
- **Grain Overlay:** Light noise texture (opacity 0.04) for premium feel
- **Edge Glow:** Subtle inner glow on dark backgrounds (not a hard border)
- **Text Entrance:** Words slide up 24px + fade in, staggered 4 frames apart
- **No Chromatic Aberration:** Too noisy at square format — skip it

---

## Audio Design

### Voice
- Tone: Punchy, direct, confident — like a news anchor's teaser
- Pace: 170–190 WPM (fast — matches short format and feed scroll energy)
- Energy: High, clipped, no fluff

### TTS Voice Personality

Punchy and direct — like a news anchor's teaser. High energy, no fluff.

| Provider | Recommended Voice | Notes |
|:---|:---|:---|
| ElevenLabs | Callum | Crisp, assertive, punchy |
| ElevenLabs (alt) | Charlotte | Confident, dynamic |
| macOS | `Flo (English (US)) -r 185` | Fast and clipped |
| OpenAI TTS | `echo` or `fable` at 1.05× | Sharp delivery |

For provider setup, API keys, recording commands, and calibration see `tts-providers.md`.

### Export Settings
- Format: WAV
- Sample Rate: 48kHz
- Bit Depth: 24-bit
- Channels: Mono (Instagram normalizes audio anyway)

### Audio Levels
- Narration: 100% (1.0)
- Background Music: 0% — **no background music by default**
  (Instagram auto-mutes video on feed; design for silent viewing first)
- SFX: 30% (0.30) — one punchy "thud" or "tick" per stat reveal
- **Caption Subtitle Track:** Always generate — most IG views are muted

### Silent-First Design Rule
Because most Instagram videos autoplay muted, every key message must
be readable on screen without audio. All voiceover content must have
a corresponding on-screen text element.

---

## Script Format

```
[00:00-00:03] HOOK HEADLINE
Scene: Single bold statement on solid background
Voiceover: "[Shocking stat or bold claim]"
[VISUAL: Headline animates in word-by-word at 12fps]
[SUBTITLE: Full text on screen]
[SFX: low thud on impact]
[Words: ~8 | 180 WPM = ~3s]

[00:03-00:20] CONTEXT
Scene: Supporting visual or stat card
Voiceover: "[Explains the headline in 2–3 sentences]"
[VISUAL: Stat card or logo lockup]
[HIGHLIGHT: Key number or company name]
[SUBTITLE: Full text on screen]
[Words: ~35 | 180 WPM = ~12s]

[00:20-00:45] THE STORY / DETAIL
Scene: 2–3 quick visual beats
Voiceover: "[What happened, what it means, what's next]"
[VISUAL: Sequence of bold text cards or icon grid]
[ANIMATE: Each beat at 12fps with spring entrance]
[SUBTITLE: Full text on screen]
[Words: ~65 | 180 WPM = ~22s]

[00:45-01:00] TAKEAWAY + SOURCE
Scene: Final headline reprise or "What this means for you"
Voiceover: "[One-line takeaway]"
[VISUAL: Bold text + source attribution in bottom bar]
[SUBTITLE: Full text on screen]
[SFX: tick on final word]
[Words: ~22 | 180 WPM = ~7s]
```

---

## Subtitle / Caption Guidelines

Generate a subtitle layer as a separate Remotion component:

```tsx
<SubtitleTrack
  captions={captionsFromAudioTimings}
  style="bottom-center"           // bottom 200px, centered
  fontSize={36}
  background="rgba(0,0,0,0.6)"   // pill background
  color="#FFFFFF"
  maxCharsPerLine={32}
/>
```

Export a `.srt` file alongside the video for manual upload to Instagram.

---

## Remotion Configuration

```typescript
// Root.tsx
<Composition
  id="InstagramSquare"
  component={InstagramSquare}
  durationInFrames={totalFrames}   // from AUDIO_TIMING.md
  fps={30}
  width={1080}
  height={1080}
/>
```

### Project Structure

```
/instagram-square
├── public/
│   ├── audio/
│   │   ├── full_narration.wav
│   │   └── sfx/ (thud.wav, tick.wav)
│   └── logos/                  ← company/product logos (.svg or .png)
├── src/
│   ├── Root.tsx
│   ├── InstagramSquare.tsx
│   ├── audioTimings.ts
│   ├── scenes/
│   │   ├── Scene_Hook.tsx
│   │   ├── Scene_Context.tsx
│   │   ├── Scene_Story.tsx
│   │   └── Scene_Takeaway.tsx
│   └── components/
│       ├── HookHeadline.tsx      ← word-by-word animated headline
│       ├── StatCard.tsx          ← large number + label
│       ├── LogoLockup.tsx
│       ├── SubtitleTrack.tsx     ← always-on caption layer
│       ├── ProgressBar.tsx       ← bottom fill bar
│       └── GrainOverlay.tsx
```

### Key Components

**HookHeadline** — word-by-word spring entrance at 12fps:
```tsx
<HookHeadline
  words={["GitHub", "just", "broke", "records"]}
  startFrame={0}
  color="#FFFFFF"
  fontSize={88}
/>
```

**StatCard** — large data callout:
```tsx
<StatCard
  value="$29B"
  label="Valuation in 2025"
  accentColor="#F5A623"
  triggerFrame={90}
/>
```

---

## Render Instructions

```bash
# Preview
npm start

# Full render
npx remotion render InstagramSquare out/final.mp4 \
  --codec=h264 \
  --crf=18 \
  --audio-codec=aac \
  --audio-bitrate=192k

# Export SRT captions (custom script)
node scripts/export-srt.js

# Test
npx remotion render InstagramSquare out/test.mp4 --frames=0-90
```

**Output:** 1080×1080 MP4, H.264, AAC 192kbps + `.srt` caption file

## Tailwind Library Reference

Tailwind is loaded globally via `src/index.css`. Use `className` for all static visual
properties. Use `style={{}}` only for values driven by `useCurrentFrame()`.

### Color Tokens — 5 News Palettes

Pick the palette that matches the scene type. All dark-bg scenes use `text-white` for
headline; the opinion scene flips to `text-ig-opinion-text`.

| Scene type | bg className | headline | accent className |
|:---|:---|:---|:---|
| Breaking news | `bg-ig-breaking-bg` | `text-white` | `text-ig-breaking` |
| Product launch | `bg-ig-launch-bg` | `text-white` | `text-ig-launch` |
| Stats / data | `bg-ig-stats-bg` | `text-white` | `text-ig-stats` |
| Opinion / take | `bg-ig-opinion-bg` | `text-ig-opinion-text` | `text-ig-opinion` |
| Tool / resource | `bg-ig-tool-bg` | `text-ig-tool-text` | `text-ig-tool` |

### Typography Tokens

| Role | className |
|:---|:---|
| Data callout (96–120px) | `text-hero font-black` |
| Headline (72–88px) | `text-headline font-black` |
| Subhead / context (32–36px) | `text-subhead font-semibold` |
| Source tag (20px) | `text-label font-normal` |

### Scene Layout Patterns

```tsx
// ── Breaking news hook: full-bleed dark, centered headline
<AbsoluteFill className="bg-ig-breaking-bg">
  <div className="scene-safe-square scene-center h-full gap-6">
    <h1 className="text-headline font-black text-white text-center leading-none"
        style={{ opacity: interpolate(frame, [0, 4], [0, 1]) }}>
      GitHub Just Broke Records
    </h1>
    <span className="text-subhead font-semibold text-ig-breaking">
      2 million repos in one day
    </span>
  </div>
</AbsoluteFill>

// ── Stat card: large number + label
<div className="stat-card">
  <span className="text-hero font-black text-ig-stats"
        style={{ transform: `scale(${scale})` }}>
    $29B
  </span>
  <span className="text-subhead font-semibold text-white">
    Valuation in 2025
  </span>
</div>

// ── Bottom bar: source attribution
<div className="bottom-bar bg-black/40">
  <span className="text-label text-white/60">Source: Bloomberg</span>
</div>

// ── Progress bar: fill driven by frame, color static
<div className="absolute bottom-[80px] left-0 right-0 h-[4px] bg-white/20">
  <div className="h-full bg-ig-stats"
       style={{ width: `${interpolate(frame, [0, totalFrames], [0, 100])}%` }} />
</div>

// ── Opinion scene: light background flip
<AbsoluteFill className="bg-ig-opinion-bg">
  <div className="scene-safe-square scene-center h-full gap-6">
    <h1 className="text-headline font-black text-ig-opinion-text text-center">
      Hot take here.
    </h1>
    <span className="text-subhead font-semibold text-ig-opinion">
      Supporting context
    </span>
  </div>
</AbsoluteFill>
```

---

### Instagram Upload Checklist
- [ ] Video is exactly 1080×1080
- [ ] Duration is 60s or under (for feed posts)
- [ ] `.srt` captions uploaded manually in Meta Business Suite
- [ ] Thumbnail frame selected (usually frame 30–60 — bold headline visible)
- [ ] Audio mix tested both with and without sound
