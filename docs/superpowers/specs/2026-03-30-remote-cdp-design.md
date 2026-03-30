# Remote CDP Chrome Support + Lightweight Distribution

## Goal

Fork mermaid-cli to support connecting to a remote Chrome instance via CDP (Chrome DevTools Protocol)
instead of only supporting `puppeteer.launch()`. Additionally, switch to `puppeteer-core` for
lightweight distribution without bundled Chromium (~2MB vs ~300MB).

## Approach: Config-Driven Connect

If the `puppeteerConfigFile` JSON contains a `browserWSEndpoint` key, use `puppeteer.connect()`
instead of `puppeteer.launch()`. This is the minimal-diff approach (~20 lines changed).

## Changes

### `package.json`
- Remove `puppeteer` from `peerDependencies`
- Add `puppeteer-core` to `dependencies`

### `src/index.js`
- Change import from `puppeteer` → `puppeteer-core`
- In `run()`: branch on `puppeteerConfig.browserWSEndpoint` presence
  - If present: `puppeteer.connect(puppeteerConfig)`
  - If absent: `puppeteer.launch(puppeteerConfig)`
- In `finally` block: `disconnect()` for remote, `close()` for local
- Same pattern in the markdown processing path (lazy browser creation)

## Usage

### Remote Chrome (OpenClaw Docker sidecar)
```json
{ "browserWSEndpoint": "ws://chrome-sidecar:9222" }
```

### Local Chrome (no bundled Chromium)
```json
{ "executablePath": "/usr/bin/chromium-browser", "args": ["--no-sandbox"] }
```

## Backward Compatibility

Users who install `puppeteer` (full package) alongside this fork still get the
auto-download Chromium behavior. The `puppeteer-core` API is identical.
