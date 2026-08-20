# Instructor notes — Dark Matter Subhalos tutorial

Private-ish notes for whoever is leading the session. Not part of the
participant-facing flow.

## Session arc (suggested ~45–60 min)

| Time | What | Where |
| --- | --- | --- |
| 0:00 | Motivation: subhalos as a dark matter probe; Σ_sub in lensing (Gilman+2020) | slides / talk |
| 0:05 | Everyone opens a Codespace on `kicp-2026-dark-matter-probes` | GitHub |
| 0:10 | `pip install galacticus==0.9.12`; look at `galacticus info` | `01-setup-and-run.md` §1 |
| 0:15 | Walk through the parameter file; demo schema autocompletion and `Ctrl+K Ctrl+G` docs jump | `01` §3 |
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

**Measured on a 4-core Codespace** (`galacticus run --no-tools …`, wall-clock,
with `galacticus==0.9.12`):

| Parameter file | Time |
| --- | --- |
| `subhalos_1e13_z0.5.xml` (fiducial CDM) | **under 6 min** total: **23 s** of one-time downloads + **5 min 15 s** of model |
| `subhalos_1e13_z0.5_WDM.xml` | **just over 2 min** |
| `subhalos_1e13_z0.5_ludlow.xml` | **5.5 min** |

The fiducial run was ~11 min under `0.9.9` and just under 7 min under `0.9.11`.
Two things bought the difference: `--no-tools` (new in `0.9.11`), and the
first-run download work in `0.9.12` — which is why the download is now a
23-second footnote rather than the headline wait. **The model evolution itself
has not got faster**, so the 5 min 15 s is now the whole story: the knobs below
are the only way to move it.

The two extension timings were measured under `0.9.11` and have **not** been
re-measured under `0.9.12`. They should be unchanged — they reuse the same
download, and `v0.9.11` and `v0.9.12` pin the *same* datasets commit
(`8e0e6bdc`) — but they are inherited numbers, not fresh ones.

**First-run download** (sizes are what crosses the network, taken from the
`v0.9.12` release assets; the unpacked figures are what lands on the Codespace
disk):

| Asset | Download | Unpacked |
| --- | --- | --- |
| `Galacticus.exe` | 290 MB | ~500 MB |
| `datasets.tar.zst` | 193 MB | ~1.7 GB |
| parameter catalog | 2.4 MiB | — |
| **total with `--no-tools`** | **~485 MB** | **~2.3 GB** |
| `tools.tar.zst` — *skipped* by `--no-tools` | 1.83 GB | ~4.1 GB |
| total *without* `--no-tools` | ~2.3 GB | ~6.4 GB |

- Watch the units when quoting these: the `~6 GB` in the pre-`0.9.11` notes was
  the **unpacked on-disk** total, not the download. `--no-tools` now removes
  about **four fifths** of the bytes downloaded (1.8 GB of 2.3 GB) and still
  cuts the disk footprint by about two thirds.
- **The datasets download shrank 1.2 GB → 193 MB in `0.9.12`** without losing
  any data. Earlier releases pulled the `datasets` repository through GitHub's
  on-the-fly zip endpoint, which compresses each file separately; `0.9.12`
  publishes the same tree as a single zstd archive, which finds the redundancy
  *between* files. Same content, same pinned commit — a sixth of the bytes.
- **The old "no percentage bar" caveat no longer applies.** The datasets archive
  is now a release asset with a `Content-Length`, so every component shows a
  real percentage bar. If you see a bar sitting still, that is a stall, not the
  display quirk it used to be.
- For reference, model evolution alone on a 20-core workstation: fiducial
  **7.9 min core-time** (8 trees, mean 60 s/tree), WDM **3.7 min**, Ludlow
  **7.5 min**. Core-time over core count is only a rough guide to Codespace
  wall-clock — Codespace cores are slower — so prefer the measured table above.
- **How we got here:** `treeCount` was reduced 16 → **8** (halves evolution), the
  devcontainer requests a **4-core** machine (`hostRequirements.cpus: 4`), the
  transfer function is the `eisensteinHu1999` fitting formula rather than `CAMB`
  (removes minutes of model setup), Part 1 runs with **`--no-tools`**, and
  `0.9.12` cut the remaining download to 23 s (parallel component downloads,
  multi-connection transfers, and the smaller datasets archive above).
- **`--no-tools`** (new in the `0.9.11` launcher) skips the pre-built tools
  archive — CAMB, CLASS, Cloudy, … — which none of our three models needs. It is
  the single biggest first-run saving we have. The launcher *remembers* the
  choice, so the extension runs stay tool-free too; `galacticus install --tools`
  adds them back if a participant wanders into a model that needs them.
- The download is **not** sped up by cores or fewer trees, but at 23 s it is no
  longer worth engineering around. The old lever — a **Codespaces prebuild**
  running `galacticus install --no-tools` to bake the binary and datasets into
  the image — now saves less than half a minute and costs the "install it live"
  teaching moment. Keep it in your back pocket only for a session on a badly
  degraded network.
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
which Andrew was fixing separately. **Not re-checked against `v0.9.12`**, and it
does not matter here: we keep the fast collisionless setup for the speed reason
above regardless. It would also need `--tools`, since that solver's growth
computation goes through CAMB.)

## Parameter-file migrations: we deliberately do *not* migrate

Between `v0.9.9` and `v0.9.12` three migrations touch our files, and all three
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

**Checked for the `v0.9.11` → `v0.9.12` bump:** `scripts/aux/migrations.xml` is
byte-identical between the two tags, so `v0.9.12` adds no migrations at all, and
the `544daf1f` stamp still stands — the parameter files needed no edit for this
bump. Confirmed directly as well, by running the `v0.9.12` migrator over all
three files: no physics translation in any of them, only the `<lastModified>`
refresh and the same cosmetic re-indentation in `_ludlow` noted above. (The
vendored `.vscode/schema/parameters.xsd` is likewise already byte-identical to
`v0.9.12`; `schema/parameters.xsd` did not change between the two releases.)

Under `v0.9.11` all three files passed `galacticus validate` with no migration,
deprecation, or `unrecognized parameter` warning in any run log. Under `v0.9.12`
only the **fiducial CDM** file has been re-run — it validates and runs clean
(`galacticus run` validates before dispatching). The other two have not been
re-run; given that no migration or schema change separates the releases, expect
them to be clean, but re-run them before a delivery if you want that confirmed.

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
- **The 3D Plotly plot doesn't render**, with *"No renderer could be found for
  mimetype `application/vnd.plotly.v1+json`"*. The **Jupyter Notebook Renderers**
  extension (`ms-toolsai.jupyter-renderers`) is missing. It used to come along
  with `ms-toolsai.jupyter`; it no longer reliably does, so it is now listed
  explicitly in `.devcontainer/devcontainer.json` and `.vscode/extensions.json`.
  Part 2's plot cell also sets
  `pio.renderers.default = "plotly_mimetype+notebook_connected"`, which emits an
  HTML fallback (plotly.js from the CDN) alongside the mime type, so the plot
  should draw even in a Codespace where the extension went missing. If it still
  doesn't, have them install the extension from the Extensions panel and re-run
  the cell.
- **`Ctrl+K Ctrl+G` does nothing in the parameter file.** It needs
  `GalacticusOrg.galacticus-code` **v0.3.0 or later** — the command was
  `.F90`-only before that, so an older build simply ignores the key in an XML
  file. Check the installed version in the Extensions panel. The extension is
  listed in `.devcontainer/devcontainer.json`, so a fresh Codespace gets
  whatever the Marketplace currently serves.
- **"It's stuck."** Since `0.9.12` this is much less likely to be the download,
  which finishes in ~23 s with a real percentage bar on every component — if a
  bar is genuinely not moving, suspect the network rather than the display. The
  long quiet stretch is now the **model run** (~5 min 15 s), which prints little
  while it evolves trees. `galacticus info` shows the cache size growing during
  the download; after that, reassure and continue.
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
- [ ] The Part 1 run command still carries **`--no-tools`** in
      `01-setup-and-run.md` (twice: Step 4 and the quick reference) and in the
      session-arc table above. Dropping it silently adds ~1.8 GB to the
      first-run download (~4 GB on disk) — which, now that the rest of the
      download is only ~485 MB, would be roughly five times the whole transfer.
- [ ] Both notebooks still fall back cleanly when the live output file is locked
      or unfinished (see "Notebook run while the model is still running" above).
- [ ] The Part 2 3D Plotly figure actually renders in a fresh Codespace, and
      `ms-toolsai.jupyter-renderers` is still listed in
      `.devcontainer/devcontainer.json` and `.vscode/extensions.json`.
- [ ] `Ctrl+K Ctrl+G` on `<darkMatterProfileScaleRadius value="johnson2021">`
      opens the docs in a fresh Codespace — i.e. the Marketplace is serving
      `GalacticusOrg.galacticus-code` **>= 0.3.0**. If it is not, drop the
      "Jump to the documentation" bullet from `01-setup-and-run.md` §3 before
      the session rather than demoing something that does nothing.
- [ ] Vendored schema `.vscode/schema/parameters.xsd` isn't badly out of date
      vs the upstream Galacticus release you're demoing (it is a snapshot of
      `schema/parameters.xsd` at the pinned tag, currently `v0.9.12`).
- [ ] The **pinned Galacticus version is consistent everywhere**. It appears in
      `requirements.txt`, `README.md`, `docs/references.md`, the session-arc
      table above, the schema comment in `.vscode/settings.json`, and *four*
      times in `01-setup-and-run.md` (Step 1's command, the "we pin the version"
      paragraph, the 💡 callout, and the quick reference). Check with
      `grep -rn '0\.9\.[0-9]' . | grep -v '^\./\.git/'` — note the deliberately
      **unpinned** file globs: the last bump missed `.vscode/settings.json`
      precisely because the check only looked at `*.md` and `*.txt`.

      Every hit should be either the current pin or a reference that is
      *meant* to name an older release. The latter are: the timings history in
      "Timings", the `--no-tools` "new in `0.9.11`" note, the `544daf1f` stamp
      (a `v0.9.11` commit), and the `Reviewed against Galacticus v0.9.11
      (544daf1f)` comment heading each of the three parameter files — that
      comment pairs with the `<lastModified>` stamp in the same file, so bump
      the two together or neither. We pin so that future
      Galacticus releases cannot break the tutorial — the launcher version
      selects the matching release of the executable *and* datasets. If you bump
      the pin, re-run the three parameter files, refresh the precomputed files in
      `tutorial/data/`, re-execute both notebooks, and re-check the measured
      numbers quoted in these notes.
