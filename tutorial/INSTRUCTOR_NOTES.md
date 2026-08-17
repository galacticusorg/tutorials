# Instructor notes — Dark Matter Subhalos tutorial

Private-ish notes for whoever is leading the session. Not part of the
participant-facing flow.

## Session arc (suggested ~45–60 min)

| Time | What | Where |
| --- | --- | --- |
| 0:00 | Motivation: subhalos as a dark matter probe; Σ_sub in lensing (Gilman+2020) | slides / talk |
| 0:05 | Everyone opens a Codespace on `kicp-2026-dark-matter-probes` | GitHub |
| 0:10 | `pip install galacticus==0.9.9`; look at `galacticus info` | `01-setup-and-run.md` §1 |
| 0:15 | Walk through the parameter file; demo schema autocompletion in VS Code | `01` §3 |
| 0:22 | **Kick off the run** (`galacticus run …`) — it runs while you talk | `01` §4 |
| 0:25 | While it runs: recap the physics (orbiting satellites, tidal stripping) | talk |
| 0:32 | Open the notebook; compute Σ_sub | `02-analysis.ipynb` |
| 0:45 | 3D visualisation; discuss scatter, resolution, CDM vs WDM extension | `02` |

**Start the model run before you explain the analysis** — it means the output
is ready by the time you get to the notebook. Anyone whose run is slow uses the
shipped precomputed file (the notebook falls back automatically).

## Timings (measured; re-check on a real Codespace)

Committed parameter files use `massResolution = 3e7`, `treeCount = 8`, the
collisionless solver, and the Eisenstein & Hu (1999) transfer function.

**Measured on a real Codespace** (`galacticus run …`, wall-clock):
- **First CDM run, 2-core Codespace, including the download: ~27 min** — too slow.
  This is dominated by the one-time download of the prebuilt binary (~250 MB) and
  datasets (~6 GB) plus tree evolution; Codespace cores are much slower per-core
  than a workstation. The WDM run (later, no download) took **~9 min** and the
  concentration run **~22 min** at `treeCount = 16`.
- **Mitigations we've applied:** `treeCount` reduced 16 → **8** (halves
  evolution), and the devcontainer now requests a **4-core** machine
  (`hostRequirements.cpus: 4`), which roughly halves evolution again. Expect the
  first CDM run to land around **10–15 min** on 4 cores; the WDM and concentration
  runs are shorter (fewer/faster subhalos, and no re-download).
- The download itself is **not** sped up by cores or fewer trees — it's the
  irreducible part of the first run. If you ever want to eliminate that wait,
  enable a **Codespaces prebuild** that runs `galacticus install` (downloads the
  binary + datasets into the image). That trades away the "install it live"
  teaching moment, so we don't do it by default — but it's the lever if the
  download becomes a problem on the day.
- The analysis notebooks run in seconds.
- **Those Codespace timings were measured when the parameter files still used the
  `CAMB` transfer function.** We have since switched to `eisensteinHu1999`, which
  removes the CAMB setup entirely, so the numbers above are upper limits — first-run
  wall-clock should now be a little shorter (the download still dominates). Worth
  re-measuring on a Codespace before the session.

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
  subhalos drop ~39k (CDM) → ~12k (WDM), and Σ_sub at 10⁸ M☉ falls by ~8×
  (WDM/CDM ≈ 0.13 within R_vir), converging to CDM at high mass — the exact
  effect Gilman et al. use to constrain WDM. The headline dark-matter-probe
  payoff; run it live if time allows. Warmer particle (lower `mass`) = deeper
  suppression.
- **Concentration** (`subhalos_1e13_z0.5_ludlow.xml`) — analytic Ludlow 2016
  c(M,z) with normalisation `C` lowered from ~650 to **300**. **Measured:** this
  makes halos clearly less concentrated (median c near 1e8 drops ~10.4 → ~7.3),
  yet Σ_sub barely moves (0.0021 vs 0.0020) — a strong robustness result: unlike
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
- [ ] Timing number above is filled in from a real Codespace.
- [ ] Vendored schema `.vscode/schema/parameters.xsd` isn't badly out of date
      vs the upstream Galacticus release you're demoing (it is a snapshot of
      `schema/parameters.xsd` at the pinned tag, currently `v0.9.9`).
- [ ] The **pinned Galacticus version is consistent everywhere**:
      `requirements.txt`, `01-setup-and-run.md` (twice), `README.md`, and the
      session-arc table above all say `galacticus==0.9.9`. We pin so that future
      Galacticus releases cannot break the tutorial — the launcher version
      selects the matching release of the executable *and* datasets. If you bump
      the pin, re-run the three parameter files, refresh the precomputed files in
      `tutorial/data/`, re-execute both notebooks, and re-check the measured
      numbers quoted in these notes.
