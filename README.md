# PulseGrid — Live Link Diagnostics

A single-file, browser-native network performance dashboard styled after Task
Manager's Performance tab. No build step, no dependencies to install — open
`pulsegrid_wifi_monitor.html` in a browser and it runs.

## What it actually measures

Browsers are sandboxed from the OS network stack, so **no web page can read
your real WiFi radio stats** — signal strength/RSSI, SSID, channel, adapter
driver info. That's a security boundary, not something this tool works
around. What it measures instead is all real, live, network-layer data:

- **Latency** — round-trip time to a test endpoint
- **Jitter** — mean variation between consecutive latency samples
- **Packet loss** — % of failed pings in a rolling window
- **Download / Upload throughput** — real multi-threaded transfers against
  a public CDN endpoint (default: `speed.cloudflare.com`)
- **DNS lookup time** — via the Resource Timing API (see caveat below)
- **WebSocket latency** — round-trip time over a persistent WS connection,
  for comparison against plain HTTP latency

## Running it

**Just open the file.** Double-click it, or drag it into a browser tab.
Everything runs client-side — nothing is installed, nothing is sent
anywhere except the live network tests you trigger.

For the PWA install feature specifically (see below), opening the raw file
via `file://` or an embedded preview frame usually won't be enough — serve
it locally instead:

```bash
python3 -m http.server 8000
# then open http://localhost:8000/pulsegrid_wifi_monitor.html
```

## Features

**Core monitoring**
- Live scrolling time-series graphs (60s / 5m / 30m windows) for all six
  metrics, plus per-metric histograms for distribution shape
- Task-Manager-style rail of sparkline tiles, click any to make it the main
  graph
- Connection Stability Score — a weighted heuristic (not a standard
  metric) combining loss, jitter, and latency consistency
- CSV / JSON / PNG export of the full session

**Speed testing**
- Manual on-demand, with optional auto-repeat (off by default — these use
  real bandwidth)
- Multi-threaded (1/4/6 parallel streams) so a single connection's
  congestion window doesn't undersell your real throughput

**Advanced testing**
- Custom ping targets (Cloudflare / Google / any URL you enter)
- DNS resolution timing
- HTTP vs. WebSocket latency comparison (opt-in, uses a free third-party
  echo server)
- Network change alerts (latency spikes, loss thresholds, connection-type
  changes, offline events) — logged in-app always, as OS notifications if
  you grant permission

**Extras**
- Simulated stream buffer panel — models what streaming might feel like
  right now, driven by your real measured throughput/jitter/loss. Downloads
  nothing, plays nothing — illustrative, not diagnostic. Clearly labeled
  `SIMULATED` in the UI.
- Installable PWA shell + optional screen wake lock while monitoring

## Known limitations (read before relying on this for anything important)

- **No WiFi radio access.** See above — this is a hard browser limit, not a
  bug.
- **DNS timing often reads "n/a."** It depends on the target server sending
  a `Timing-Allow-Origin` header. Most servers don't. This isn't broken;
  it's the browser correctly withholding cross-origin timing data.
- **Custom/Google ping targets run in `no-cors` mode.** You get real
  latency timing but no byte-count, because the browser can't read opaque
  cross-origin response bodies.
- **WebSocket comparison depends on a free third-party echo server**
  (`echo.websocket.org`). It can go down or change hands; the tool
  auto-reconnects but can't guarantee uptime that isn't ours to guarantee.
- **The PWA install flow needs a real top-level origin.** Inside a
  sandboxed preview iframe (including most in-chat artifact previews),
  `beforeinstallprompt` and service worker registration typically won't
  fire at all. Open the downloaded file directly, or serve it locally, to
  actually test installability. The panel detects and flags iframe
  contexts on load.
- **"Installable" is not "runs in the background."** No browser gives a
  PWA true persistent background execution once its window is closed —
  that's a platform boundary, not a shortfall of this build. Installing
  gets you an app-like window and slightly better survival while
  minimized, nothing more. For genuine always-on monitoring, run a small
  script on a machine that stays on (cron/systemd), not a browser tab.
- **The stability score and buffer simulation are heuristics/models**, not
  standardized metrics or real playback. Treat them as illustrative.
- **Throughput tests consume real bandwidth.** Auto-repeat defaults off on
  purpose.

## Browser support notes

- `navigator.connection` (Network Information API) is Chromium-only —
  Firefox and Safari will show "unsupported" in the Connection Info panel.
- Wake Lock API support varies; the panel reports "Not supported" plainly
  if your browser lacks it rather than failing silently.
- Everything else (canvas rendering, fetch, WebSocket, Resource Timing,
  Notifications) has broad modern-browser support.

## Privacy

- Public IP lookup is click-to-reveal only — nothing is requested until
  you explicitly ask for it.
- No data leaves your browser except the live test requests you trigger
  (pings, speed tests, DNS/WS checks) and the one-off IP lookup if you use
  it.
- CSV/JSON exports are generated and downloaded locally; nothing is
  uploaded anywhere.
