# Instructor notes — Dark Matter Subhalos tutorial

Private-ish notes for whoever is leading the session. Not part of the
participant-facing flow.

## Session arc (suggested ~45–60 min)

| Time | What | Where |
| --- | --- | --- |
| 0:00 | Motivation: subhalos as a dark matter probe; Σ_sub in lensing (Gilman+2020) | slides / talk |
| 0:05 | Everyone opens a Codespace on `kicp-2026-dark-matter-probes` | GitHub |
| 0:10 | `pip install galacticus`; look at `galacticus info` | `01-setup-and-run.md` §1 |
| 0:15 | Walk through the parameter file; demo schema autocompletion in VS Code | `01` §3 |
| 0:22 | **Kick off the run** (`galacticus run …`) — it runs while you talk | `01` §4 |
| 0:25 | While it runs: recap the physics (orbiting satellites, tidal stripping) | talk |
| 0:32 | Open the notebook; compute Σ_sub | `02-analysis.ipynb` |
| 0:45 | 3D visualisation; discuss scatter, resolution, CDM vs WDM extension | `02` |

**Start the model run before you explain the analysis** — it means the output
is ready by the time you get to the notebook. Anyone whose run is slow uses the
shipped precomputed file (the notebook falls back automatically).

## Timings (measured; re-check on a real Codespace)

Measured on a 20-core workstation with the committed parameter file
(`massResolution = 3e7`, `treeCount = 16`, collisionless solver, CAMB transfer):

- `pip install galacticus`: a few seconds.
- **First** `galacticus run` in a fresh environment pays three one-time costs:
  1. downloading the prebuilt binary (~250 MB) and datasets (~6 GB) — **the
     dominant cost on a Codespace**; budget a few minutes and mind the disk;
  2. the CAMB transfer-function computation: **~50 s** (cached afterward);
  3. tree evolution (below).
- **Tree evolution scales per tree, not per core.** Galacticus parallelises
  *across* trees, so each tree runs on one core (~36 s/tree at `3e7`), and a
  2-core Codespace runs 2 trees at a time. 16 trees ⇒ ~8 batches ⇒ **≈ 5 min**
  (some trees are slower; the slowest single tree we saw was ~110 s).
- The analysis notebook runs in seconds.

**Net: have participants `pip install` and kick off the run early** (during the
parameter-file walk-through). By the time you reach the notebook (~10 min later)
it's done; anyone still waiting uses the shipped precomputed file.

> **Knobs to trade statistics vs. wall-clock** (both in the parameter file):
> `massResolution` (finer = resolves lower-mass subhalos = many more objects;
> must stay **below 1e8** so the 1e8 subhalos are complete — verified complete at
> `3e7`) and `treeCount` (more realisations = smoother Σ_sub, linear in time).
> To go faster: raise `massResolution` toward `1e8` or lower `treeCount`.
> To go faster on *setup*: switch `transferFunction` to `eisensteinHu1999`
> (a fitting formula, no CAMB) — cuts the ~50 s cold-start to ~3 s at the cost of
> a slightly less accurate power spectrum. Fine for this demo.

## Why this file differs from the canonical `darkMatterOnlySubHalos.xml`

The upstream canonical file uses the full baryons + dark-energy structure-
formation physics: `baryonsDarkMatter` linear growth (computed via CAMB) and the
`sphericalCollapseBrynsDrkMttrDrkEnrgy` critical-overdensity / virial-density
solver. **For a dark-matter-only tutorial that has to run live, that machinery is
unnecessarily slow** — the CAMB-based growth computation alone adds several
minutes to first-run model setup. So this file uses the collisionless-matter
spherical-collapse solver (`sphericalCollapseClsnlssMttrCsmlgclCnstnt`) and
analytic `collisionlessMatter` growth: physically appropriate for DMO, and it
keeps setup near-instant. **That speed is the reason we use it.**

(Aside: the canonical baryons+dark-energy solver also currently crashes on the
v0.9.7 pip binary — `Fatal error: initial overdensity of perturbation should be
small` — which Andrew is fixing separately. Even once that's fixed, we keep the
fast collisionless setup here for the speed reason above.)

## The Σ_sub calculation, in one paragraph

Σ_sub is the amplitude of the projected subhalo mass function at a pivot mass of
10⁸ M☉ (Gilman et al. 2020): d²N/(dA dm) = Σ_sub/m₀ (m/m₀)^(−α) with m₀ = 10⁸ M☉.
In the notebook we project the 3D subhalo positions (`positionOrbital*`) onto a
plane, select subhalos in a mass window around 10⁸ M☉ (using **infall mass**,
`basicMass`, following Gilman et al. 2025), count them within a projected
aperture around the host centre, and divide by (area × mass-bin width).
Averaging over the `treeCount` realisations gives Σ_sub and its tree-to-tree
scatter. This is a simplified, pedagogical version of the lensing-region
measurement — good enough to get the number and the intuition.

Talking points the notebook sets up:
- **We get a few × 10⁻³ kpc⁻² (halo-averaged); Gilman et al. 2025 infer ~0.27
  kpc⁻².** The gap is a feature, not a bug — lensing measures the dense central
  region with different mass/area definitions and includes line-of-sight halos.
  It's a natural hook for "how do we compare models to these constraints?"
- **The surviving *infall*-mass function is shallower than ~1.9 (we fit ~1.7)**
  and flattens below 10⁸ M☉ because tidal destruction preferentially removes
  low-mass subhalos. Switching to *bound* mass (a suggested exercise) recovers a
  clean ≈1.9 slope.

## Common gotchas

- **Wrong kernel in the notebook.** VS Code must use the Python interpreter the
  packages were `pip install`ed into. If imports fail, use the kernel picker
  (top-right of the notebook) to select the right interpreter, or restart.
- **`galacticus.hdf5` in the wrong place.** Output lands in the *current working
  directory*. If someone `cd`s elsewhere, the notebook won't find it — the
  notebook searches a few sensible locations and then falls back to the shipped
  file.
- **"It's stuck."** Almost always the first-run download. `galacticus info`
  shows cache size growing. Reassure and continue.
- **Codespace rebuild wipes the binary.** The downloaded binary/datasets live in
  the container's home cache, not the repo, so a fresh Codespace re-downloads on
  first run. That's fine — it's why we start the run early.

## Extensions — Part 3 (`03-extensions.ipynb`)

Two ready-to-run variant parameter files, compared in `03-extensions.ipynb`
(which falls back to shipped precomputed outputs like Part 2):

- **CDM vs WDM** (`subhalos_1e13_z0.5_WDM.xml`) — a 3 keV thermal relic
  (bode2001 transfer + barkana2001WDM barrier + sharp-k window). **Measured:**
  subhalos drop from ~71k (CDM) to ~24k (WDM), and Σ_sub at 10⁸ M☉ falls by
  ~8× (WDM/CDM ≈ 0.13 within R_vir), converging to CDM at high mass — the exact
  effect Gilman et al. use to constrain WDM. The headline dark-matter-probe
  payoff; run it live if time allows (~30 s of evolution). Warmer particle
  (lower `mass`) = deeper suppression.
- **Concentration model** (`subhalos_1e13_z0.5_ludlow.xml`) — swaps the fiducial
  layered scale-radius model for the analytic Ludlow 2016 c(M,z). **Measured:**
  Σ_sub barely changes (0.0019 → 0.0020) — a robustness result worth stating
  (not every ingredient matters as much as the DM particle).

Other quick ideas to suggest verbally:
- **Host mass / redshift dependence.** Vary `massTree` or the base redshift.
- **Radial distribution.** Plot subhalo number vs projected radius.
- **WDM mass scan.** Vary the WDM `mass` (2–10 keV) → this *is* the constraint curve.

## Files to double-check before each delivery

- [ ] All three parameter files still validate, e.g.
      `galacticus validate tutorial/parameters/subhalos_1e13_z0.5.xml`
      (also `_WDM.xml` and `_ludlow.xml`).
- [ ] Precomputed fallbacks present in `tutorial/data/` (fiducial, `_WDM`,
      `_ludlow`) and current.
- [ ] Timing number above is filled in from a real Codespace.
- [ ] Vendored schema `.vscode/schema/parameters.xsd` isn't badly out of date
      vs the upstream Galacticus release you're demoing.
