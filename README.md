# Galacticus Tutorials

Hands-on, interactive tutorials for the [Galacticus](https://github.com/galacticusorg/galacticus)
semi-analytic model of galaxy and dark-matter-halo formation. Each tutorial runs
**entirely in your browser** using a [GitHub
Codespace](https://docs.github.com/en/codespaces/overview) — no local install, no
compiler, nothing to set up in advance.

> **One branch per tutorial.** This `main` branch is just the landing page and a
> shared Codespace skeleton. Each tutorial lives on its own branch — switch to
> the branch for the tutorial you want, then open it in a Codespace.

---

## Tutorials

| Branch | Tutorial | Venue |
| --- | --- | --- |
| [`kicp-2026-dark-matter-probes`](../../tree/kicp-2026-dark-matter-probes) | Dark matter subhalos & the projected subhalo density Σ<sub>sub</sub> | [KICP Workshop: Dark Matter Probes](https://indico.uchicago.edu/event/580/overview) |

---

## How to start a tutorial

1. Use the branch dropdown (top-left of the GitHub file list) to switch to the
   tutorial's branch.
2. Click the green **`< > Code`** button → **Codespaces** tab → **Create
   codespace on _\<branch\>_**.
3. Wait ~1 minute for the container to build; VS Code opens in your browser.
4. Open the tutorial's `tutorial/01-...` file and follow along.

You'll have a full Linux environment with Python, Jupyter, and VS Code, ready to
install and run Galacticus.

## What you'll need

- A (free) GitHub account. The [Codespaces free
  tier](https://github.com/features/codespaces) is plenty.
- A modern web browser. Nothing else.

## Prefer to run locally?

Open any tutorial branch in VS Code with the [Dev Containers
extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)
(**Dev Containers: Reopen in Container**) to reproduce the same environment. The
pip-installed Galacticus binary supports Linux (x86-64) and macOS (Intel & Apple
Silicon).

## Documentation & help

- Galacticus documentation: <https://galacticus.readthedocs.io/>
- Galacticus wiki: <https://github.com/galacticusorg/galacticus/wiki>
- Discussion forum: <https://github.com/galacticusorg/galacticus/discussions>
- Dendros (analysis toolkit): <https://dendros.readthedocs.io/>

---

## Adding a new tutorial (maintainer note)

Branch from `main` (you inherit the `.devcontainer` and `.vscode` skeleton), add a
`tutorial/` directory with your parameter files, notebook, and a numbered
walk-through, then add a row to the table above. Keep participant-facing content
on the branch; keep `main` as the index.
