### 🔬 Browser-only Insight demo
A zero-backend Pareto explorer lives in
`demos/alpha_agi_insight_v1/insight_browser_v1/`.

## Prerequisites
- **Node.js ≥20** is required for offline PWA support and by `manual_build.py`  
  to generate the service worker.
- **Python ≥3.11** is required when using `manual_build.py`.

Verify your Node.js version before running the build script:

```bash
node --version
```
The output must start with `v20` or higher. Only run `manual_build.py` when this
requirement is met.

## Environment Setup
Copy [`.env.sample`](.env.sample) to `.env` then review the variables:

- `PINNER_TOKEN` – Web3.Storage token used to pin results.
- `OPENAI_API_KEY` – optional OpenAI key for chat prompts.
- `IPFS_GATEWAY` – base URL of the IPFS gateway used to fetch pinned runs.
- `OTEL_ENDPOINT` – OTLP/HTTP endpoint for anonymous telemetry (leave blank to disable).
- `WEB3_STORAGE_TOKEN` – build script token consumed by `npm run build`.

See [`.env.sample`](.env.sample) for the full list of supported variables.

## Build & Run
Before running the build script you **must** replace the placeholder
WebAssembly artifacts. Fetch the real assets with:

```bash
python ../../../scripts/fetch_assets.py
```

Once the wasm files are in place run:
```bash
npm ci           # deterministic install
npm run build    # compile to dist/ and embed env vars
```
The build script reads `.env` automatically and injects the values into
`dist/index.html` as `window.PINNER_TOKEN`, `window.OPENAI_API_KEY`,
`window.IPFS_GATEWAY` and `window.OTEL_ENDPOINT`.
The unbuilt `index.html` falls back to `'self'` for the IPFS and telemetry
origins, but running `npm run build` (or `python manual_build.py`) replaces
these defaults with the real values from `.env`.
Place the Pyodide 0.25 files in `wasm/` before building. The script copies them
to `dist/wasm` so the demo can run offline. When preparing the environment
offline run:

```bash
python ../../../scripts/fetch_assets.py
```

This downloads the Pyodide runtime and `wasm-gpt2` model from IPFS into
`wasm/` and `wasm_llm/`.
It also retrieves `lib/bundle.esm.min.js` from the mirror. The build and
`manual_build.py` scripts scan every downloaded asset for the word
`"placeholder"` and abort when any file still contains that marker.
Run `scripts/fetch_assets.py` if you encounter this error.
```bash
PINNER_TOKEN=<token> npm start
```
`npm start` serves the `dist/` folder on `http://localhost:3000` by default.
Set `PINNER_TOKEN` to your [Web3.Storage](https://web3.storage/) token so
exported JSON results can be pinned.

If `OPENAI_API_KEY` is saved in `localStorage`, the demo uses the OpenAI API for
chat prompts. When the key is absent a lightweight GPT‑2 model under
`wasm_llm/` runs locally.

Open `index.html` directly in your browser or pin the folder to IPFS
(`ipfs add -r insight_browser_v1`) and share the CID.
The URL fragment encodes parameters such as `#/s=42&p=120&g=80`.

## Manual Build
Use `manual_build.py` for air‑gapped environments:

1. `cp .env.sample .env` and edit the values if you haven't already.
2. `python ../../../scripts/fetch_assets.py` to fetch Pyodide and the GPT‑2 model.
   The build scripts verify these files no longer contain the word `"placeholder"`.
3. Confirm `node --version` reports **v20** or newer. `manual_build.py` exits if
   Node.js is missing or too old.
4. `python manual_build.py` – bundles the app, generates `dist/sw.js` and embeds
   your `.env` settings.
5. `npm start` or open `dist/index.html` directly to run the demo.

The script requires Python ≥3.11. It loads `.env` automatically and injects
`PINNER_TOKEN`, `OPENAI_API_KEY`, `IPFS_GATEWAY` and `OTEL_ENDPOINT` into
`dist/index.html`, mirroring `npm run build`.

## Toolbar & Controls
- **CSV** – export the current population as `population.csv`.
- **PNG** – download a `frontier.png` screenshot of the chart.
- **Share** – copy a permalink to the clipboard. When `PINNER_TOKEN` is set,
  exported JSON is pinned to Web3.Storage and the CID appears in a toast.
- **Theme** – toggle between light and dark mode.

Drag a previously exported JSON state onto the drop zone to restore a
simulation.

## Darwin-Archive
Every completed run is stored locally using IndexedDB. The **Evolution** panel
lists archived runs with their score and novelty. Click **Re-spawn** next to a
row to restart the simulation using that seed.

## Arena & Meme Cloud
The **Arena panel** allows quick debates between roles on any candidate in the
frontier. Results appear ranked within the panel. The **Meme Cloud** below the
archive table visualizes common strategy transitions across runs.

## Running a Simulation
Use the **Simulator** panel to launch a full run. Adjust the seed list, population
size and number of generations, then press **Start**. When `PINNER_TOKEN` is set,
the resulting `replay.json` is pinned to Web3.Storage and the CID appears once
the run finishes. Keep this CID handy to share or reload the simulation later.

## Load via CID
Append `#/cid=&lt;CID&gt;` to the URL (or use the **Share** permalink) to replay a
previous run. The simulator fetches the JSON from IPFS and populates the chart.

## Privacy
Anonymous telemetry is optional. On first use a random ID is generated and
hashed with SHA-256 using the static salt `"insight"`. Only this salted hash and
basic usage metrics are sent to the OTLP endpoint. Clearing browser storage
resets the identifier.

See **Environment Setup** above for the list of supported variables.

## Safari/iOS Support
Pyodide is disabled on Safari and iOS devices because the runtime fails to load
reliably. The demo automatically falls back to the JavaScript engine instead of
executing Python code in the browser. Expect noticeably slower performance for
LLM tasks and the absence of features that rely on the Python bridge, such as
the local GPT‑2 critic.
