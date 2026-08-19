# Instructor notes — Dark Matter Subhalos tutorial

Private-ish notes for whoever is leading the session. Not part of the
participant-facing flow.

## Session arc (suggested ~45–60 min)

| Time | What | Where |
| --- | --- | --- |
| 0:00 | Motivation: subhalos as a dark matter probe; Σ_sub in lensing (Gilman+2020) | slides / talk |
| 0:05 | Everyone opens a Codespace on `kicp-2026-dark-matter-probes` | GitHub |
| 0:10 | `pip install galacticus==0.9.11`; look at `galacticus info` | `01-setup-and-run.md` §1 |
| 0:15 | Walk through the parameter file; demo schema autocompletion in VS Code | `01` §3 |
| 0:22 | **Kick off the run** (`galacticus run --no-tools …`) — it runs while you talk | `01` §4 |
| 0:25 | While it runs: recap the physics (orbiting satellites, tidal stripping) | talk |
| 0:32 | Open the notebook; compute Σ_sub | `02-analysis.ipynb` |
| 0:45 | 3D visualisation; discuss scatter, resolution, CDM vs WDM extension | `02` |

**Start the model run before you explain the analysis** — it means the output
is ready by the time you get to the notebook. Anyone whose run is slow uses the
shipped precomputed file (the notebook falls back automatically).

## Timings

Committed parameter files use `massResolution = 3e7`, `treeCount = 8`, the
collisionless solver, and the Eisenstein & Hu (1999) transfer function.

| Parameter file | Time on a 4-core Codespace |
| --- | --- |
| `subhalos_1e13_z0.5.xml` (fiducial CDM) | **~5 min**, *including* the one-time downloads — ⚠️ estimate, see below |
| `subhalos_1e13_z0.5_WDM.xml` | **~2 min** |
| `subhalos_1e13_z0.5_ludlow.xml` | **~5–6 min** |

> ⚠️ **The fiducial number is an estimate, not a measurement.** The `~11 min`
> that was measured on a Codespace under `galacticus==0.9.9` was dominated by a
> ~6 GB first-run download. Since we now pass **`--no-tools`** (§ below) that
> download is ~2 GB, so most of that wait is gone — but the new figure has not
> been re-measured on a Codespace. **Re-time it before the next delivery.** The
> two extension numbers are the previous Codespace measurements; they never
> included the download, and the model evolution itself is essentially unchanged
> under `0.9.11`, so they should still hold.

**Measured under `galacticus==0.9.11`** on a 20-core workstation, which is what
the estimate above is scaled from:

| Quantity | Measured |
| --- | --- |
| First-run download, `--no-tools` | 276 MiB executable + 1.7 GiB datasets ≈ **2.0 GB** |
| Tools archive (what `--no-tools` skips) | a further **~4 GB** |
| `subhalos_1e13_z0.5.xml` evolution | 1.1 min wall / **7.9 min core-time** (8 trees, mean 60 s/tree) |
| `subhalos_1e13_z0.5_WDM.xml` evolution | **3.7 min core-time** |
| `subhalos_1e13_z0.5_ludlow.xml` evolution | **7.5 min core-time** |

- Core-time divided by the core count is only a rough guide to Codespace
  wall-clock — Codespace cores are slower, and the old measurements do not scale
  that way cleanly. That is why the fiducial row above is flagged.
- **How we got here:** `treeCount` was reduced 16 → **8** (halves evolution), the
  devcontainer requests a **4-core** machine (`hostRequirements.cpus: 4`), the
  transfer function is the `eisensteinHu1999` fitting formula rather than `CAMB`
  (removes minutes of model setup), and Part 1 now runs with **`--no-tools`**.
- **`--no-tools`** (new in the `0.9.11` launcher) skips the pre-built tools
  archive — CAMB, CLASS, Cloudy, … — which none of our three models needs. It is
  the single biggest first-run saving we have. The launcher *remembers* the
  choice, so the extension runs stay tool-free too; `galacticus install --tools`
  adds them back if a participant wanders into a model that needs them.
- The download itself is **not** sped up by cores or fewer trees — it's the
  irreducible part of the first run. If you ever want to eliminate that wait,
  enable a **Codespaces prebuild** that runs `galacticus install --no-tools`
  (downloads the binary + datasets into the image). That trades away the
  "install it live" teaching moment, so we don't do it by default — but it's the
  lever if the download becomes a problem on the day.
- The analysis notebooks run in seconds.

**Net: have participants `pip install` and kick off the run early** (during the
parameter-file walk-through and while you talk through the physics), and use a
4-core Codespace. Anyone still waiting uses the shipped precomputed files.

> **Knobs to trade statistics vs. wall-clock** (in the parameter file):
> `treeCount` (more realisations = smoother Σ_sub, linear in time — drop to 4–6
> to go faster, raise for smoother statistics) and `massResolution` (finer =
> resolves lower-mass subhalos = many more objects; must stay **below 1e8** so
> the 1e8 subhalos are complete — `3e7` is verified complete; **note that even
> `3e7` makes Σ_sub an underestimate**, see the resolution discussion in Part 2).
> Setup is already near-instant: we use the `eisensteinHu1999` transfer function
> (a fitting formula) rather than `CAMB`, which would add minutes of setup for
> accuracy this tutorial does not need.

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

(Aside: the canonical baryons+dark-energy solver crashed on the v0.9.7 pip
binary — `Fatal error: initial overdensity of perturbation should be small` —
which Andrew was fixing separately. **Not re-checked against `v0.9.11`**, and it
does not matter here: we keep the fast collisionless setup for the speed reason
above regardless. It would also need `--tools`, since that solver's growth
computation goes through CAMB.)

## Parameter-file migrations: we deliberately do *not* migrate

Between `v0.9.9` and `v0.9.11` three migrations touch our files, and all three
exist to **preserve the old behaviour** across a changed default:

| Migration | Would pin |
| --- | --- |
| `vitvitska_subresolution_method` + `…_enum` | `subresolutionAngularMomentumMethod="original"` on `haloAngularMomentumVitvitska2002` |
| `johnson2021_mass_function_slope` | `massFunctionSlopeLogarithmic="-1.8"` on `darkMatterProfileScaleRadius value="johnson2021"` |

The new defaults — the resolution-convergent stochastic angular-momentum
variance, and the unified δ = −1.9 sub-resolution slope — are exactly the
improvements we bumped the pin for, so **we take them**. The files were therefore
*not* put through `parametersMigrate.py`; instead their `<lastModified>` stamp was
advanced by hand to `544daf1f` (`v0.9.11`) with a comment saying why. Verified:
with that stamp the migrator applies no physics translation to any of the three
files (only a timestamp refresh, and some cosmetic re-indentation in `_ludlow`).

⚠️ **If you bump the pin again, re-read the new migrations before running
`parametersMigrate.py`.** Running it blind against a *stale* stamp would quietly
reinstate the old behaviour, and nothing in the run output would tell you.

The runs are clean under `v0.9.11`: all three files pass `galacticus validate`,
and none of the three run logs contains a migration, deprecation, or
`unrecognized parameter` warning.

## Run-to-run variation

Two runs of the same file with the same seed do **not** give bit-identical
output — thread scheduling perturbs the random-number stream. Measured on the
fiducial CDM file: 39055 vs 38859 subhalos (~0.5 %), median concentration 10.44
vs 10.36; Σ_sub agrees to the four decimal places the notebook prints. So expect
a participant's own numbers to differ slightly from the shipped file's, and say
so if anyone asks.

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
- **Notebook run while the model is still running.** Both notebooks handle this:
  a file that Galacticus still holds open cannot be opened for reading (h5py
  raises `BlockingIOError`), and a file left behind by an interrupted or killed
  run opens but has no `statusCompletion`. Either way the notebook prints a
  `skipping …` line and falls through to the shipped copy in `tutorial/data/`,
  so the session keeps moving. If someone reports a `BlockingIOError` or a
  `RuntimeError: … incomplete or missing statusCompletion` traceback, they are
  on an **old copy of the notebooks** — have them pull.
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
  subhalos drop ~39k (CDM) → ~13k (WDM), and Σ_sub at 10⁸ M☉ falls by ~7×
  (WDM/CDM ≈ 0.14 within R_vir), converging to CDM at high mass — the exact
  effect Gilman et al. use to constrain WDM. The headline dark-matter-probe
  payoff; run it live if time allows. Warmer particle (lower `mass`) = deeper
  suppression.
- **Concentration** (`subhalos_1e13_z0.5_ludlow.xml`) — analytic Ludlow 2016
  c(M,z) with normalisation `C` lowered from ~650 to **300**. **Measured:** this
  makes halos clearly less concentrated (median c near 1e8 drops ~10.4 → ~7.0),
  yet Σ_sub barely moves (0.0021 vs 0.0021) — a strong robustness result: unlike
  the DM particle, concentration hardly touches Σ_sub here. The concentration
  plot shows the shift; the Σ_sub table shows the robustness.

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
- [ ] Timings above still hold — re-measure on a Codespace if you change the
      pinned version, `treeCount`, `massResolution`, or the machine size.
      **The fiducial timing is currently an un-re-measured estimate** (see the
      warning in the Timings section) — re-time it.
- [ ] The Part 1 run command still carries **`--no-tools`** in
      `01-setup-and-run.md` (twice: Step 4 and the quick reference) and in the
      session-arc table above. Dropping it silently adds ~4 GB to the first-run
      download.
- [ ] Both notebooks still fall back cleanly when the live output file is locked
      or unfinished (see "Notebook run while the model is still running" above).
- [ ] Vendored schema `.vscode/schema/parameters.xsd` isn't badly out of date
      vs the upstream Galacticus release you're demoing (it is a snapshot of
      `schema/parameters.xsd` at the pinned tag, currently `v0.9.11`).
- [ ] The **pinned Galacticus version is consistent everywhere**:
      `requirements.txt`, `01-setup-and-run.md` (twice), `README.md`, and the
      session-arc table above all say `galacticus==0.9.11`. We pin so that future
      Galacticus releases cannot break the tutorial — the launcher version
      selects the matching release of the executable *and* datasets. If you bump
      the pin, re-run the three parameter files, refresh the precomputed files in
      `tutorial/data/`, re-execute both notebooks, and re-check the measured
      numbers quoted in these notes.
