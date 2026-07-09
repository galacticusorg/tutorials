# References, documentation & going further

## Galacticus

| Resource | Link |
| --- | --- |
| Documentation (Read the Docs) | <https://galacticus.readthedocs.io/> |
| Source code & releases | <https://github.com/galacticusorg/galacticus> |
| Wiki (tutorials, how-tos, physics notes) | <https://github.com/galacticusorg/galacticus/wiki> |
| Discussion forum (ask questions here!) | <https://github.com/galacticusorg/galacticus/discussions> |
| Issue tracker | <https://github.com/galacticusorg/galacticus/issues> |
| Description paper (Benson 2012) | <https://arxiv.org/abs/1008.1786> |
| PyPI package | <https://pypi.org/project/galacticus/> |

### The `galacticus` command-line tool

Once `pip install galacticus` is done, the `galacticus` launcher gives you:

| Command | What it does |
| --- | --- |
| `galacticus run <file.xml>` | Validate and run a parameter file (also just `galacticus <file.xml>`) |
| `galacticus validate <file.xml>` | Check a parameter file without running it |
| `galacticus resolve <file.xml> -o <out.xml>` | Expand XInclude/change files into one standalone file |
| `galacticus install` | Pre-download the binary, datasets & tools (otherwise done on first run) |
| `galacticus update` | Re-download binaries for the installed version |
| `galacticus info` | Show install location, environment variables, cache size |
| `galacticus clean` | Remove regenerable cached data |

Pass `--help` to any of these for details. Arguments after the parameter file
are forwarded to the underlying executable.

## Dendros (analysis toolkit)

| Resource | Link |
| --- | --- |
| Documentation | <https://dendros.readthedocs.io/> |
| Source code | <https://github.com/galacticusorg/dendros> |
| PyPI package | <https://pypi.org/project/dendros/> |

Dendros reads Galacticus HDF5 outputs (and MCMC chain logs). The key entry
point is `open_outputs("file.hdf5")`, which returns a `Collection` with methods
like `list_outputs()`, `list_properties()`, and `read()`. Datasets come back as
[`astropy` quantities](https://docs.astropy.org/en/stable/units/) with units
attached.

## The parameter-file schema (editor autocompletion)

Galacticus parameter files are XML, and Galacticus ships an XML Schema
(`schema/parameters.xsd`) generated from its parameter catalog. If you install
the [Red Hat XML extension](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-xml)
(already recommended in this repo) and associate parameter files with the
schema, VS Code gives you **live validation and autocompletion** of parameter
names and allowed values as you type.

This repo wires that up for you in [`.vscode/settings.json`](../.vscode/settings.json),
pointing at a vendored snapshot of the schema in `.vscode/schema/parameters.xsd`.
The upstream, always-current schema lives at
<https://github.com/galacticusorg/galacticus/blob/master/schema/parameters.xsd>.
See the "Editor validation and autocompletion" section of the Galacticus
user-guide for the details.

## Full (developer) install from source

The pip install used in this tutorial gives you a ready-to-run **prebuilt
binary** — perfect for running models. If you want to *modify or extend*
Galacticus (add physics, change the code), you need a source build. In brief you
need a modern Fortran compiler (`gfortran` ≥ 16), `make`, and the GSL, zlib,
HDF5, FoX, and BLAS libraries (HDF5 and FoX built with the same compiler), plus
Python 3. Then:

```bash
git clone https://github.com/galacticusorg/galacticus.git
cd galacticus
# install Python build/support dependencies
pip install -e .
# build the executable
make Galacticus.exe
```

The authoritative, platform-specific instructions (including a ready-made build
environment container image, `ghcr.io/galacticusorg/buildenv`) are in the
[installation guide](https://galacticus.readthedocs.io/en/latest/manuals/user-guide/installation/)
and on the [wiki](https://github.com/galacticusorg/galacticus/wiki). For a
zero-setup source-build environment, the Galacticus repository itself ships a
`.devcontainer` you can open in a Codespace.

## The science: Σ<sub>sub</sub> and dark matter

- Gilman et al. (2020), *Warm dark matter chills out: constraints on the halo
  mass function and the free-streaming length of dark matter with eight rogue
  quasar lenses* — <https://ui.adsabs.harvard.edu/abs/2020MNRAS.491.6077G>.
  Defines Σ<sub>sub</sub>, the projected subhalo number density normalisation at
  10<sup>8</sup> M<sub>☉</sub>, used as a lensing dark-matter probe.
- The subhalo orbital dynamics, tidal mass loss, and tidal heating in the model
  you run come from the `orbiting` satellite component; the relevant
  calibrations are cited inline in the parameter file.
