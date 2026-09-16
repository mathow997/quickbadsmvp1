# BADs Quick MVP — Decision Log

Rule: append an entry whenever a new call is made or we change path — not once per session.
Format: date · decision · why · what it kills/defers. Newest at the bottom.

## Retro (reconstructed 2026-09-16 from prior sessions)

- 2026-09-1x · Measure actual rooms, not fixed minimum boxes. Boxes snap to wall faces and show
  actual `Wmm x Dmm (m²)`; pass/fail is only a flag. Why: BADs minimums are the test, the room
  is the subject; oversized compliant rooms must read as compliant. Kills: fixed-size place boxes.
- 2026-09-1x · Compliant graphics match the guideline doc (dark dotted outline + caps label);
  only fail state deviates (red dotted + FAIL). Why: output must look like the statutory examples.
  Kills: green/red fill styling.
- 2026-09-1x · Exclusions are a working layer only: hide toggle, never exported. Export = original
  PDF vectors + dotted boxes as the sole addition (pdf-lib overlay). Why: tribunal exhibit must be
  the author's plan plus our measurement, nothing else.
- 2026-09-1x · Auto-first, manual-fallback. `Auto-measure` drops everything; user only drags edges
  or fixes a category when wrong. Why: 1-click is the whole point of stage 2.

## 2026-09-16 — solo / bedrooms / vector pivot

- Solo apartments only; delete the blue-box unit finder (find/prev/next, trace polygon,
  rect→poly, wall-snap poly, lock blues, unit card). Why: new inputs are one apartment per PDF;
  the FH 6-pack rectangles were tuned to a single sheet and piled onto unrelated plans.
  Kills: `units[]`, `activeUnitId`, all polygon-unit code. Old JSON `units` blocks ignored on load.
- Bedrooms only; drop living-room categorising everywhere. Biggest bedroom = Main (3.0×3.4),
  rest = Other (3.0×3.0). Why: scope is bedroom compliance proof. Kills: `living` type, living
  placements, living branches in checks and export.
- Kitchen exclusions removed; robe exclusions stay (BADs counts robes as extra). Why: no open-plan
  kitchen stripping in bedroom-only flow. Kills: `KITCHEN (excluded)` creation; `+ Exclusion`
  defaults to robe.
- No backwards compat with old JSONs. Why: prototype speed; old files were raster-era guesses.
  Loader reads `rooms`/`excls` only and ignores everything else.
- Vector-only for all measurement and logic. Why: prototype for a Revit plugin; walls/beds/scale
  are objects, not pixels — no render-resolution or calibration error. Defers: raster canvas stays
  temporarily as the *viewing* layer until SVG vector display replaces it.
- No Revit view template, no reliance on room tags (practices misuse them for whole-apartment
  grouping). Bedrooms seeded from bed symbols + wall enclosure; walls self-calibrated per sheet
  (top-quartile dark widths; parallel runs within ~300mm merged) so solid-fill / outline /
  layered-construction styles all work.
- Push to `github.com/mathow997/quickbadsmvp1` on every major change. `git` here resolves via
  GitHub Desktop's bundled binary (not on PATH).

## 2026-09-16 — stage-2 strip + vector inspector (built, pushed)

- Rewrote `bads-stage2.html`: no units, no living, no kitchen, PDF-only input, page default 1,
  bedcount select (1/2/3) driving auto-measure, robe-only exclusions, loader drops `living`.
  Raster canvas retained as viewing layer only (deprecated, pending SVG vector display).
- Added vector inspector (`getOperatorList` dump: path-op counts, line-width + color histograms,
  page size, title scale, BED/ROBE/WIR text hits, first 40 text items). Run it on `trial plans.pdf`
  and one FH solo next; thresholds for wall self-calibration come from real data, not guesses.
