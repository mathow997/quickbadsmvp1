# BADs Quick MVP — Decision Log

Rule: append an entry whenever a new call is made or we change path — not once per session.
Format: date · decision · why · what it kills/defers. Newest at the bottom.
Entries stay chronological; the live question is always the last entry.

## Session restart (read this first)

- Workdir: `C:\Users\mhowa\Documents\IdeasGuy\badsdiagrams`; repo = `quickbadsmvp1/` subfolder,
  pushed to `github.com/mathow997/quickbadsmvp1`, branch `main`. Root and repo copies of
  `bads-stage2.html` + `DECISIONS.md` are kept IDENTICAL — edit the repo copy, Copy-Item to root,
  commit, push. Never let them diverge.
- `git` is NOT on PATH. Use the GitHub Desktop bundle:
  `%LOCALAPPDATA%\GitHubDesktop\app-3.6.5\resources\app\git\cmd\git.exe` with `-C quickbadsmvp1`.
- App: open `bads-stage2.html` from disk (file://), load PDFs via picker. No build step.
- Key test files (workdir root): `trial plans.pdf` (Revit A01 1-bed, 1:20 — main sheet),
  `FH-Design-C-NSG_DIP_Final 7.pdf` (1-bed presentation, 1:50),
  `trial plans BADs.pdf` (reference exhibit of correct box placement),
  `Apartment-Design-Guidelines-for-Victoria.pdf` (statutory source).
- Verify before push: inline-script brace/paren balance (PowerShell counter) + every
  getElementById target exists. NO apostrophes in JS comments — the counter treats `'` as a
  string delimiter and lies (see v5 entry).
- Current HEAD: `2ee6d6b` (v5.4). Live question: last entry below.

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

## 2026-09-16 — inspector findings (v1 dumps)

- `trial plans.pdf` (Revit A01, 1:20, A0): 1648 ops · strokes 594 / fills 49 · black-only
  (white fills only) · widths 0.24–1.98pt (thin 0.24–0.43, mid 0.71–0.85, heavy 1.42–1.98) ·
  53 texts, all title block, zero room labels. Walls are outline-stroke pairs; pen weight marks
  cut-vs-projection, NOT wall thickness. Bed detection must be structural (no color separation).
- `FH-Design-C-NSG_DIP_Final 7.pdf` (1-bed presentation, 1:50, A1): 13143 ops · strokes 3722 /
  eoFill 280 · grey palette (grey strokes 85/102/128/179/192, dark fills 25/35/black, white 51) ·
  widths 0.3–4.2pt · 35 texts (title + disclaimer, letter-spaced extraction), no room labels.
  eoFill runs are likely robe/wall hatching — need fill bounding boxes to confirm.
- Consequence: color is a hint on FH but absent on Revit, so the adapter cannot depend on it.
  (Refinement: Revit CAN emit colour — these two sheets just don't. Colour stays demotion-only
  hint, never a primary signal, since its meaning varies per practice.)
  Width counts are `setLineWidth` calls, not strokes — v2 must attribute strokes per width.
  Geometry (segments, pairs, rects, fill boxes) is required next; counts alone can't place boxes.

## 2026-09-16 — inspector v2 (geometry) — pushed as aa17f92 follow-up

- Parses `constructPath(ops, coords)` per pdf.js 3.11.174 svg.js (verified against source):
  tracks save/restore/transform/form-XObject CTM, line width, stroke/fill colour; commits paths
  on stroke/fill/clip/endPath. Outputs: H/V segment split, dark parallel-pair search (60–350mm,
  >50% overlap), closed-4-corner + `rectangle` rect capture with real dims, bed match
  (800–1950 × 1750–2150mm) with bedside-square support count (<800mm), top fills by area.
- Parsed geometry stashed on `window.lastVec` — next step (auto-seed bedrooms from bed rects)
  builds on it, no re-parse.

## 2026-09-16 — trial A01 v2 dump (unpushed; ships with v3)

- 1484 segs (335 H / 390 V, ~759 angled — likely door-swing arcs, future entry-door signal).
- Wall pairs over-fire: H=726/V=866, dominated by 0.24pt hairlines (842/1398 incidences) vs
  1.98pt (221/144). Fix: width-gate pairing to mid+heavy bands; hairlines excluded.
- Black solid fills ARE walls here too: 200×6400, 200×6200, 6600×200 etc. — Revit mixes
  outline pairs and 100–200mm solid poche. Fix: treat dark fills 100–300mm thick as walls directly.
- Bed rects = 0 of 49 rects. Beds are not closed axis-aligned rects in this sheet (rounded
  bedding / unclosed segment groups suspected). Fix v3: print full rect-dim histogram,
  add unclosed 4-segment rect assembly + arc/circle detection before auto-seed.

## 2026-09-16 — FH C-solo v2 dump

- 6763 segs (922 H / 2570 V — V-heavy: balcony decking, per later screenshot).
- Bed rects = 3: 2× 1530×2040 sup8 (queens, exact) + 1× 1266×2112 sup0 — in a 1-bed Type 1A.
  Queens are double-drawn (plan + bedding overlap, per screenshot); dedupe by overlap needed.
- Same hairline over-pairing (0.3pt dominates; 2.1/4.2pt never pair) — so width-gating to heavy
  is WRONG for FH. Fix v3: gap histogram (20mm bins) per direction; wall thicknesses = peaks
  (expect 90/190/270), furniture scatter = background. Width stays out of the decision.
- Colour bug: this sheet emits 0–255 range color args, v2 normC assumed 0–1 (hence 65025 fills).
  Fix v3: values >1 pass through unscaled. Dark detection itself was unaffected (black is 0 either way).
- Top fills are all white/grey room backgrounds; dark fills (25/25/25 etc.) need their own
  listing — v3 prints dark fills separately as wall-poche / robe-hatch candidates.

## 2026-09-16 — v3: assembly, dedupe, gap histogram, auto-seed (built, pushed)

- Unclosed-rect assembly: H endpoints × V endpoints within ~2mm real form corners; 4-corner
  cycles sharing segments become rects (`open:true`). Guarded: skipped past 60M endpoint pairs.
- Dedupe: same dims (±5%/60mm) + >50% overlap = one rect. Beds additionally require support≥2
  (squares 280–650mm within 800mm) — kills double-drawn queens and dining tables per screenshots.
- Gap histogram (20mm bins, H+V pooled) replaces width-gating; peaks = wall thicknesses.
- Dark fills listed separately (poche / robe-hatch candidates); normC passes through >1 values.
- Auto-seed: beds found → boxes centered on up to 3 biggest (size = bed+800mm, ≥ minimums);
  none found → page-fraction fallback. Robes stay fraction-placed (dark-fill seeding is v4).
- Screenshots confirmed: FH has exactly 1 bed (double-draw proved); A01 bed is unclosed
  segments (assembly proved necessary); entry door swing visible (future unit signal);
  balcony decking explains FH V-heavy segments.

## 2026-09-16 — FH v3 dump (unpushed; ships with v4)

- Beds 3→1 (1530×2040 sup4): dedupe + support≥2 work exactly as designed. 1266×2112 sup0
  (table) and 600×1870 sup1 (robe) correctly excluded. Unexplained: 2000×2300 closed sup4.
- normC fix confirmed (fills read 255/25/35 properly). Side effect: 85-grey strokes now count
  dark (555→595 dark H) — consider tightening dark to ≤60 in v4 (walls are black; 85 is furniture).
- Gap histogram is readable-but-flat (60–120 furniture+stud band, 200–220 and 300–320 wall
  bumps). Peaks usable as priors, not sole signal.
- Dark fills ARE walls, directly: 8700×300, 3800×300, 4500×200 etc. v4 fits boxes to these +
  wall pairs instead of bed+800mm guesses.

## 2026-09-16 — A01 v3 dump (unpushed; ships with v4)

- Bed 0→1 (1600×1800 open sup4): unclosed assembly works. 1800 long is short of a real queen
  (2030) — likely bedding inner, not frame. Position right; size still gets dragged. Fine.
- Gap histogram SHARP on Revit: 80–100×297 (stud ~90), 200–240×398 (concrete ~210–230).
  Self-calibration validated — peaks, not widths, carry wall identity on both sheets.
- Dark fills confirm 100mm + 200mm solid walls (6600×200, 5500×100…).
- Dark threshold stays 90 (histogram does the filtering; tightening deferred until furniture
  demonstrably corrupts a fit). 1000×2000 sup0 correctly excluded (wardrobe/desk).
- v4 = wall-fit: grow seeded boxes to nearest enclosing dark lines/fills, cap 6000mm/side,
  keep seeded size when no wall in range (open-plan side). Robe auto-seed deferred.

## 2026-09-16 — bedroom markup rules (from 1BED screenshot, hard constraints)

- INNER WALL FACE IS HARD: box edges stop at the first dark line/fill edge from room center.
  Never pair-midline, never outer face.
- Door openings: wall is interrupted but the box follows the WALL PLANE across the gap —
  so fit to colinear planes with union extents, not nearest segments.
  Door-swing arcs explicitly ignored: tag curve-derived segments, exclude from planes.
- Robe fronts non-critical: near-flush acceptable, minimums met is what counts. No robe
  auto-precision work; fraction defaults + drag stand.
- Wall evidence priority: dark fill edges (unambiguous poche) first, then dark strokes
  ≥0.6pt (excludes 0.24–0.43 furniture hairlines; FH thin outlines covered via its fills).

## 2026-09-16 — v4 built (planes + fit, pushed)

- Curve-derived segments tagged (cu=1), excluded from H/V — hence from pairing, planes and
  assembly. Counted separately (cuN) to preserve the future entry-door signal.
- Planes: dark fill edges (len>200mm) + dark strokes ≥0.6pt, 30mm colinear merge, union extents
  (span openings), support = summed length. Fit needs sup≥1000mm, side ≤4500mm away, axis ≤6000.
- Caught pre-push: span-check vs distance axes were mixed in pick() — separated.
- Auto-measure now seeds→fits→checks in one click; manual "Fit to walls" button added.

## 2026-09-16 — v4.1: off-target box fix (built, pushed)

- Symptom: fitted box landed bottom-left off-sheet on trial A01 despite good seeds/planes.
- Causes addressed: (1) `lastVec` never invalidated — inspecting sheet X then measuring sheet Y
  used X's geometry. Now: render clears it, auto-measure re-parses when stale (keyed
  file|page). Two-step dependency gone. (2) Sheet frame / title rules hijacking planes:
  30mm page-margin exclusion on plane candidates. (3) Failures were silent: fit now records
  per-side plane distances (mm) onto each room, printed in the auto message.
- Added 📋 Copy inspector output (clipboard API + execCommand fallback for file://).

## 2026-09-16 — v4.2: binned planes + position diagnostics (built, pushed)

- Chained 30mm clustering merged everything into 4 mega-planes → replaced with fixed 10mm
  bins (sup<300 dropped). Same change kills cross-page chaining.
- Position diagnostics: bed + near-band rects print sheet fractions (@x%,y-from-top);
  auto message prints seed box positions in canvas %. Next off-target report pinpoints
  detection (wrong @) vs mapping (right @, wrong box) vs fit (right seed, wrong planes).

## 2026-09-16 — v4.3: scope + tolerant corners (built, pushed)

- Root cause of off-target box: detected "bed" sat at sheet (5%,86%) with a sibling at
  (-15%,85%) — legend/off-page junk, not the apartment bed. Fit correctly refused (0/1).
- Fixes: endpoint-to-segment corners (10mm tol; overshoot + T-junction tolerant, replaces
  endpoint-endpoint 2mm); page-bounds clip on corners and rects (kills -15% class);
  beds require wall proximity (<1500mm from a sup≥1000 plane — kills legend class).
- Near-band list marks NOPROX so rejected candidates stay visible for tuning.

## 2026-09-16 — v4.4: enclosure filter + biggest-rects diagnostic (built, pushed)

- Near-band revealed a legend row (six 1600-tall swatches, bottom-left paper) that assembles
  perfectly — shape filters can't beat legends. Replaced proximity with enclosure: bed needs
  walls on 3+ of 4 ray directions (<6m, sup≥1000 planes). Legend in open paper fails it.
- Biggest-12 rects printed with enclosure flags: shows whether the real bed assembles at all
  and which filter rejects it. Caught pre-push: marker still called removed proxOK — fixed.

## 2026-09-16 — v4.5: overlay + tolerant corners (built, pushed)

- Fit detail `[T240 B1960 L1298 R1302]` proved the machine fits fine — it fit the wrong room
  because the seed was legend junk. Enclosure now blocks that class (Bed rects: 0).
- Detection overlay: beds green, bed-band yellow, sup≥1000 planes blue (toggle). One screenshot
  replaces paste cycles from here on.
- Corner tol 2→20mm; corner count printed. Real bed still not assembling — overlay shows where
  its lines go missing.

## 2026-09-16 — v4.6: gap-bridging + corner dots (built, pushed)

- Overlay verdict: legend filtered, bedroom has no yellow at all — bed lines don't meet.
  Fix: merge colinear runs across <100mm gaps before cornering (10mm bins).
- Corners plotted magenta (capped 3000) — next screenshot shows whether the bedroom rings.

## 2026-09-16 — v5: click-seed + local plane support (built, pushed)

- Furniture-corner chasing stalled (bed lines will not meet). Unblocked differently: one click
  in the bedroom seeds a box, fitRoom grows it to inner faces. Auto bed-seed stays where it
  works (FH queens); A01 gets one click instead of dragging four edges.
- Planes now score by member length within 3m of the seed (≥500mm) — distant colinear lines
  no longer hijack fits (the 7m-box class).
- Meta: brace-check false alarm — a `don't` in a comment broke the naive quote-stripper.
  Avoid apostrophes in JS comments or the static check lies.

## 2026-09-16 — v5.2: clip fix + plane listing (built, pushed)

- Overlay verdict: blue misses the bedroom walls entirely. Suspect: I cleared the current path
  on `clip`, but PDF clip does not consume the path — clipped-then-painted walls were dropped.
  Fixed to keep accumulating (matches pdf.js SVG backend behavior).
- Per-plane listing (offset mm from top/left, extent, support) added — next paste shows exactly
  which planes exist and whether bedroom walls are among them.

## 2026-09-16 — v5.3: wall-evidence map (built, pushed as ca5a442)

- Added dark-fills-with-positions (top 20) + heavy-stroke metres per 3×3 cell.
- Finding: ALL dark fills sit bottom-left/off-page; heavy strokes avoid the top row and right
  column entirely — yet the bedroom walls render thick black. Open question (live): what draws
  the bedroom walls with no dark fill and no heavy stroke? v5.4 overlay answers by class.

## 2026-09-16 — v5.4: evidence-class overlay (built, pushed as 2ee6d6b)

- Overlay draws fills red, heavy dark strokes orange, planes blue, beds green, bed-band yellow.
- Purpose: one zoomed screenshot shows which bedroom walls parse as what — no more aggregate
  table guessing. Live: awaiting that screenshot.

## 2026-09-16 — v5.6: ExtGState line widths (built, pushed)

- New suspect for missing bedroom walls: weights set via `/GS gs` (LW in graphics state),
  which the parser ignored — those strokes kept whatever width came before (often 0.24),
  inflating hairline pairs and starving planes. Now LW is extracted (counter printed).
- Raster table ruled out a raster base (8.5% peak = linework density, not solid walls).

## 2026-09-16 — v5.7: room-band diagnostic (built, pushed as b14f105)

- 0 gstate-LW killed the ExtGState theory. Standing model: the apartment is drawn in thin
  lines throughout (outlines + dense hatch reading black); fills and heavies belong to
  legend/off-page content. Prints 2.4–6m enclosed rects — if the bedroom outline assembles,
  v6 seeds rooms from it instead of chasing beds.

## 2026-09-16 — v5.5: raster-or-vector test (built, pushed)

- Zoomed overlay: bedroom walls carry NO overlay class at all — absent from parse, not misplaced.
  Yet they render thick black. Remaining hypothesis: the plan base is a RASTER image
  (1 paintImageXObject on sheet) with vector furniture/title over it.
- Added rasterCheck: downsampled dark-pixel % per 3×3 cell. If top-right is dark in pixels but
  empty in vectors, walls are raster and v6 goes hybrid (raster wall-maps + vector seeds/export).
