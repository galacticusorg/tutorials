# Dark Matter Subhalos with Galacticus

### A hands-on tutorial for the [KICP Workshop: Dark Matter Probes](https://indico.uchicago.edu/event/580/overview)

In this tutorial you will use [Galacticus](https://github.com/galacticusorg/galacticus)
to model the population of **dark matter subhalos** orbiting inside a
10<sup>13</sup> M<sub>☉</sub> host halo at *z* = 0.5, and then measure
**Σ<sub>sub</sub>** — the projected number density of 10<sup>8</sup> M<sub>☉</sub>
subhalos that appears in strong-lensing dark-matter constraints
([Gilman et al. 2020](https://ui.adsabs.harvard.edu/abs/2020MNRAS.491.6077G)).

Everything runs in your browser in a GitHub Codespace. You will:

1. **Install Galacticus** from PyPI (`pip install galacticus==0.9.12`) — a
   prebuilt binary, no compiler needed. (The version is pinned so the tutorial
   stays reproducible.)
2. **Run a dark-matter-only model** of a single host halo and its subhalos.
3. **Analyse the output** in a Jupyter notebook with
   [Dendros](https://dendros.readthedocs.io/), compute Σ<sub>sub</sub>, and make
   an interactive **3D visualisation** of the orbiting subhalos.
4. **Explore extensions** (optional): compare **cold vs. warm dark matter** —
   the headline probe application, where WDM suppresses low-mass subhalos — and
   test how sensitive Σ<sub>sub</sub> is to the halo concentration model.

---

## Getting started (≈ 1 minute)

1. Make sure you are on the **`kicp-2026-dark-matter-probes`** branch (you are,
   if you are reading this in the Codespace).
2. Open **[`tutorial/01-setup-and-run.md`](tutorial/01-setup-and-run.md)** and
   follow it top to bottom.
3. Then open the notebook **[`tutorial/02-analysis.ipynb`](tutorial/02-analysis.ipynb)**.

> **Not in a Codespace yet?** From the GitHub page for this branch, click
> **`< > Code` → Codespaces → Create codespace on kicp-2026-dark-matter-probes**.
> Wait for VS Code to open in your browser, then come back to step 2. (This repo
> requests a **4-core** machine so the model runs faster — the model run is the
> main wait at ~5 minutes, so start it early.)

## What's in this repository

```
tutorial/
  01-setup-and-run.md          Step-by-step: install, inspect the parameter file, run the model
  02-analysis.ipynb            Notebook: compute Σ_sub + interactive 3D subhalo plot
  03-extensions.ipynb          Notebook: cold vs warm dark matter, and the concentration model
  parameters/
    subhalos_1e13_z0.5.xml         The fiducial CDM model you run in Part 1
    subhalos_1e13_z0.5_WDM.xml     Warm dark matter variant (Part 3)
    subhalos_1e13_z0.5_ludlow.xml  Alternative concentration model (Part 3)
  data/                        Precomputed outputs — fallbacks if a live run is slow
  INSTRUCTOR_NOTES.md          Notes for whoever is leading the session (timings, gotchas)
docs/
  references.md                Where to find documentation, and how to do a full dev install
.vscode/                       Editor config: schema-based highlighting for parameter files
.devcontainer/                 The Codespace definition (Python + Jupyter, nothing else)
requirements.txt               The Python packages this tutorial uses
```

## Documentation & help

- Galacticus docs: <https://galacticus.readthedocs.io/> · wiki:
  <https://github.com/galacticusorg/galacticus/wiki> · forum:
  <https://github.com/galacticusorg/galacticus/discussions>
- Dendros docs: <https://dendros.readthedocs.io/>
- More pointers, and a full source (developer) install:
  [`docs/references.md`](docs/references.md)
