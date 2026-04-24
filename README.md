# Amplitude Instrumentation Walkthrough

A single-file interactive demo for walking customers through Amplitude SDK instrumentation. Built for use on customer calls — open the HTML file in any browser, paste an API key, and fire live events to Amplitude in real time.

---

## What it is

A guided, step-by-step walkthrough across four Amplitude products, each with a sidebar of numbered steps, working code snippets that update live as you type, and an event stream that shows SDK calls as they happen.

**Products covered:**

| Tab | Steps | Key topics |
|-----|-------|------------|
| Analytics SDK | 8 | Install methods, autocapture, identify, track, revenue, groups |
| Experiment SDK | 8 | Flags vs experiments, providers, remote vs local evaluation, fetch, exposure |
| Guides & Surveys | 7 | Three providers, plugin vs standalone, triggers, targeting, debug |
| Session Replay | 7 | Sample rate, session linking, privacy, masking, install methods |

---

## Usage

No build step, no dependencies, no server required.

**Hosted on GitHub Pages:** [https://amp-anuj.github.io/Amplitude-Instrumentation-Walkthrough/amplitude-sdk-demo.html](https://amp-anuj.github.io/Amplitude-Instrumentation-Walkthrough/amplitude-sdk-demo.html)

```bash
# Clone or download, then just open the file
open amplitude-sdk-demo.html
```

Or serve it locally if you prefer:

```bash
python3 -m http.server 8000
# then open http://localhost:8000/amplitude-sdk-demo.html
```

### On a customer call

1. Open the file in Chrome (recommended — the [Amplitude Chrome Extension](https://amplitude.com/docs/data/chrome-extension-debug) works alongside it for live debugging)
2. Navigate to the **Analytics tab → Step 3: Initialize SDK**
3. Paste the customer's Amplitude API key and click **Initialize SDK**
4. Session Replay starts recording automatically in the background (using the same API key)
5. Walk through each step in order — events fire to the customer's real Amplitude project
6. By the time you reach the **Session Replay tab**, there's already a live recording of the entire walkthrough session

---

## Customizing for a customer

The demo can be tailored for a specific customer — showing only the products they've contracted and pre-selecting the right integration paths. Two ways to do it:

### Option A — Claude skill (recommended)

Install the skill once, then generate customer-specific files on demand before any call.

**Install:**
```bash
mkdir -p ~/.claude/plugins/marketplaces/claude-plugins-official/plugins/amplitude-tools/.claude-plugin
mkdir -p ~/.claude/plugins/marketplaces/claude-plugins-official/plugins/amplitude-tools/skills/instrumentation-walkthrough
```

Copy [`skills/instrumentation-walkthrough/SKILL.md`](skills/instrumentation-walkthrough/SKILL.md) from this repo into:
```
~/.claude/plugins/marketplaces/claude-plugins-official/plugins/amplitude-tools/skills/instrumentation-walkthrough/SKILL.md
```

Also copy [`skills/instrumentation-walkthrough/../.claude-plugin/plugin.json`](.claude-plugin/plugin.json) to:
```
~/.claude/plugins/marketplaces/claude-plugins-official/plugins/amplitude-tools/.claude-plugin/plugin.json
```

Restart Claude Code, then trigger with natural language:
> "build me an interactive demo for Acme Corp"

Claude will look up the customer in Salesforce, confirm the details with you, and generate a pre-configured `acme-corp-demo.html` in your current directory.

### Option B — Manual config

Open `amplitude-sdk-demo.html` and find the `CUSTOMER_CONFIG` block near the top of the `<script>` tag:

```js
const CUSTOMER_CONFIG = {
  customer: "",          // shown in the header — e.g. "Acme Corp"
  products: ["analytics", "experiment", "guides", "session_replay"],
  cdp: "amplitude",     // "amplitude" | "segment" | "rudderstack" | "mparticle"
  apiKey: "",           // pre-filled into all SDK init inputs
};
```

Edit the values, save as `[customer]-demo.html`, and open in Chrome.

**`products`** controls which tabs are shown. Remove any products not on the customer's contract:
- `"analytics"` — Analytics SDK tab
- `"experiment"` — Experiment SDK tab
- `"guides"` — Guides & Surveys tab
- `"session_replay"` — Session Replay tab

**`cdp`** controls which integration paths are pre-selected and which third-party content is shown:
- `"amplitude"` — hides all Segment/standalone paths and third-party content; shows only native Amplitude paths
- `"segment"` / `"rudderstack"` / `"mparticle"` — shows standalone/Segment paths throughout

**`apiKey`** — if provided, pre-fills the customer's API key into all SDK init inputs so you don't need to paste it during the call.

---

## Features

### Analytics tab
- **Three install paths** — Unified Script (CDN), npm (`@amplitude/unified`), Pinned/Minified — with a product support comparison table
- **Interactive autocapture toggles** — flip any of the 9 autocapture options; the code snippet updates live. Reset button reverts to defaults (`autocapture: true`)
- **Live code snippets** — API key input updates all code blocks in real time
- **Identify, Track, Revenue, Groups** — all interactive with try-it cards that fire real SDK calls
- **Session Replay auto-init** — initialized silently when the Analytics SDK loads, recording the entire walkthrough session

### Experiment tab
- **Path selector** — Amplitude Analytics vs Segment (standalone)
- **Two providers explained** — `ExperimentUserProvider` and `ExposureTrackingProvider` with a comparison table across Amplitude, Segment cloud-mode, Segment device-mode, and other CDPs
- **Remote vs Local evaluation** — concept cards and comparison table (latency, cohort support, sticky bucketing, bucketing keys)
- **Full working Segment integration** — loads `analytics.js` dynamically from the Segment CDN, wires both providers, fires `$exposure` automatically via `ExposureTrackingProvider` (no manual tracking needed)
- **`console.log` in snippet** — customers can open DevTools and see `Variant: treatment` or `Variant: control` immediately

### Guides & Surveys tab
- **Path selector** — Browser SDK plugin vs Standalone (Segment)
- **Three providers** — User, Tracking, and Event Provider with an implementation matrix
- **Standalone Segment path** — full `analytics.js` snippet, `analytics.ready()` wrapper, `forwardEvent()` for real-time triggers
- **`engagement.gs.show(key)`** — QA utility to display any guide by key on demand
- **`_debugStatus()`** — run live from the demo to verify SDK initialization state

### Session Replay tab
- **Three install paths** — SR Plugin (Browser SDK), Standalone (Segment), GTM
- **Standalone Segment path** — full `analytics.js` snippet, `analytics.ready()` wrapper, `addSourceMiddleware` to inject `[Amplitude] Session Replay ID` into all Segment events
- **Privacy masking config** — `blockSelector`, `maskSelector`, `defaultMaskLevel`, `amp-block` / `amp-unmask` HTML attributes
- **`getSessionReplayProperties()`** — call live to see the Replay ID for the current session

---

## Credentials used in this demo

The following test credentials are pre-filled in certain steps for demonstration:

| Field | Value | Used in |
|-------|-------|---------|
| Flag key | `instrumentation-demo` | Experiment → Fetch variants |
| Segment Write Key | `6cHFE1dyrahopIUlkvve2xrT3wCokWwU` | Experiment → Segment path |
| Deployment key | `17a54b1a4c34c4447a53c73e6f39bd03` | Experiment → Initialize |

For other tabs, use your own Amplitude API key from any project in your org.

---

## Reference material

Built from four internal instrumentation decks:

- **Analytics SDK** — Browser SDK overview, autocapture vs precision, identify, track, revenue, groups, GTM/Tealium, Ampli
- **Experiment SDK** — Flags vs experiments, deployments, assignment vs exposure, remote vs local evaluation, providers, Segment/RudderStack integration, cohort targeting, holdout groups, sticky bucketing
- **Guides & Surveys** — Three providers, plugin vs standalone, taxonomy considerations, triggers, targeting, platform enablement, CSP, `_debugStatus()`, `forwardEvent()`
- **Session Replay** — Plugin vs standalone vs GTM, mobile paths, sample rate, Session Replay ID, privacy/masking, opt-out, data retention, bot blocking

External docs referenced:
- [Browser SDK 2](https://amplitude.com/docs/sdks/analytics/browser/browser-sdk-2)
- [Browser Unified SDK](https://amplitude.com/docs/sdks/analytics/browser/browser-unified-sdk)
- [Experiment JS Client](https://amplitude.com/docs/sdks/experiment-sdks/experiment-javascript)
- [Session Replay Browser SDK](https://amplitude.com/docs/sdks/session-replay/session-replay-standalone-sdk)
- [Segment Analytics.js Quickstart](https://www.twilio.com/docs/segment/connections/sources/catalog/libraries/website/javascript/quickstart)
- [Amplitude Brand Guide](https://brand.amplitude.com/creative/logo)

---

## File structure

```
amplitude-sdk-demo.html                          # The entire app — single self-contained file
README.md                                        # This file
skills/
  instrumentation-walkthrough/
    SKILL.md                                     # Claude skill for generating customer demos
.claude-plugin/
  plugin.json                                    # Plugin metadata for Claude Code
```

Everything — HTML, CSS, and JavaScript — lives in a single file with no external dependencies beyond Google Fonts (DM Sans) and the Amplitude/Segment CDN scripts loaded at runtime when a key is entered.

---

## Notes

- **The `__inject.bundle.js` error** is cosmetic — thrown by webpack inside the Amplitude CDN bundle when `document.currentScript` isn't available (sandboxed iframes, `file://` protocol). Doesn't affect SDK functionality.
- **Session Replay** requires `sessionReplay` to be bundled in your project's Unified Script. If the plugin isn't available after init, the SR status shows "Use SR tab" and the standalone path on the Session Replay tab can be used instead.
- **Segment paths** require a valid Segment Write Key and a connected Amplitude destination in your Segment workspace to route events through.
