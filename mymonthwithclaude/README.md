# mymonthwithclaude

Static page that lets a visitor drop their `~/.claude/projects` folder, get a
LinkedIn-style PNG summary rendered in-browser via Pyodide + matplotlib, and
optionally submit name/email/stats to a Google Form.

## Files

- `index.html` — page layout and form
- `style.css` — black/mint palette matching the rendered graphic
- `config.js` — Google Form ID + entry IDs
- `aggregate.js` — JS port of `aggregate.py` (subset). Iterates JSONL files in
  the picked directory, computes totals + day series.
- `render.js` — Pyodide bootstrap. Loads matplotlib + numpy, mounts
  `build_linkedin.py` and the Claude logo into Pyodide FS, returns a PNG Blob.
- `build_linkedin.py` — sanitized renderer (no hardcoded paths, reads
  `/stats.json` from Pyodide FS, env vars for `DISPLAY_NAME`, `USE_LOGO`,
  `WINDOW_DAYS`, writes `/output.png`).
- `main.js` — wires up the UI.
- `assets/Claude_AI_logo.svg.png` — used in the "Logo" style title row.

## Local development

`showDirectoryPicker()` does not work over `file://`. Serve over http:

```sh
cd /tmp/tikivc        # parent directory
python3 -m http.server 8000
# then open http://localhost:8000/mymonthwithclaude/
```

You also need a Chromium-based browser (Chrome/Edge/Arc/Brave) for the File
System Access API.

## Deploy

Push to `main` on `chandr3w/tikivc` — GitHub Pages auto-deploys at
`tiki.vc/mymonthwithclaude/`.

## Notes / tradeoffs

- Pyodide's first load is ~10MB of WASM + matplotlib. Cached after first visit.
- The renderer uses `sans-serif` / `monospace` fallbacks instead of Inter /
  Andale Mono — installing custom fonts in Pyodide is finicky and not worth the
  payload weight.
- `aggregate.js` only computes the subset of stats `build_linkedin.py` actually
  consumes (totals + day series). The full Python aggregator computes more
  (project breakdowns, session durations, hour buckets, etc.) which we don't
  need here.
- Google Form `fetch` is `mode: 'no-cors'`. Response is opaque; we
  optimistically show "Thanks!" on click.
