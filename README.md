# QuickBADs Demo

Measure Better Apartment Design Standards (BADS) bedroom compliance on apartment PDFs:
load a plan → auto-measure or click-seed bedrooms → boxes fit to inner wall faces →
live PASS/FAIL against minima (Main 3.0×3.4, Other 3.0×3.0, robes excluded) →
export a vector PDF exhibit (original plan + dotted boxes only).

## Run it

- Online (GitHub Pages): https://mathow997.github.io/quickbadsmvp1/ — pick a sample thumbnail.
- Locally: open `bads-stage2.html` in a browser (file works; Upload any PDF).
  Sample cards need hosting to fetch — locally, use Upload instead.

## Samples

`samples/` holds demo PDFs (`trial-a01.pdf`, `fh-c-1bed.pdf`, `fh-c-2.pdf`).
`samples/expected/` holds pre-built correct exhibits — the on-rails backup if a live run stalls.

## Fake-it demo mode

The 🎭 Fake it button (`?fake`) turns the app deterministic: the 3 sample plans load
pre-verified boxes instead of detecting anything — always correct, flagged on screen as
"★ On-rails demo result". Uploads and detection settings hide; click tools still work.

Authoring a verified exhibit (do once per sample, in a desktop browser):
1. Load the sample, auto-measure / click-seed / drag everything to correct.
2. Export JSON, rename to `<sample>.json` (e.g. `trial-a01.json`), drop it in
   `samples/expected/`, commit + push. Fake-it picks it up automatically.

## Publish a new version

Push to `main`, then enable once: repo Settings → Pages → Deploy from branch (`main`, `/ (root)`).
Every push after that redeploys automatically.

## Decisions

See `DECISIONS.md` — every call and path change, newest last.
