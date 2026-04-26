# Asset Sourcing Guide

This file is the single source of truth for sourcing visual assets (photos, logos, GIFs)
during Step 4 of the video-creator workflow.

Assets are downloaded into `public/images/` and referenced in Remotion via `/images/<filename>`.

---

## When to Run Asset Sourcing

After script approval (Gate A), before TTS (Step 5). Walk through the approved script
scene-by-scene and identify every visual that is **not** a code block or kinetic text.
For each one, determine which source applies using the priority order below and fetch it.

---

## Source Priority

| Priority | Source | Best For |
|:---:|---|---|
| 1 | **SimpleIcons** | Brand/tech logos — no API key needed |
| 2 | **Unsplash** | Photos, people, backgrounds, product shots |
| 3 | **Giphy** | Reaction clips / humor beats |
| 4 | **meme-factory skill** | Custom text-on-template memes |
| 5 | **WebSearch fallback** | Anything not found above |

Work through this priority order for every asset. Do not jump to a lower-priority
source if a higher one is available.

---

## Filename Convention

All downloaded assets must follow this naming pattern:

```
public/images/scene-{scene_id}-{slug}.{ext}
```

Examples:
```
public/images/scene-2-react.svg
public/images/scene-3-hero-photo.jpg
public/images/scene-5-mind-blown.mp4
```

Where `slug` is a short, lowercase, hyphenated description of the asset.

---

## Download Verification

After **every** download, verify the file exists and is non-zero bytes:

```bash
[ -s public/images/FILENAME.ext ] \
  && echo "✅ Downloaded: FILENAME.ext" \
  || echo "❌ Failed — file missing or empty: FILENAME.ext"
```

If verification fails: retry once, then fall back to the next priority source.
Do not proceed with a broken or empty file.

---

## Image Resizing

After downloading any photo or raster image (`.jpg`, `.png`, `.webp`), resize it
to a maximum of 1920px on the longest edge to avoid bloating the Remotion bundle
and slowing renders:

```bash
ffmpeg -i public/images/FILENAME.jpg \
  -vf "scale='min(1920,iw)':-2" \
  public/images/FILENAME.jpg -y
```

SVGs and MP4s do not need resizing.

---

## Source 1: SimpleIcons — Brand & Tech Logos

Use for: programming language logos, framework logos, company/product logos, tool icons.

**No API key required. Free. No rate limits.**

### Finding the correct slug

Browse https://simpleicons.org — search by name, the slug is shown on the card.
Common slugs: `react`, `typescript`, `rust`, `go`, `python`, `nodejs`, `bun`,
`docker`, `kubernetes`, `github`, `vercel`, `postgresql`, `redis`, `graphql`,
`tailwindcss`, `nextdotjs`, `vitejs`, `tanstackquery`

### Download

```bash
curl -L "https://cdn.simpleicons.org/ICON_SLUG/000000" \
  -o public/images/scene-{scene_id}-ICON_SLUG.svg
```

The trailing `/000000` sets the icon fill to black. Replace with any hex color,
or use the brand color (see below).

### Getting the official brand color

SimpleIcons publishes brand hex colors in their data file on npm. Fetch it with:

```bash
curl -s "https://raw.githubusercontent.com/simple-icons/simple-icons/develop/_data/simple-icons.json" \
  | jq '.icons[] | select(.slug == "ICON_SLUG") | .hex'
# Returns e.g. "61DAFB" for React
```

Then re-fetch the icon with the brand color:

```bash
curl -L "https://cdn.simpleicons.org/ICON_SLUG/61DAFB" \
  -o public/images/scene-{scene_id}-ICON_SLUG.svg
```

### Usage in Remotion

```tsx
<Img src="/images/scene-2-react.svg" style={{ width: 80, height: 80 }} />
```

### License note

SimpleIcons artwork is **CC0**. However, the brand logos they depict are
**trademarked** by their respective owners. Use is acceptable for editorial
and educational videos. Consult your client before using in commercial promotion.

---

## Source 2: Unsplash — Photos & Backgrounds

Use for: people, places, abstract backgrounds, product lifestyle shots.

**Requires:** `UNSPLASH_ACCESS_KEY` — free tier available at unsplash.com/developers
(50 requests/hour on the demo key, 5,000/hour on a registered app key).

If `UNSPLASH_ACCESS_KEY` is not set: skip Unsplash entirely and proceed to the
next priority source. Do not attempt unauthenticated requests.

If the rate limit is hit mid-workflow: pause, notify the user, and offer to
continue with WebSearch fallback for remaining assets.

### Search

```bash
curl "https://api.unsplash.com/search/photos?query=QUERY&per_page=5&client_id=$UNSPLASH_ACCESS_KEY" \
  | jq '.results[] | {id, description: .alt_description, url: .urls.regular, credit: .user.name}'
```

Pick the most visually appropriate result. Prefer `.urls.regular` (1080px) over
`.urls.full` to keep file sizes reasonable before the resize step.

### Download

```bash
curl -L "IMAGE_URL" -o public/images/scene-{scene_id}-SLUG.jpg
```

Then resize per the Image Resizing rule above.

### Attribution

The Unsplash License does not require on-screen credit in published videos, but
always record the photographer in the asset manifest and in a code comment
in the Remotion component:

```tsx
{/* Photo: "Description" by Photographer Name — Unsplash License */}
```

---

## Source 3: Giphy — Reaction Clips

Use for: humor beats, absurd facts, cultural references, emotional punctuation.

**Requires:** `GIPHY_API_KEY` — free Developer tier available at developers.giphy.com.
Check the Giphy developer dashboard for current rate limits, as they change over time.

If `GIPHY_API_KEY` is not set: skip Giphy entirely and proceed to the next priority source.

> **Giphy vs. meme-factory:** Use Giphy for raw reaction clips with no custom text.
> Use meme-factory when you need custom text overlaid on a meme template.

### Search

```bash
curl "https://api.giphy.com/v1/gifs/search?api_key=$GIPHY_API_KEY&q=QUERY&limit=5&rating=g" \
  | jq '.data[] | {id, title, mp4: .images.fixed_height.mp4, gif: .images.fixed_height.url}'
```

### Download

Always prefer MP4 over GIF — smaller file size, smoother playback in Remotion.
Only fall back to GIF if the `mp4` field is absent in the API response.

```bash
# Preferred: MP4
curl -L "MP4_URL" -o public/images/scene-{scene_id}-SLUG.mp4

# Fallback only: GIF
curl -L "GIF_URL" -o public/images/scene-{scene_id}-SLUG.gif
```

### If you downloaded a raw GIF anyway

Convert it to MP4 immediately after download — do not use `.gif` files directly in Remotion:

```bash
ffmpeg -i public/images/scene-{scene_id}-SLUG.gif \
  -vf "scale=trunc(iw/2)*2:trunc(ih/2)*2" \
  -c:v libx264 -pix_fmt yuv420p -movflags +faststart \
  public/images/scene-{scene_id}-SLUG.mp4 -y
```

Then update the manifest entry to point at the `.mp4` and delete the `.gif`.

### Usage in Remotion

Always use `<OffthreadVideo>` — **never** `<Video>`, `<Img>`, or `<Gif>` for animation clips.

```tsx
import { OffthreadVideo, Sequence, staticFile } from "remotion";

{/* Clip that starts at scene frame 0 */}
<OffthreadVideo
  src={staticFile("images/scene-5-mind-blown.mp4")}
  style={{ width: 700, height: 420, display: "block" }}
/>

{/* Clip that first appears at a later frame (e.g. frame 220) — MUST wrap in Sequence */}
{/* Without Sequence, useCurrentFrame() is 220 at appearance time, seeking 7s into a 5s clip */}
<Sequence from={220} layout="none">
  <OffthreadVideo
    src={staticFile("images/scene-5-mind-blown.mp4")}
    style={{ width: 700, height: 420, display: "block" }}
  />
</Sequence>
```

> **Why `<OffthreadVideo>` and not `<Gif>`?** `@remotion/gif` renders to a `<canvas>` and
> relies on a `ResizeObserver` to get its own size before drawing. When a CSS `transform: scale()`
> exists on any ancestor (common for spring-in animations), `getClientRects()` returns near-zero
> dimensions and the draw call is silently skipped. Remotion may snapshot the frame before the
> observer re-fires with the correct size. Result: blank canvas in the rendered video with no error.
> `<OffthreadVideo>` has none of these issues — it extracts frames server-side, deterministically.

> **Why `<Sequence from={N}>`?** `<OffthreadVideo>` uses `useCurrentFrame()` to seek into the
> video. If the clip appears mid-scene at frame N but there's no wrapping `<Sequence>`,
> the video seeks to `N/fps` seconds on its first visible frame — past the end for short clips.
> `<Sequence from={N} layout="none">` resets the frame counter to 0 at the appearance point.

---

## Source 4: meme-factory Skill

Use for: custom text overlaid on a standard meme template (Drake, Distracted Boyfriend, etc.).

Invoke the `meme-factory` skill directly, passing the template name and text strings.
The skill will return a rendered image file — save it to `public/images/` following
the filename convention and add it to the asset manifest as any other asset.

See the meme-factory skill documentation for invocation details and available templates.

---

## Source 5: WebSearch Fallback

Use when no higher-priority source yields a usable asset.

### Search

Use the `WebSearch` tool with descriptive queries. Prefer results from known
permissive-license sources (Wikimedia Commons, Pexels, Pixabay, official press kits).

### License Verification

Before downloading, confirm the license. Only the following are acceptable:

| License | Acceptable |
|---|:---:|
| CC0 (Public Domain) | ✅ |
| Unsplash License | ✅ |
| Pexels License | ✅ |
| Pixabay License | ✅ |
| Explicit "free for commercial use" statement | ✅ |
| CC BY (attribution required in video) | ⚠️ Flag for user decision |
| CC BY-NC (non-commercial only) | ❌ |
| No license stated | ❌ Assume all rights reserved |

If the license is unclear, do not download. Mark the asset as `placeholder` in
the manifest and flag it for the user to resolve.

### Download

```bash
curl -L "IMAGE_URL" -o public/images/scene-{scene_id}-SLUG.ext
```

Then resize if raster, and verify per the rules above.

---

## All-Sources-Fail Fallback

If every source is exhausted and no usable asset is found:

1. Do **not** block the workflow
2. Generate a placeholder at render time — a solid colored rectangle with the
   asset label as white text, implementable directly in Remotion:
   ```tsx
   {/* PLACEHOLDER: replace before final render */}
   <div style={{ background: '#333', color: '#fff', display: 'flex',
     alignItems: 'center', justifyContent: 'center', width: '100%', height: '100%' }}>
     scene-{scene_id}: {assetLabel}
   </div>
   ```
3. Mark the asset as `"status": "placeholder"` in the manifest
4. List all placeholders in a summary after the manifest is generated so the
   user can resolve them before final render

---

## Asset Manifest

After all assets are sourced, generate `public/images/asset-manifest.json`.
This is the authoritative record used by the Remotion code generation step
to reference assets by scene.

### Schema

Every entry must include all of the following fields:

```json
{
  "asset_id": "scene-2-react",
  "scene_id": 2,
  "file": "public/images/scene-2-react.svg",
  "source": "simpleicons",
  "credit": "SimpleIcons — CC0",
  "license": "CC0",
  "status": "ready"
}
```

### `status` values

| Value | Meaning |
|---|---|
| `ready` | Downloaded, verified, resized — safe to use |
| `placeholder` | No asset found — Remotion placeholder in use, needs replacement |
| `needs-review` | License unclear — flagged for user decision before final render |

### Full example

```json
[
  {
    "asset_id": "scene-1-typescript",
    "scene_id": 1,
    "file": "public/images/scene-1-typescript.svg",
    "source": "simpleicons",
    "credit": "SimpleIcons — CC0",
    "license": "CC0",
    "status": "ready"
  },
  {
    "asset_id": "scene-2-hero-photo",
    "scene_id": 2,
    "file": "public/images/scene-2-hero-photo.jpg",
    "source": "unsplash",
    "credit": "Jane Doe — Unsplash License",
    "license": "Unsplash License",
    "status": "ready"
  },
  {
    "asset_id": "scene-3-mind-blown",
    "scene_id": 3,
    "file": "public/images/scene-3-mind-blown.mp4",
    "source": "giphy",
    "credit": "Giphy — https://giphy.com/gifs/GIPHY_ID",
    "license": "Giphy Terms of Service",
    "status": "ready"
  },
  {
    "asset_id": "scene-4-custom-diagram",
    "scene_id": 4,
    "file": "public/images/scene-4-custom-diagram.png",
    "source": "websearch",
    "credit": "Example Corp Press Kit",
    "license": "CC0",
    "status": "needs-review"
  }
]
```

### Post-manifest summary

After generating the manifest, print a plain-text summary for quick human review:

```
ASSET MANIFEST SUMMARY
──────────────────────
Total assets:     4
Ready:            3
Needs review:     1
Placeholders:     0

⚠️  needs-review assets must be resolved before final render:
  - scene-4-custom-diagram.png (source: websearch — license unconfirmed)
```

