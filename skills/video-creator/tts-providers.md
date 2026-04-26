# TTS Provider Configuration

This file is the single source of truth for TTS setup across all video-creator style guides.
Each style guide specifies *which voices fit its tone* — this file specifies *how to use them*.

---

## Provider Selection

When Step 4 begins, always ask:

```
Which TTS provider do you want to use?

1. 🎙️  ElevenLabs  — highest quality, most natural, requires API key
2. 🍎  macOS say   — free, offline, instant, good enough for drafts
3. 🤖  OpenAI TTS  — solid quality, simple API, requires OpenAI key

Reply with a number or name.
Then confirm which voice you'd like (recommendations below per format).
```

---

## 1. ElevenLabs

### Setup

```bash
# Set your API key (add to ~/.zshrc or ~/.bash_profile to persist)
export ELEVENLABS_API_KEY="your_key_here"

# Verify it's set
echo $ELEVENLABS_API_KEY
```

Get your key at: elevenlabs.io → Profile → API Keys

### Finding Voice IDs

```bash
# List all available voices and their IDs
curl -s -H "xi-api-key: $ELEVENLABS_API_KEY" \
  "https://api.elevenlabs.io/v1/voices" | \
  python3 -c "
import json, sys
voices = json.load(sys.stdin)['voices']
for v in sorted(voices, key=lambda x: x['name']):
    print(f\"{v['name']:30} {v['voice_id']}\")
"
```

### Recommended Voices by Format

| Format | Voice Name | Tone |
|:---|:---|:---|
| Fireship | George | Authoritative, dry, slightly sardonic |
| Fireship (alt) | Daniel | British, wry, matter-of-fact |
| YouTube Shorts | Josh | Engaging, clear, energetic |
| YouTube Shorts (alt) | Bella | Warm, conversational |
| Instructional | Adam | Professional, neutral, trustworthy |
| Instructional (alt) | Rachel | Clear, composed, corporate-friendly |
| Instagram | Callum | Crisp, assertive, punchy |
| Instagram (alt) | Charlotte | Confident, dynamic |

### Voice Settings by Format

| Format | stability | similarity_boost | style | speed |
|:---|:---:|:---:|:---:|:---:|
| Fireship | 0.65 | 0.75 | 0.0 | 1.0 |
| YouTube Shorts | 0.50 | 0.75 | 0.20 | 1.0 |
| Instructional | 0.75 | 0.80 | 0.0 | 0.95 |
| Instagram | 0.55 | 0.70 | 0.15 | 1.05 |

- **stability** — lower = more expressive/variable; higher = more consistent/robotic
- **similarity_boost** — higher = closer to the reference voice
- **style** — exaggerates the voice's natural style; keep at 0.0 for dry delivery (Fireship)
- **speed** — 1.0 is normal; Fireship-style tightness is achieved through the script, not speed

### Model Selection

| Model | Use case |
|:---|:---|
| `eleven_turbo_v2_5` | Fast iteration, drafts, short clips |
| `eleven_multilingual_v2` | Final render — highest quality |

Use `eleven_turbo_v2_5` for all drafts. Switch to `eleven_multilingual_v2` for the final render only.

### Recording: One File Per Sentence

```bash
# Function to generate a single sentence
# Usage: el_say "voice_id" "text" "output_filename.wav"
el_say() {
  local VOICE_ID="$1"
  local TEXT="$2"
  local OUTPUT="$3"
  local MODEL="${4:-eleven_turbo_v2_5}"

  curl -s -X POST \
    "https://api.elevenlabs.io/v1/text-to-speech/${VOICE_ID}/stream" \
    -H "xi-api-key: $ELEVENLABS_API_KEY" \
    -H "Content-Type: application/json" \
    -d "{
      \"text\": \"${TEXT}\",
      \"model_id\": \"${MODEL}\",
      \"voice_settings\": {
        \"stability\": 0.65,
        \"similarity_boost\": 0.75,
        \"style\": 0.0,
        \"use_speaker_boost\": true
      },
      \"output_format\": \"mp3_44100_128\"
    }" | \
    ffmpeg -y -i pipe:0 -ar 48000 -ac 2 -sample_fmt s16 "${OUTPUT}" -loglevel quiet

  echo "✓ ${OUTPUT}"
}

# Export the function so it works in subshells
export -f el_say
```

> **Requires ffmpeg** — install with `brew install ffmpeg` if not present.
> The API returns MP3; ffmpeg converts it to 48kHz stereo WAV (Remotion-compatible).

### Batch Recording Example

```bash
# Get your voice ID first (run the voice list command above, find your voice)
VOICE_ID="your_voice_id_here"
OUT_DIR="public/audio/[topic-name]"
mkdir -p "$OUT_DIR"

el_say "$VOICE_ID" "Your sentence one." "$OUT_DIR/vo_01.wav"
el_say "$VOICE_ID" "Your sentence two." "$OUT_DIR/vo_02.wav"
# ... etc

echo "All files recorded."
```

### Measuring Duration After Recording

```bash
# Measure all WAV files in order
for f in public/audio/[topic-name]/vo_*.wav; do
  dur=$(ffprobe -v error -show_entries format=duration \
    -of default=noprint_wrappers=1:nokey=1 "$f")
  echo "$f: ${dur}s"
done
```

### Duration Calibration

ElevenLabs duration is more predictable than macOS `say`, but still verify total before building
frame timings:

1. Record all sentences
2. Sum durations: `total = sum of all vo_XX.wav durations + (num_sentences × 0.05s gap)`
3. If total deviates >3s from target, adjust via `style.speed` (0.95–1.10) and re-record
4. If over budget: cut sentences from the script — don't speed-hack your way to the target

---

## 2. macOS `say`

No API key needed. Good for offline drafts or fast iteration.

### Setup

No setup required. Voices are built into macOS.

```bash
# List all available voices
say -v '?'

# Preview a voice
say -v Daniel "Hello, I'm Daniel."
```

### Recommended Voices by Format

| Format | Command | Notes |
|:---|:---|:---|
| Fireship | `say -v Daniel -r 270` | British, dry delivery — empirically calibrated |
| YouTube Shorts | `say -v "Samantha (Enhanced)" -r 165` | Clear, engaging |
| Instructional | `say -v "Reed (English (US))" -r 150` | Professional, measured |
| Instagram | `say -v "Flo (English (US))" -r 185` | Crisp, punchy |

### Recording to WAV

macOS `say` outputs AIFF by default. Convert to WAV for Remotion:

```bash
# Record and convert in one step
say -v Daniel -r 270 "Your sentence here." -o /tmp/temp.aiff && \
  ffmpeg -y -i /tmp/temp.aiff -ar 48000 -ac 2 -sample_fmt s16 output.wav -loglevel quiet
```

### Duration Calibration (CRITICAL for macOS)

macOS `-r N` is NOT literal WPM — punctuation adds silent pauses that inflate duration
significantly. Always calibrate before recording the full script:

```bash
# 1. Record one representative sentence
say -v Daniel -r 270 "Your calibration sentence here." -o /tmp/cal.aiff

# 2. Measure actual duration
afinfo /tmp/cal.aiff | grep "estimated duration"

# 3. Calculate actual WPM and scale:
#    required_rate = current_rate × (actual_duration / expected_duration)
#    e.g. if 24-word sentence at -r 270 takes 5.9s but expected 5.0s:
#    required_rate = 270 × (5.0 / 5.9) = 229

# 4. Re-record calibration sample at new rate to verify, then record all files
```

### Word Count Budgets (macOS `say` with Daniel -r 270)

| Target duration | Max VO words | Notes |
|:---|---:|:---|
| 60 seconds | ~130 words | Leaves ~3s for meme beats + gaps |
| 100 seconds | ~220 words | Leaves ~5s for meme beats + gaps |
| 120 seconds | ~265 words | Leaves ~5s for meme beats + gaps |

---

## 3. OpenAI TTS

### Setup

```bash
export OPENAI_API_KEY="your_key_here"
```

### Recommended Voices by Format

| Format | Voice | Speed | Notes |
|:---|:---|:---:|:---|
| Fireship | `onyx` | 1.10× | Deep, authoritative, dry |
| YouTube Shorts | `alloy` or `nova` | 0.95× | Clear and engaging |
| Instructional | `onyx` or `shimmer` | 0.90× | Professional, measured |
| Instagram | `echo` or `fable` | 1.05× | Crisp and assertive |

### Recording: One File Per Sentence

```bash
# Function to generate a single sentence
# Usage: oai_say "voice" speed "text" "output.wav"
oai_say() {
  local VOICE="$1"
  local SPEED="$2"
  local TEXT="$3"
  local OUTPUT="$4"

  curl -s -X POST "https://api.openai.com/v1/audio/speech" \
    -H "Authorization: Bearer $OPENAI_API_KEY" \
    -H "Content-Type: application/json" \
    -d "{
      \"model\": \"tts-1-hd\",
      \"voice\": \"${VOICE}\",
      \"input\": \"${TEXT}\",
      \"speed\": ${SPEED},
      \"response_format\": \"wav\"
    }" | \
    ffmpeg -y -i pipe:0 -ar 48000 -ac 2 -sample_fmt s16 "${OUTPUT}" -loglevel quiet

  echo "✓ ${OUTPUT}"
}
export -f oai_say
```

### Batch Recording Example

```bash
OUT_DIR="public/audio/[topic-name]"
mkdir -p "$OUT_DIR"

oai_say "onyx" 1.10 "Your sentence one." "$OUT_DIR/vo_01.wav"
oai_say "onyx" 1.10 "Your sentence two." "$OUT_DIR/vo_02.wav"
```

---

## Measuring All Durations (Any Provider)

After all files are recorded:

```bash
# Print durations for all sentences (requires ffprobe)
echo "=== Duration Report ==="
total=0
i=1
for f in public/audio/[topic-name]/vo_*.wav; do
  dur=$(ffprobe -v error -show_entries format=duration \
    -of default=noprint_wrappers=1:nokey=1 "$f")
  printf "vo_%02d.wav: %.3fs\n" $i $dur
  total=$(python3 -c "print($total + $dur)")
  ((i++))
done
echo "─────────────────────"
echo "Total VO duration: ${total}s"
echo "(Add ~0.05s × num_sentences for gaps)"
```

---

## Frame Timing Calculation

Once durations are measured, calculate frame positions at the composition's FPS:

```bash
FPS=60  # or 30 for non-Fireship formats

# Convert duration to frames
# frames = round(start_time_seconds × FPS)

# Example at 60fps:
# vo_01.wav starts at 0s     → frame 0
# vo_02.wav starts at 1.2s   → frame 72   (1.2 × 60)
# vo_03.wav starts at 2.8s   → frame 168  (2.8 × 60)
# etc.
```

Record these in `audioTimings.ts` in the topic's source directory.

---

## Final Format Checklist

Before handing audio to Remotion:
- [ ] All files are WAV (not AIFF, MP3, or M4A)
- [ ] Sample rate: 48kHz
- [ ] Bit depth: 16-bit or 24-bit
- [ ] Channels: stereo (2)
- [ ] Total duration is within ±2s of target
- [ ] Files are named `vo_01.wav`, `vo_02.wav`, … (zero-padded)
- [ ] Files live in `public/audio/[topic-name]/`
