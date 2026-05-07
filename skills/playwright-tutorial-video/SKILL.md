---
name: playwright-tutorial-video
description: >
  Record a real user flow with Playwright, then generate a Remotion tutorial video.
  Two entry points: scripted-recording (a Playwright script automates the flow
  end-to-end) and interactive-recording (you drive Chromium manually while
  Playwright records, auto-capturing click coordinates). Both produce a single
  continuous video with timestamp-synced caption + highlight overlays and TTS
  narration. No new UI code is created.
metadata:
  tags: playwright, remotion, tutorial, automation, recording, video, interactive, onboarding
---

# Claude Code Skill: Playwright Tutorial Video

Bridges **Playwright recording** and **Remotion overlays** in one pipeline.

Playwright records a real browser session at exactly 1920×1080. Element coordinates are captured live via `getBoundingClientRect()` in the same pixel space, so highlights line up perfectly with the recorded video. The manifest feeds a single Remotion `VideoComposition` that plays `OffthreadVideo` with `HighlightBox`, `Caption`, and `Audio` overlays sequenced by timestamp.

```
Browser session (Playwright @ 1920×1080)
  → manifest.json (timed steps + pixel-perfect coords)
    → Remotion VideoComposition (video + overlays + narration)
      → tutorial.mp4
```

---

## Entry Points

| Entry | When to use |
|---|---|
| `scripted-recording` | You can write a Playwright script that performs the flow — fully automated, repeatable, headless |
| `interactive-recording` | The flow needs human judgement (2FA, exploratory paths, third-party UIs) — you drive Chromium by hand, every click is auto-captured with bounding-box coords |

Both entries produce the **same downstream artifacts**: one `recording.webm`, one `manifest.json`, one `VideoComposition.tsx`. All steps after the recording phase are shared.

**Choosing between entries:**
- `scripted-recording` — your app is running locally, the flow is deterministic (forms, button clicks, navigation), and you want it repeatable for re-renders. Coordinates come from `locator.boundingBox()` calls in the script.
- `interactive-recording` — the target needs login, the path branches, or you simply want to demonstrate from your own session without writing automation. Coordinates come from real `getBoundingClientRect()` calls injected into every click event.

---

## Activation

```
🎭 Activate Playwright Tutorial

Entry: [scripted-recording | interactive-recording]

# For scripted-recording entry:
App URL:    [e.g. http://localhost:3000]
Task:       [what the user does — the full flow to record end-to-end]
Selectors:  [optional — Claude infers from the task]
Setup:      [auto (default) — open Chromium first to confirm/log-in/navigate to starting page,
             then run the headless recording with the captured session
            | skip — go straight to headless recording (only if URL is public + already at start)]

# For interactive-recording entry (Claude opens Chromium, you drive it manually):
App URL:    [e.g. https://linear.app — where Chromium should open]
Task:       [overall description of what you'll demonstrate]
```

> `🎭 Activate Playwright Tutorial` is the **only** trigger.
> Do not activate from unrelated messages.

---

## Session Context

Initialize at activation and maintain across all steps:

```
entry:            <scripted-recording | interactive-recording>
app_url:          <URL of running app or live site>
task:             <what the user is doing>
flow_slug:        <kebab-case — e.g. "onboarding-checklist">
viewport:         { width: 1920, height: 1080 }   ← always
video:            { width: 1920, height: 1080, fps: 30 }
manifest_path:     public/recordings/<flow-slug>/manifest.json
recording_path:    public/recordings/<flow-slug>/recording.webm
storage_state_path: public/recordings/<flow-slug>/storage-state.json   ← written by setup phase if used
start_url_path:    public/recordings/<flow-slug>/start-url.txt          ← URL the user reached at end of setup
render_output:     output/<flow-slug>-<YYYYMMDD>.mp4
setup_pid:         <background task id of recording-setup.ts — scripted-recording when Setup: auto>
recorder_pid:      <background task id of interactive-record.ts — interactive-recording only>
```

---

## Autonomy Rules

### Claude decides autonomously
- Highlight overlay timing (delay after step start, fade-in, hold duration)
- Caption typewriter speed and start frame
- Audio sequencing and trim padding
- Translating captured click labels into viewer-friendly captions

### Claude must ask the user
- Entry point and activation inputs
- Confirm the proposed caption + highlight schedule before generating the composition (**Caption Gate**)
- Confirm "go" before render (**Render Gate**)
- Any ambiguous selector (asks once, does not re-ask)

---

## Step 0: Preflight

### scripted-recording entry

```bash
# Playwright + chromium
npx playwright --version
npx playwright install chromium --dry-run 2>&1 | head -1

# App is reachable
curl -s -o /dev/null -w "%{http_code}" <APP_URL>

# Remotion project initialized
node -e "require('./package.json').dependencies['remotion'] && console.log('ok')"

# tsx for running the recording script
npx tsx --version
```

### interactive-recording entry

No app dev-server check — the user navigates to a live site (their own app, Linear, Notion, anything reachable in a browser).

```bash
npx playwright --version
npx playwright install chromium --dry-run 2>&1 | head -1
npx tsx --version
node -e "require('./package.json').dependencies['remotion'] && console.log('ok')"
```

Confirm the URL is reachable from a normal browser (don't `curl` — Linear, Notion etc. require login and may return 4xx without cookies).

```
✅ Preflight passed. Proceeding with flow: [FLOW-SLUG]
```

---

## Step 0.5a: Setup Phase — scripted-recording entry

**Default behavior (`Setup: auto`).** Before any scripted recording runs, open headed Chromium at `App URL` so the user can:
- confirm the URL is the right starting page
- log in / dismiss cookie banners / dismiss onboarding modals
- navigate to a deeper starting state (e.g. open a specific project, then start)

When the user signals ready, save `storageState` (cookies + localStorage) and the final URL. The flow-specific recording script in Step 1a loads both — so the headless run starts authenticated, on the same page, with the same client state.

**Skip this section entirely if `Setup: skip`** was specified at activation.

### Generate the setup helper (once per project)

Write `scripts/recording-setup.ts` to the project **once** — skip if already present.

```typescript
// scripts/recording-setup.ts
// Usage: npx tsx scripts/recording-setup.ts <flow-slug> <start-url>
//
// Drive from chat: touch public/recordings/<flow-slug>/.ir-start once the user
// confirms they're on the correct starting page (logged in, modals dismissed, etc).

import { chromium } from 'playwright';
import * as fs from 'fs';
import * as path from 'path';
import * as readline from 'readline';

const [, , FLOW_SLUG, START_URL] = process.argv;
if (!FLOW_SLUG || !START_URL) {
  console.error('\nUsage: npx tsx scripts/recording-setup.ts <flow-slug> <start-url>\n');
  process.exit(1);
}

const RECORDING_DIR  = path.join('public', 'recordings', FLOW_SLUG);
const STORAGE_PATH   = path.join(RECORDING_DIR, 'storage-state.json');
const START_URL_PATH = path.join(RECORDING_DIR, 'start-url.txt');
const SIGNAL_START   = path.join(RECORDING_DIR, '.ir-start');
const VIEWPORT       = { width: 1920, height: 1080 } as const;

function waitForSignal(): Promise<void> {
  fs.mkdirSync(RECORDING_DIR, { recursive: true });
  if (fs.existsSync(SIGNAL_START)) fs.unlinkSync(SIGNAL_START);

  if (process.stdin.isTTY) {
    return new Promise(resolve => {
      const rl = readline.createInterface({ input: process.stdin, output: process.stdout });
      process.stdout.write('\n  On the correct starting page? Press ENTER › ');
      rl.once('line', () => { rl.close(); resolve(); });
    });
  }

  return new Promise(resolve => {
    const interval = setInterval(() => {
      if (fs.existsSync(SIGNAL_START)) {
        fs.unlinkSync(SIGNAL_START);
        clearInterval(interval);
        resolve();
      }
    }, 250);
  });
}

(async () => {
  console.log('\n┌──────────────────────────────────────────────────────────┐');
  console.log('│  🎭  Setup Phase — confirm starting page                 │');
  console.log('│  Browser is open. Log in, dismiss banners, navigate to   │');
  console.log('│  the exact page where the recording should begin.        │');
  console.log('└──────────────────────────────────────────────────────────┘\n');
  console.log(`  Signal (non-TTY): touch ${SIGNAL_START}\n`);

  const browser = await chromium.launch({
    headless: false,
    args: ['--disable-infobars', '--no-default-browser-check'],
  });
  const ctx  = await browser.newContext({ viewport: VIEWPORT });
  const page = await ctx.newPage();
  await page.goto(START_URL, { waitUntil: 'domcontentloaded' });

  await waitForSignal();

  const finalUrl     = page.url();
  const storageState = await ctx.storageState();
  fs.mkdirSync(RECORDING_DIR, { recursive: true });
  fs.writeFileSync(STORAGE_PATH,   JSON.stringify(storageState, null, 2));
  fs.writeFileSync(START_URL_PATH, finalUrl);

  await ctx.close();
  await browser.close();

  console.log(`\n✅ Setup complete`);
  console.log(`   Start URL     ${finalUrl}`);
  console.log(`   Storage state ${STORAGE_PATH}\n`);
})();
```

### Run the setup as a background task

```bash
mkdir -p public/recordings/<flow-slug>
npx tsx scripts/recording-setup.ts <flow-slug> <app-url>
```

### The conversation pattern

```
1. Claude:   "Chromium is open at <app-url>. Confirm this is the correct starting
              page — log in or navigate if needed. Tell me when you're ready."
2. User:     "Looks right" / "ok I'm logged in and on the project page"
3. Claude:   touch public/recordings/<flow-slug>/.ir-start
             [waits for the background task to report completion]
4. Claude:   "Setup saved. Start URL: <captured-url>. Generating recording script..."
```

Both `storage-state.json` and `start-url.txt` are now in place. Step 1a's generated recording script will load them automatically.

If the user navigates somewhere different than `App URL`, the captured `start-url.txt` becomes the actual recording start URL — so the script doesn't need to re-do navigation.

---

## Step 1a: Generate Recording Script — scripted-recording entry

Write `scripts/playwright-capture.ts` to the project **once** — skip if already present. This is the reusable recording helper.

```typescript
// scripts/playwright-capture.ts
import { type Page } from 'playwright';
import * as fs from 'fs';
import * as path from 'path';

export interface CapturedElement {
  selector: string;
  x: number;
  y: number;
  width: number;
  height: number;
}

export interface TimedStep {
  step: number;
  label: string;
  caption: string;
  startMs: number;
  interaction: 'click' | 'fill' | 'select' | 'none';
  element: CapturedElement | null;
  typedValue?: string;
}

export interface RecordingManifest {
  flowSlug: string;
  capturedAt: string;
  recordingFile: string;
  videoDurationMs: number;
  viewport: { width: number; height: number };
  steps: TimedStep[];
}

export class TutorialCaptureRecording {
  private steps: TimedStep[] = [];
  private stepIndex = 0;
  public t0 = 0;
  private manifestPath: string;

  constructor(private page: Page, private flowSlug: string) {
    const dir = path.join('public', 'recordings', flowSlug);
    fs.mkdirSync(dir, { recursive: true });
    this.manifestPath = path.join(dir, 'manifest.json');
  }

  start(): void { this.t0 = Date.now(); }
  private elapsed(): number { return Date.now() - this.t0; }

  private async getBox(selector: string): Promise<CapturedElement | null> {
    try {
      const locator = this.page.locator(selector).first();
      await locator.scrollIntoViewIfNeeded();
      const box = await locator.boundingBox();
      if (!box) return null;
      return { selector, x: box.x, y: box.y, width: box.width, height: box.height };
    } catch { return null; }
  }

  /** Mark a moment in the recording with an optional element to highlight — no interaction. */
  async mark(label: string, caption: string, selector?: string): Promise<void> {
    const startMs = this.elapsed();
    const element = selector ? await this.getBox(selector) : null;
    this.steps.push({ step: ++this.stepIndex, label, caption, startMs, interaction: 'none', element });
  }

  async click(selector: string, label: string, caption: string): Promise<void> {
    const startMs = this.elapsed();
    const element = await this.getBox(selector);
    this.steps.push({ step: ++this.stepIndex, label, caption, startMs, interaction: 'click', element });
    await this.page.locator(selector).first().click();
  }

  async select(selector: string, value: string, label: string, caption: string): Promise<void> {
    const startMs = this.elapsed();
    const element = await this.getBox(selector);
    this.steps.push({ step: ++this.stepIndex, label, caption, startMs, interaction: 'select', element, typedValue: value });
    await this.page.locator(selector).first().selectOption(value);
  }

  async fill(selector: string, value: string, label: string, caption: string): Promise<void> {
    const startMs = this.elapsed();
    const element = await this.getBox(selector);
    this.steps.push({ step: ++this.stepIndex, label, caption, startMs, interaction: 'fill', element, typedValue: value });
    await this.page.locator(selector).first().fill(value);
  }

  async save(recordingWebmPath: string, videoDurationMs: number): Promise<void> {
    const vp = this.page.viewportSize() ?? { width: 1920, height: 1080 };
    const manifest: RecordingManifest = {
      flowSlug: this.flowSlug,
      capturedAt: new Date().toISOString(),
      recordingFile: recordingWebmPath,
      videoDurationMs,
      viewport: vp,
      steps: this.steps,
    };
    fs.writeFileSync(this.manifestPath, JSON.stringify(manifest, null, 2));
    console.log(`\n✅ Recording manifest → ${this.manifestPath}`);
    console.log(`   Video: ${recordingWebmPath}  (${(videoDurationMs / 1000).toFixed(1)}s)`);
    this.steps.forEach(s =>
      console.log(`   Step ${s.step} @${(s.startMs / 1000).toFixed(2)}s: [${s.interaction}] ${s.label}`)
    );
  }
}
```

Then generate the flow-specific recording script:

```typescript
// scripts/<flow-slug>-recording.ts
import { chromium } from 'playwright';
import { TutorialCaptureRecording } from './playwright-capture';
import * as fs from 'fs';
import * as path from 'path';

const FLOW_SLUG = '<flow-slug>';
const APP_URL   = '<app-url>';

const RECORDING_DIR  = `public/recordings/${FLOW_SLUG}`;
const STORAGE_PATH   = path.join(RECORDING_DIR, 'storage-state.json');
const START_URL_PATH = path.join(RECORDING_DIR, 'start-url.txt');

(async () => {
  // If Step 0.5a (Setup Phase) ran, load the captured session + start URL.
  // Otherwise fall back to APP_URL with no auth state.
  const storageState = fs.existsSync(STORAGE_PATH)   ? STORAGE_PATH                                : undefined;
  const startUrl     = fs.existsSync(START_URL_PATH) ? fs.readFileSync(START_URL_PATH, 'utf8').trim() : APP_URL;

  // headless required — headed mode crashes during recordVideo on many setups.
  // Viewport matches Remotion composition size — element coordinates are native
  // 1920×1080 pixels, no scaling needed.
  const browser = await chromium.launch({ headless: true });
  const context = await browser.newContext({
    viewport: { width: 1920, height: 1080 },
    storageState,                          // restores cookies + localStorage from setup phase
    recordVideo: {
      dir: `${RECORDING_DIR}/`,
      size: { width: 1920, height: 1080 },
    },
  });

  const page    = await context.newPage();
  const capture = new TutorialCaptureRecording(page, FLOW_SLUG);

  await page.goto(startUrl);
  // Use domcontentloaded — networkidle never fires with WebSocket-based apps (Convex, Supabase, etc.)
  await page.waitForLoadState('domcontentloaded');
  await page.waitForSelector('<first-visible-selector>', { timeout: 10000 });
  await page.waitForTimeout(1000); // let initial animations settle before starting the clock

  capture.start(); // ← t = 0 starts here, not at page.goto()

  // ── Step 1: orientation ──────────────────────────────────────────
  await capture.mark('Overview', 'caption...', 'selector-to-highlight');
  await page.waitForTimeout(3000);

  // ── Step 2: first interaction ────────────────────────────────────
  await capture.click('button:has-text("...")', 'Label', 'Caption...');
  await page.waitForTimeout(500);
  await capture.mark('Result state', 'What changed...', 'result-selector');
  await page.waitForTimeout(3000);

  // … one call per step …

  const endMs = Date.now();
  await context.close(); // finalizes the .webm

  const videoPath = await page.video()!.path();
  const videoDurationMs = endMs - capture.t0;

  fs.mkdirSync(path.dirname(`public/recordings/${FLOW_SLUG}/recording.webm`), { recursive: true });
  fs.renameSync(videoPath, `public/recordings/${FLOW_SLUG}/recording.webm`);

  await capture.save(`public/recordings/${FLOW_SLUG}/recording.webm`, videoDurationMs);
  await browser.close();
})();
```

### Selector inference rules (when `Selectors` not provided at activation)

| Element type | Preferred selector |
|---|---|
| Button with visible text | `role=button[name="Submit"]` |
| Input with placeholder | `[placeholder="Project name"]` |
| Input with label | `role=textbox[name="Email"]` |
| `data-testid` present | `[data-testid="submit-btn"]` |
| Link | `role=link[name="Settings"]` |
| Fallback | `.class-name` (least preferred — note in script comment) |

### Pacing guide

The single most common mistake is moving too fast — the viewer can't read a caption that's still typing when the next step starts. Minimum wait times:

| After this call | `waitForTimeout` | Why |
|---|---|---|
| Initial `mark()` (orientation) | **3000ms** | Caption types ~2s, reader needs ~1s after |
| `select()` or `click()` | **400–500ms** | Let the UI finish updating |
| `mark()` after a result | **3000ms** | Hold long enough to read |
| `click()` for a reset/clear | **1500ms** | Transition only — no caption to read |
| Final `mark()` | **3500ms** | Last frame — hold extra long |

**Why these numbers:** the typewriter runs at ~40 chars/sec. A 70-char caption takes ~1.75s to type. Viewer needs ~1s to absorb. 3s minimum is the floor, not the target.

**When audio is added (recommended):** replace fixed waits with `audioDurationMs[i] + 800`. Measure first with ffprobe, then set wait times accordingly — see the Audio section.

For `select:nth-of-type()` selectors: unreliable when selects live in separate parent elements. Use `.nth(index)` via an index parameter instead.

---

## Step 2a: Run Recording Script — scripted-recording entry

Present the generated script to the user and ask:

> "Does this look right? Adjust selectors or steps, then run it."

Once approved:

```bash
npx tsx scripts/<flow-slug>-recording.ts
```

After completion, report:

```
Recording complete: public/recordings/<flow-slug>/recording.webm   (Xs)
Manifest:           public/recordings/<flow-slug>/manifest.json
Steps captured: N

  1  @0.0s     [none]   Overview
  2  @3.0s     [click]  Click 'Add new issue'    → x:611  y:769  w:320  h:28
  ...
```

Then continue to the **Caption Gate**.

---

## Step 1b: Generate Recorder Script — interactive-recording entry

Write `scripts/interactive-record.ts` to the project **once** — skip if already present.

This recorder:
- Launches headed Chromium at exactly **1920×1080**
- Phase 1: lets the user log in / navigate (no recording yet)
- Phase 2: reopens with `recordVideo` active, restores `storageState` (cookies + localStorage), navigates back to where the user left Phase 1
- Captures every click via injected `addEventListener('click', …, true)` → calls `exposeFunction` callbacks with `getBoundingClientRect()`, text, aria-label
- Captures SPA route changes via `MutationObserver`
- Writes a `RecordingManifest`-compatible JSON with pixel-perfect coordinates

**Two modes for advancing phases:**
- **TTY mode** (real terminal): readline `prompt()` — user presses ENTER
- **File-signal mode** (Claude Code, where stdin isn't a TTY): polls for `.ir-start` / `.ir-stop` files. Claude advances phases by `touch`-ing those files.

The script auto-detects `process.stdin.isTTY` and switches mode.

```typescript
// scripts/interactive-record.ts
// Usage: npx tsx scripts/interactive-record.ts <flow-slug> <start-url>
//
// Drive from chat: touch public/recordings/<flow-slug>/.ir-start to begin
// recording, then .ir-stop when the flow is complete.

import { chromium, type BrowserContext } from 'playwright';
import * as fs from 'fs';
import * as path from 'path';
import * as readline from 'readline';

const [, , FLOW_SLUG, START_URL] = process.argv;
if (!FLOW_SLUG || !START_URL) {
  console.error('\nUsage: npx tsx scripts/interactive-record.ts <flow-slug> <start-url>\n');
  process.exit(1);
}

const RECORDING_DIR = path.join('public', 'recordings', FLOW_SLUG);
const VIDEO_FINAL   = path.join(RECORDING_DIR, 'recording.webm');
const MANIFEST_PATH = path.join(RECORDING_DIR, 'manifest.json');
const VIEWPORT      = { width: 1920, height: 1080 } as const;
const SIGNAL_START  = path.join(RECORDING_DIR, '.ir-start');
const SIGNAL_STOP   = path.join(RECORDING_DIR, '.ir-stop');

function waitForSignal(signalFile: string, ttyPrompt: string): Promise<void> {
  fs.mkdirSync(path.dirname(signalFile), { recursive: true });
  if (fs.existsSync(signalFile)) fs.unlinkSync(signalFile);

  if (process.stdin.isTTY) {
    return new Promise(resolve => {
      const rl = readline.createInterface({ input: process.stdin, output: process.stdout });
      process.stdout.write(ttyPrompt);
      rl.once('line', () => { rl.close(); resolve(); });
    });
  }

  return new Promise(resolve => {
    const interval = setInterval(() => {
      if (fs.existsSync(signalFile)) {
        fs.unlinkSync(signalFile);
        clearInterval(interval);
        resolve();
      }
    }, 250);
  });
}

interface RawClick { timestamp: number; x: number; y: number; width: number; height: number; text: string; ariaLabel: string; tagName: string; }
interface RawNav   { timestamp: number; url: string; }

// ── phase 1: setup, no recording ─────────────────────────────────────────
async function setup(): Promise<{ storageState: Awaited<ReturnType<BrowserContext['storageState']>>; url: string }> {
  console.log('\n┌───────────────────────────────────────────────────────┐');
  console.log('│  🎭  Interactive Recorder — Phase 1: Setup            │');
  console.log('│  Browser is open. Log in and navigate to start page.  │');
  console.log('└───────────────────────────────────────────────────────┘\n');
  console.log(`  Signals (non-TTY mode):`);
  console.log(`    Start:  touch ${SIGNAL_START}`);
  console.log(`    Stop:   touch ${SIGNAL_STOP}\n`);

  const browser = await chromium.launch({
    headless: false,
    args: ['--disable-infobars', '--no-default-browser-check'],
  });

  const ctx  = await browser.newContext({ viewport: VIEWPORT });
  const page = await ctx.newPage();
  await page.goto(START_URL, { waitUntil: 'domcontentloaded' });

  await waitForSignal(SIGNAL_START, '\n  Ready to record? Press ENTER › ');

  const url          = page.url();
  const storageState = await ctx.storageState();
  await ctx.close();
  await browser.close();

  console.log(`\n  Session saved. Recording will start at: ${url}\n`);
  return { storageState, url };
}

// ── phase 2: record ──────────────────────────────────────────────────────
async function record(
  storageState: Awaited<ReturnType<BrowserContext['storageState']>>,
  startUrl: string,
): Promise<void> {
  console.log('┌───────────────────────────────────────────────────────┐');
  console.log('│  Phase 2 — Recording                                  │');
  console.log('│  Perform your flow. Every click is captured.          │');
  console.log('└───────────────────────────────────────────────────────┘\n');

  fs.mkdirSync(RECORDING_DIR, { recursive: true });

  const browser = await chromium.launch({
    headless: false,
    args: ['--disable-infobars', '--no-default-browser-check'],
  });

  const ctx = await browser.newContext({
    viewport: VIEWPORT,
    storageState,
    recordVideo: { dir: RECORDING_DIR, size: VIEWPORT },
  });

  const page = await ctx.newPage();
  const rawClicks: RawClick[] = [];
  const rawNavs:   RawNav[]   = [];

  await page.exposeFunction('__irClick', (e: RawClick) => { rawClicks.push(e); });
  await page.exposeFunction('__irNav',   (e: RawNav)   => { rawNavs.push(e);   });

  await page.addInitScript(() => {
    document.addEventListener('click', function (ev: MouseEvent) {
      let el = ev.target as HTMLElement;
      // Walk up to a meaningful ancestor if target is a tiny inline icon
      for (let i = 0; i < 5 && el.parentElement; i++) {
        const r = el.getBoundingClientRect();
        if (r.width >= 16 && r.height >= 16) break;
        el = el.parentElement;
      }
      const r = el.getBoundingClientRect();
      if (r.width === 0 && r.height === 0) return;
      const raw = ((el as HTMLElement).innerText || el.textContent || '').replace(/\s+/g, ' ').trim();
      (window as any).__irClick({
        timestamp: Date.now(),
        x: Math.round(r.left), y: Math.round(r.top),
        width:  Math.round(Math.min(r.width,  1800)),
        height: Math.round(Math.min(r.height, 900)),
        text: raw.slice(0, 80),
        ariaLabel: el.getAttribute('aria-label') || '',
        tagName: el.tagName.toLowerCase(),
      });
    }, true);

    let _lastUrl = location.href;
    new MutationObserver(() => {
      if (location.href !== _lastUrl) {
        _lastUrl = location.href;
        (window as any).__irNav({ timestamp: Date.now(), url: location.href });
      }
    }).observe(document.documentElement, { subtree: true, childList: true });
  });

  await page.goto(startUrl, { waitUntil: 'domcontentloaded' });
  await page.waitForTimeout(1000);
  const t0 = Date.now();
  console.log('  🔴  Recording!  Do your thing...\n');

  const stopPromise   = waitForSignal(SIGNAL_STOP, '  Done? Press ENTER › ');
  const closedPromise = new Promise<void>(r => (page as any).once('close', r));
  await Promise.race([stopPromise, closedPromise]);

  const durationMs = Date.now() - t0;
  await ctx.close().catch(() => {});
  await browser.close().catch(() => {});

  // Rename raw video
  const rawFiles = fs.readdirSync(RECORDING_DIR).filter(f => f.endsWith('.webm') && f !== 'recording.webm');
  if (rawFiles.length === 0) throw new Error(`No .webm found in ${RECORDING_DIR}`);
  fs.renameSync(path.join(RECORDING_DIR, rawFiles[0]), VIDEO_FINAL);

  // Build manifest
  type Step = {
    step: number; label: string; caption: string; startMs: number;
    interaction: 'click' | 'none';
    element: { selector: string; x: number; y: number; width: number; height: number } | null;
  };
  const timeline = [
    ...rawClicks.map(c => ({ kind: 'click' as const, ...c })),
    ...rawNavs.map(n   => ({ kind: 'nav'   as const, ...n })),
  ].sort((a, b) => a.timestamp - b.timestamp);

  const steps: Step[] = [];
  for (let i = 0; i < timeline.length; i++) {
    const ev = timeline[i];
    if (ev.timestamp < t0) continue;
    const startMs = ev.timestamp - t0;

    if (ev.kind === 'nav') {
      steps.push({ step: 0, label: `Navigate → ${new URL(ev.url).pathname.slice(0, 50)}`, caption: '', startMs, interaction: 'none', element: null });
      continue;
    }
    if (ev.width  > VIEWPORT.width  * 0.9) continue;
    if (ev.height > VIEWPORT.height * 0.9) continue;
    const prev = timeline[i - 1];
    if (prev?.kind === 'click' && ev.timestamp - prev.timestamp < 200) continue;

    steps.push({
      step: 0,
      label: (ev.ariaLabel || ev.text || `${ev.tagName} click`).slice(0, 60),
      caption: '', startMs, interaction: 'click',
      element: { selector: 'captured', x: ev.x, y: ev.y, width: ev.width, height: ev.height },
    });
  }
  steps.forEach((s, i) => { s.step = i + 1; });

  const manifest = {
    flowSlug:        FLOW_SLUG,
    capturedAt:      new Date().toISOString(),
    recordingFile:   VIDEO_FINAL,
    videoDurationMs: durationMs,
    viewport:        VIEWPORT,
    steps,
  };
  fs.writeFileSync(MANIFEST_PATH, JSON.stringify(manifest, null, 2));

  console.log(`\n✅  Recording complete!`);
  console.log(`   Video     ${VIDEO_FINAL}   (${(durationMs / 1000).toFixed(1)}s)`);
  console.log(`   Manifest  ${MANIFEST_PATH}`);
  console.log(`   Steps     ${steps.length} captured\n`);
  steps.forEach(s => {
    const t = (s.startMs / 1000).toFixed(2).padStart(7);
    const el = s.element ? `  [${s.element.x}, ${s.element.y}, ${s.element.width}×${s.element.height}]` : '';
    console.log(`   ${String(s.step).padStart(2)}.  ${t}s  ${s.label}${el}`);
  });
}

(async () => {
  try {
    const { storageState, url } = await setup();
    await record(storageState, url);
  } catch (err) {
    console.error('\n❌ Error:', (err as Error).message);
    process.exit(1);
  }
})();
```

This file is generated **once per project**. If `scripts/interactive-record.ts` already exists, skip generation.

---

## Step 2b: Run Recorder + Drive From Chat — interactive-recording entry

Run the script as a **background task** so it stays alive while Claude waits for user signals:

```bash
mkdir -p public/recordings/<flow-slug>
npx tsx scripts/interactive-record.ts <flow-slug> <start-url>
```

Then drive the two phases by writing signal files at the right moments.

### The conversation pattern

```
1. Claude:   "Chromium is open. Log in and navigate to the right starting point.
              Tell me when you're ready to record."
2. User:     "I'm at the right page"
3. Claude:   touch public/recordings/<flow-slug>/.ir-start
             "Recording is live. Do your flow naturally — every click is captured.
              Tell me when you're done."
4. User:     "done"
5. Claude:   touch public/recordings/<flow-slug>/.ir-stop
             [waits for the script to finalise, reads the produced manifest]
6. Claude:   "Captured N clicks. Manifest at <path>. Continuing to Caption Gate..."
```

### What the user provides
- `App URL:` — passed as `<start-url>` to the script
- `Task:` — informs caption text and flow naming

### What the user does NOT provide
- No video file (Playwright records it)
- No step descriptions (clicks are auto-captured)
- No coordinate guesses (real `getBoundingClientRect()` calls)

### Why coordinates are pixel-perfect
- Viewport is exactly 1920×1080
- `recordVideo.size` is exactly 1920×1080
- `getBoundingClientRect()` returns CSS pixel coordinates in the same 1920×1080 space
- Three coordinate spaces collapse into one — no scaling, no math, no estimation

### Notes for Claude
- After `touch .ir-stop`, **wait until the background task reports completion** before reading the manifest. The `.webm` is finalised on context close, which takes 1–3s.
- If the user closes the Chromium window manually, the script handles that path via `page.once('close')`. Manifest is still written.
- The recorder rejects clicks covering >90% of the viewport (background clicks) and clicks within 200ms of the previous one (double-click noise).
- If a recording produces zero usable steps (user only navigated, never clicked), offer to re-run rather than building an empty composition.

After completion, read the manifest:

```bash
cat public/recordings/<flow-slug>/manifest.json
```

Report:

```
Manifest read: public/recordings/<flow-slug>/manifest.json
Recorded: <duration>s at 1920×1080
Steps: N captured

  1  [click] @0.0s   <label>      → x:..., y:..., w×h
  2  [click] @2.1s   <label>      → ...
  ...
```

Captured `label` is element text or aria-label — useful but not viewer-friendly. Captions are empty in the manifest. Continue to the **Caption Gate**.

---

## Caption Gate

Both entries land here with a `manifest.json`. Refine captions before generating the composition:

- Combine multiple captured clicks into one caption phase when they're part of one logical step (e.g. open dropdown + select option = one caption "Set priority to High")
- Write captions in the second person, present tense: "Click 'Add new issue' to start creating one."
- Keep captions ≤ 90 chars where possible — typewriter runs at ~40 chars/sec

Show the proposed caption + highlight schedule and gate before generating files:

```
Captions (N phases):
  0–13.1s    Linear's board organises every issue by workflow status...
  13.1–16.2s Click 'Add new issue' to create one in the current column.
  ...

Highlights (N regions):
  h-add-issue     13.1–16.2s   x:611  y:769  w:320  h:28
  h-title         16.2–21.5s   x:604  y:198  w:712  h:24
  ...

Does this look right?
```

Await:
- ✅ **APPROVED** — Proceed to TTS audio + composition
- 🔄 **REVISE** — apply specific edits, re-present
- ❌ **RESTART** — re-record

---

## Step 3: macOS TTS Audio (recommended)

Generate one narration audio file per step. Narration text is the step's `caption`.

### Voice selection

Default: `Samantha (Enhanced)` — clear, professional US English. No API key required.

```bash
say -v '?' | grep "en_US"   # list voices (reference only)
```

### Generate AIFFs in parallel, convert to MP3

```bash
mkdir -p audio/<flow-slug> public/audio/<flow-slug>

say -v "Samantha (Enhanced)" -o audio/<flow-slug>/step-01.aiff "Step 1 caption text" &
say -v "Samantha (Enhanced)" -o audio/<flow-slug>/step-02.aiff "Step 2 caption text" &
# ... one per step
wait

# macOS say outputs 22050Hz mono — re-encode to 44100Hz stereo for Remotion Studio
for i in $(seq -w 1 <N>); do
  ffmpeg -y -i audio/<flow-slug>/step-$i.aiff \
    -ar 44100 -ac 2 -codec:a libmp3lame -qscale:a 2 \
    public/audio/<flow-slug>/step-$i.mp3 -loglevel error
done
```

### Measure durations

```bash
for i in $(seq -w 1 <N>); do
  printf "step-%s: " $i
  ffprobe -v error -show_entries format=duration \
    -of default=noprint_wrappers=1:nokey=1 \
    public/audio/<flow-slug>/step-$i.mp3
done
```

### Audio timing rules

| Step type | Audio wait | Why |
|---|---|---|
| `mark()` / `none` | `audioDurationMs + 800` | Audio must finish before next step |
| `select()` / `click()` (interaction) | `400` | React settles quickly; audio overlaps the next `mark()` |
| `mark()` after interaction | `audioDurationMs + 800` | Audio for the result caption |
| Final `mark()` | `audioDurationMs + 1500` | Extra tail at the end |

**Workflow with audio (scripted-recording):**
1. First pass: write the recording script with placeholder `HOLD_STEP = 4000`
2. Run dry-run to get caption text into `flow.ts`
3. Generate audio and measure durations
4. Replace `HOLD_STEP` with per-step values: `audioDurationMs[i] + 800`
5. Re-run the recording with audio-informed timing

**Workflow with audio (interactive-recording):** the recording is already done — generate audio after Caption Gate, then sequence audio in the Remotion composition by `captionStart` (no second recording pass needed).

---

## Step 4: Recording Mode Remotion Composition

After the manifest is approved, create a single `VideoComposition.tsx`.

### `msToFrame` utility

```typescript
// src/flows/<flow-slug>/remotion/msToFrame.ts
const FPS = 30;
export const msToFrame = (ms: number): number => Math.round(ms / 1000 * FPS);
```

### Timed step data

```typescript
// src/flows/<flow-slug>/data/flow.ts
import { msToFrame } from '../remotion/msToFrame';

export interface TimedStep {
  id: string;
  label: string;
  caption: string;
  startMs: number;
  overlayDelayMs: number;     // how long after startMs before overlay appears
  overlayDurationMs: number;
  audioDurationMs: number;    // measured from ffprobe
  highlight?: { x: number; y: number; w: number; h: number };
}

// Coordinates are already in 1920×1080 space — no scaling needed.
export const TIMED_STEPS: TimedStep[] = [
  {
    id: 'step-01',
    label: 'Overview',
    caption: '...',
    startMs: 0,
    overlayDelayMs: 200,
    overlayDurationMs: 2800,
    audioDurationMs: 3200,
    highlight: { x: 36, y: 78, w: 101, h: 65 },
  },
  // ...
];

// Derive frame numbers from timestamps — no hardcoding needed
export const STEPS_WITH_FRAMES = TIMED_STEPS.map(s => ({
  ...s,
  overlayStart:    msToFrame(s.startMs + s.overlayDelayMs),
  overlayDuration: msToFrame(s.overlayDurationMs),
  captionStart:    msToFrame(s.startMs),
  captionDuration: msToFrame(s.overlayDelayMs + s.overlayDurationMs),
  audioDuration:   msToFrame(s.audioDurationMs),
}));
```

### `RecordingCaption.tsx`

**Do not use sequenced `<Caption>` components.** Mounting a new `Caption` at each step boundary triggers the slide-up entrance repeatedly, and the small inter-step gap causes the bar to flash off and on — visible as glitching on every transition.

Use a single persistent component that stays mounted for the full video duration:

```tsx
// src/flows/<flow-slug>/remotion/RecordingCaption.tsx
import React from 'react';
import { useCurrentFrame, interpolate } from 'remotion';
import { STEPS_WITH_FRAMES } from '../data/flow';

// Never unmounts. Finds the active step from the frame and restarts
// the typewriter at each step boundary without any slide-up animation.
export const RecordingCaption: React.FC = () => {
  const frame = useCurrentFrame();

  // Skip steps where audio doesn't fit in the caption window —
  // they are brief interaction steps (~450ms) with no room for narration.
  const step = [...STEPS_WITH_FRAMES].reverse().find(
    s => frame >= s.captionStart && s.audioDuration <= s.captionDuration
  );
  if (!step) return null;

  const localFrame = frame - step.captionStart;
  const { caption } = step;
  const charCount = Math.floor(
    interpolate(localFrame, [0, caption.length * 0.75], [0, caption.length], {
      extrapolateLeft:  'clamp',  // required — without this, negative charCount causes slice(0,-N)
      extrapolateRight: 'clamp',  // which returns nearly the full string, flashing before typewriter
    })
  );
  const opacity = interpolate(frame, [step.captionStart, step.captionStart + 10], [0, 1], {
    extrapolateLeft: 'clamp', extrapolateRight: 'clamp',
  });

  return (
    <div className="tut-caption" style={{ opacity }}>
      {caption.slice(0, charCount)}
    </div>
  );
};
```

### `HighlightBox.tsx`

```tsx
// src/flows/<flow-slug>/remotion/overlays/HighlightBox.tsx
import React from 'react';
import { useCurrentFrame, interpolate } from 'remotion';

interface Props { x: number; y: number; w: number; h: number; showAt: number; }

export const HighlightBox: React.FC<Props> = ({ x, y, w, h, showAt }) => {
  const frame = useCurrentFrame();
  const opacity = interpolate(frame, [showAt, showAt + 10], [0, 1], {
    extrapolateLeft: 'clamp', extrapolateRight: 'clamp',
  });
  return (
    <div
      className="tut-highlight"
      style={{ left: x, top: y, width: w, height: h, opacity }}
    />
  );
};
```

### `VideoComposition.tsx`

```tsx
// src/flows/<flow-slug>/remotion/VideoComposition.tsx
import React from 'react';
import { AbsoluteFill, OffthreadVideo, staticFile, Sequence, Audio } from 'remotion';
import { HighlightBox } from './overlays/HighlightBox';
import { RecordingCaption } from './RecordingCaption';
import { STEPS_WITH_FRAMES } from '../data/flow';

export const VideoComposition: React.FC = () => (
  <AbsoluteFill>
    {/* Continuous Playwright recording — real interactions, no jump cuts */}
    <OffthreadVideo
      src={staticFile('recordings/<flow-slug>/recording.webm')}
      style={{ width: '100%', height: '100%' }}
      muted
    />

    {/* Highlight ring per step — sequenced per interaction */}
    {STEPS_WITH_FRAMES.filter(s => s.highlight).map(step => (
      <Sequence key={`hl-${step.id}`} from={step.overlayStart} durationInFrames={step.overlayDuration}>
        <HighlightBox {...step.highlight!} showAt={0} />
      </Sequence>
    ))}

    {/* Audio narration — starts at captionStart, runs for captionDuration */}
    {STEPS_WITH_FRAMES.map((step, i) => (
      <Sequence key={`audio-${step.id}`} from={step.captionStart} durationInFrames={step.captionDuration}>
        <Audio src={staticFile(`audio/<flow-slug>/step-${String(i + 1).padStart(2, '0')}.mp3`)} />
      </Sequence>
    ))}

    {/* Single persistent caption — never mounts/unmounts, no glitch between steps */}
    <RecordingCaption />
  </AbsoluteFill>
);
```

Register in `src/Root.tsx` (or `src/remotion-root.tsx`):

```tsx
import { msToFrame } from './flows/<flow-slug>/remotion/msToFrame';

// videoDurationMs comes from the manifest
<Composition
  id="RecordingTutorial"
  component={VideoComposition}
  durationInFrames={msToFrame(videoDurationMs)}
  fps={30}
  width={1920}
  height={1080}
/>
```

### Overlay timing rules

| Step type | `overlayDelayMs` | `overlayDurationMs` |
|---|---|---|
| `mark` (orientation) | 200 | until next step − 300 |
| `click` | 400 — after click settles | until next step − 300 |
| `select` | 300 — after value updates | until next step − 300 |
| `fill` | 0 — as typing begins | until next step − 300 |
| Last step | 200 | remaining duration − 200 |

---

## Gate D — Testing Phase

**Do not start the render until this gate passes and the user confirms.**

### 1. TypeScript type-check

```bash
npx tsc --noEmit
```

All generated files (`data/flow.ts`, `remotion/msToFrame.ts`, `remotion/RecordingCaption.tsx`, `remotion/VideoComposition.tsx`, `remotion/overlays/HighlightBox.tsx`) must compile without errors.

### 2. Extract verification stills

Pick one frame from each step's overlay window (mid-overlay = `step.overlayStart + step.overlayDuration / 2`):

```bash
npx remotion still src/remotion-root.tsx RecordingTutorial \
  --frame=<mid-overlay-frame> \
  --output=public/recordings/<flow-slug>/verify-step<N>.png
```

Open them all:

```bash
open public/recordings/<flow-slug>/verify-step*.png
```

### 3. Visual verification

For each still, confirm:
- Highlight box sits on the correct element (compare against the recording at the same timestamp)
- Caption is showing the expected text and is fully typed by mid-overlay
- Audio is sequenced at the right step (`step-NN.mp3` matches caption N)

If a highlight is off, the manifest coordinates are wrong — re-record rather than try to patch coords by hand. The whole point of recording-mode is that coords come straight from the browser.

### 4. Present testing summary

```
Testing phase complete

TypeScript: ✅ 0 errors

Verification stills:
  Step 1  verify-step1.png  ✅ overlay correct
  Step 2  verify-step2.png  ✅ highlight on 'Add new issue'
  Step 3  verify-step3.png  ✅ caption matches step
  ...

All N steps verified.

Ready to render? (yes / adjust first)
```

Do not proceed until the user explicitly confirms "go".

---

## Render

### Remotion entry file requirement

The entry file passed to `remotion render` / `remotion still` **must call `registerRoot()`** — exporting the root component is not enough:

```tsx
// src/remotion-root.tsx
import { registerRoot } from 'remotion';
const RemotionRoot = () => <><Composition .../></>;
registerRoot(RemotionRoot); // ← required
```

### Full render

**Always run from the project root directory** — Remotion resolves `public/` and `remotion.config.ts` relative to the CWD. Running from a different directory causes `staticFile()` URLs to 404 silently.

```bash
cd /path/to/project   # ← must be the project root, not a parent directory
node_modules/.bin/remotion render src/remotion-root.tsx RecordingTutorial \
  output/<flow-slug>-<YYYYMMDD>.mp4 \
  --codec=h264 \
  --crf=18
```

---

## Output Structure

```
scripts/
├── playwright-capture.ts         ← reusable recording helper (scripted-recording)
├── recording-setup.ts            ← reusable auth/setup helper (scripted-recording)
├── interactive-record.ts         ← reusable interactive recorder (interactive-recording)
└── <flow-slug>-recording.ts      ← flow-specific script (scripted-recording only)

public/recordings/<flow-slug>/
├── recording.webm                ← Playwright recording at 1920×1080
├── manifest.json                 ← timed steps + highlight coords + duration
├── storage-state.json            ← cookies + localStorage from setup phase (scripted-recording)
├── start-url.txt                 ← URL captured at end of setup phase (scripted-recording)
├── .ir-start                     ← signal file (transient — both entries)
└── .ir-stop                      ← signal file (transient — interactive-recording)

public/audio/<flow-slug>/
├── step-01.mp3
├── step-02.mp3
└── ...

src/flows/<flow-slug>/
├── data/
│   └── flow.ts                   ← TIMED_STEPS + STEPS_WITH_FRAMES
└── remotion/
    ├── msToFrame.ts
    ├── RecordingCaption.tsx
    ├── VideoComposition.tsx
    └── overlays/
        └── HighlightBox.tsx

output/
└── <flow-slug>-<YYYYMMDD>.mp4   ← final render
```

No `components/` directory is ever created. Both entries produce the same single-composition shape — no per-scene files, no component discovery, no coordinate scaling.

---

## CSS Architecture Rule — MANDATORY

All CSS lives in `src/index.css`. No exceptions.

- **Never** write inline visual styles in overlay components
- `style={{}}` only for animated values driven by `useCurrentFrame()` / `interpolate()` / `spring()`, plus the absolute positioning of `HighlightBox`
- Check `src/index.css` for existing `.tut-*` classes before adding new ones

Add these to `src/index.css` under `@layer components` if not already present:

```css
/* ── Playwright Tutorial Overlays ──────────────────────────────── */
.tut-caption   { @apply absolute bottom-0 left-0 right-0 flex items-center px-8 font-sans; background: rgba(0,0,0,0.75); height: 64px; color: white; font-size: 1.125rem; }
.tut-highlight { @apply absolute rounded-lg pointer-events-none; border: 2px solid #3B82F6; background: rgba(59,130,246,0.15); }
```

---

## Final Checklist

### Setup
- [ ] Preflight passed — Playwright, tsx, Remotion verified

### Scripted Recording (scripted-recording entry)
- [ ] Setup phase decision — `auto` (default) or `skip` confirmed at activation
- [ ] **If `Setup: auto`:** `scripts/recording-setup.ts` written, run as background task, user confirmed starting page → `.ir-start` written → `storage-state.json` + `start-url.txt` present
- [ ] `scripts/playwright-capture.ts` written to project (or already present)
- [ ] `scripts/<flow-slug>-recording.ts` generated and reviewed by user
- [ ] Recording script run — `recording.webm` and `manifest.json` confirmed present

### Interactive Recording (interactive-recording entry)
- [ ] `scripts/interactive-record.ts` written to project (or already present)
- [ ] Recorder running as background task: `npx tsx scripts/interactive-record.ts <slug> <url>`
- [ ] User logged in / navigated to start page → `.ir-start` written → recording phase active
- [ ] User completed flow → `.ir-stop` written → background task reports completion
- [ ] `recording.webm` and `manifest.json` confirmed at `public/recordings/<flow-slug>/`

### Caption Gate
- [ ] Captions refined into viewer-friendly second-person prose
- [ ] Highlight schedule reviewed
- [ ] User approved schedule before generating composition

### Audio + Composition
- [ ] TTS audio generated for every caption — `public/audio/<flow-slug>/step-NN.mp3`
- [ ] Audio durations measured via ffprobe and stored in `flow.ts`
- [ ] `data/flow.ts`, `msToFrame.ts`, `RecordingCaption.tsx`, `VideoComposition.tsx`, `HighlightBox.tsx` generated
- [ ] Composition registered in `src/Root.tsx` / `src/remotion-root.tsx`
- [ ] `.tut-*` overlay classes present in `src/index.css`

### Testing (Gate D — must complete before render)
- [ ] TypeScript type-check passed — 0 errors
- [ ] Verification still extracted for every step
- [ ] All highlights and captions confirmed correct
- [ ] User confirmed "go" before render started

### Render
- [ ] Full render completed
- [ ] Output: `output/<flow-slug>-<YYYYMMDD>.mp4`
