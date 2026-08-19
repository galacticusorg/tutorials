# Part 1 — Install Galacticus, inspect the model, and run it

**Goal:** by the end of this page you will have installed Galacticus, understood
the parameter file that defines our model, and produced an HDF5 output file full
of dark matter subhalos. Part 2 (the notebook) then analyses that output.

Everything here happens in the **terminal** and **editor** of your Codespace.
Open a terminal with **Terminal → New Terminal** (or `` Ctrl+` ``).

---

## Step 1 — Install Galacticus

Galacticus is on PyPI. Installing it downloads a small launcher; the first time
you *run* a model it fetches a prebuilt binary plus datasets for your platform
(Linux, in the Codespace) — no compiler required.

```bash
pip install galacticus==0.9.11
```

We pin the version deliberately: the launcher version selects the matching
release of the Galacticus executable and datasets (here `v0.9.11`), so this
tutorial gives the same answers no matter when you run it. For your own work,
drop the `==0.9.11` to get the latest release.

Check that the launcher is available and see where it will put things:

```bash
galacticus info
```

You'll see the launcher version and the paths it manages. The executable shows
as *"not present"* — that's expected; it downloads on first run (Step 4).

> 💡 **What just happened?** `pip install galacticus==0.9.11` did *not* compile anything.
> Galacticus is a large Fortran code, but the PyPI package ships a ready-to-run
> binary. If you wanted to modify the source you'd do a full build instead — see
> [`docs/references.md`](../docs/references.md).

## Step 2 — Install the analysis tools

We'll analyse the output with **Dendros** and make an interactive 3D plot with
**Plotly**. Install both now (`nbformat` lets Plotly render inside the notebook):

```bash
pip install 'dendros[pandas,tabulate,plot]' plotly nbformat
```

(You can also install everything for this tutorial in one go with
`pip install -r requirements.txt`.)

## Step 3 — Look at the parameter file

A Galacticus model is defined entirely by an **XML parameter file**. Open ours
in the editor:

```
tutorial/parameters/subhalos_1e13_z0.5.xml
```

Click on it in the file explorer on the left. A few things to notice:

- **It's structured XML.** Each top-level element selects a piece of physics or
  a numerical choice. Elements can nest — for example the dark matter density
  profile is built up from several layered `darkMatterProfileScaleRadius`
  choices.
- **Schema-aware editing.** This repo associates parameter files with the
  Galacticus parameter *schema*, using the Red Hat XML extension (installed for
  you). Try it:
  - Put your cursor inside the `<parameters>` element on a blank line and press
    `Ctrl+Space` — VS Code offers the valid parameter names.
  - Start typing a `value="..."` for something like `<darkMatterParticle>` and
    it will suggest the allowed options (CDM, WDMThermal, …).
  - **Invalid *values* are flagged.** Change
    `<darkMatterParticle value="CDM"/>` to `value="banana"` and a red squiggle
    appears — `banana` isn't in the allowed set. Change it back.
  - Note: the schema is deliberately *permissive about names* — it will happily
    autocomplete element and attribute names, but it won't underline an
    unrecognised name (Galacticus parameter files are too deeply and
    context-dependently nested for name-level validation). The strict check is
    `galacticus validate` (next), which the launcher also runs automatically
    before every run.
- **The physics of *this* model.** Read the comments. The parameters that make
  this the model we want are:

  | Parameter | Value | Meaning |
  | --- | --- | --- |
  | `mergerTreeBuildMasses → massTree` | `1.0e13` | host halo mass, 10¹³ M☉ |
  | `mergerTreeConstructor → redshiftBase` | `0.5` | the host is defined at *z* = 0.5 |
  | `outputTimes → redshifts` | `0.5` | we output the subhalo population at *z* = 0.5 |
  | `mergerTreeMassResolution → massResolution` | `3.0e7` | below 10⁸, so the 10⁸ M☉ subhalos we count are fully resolved |
  | `mergerTreeBuildMasses → treeCount` | `8` | independent realisations of the host, for Σ_sub statistics |
  | `componentSatellite value="orbiting"` | — | each subhalo gets a live position & orbit, with tidal stripping & heating |

  Everything else (cosmology, power spectrum, tidal-stripping calibration, …) is
  the standard, published Galacticus dark-matter-only setup.

> 🎓 **Try editing (optional).** Parameter files are meant to be edited. You
> could change `massTree` to `1.0e14` for a cluster-scale host, or `redshifts`
> to `1.0`. If you experiment, save under a **new filename** so you can always
> get back to the original.

## Step 4 — Run the model

Now run it. From the repository root:

```bash
galacticus run --no-tools tutorial/parameters/subhalos_1e13_z0.5.xml
```

**What is `--no-tools`?** Alongside the executable and datasets, Galacticus
normally downloads an archive of *pre-built tools* — CAMB, CLASS, Cloudy and
friends — which some models call out to. Ours doesn't need any of them (that is
deliberate: our power spectrum uses the Eisenstein & Hu fitting formula, not
CAMB), so `--no-tools` skips that archive and cuts the download from roughly
6 GB to about 2 GB. The launcher **remembers the choice**, so later runs stay
tool-free; if you ever build a model that does need them, add them with
`galacticus install --tools`.

**The first run is the slow one.** Before building any trees, Galacticus does a
one-time download of its binary (~280 MB) and datasets (~1.7 GB) — this download
is the bulk of the wait. After that it builds and evolves each of the 8 merger
trees, following every subhalo's orbit through the host. All told, budget
**~5 minutes** for this first run on the 4-core Codespace (the download
dominates; later runs skip it and are much quicker). It's
a good moment to talk through what the model is doing — **start it early** and let
it run while you read on. If it's still going when you reach Part 2, the notebook
falls back to a shipped precomputed copy automatically.

When it finishes you'll have an output file:

```
subhalos_1e13_z0.5.hdf5
```

Confirm it's there:

```bash
ls -lh subhalos_1e13_z0.5.hdf5
```

> ⏳ **If your run is taking too long** (slow network, busy machine), don't
> worry — we ship a precomputed copy of the output at
> `tutorial/data/subhalos_1e13_z0.5.hdf5`. The notebook in Part 2 will fall back
> to it automatically if your own run isn't ready.

## Step 5 — On to the analysis

Open **[`02-analysis.ipynb`](02-analysis.ipynb)**. When VS Code asks for a
kernel, choose the Python environment you've been installing into. Then run the
cells top to bottom to compute Σ<sub>sub</sub> and visualise the subhalos.

---

### Quick reference — the commands from this page

```bash
pip install galacticus==0.9.11
pip install 'dendros[pandas,tabulate,plot]' plotly
galacticus run --no-tools tutorial/parameters/subhalos_1e13_z0.5.xml
```
