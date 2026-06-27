---
sources: [summaries/snapshot-2026-04-14T04_21_45_999Z_20260413212342.md, summaries/snapshot-2026-04-14T04_21_45_999Z_20260413212147.md, summaries/entry_points.md, summaries/top_level.md, summaries/requirements.md, summaries/DEPENDENCIES.md, summaries/installation.md]
brief: Managing Python runtimes and dependencies for isolated, reproducible projects.
---

# Python Environment Management

Python environment management encompasses the tools, strategies, and best practices for installing, isolating, and maintaining Python dependencies across projects. For tools like Luria, proper environment management ensures reproducible installations, prevents version conflicts, and supports both end-user and developer workflows. It also matters for small writing-heavy repositories: the `YC-2026` project snapshots in [[summaries/snapshot-2026-04-14T04_21_45_999Z_20260413212147]] and [[summaries/snapshot-2026-04-14T04_21_45_999Z_20260413212342]] show full `.venv` directories committed alongside Markdown application materials, illustrating how even a lightweight personal project may depend on a dedicated Python runtime for editing, notebooks, debugging, or automation.

See [[summaries/installation]] for a full breakdown of how Luria handles environment setup. For a concrete example of a pinned Jupyter/IPython environment, see [[summaries/requirements]].

## Why Environment Management Matters

- **Isolation**: Prevents dependency conflicts between projects
- **Reproducibility**: Ensures consistent behavior across machines and deployments
- **Security**: Limits the blast radius of vulnerable packages
- **Collaboration**: Makes onboarding and contribution straightforward
- **Workflow support**: Keeps notebooks, shells, debuggers, and automation tools aligned with the project they serve

The `YC-2026` snapshots are a useful reminder that environment management is not only for large software systems. A project centered on `application.md`, `README.md`, draft-history files, and personal application prep can still rely on a local Python environment containing Jupyter, IPython, and debugging tools. In practice, this makes environment management part of broader [[concepts/personal-writing-workflows]] and [[concepts/python-project-structure]]. It also overlaps with [[concepts/application-preparation]] when a repository is used to support reflective writing, drafting, and revision rather than product code alone.

## Package Managers

### uv (Recommended)

[uv](https://github.com/astral-sh/uv) is a fast, modern Python package installer and resolver written in Rust. It is the recommended tool for Luria:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
uv add luria
uv add luria@latest   # upgrade
uv remove luria       # uninstall
```

- Significantly faster than pip
- Handles virtual environments automatically
- Compatible with standard `pyproject.toml` and `requirements.txt`

See also [[concepts/uv-workspace-layout]] for project-level uv configuration.

### pip

The standard Python package manager, available everywhere Python is installed. In the requirements snapshot (see [[summaries/requirements]]), `pip==26.0.1` itself is pinned, illustrating that even the installer is version-locked for full reproducibility:

```bash
pip install luria
pip install luria[full]       # with optional extras
pip install --upgrade luria
pip install --user luria      # user-level, no sudo needed
pip uninstall luria
```

The `YC-2026` snapshots also show conventional `venv` layouts with executables such as `pip`, `pip3`, `python`, `python3`, and `jupyter` under `.venv/bin`, reinforcing how pip-centered environments remain the default shape of many local Python projects.

### Poetry

A dependency management and packaging tool with lock file support:

```bash
poetry add luria
poetry add "luria[full]"
```

### conda / mamba

Useful for data science workflows that mix Python and non-Python dependencies. Note that Luria is not yet available on conda-forge, so pip is used inside conda environments:

```bash
conda create -n luria python=3.10
conda activate luria
pip install luria
```

## Virtual Environments

Virtual environments isolate a project's Python interpreter and installed packages from the system Python:

```bash
python -m venv .venv
source .venv/bin/activate          # macOS/Linux
.venv\Scripts\activate             # Windows
```

Best practices:
- Always use a virtual environment for project work
- Add `.venv/` to `.gitignore`
- Use `pip check` to identify dependency conflicts
- Audit dependencies with `pip-audit` or `safety`
- Treat the environment as disposable infrastructure that can be rebuilt from declared dependencies

The `YC-2026` snapshots are especially instructive here. They include:
- a top-level `.venv/`
- a `pyvenv.cfg`
- a complete `site-packages/` tree
- Jupyter/IPython command-line entry points in `.venv/bin`

The newer snapshot, [[summaries/snapshot-2026-04-14T04_21_45_999Z_20260413212342]], makes this even more concrete by showing the full environment beside project materials like `application.md`, `README.md`, `CLAUDE.md`, `GRAYMATTER.md`, `pyproject.toml`, `.history/`, and `.remember/`. This shows a real-world local environment captured as a directory snapshot rather than just an abstract recommendation. It also highlights a common repository hygiene concern: while keeping `.venv` locally is normal, checking the entire environment into source control is usually discouraged unless there is a specific archival or portability reason. See [[concepts/repository-hygiene]].

## Python Version Requirements

Luria requires:
- **Minimum**: Python 3.10
- **Recommended**: Python 3.11+

Check your version:
```bash
python --version
python3 --version
```

The `YC-2026` snapshots demonstrate that some projects may target newer interpreters in practice: their `.venv` contents include Python 3.14 executables and libraries. This underscores an important principle: environment management is not just about package versions, but about pinning the interpreter version itself when reproducibility matters.

## Jupyter / IPython Kernel Environments

A common pattern in this project is a dedicated Jupyter/IPython kernel environment with fully pinned dependencies. The [[summaries/requirements]] file illustrates this pattern with a base kernel stack including:

- **ipython==9.10.0** / **ipykernel==7.2.0** — Interactive shell and kernel
- **jupyter-client==8.8.0** / **jupyter-core==5.9.1** — Messaging and core infrastructure
- **pyzmq==27.1.0** — ZeroMQ inter-process communication
- **tornado==6.5.4** — Async networking backbone
- **traitlets==5.14.3** — Configuration traits system shared across Jupyter components
- **debugpy==1.8.20** — Integrated debugger (used by VS Code and Jupyter)
- **jedi==0.19.2** — Autocompletion and static analysis
- **matplotlib-inline==0.2.1** — Inline plot rendering in notebooks
- **psutil==7.2.2** — System and process monitoring

This environment is intentionally minimal — it provides an interactive computing layer without bundling data science libraries (numpy, pandas, etc.), which are installed separately as needed. Platform-specific packages like `appnope` (macOS App Nap suppression) and `pexpect`/`ptyprocess` (Unix pseudo-terminal control) are also included to ensure cross-platform reliability.

The `YC-2026` snapshots provide concrete filesystem examples of this pattern. Their `.venv` includes:
- `ipython`, `ipykernel`, `jupyter`, and `jupyter-kernel`
- `jupyter_client` and `jupyter_core`
- `debugpy`, `traitlets`, `pyzmq`, `tornado`, and `jedi`
- `matplotlib_inline` and `psutil`
- a shared Jupyter kernelspec under `.venv/share/jupyter/kernels/python3/kernel.json`

The newer snapshot additionally shows how these packages coexist with a writing-oriented repository whose main payload is Markdown and project notes rather than a large application codebase. This shows what a self-contained notebook-capable environment looks like on disk: not just a package list, but the executables, kernel registration files, and supporting libraries that enable interactive development. It also demonstrates overlap with [[concepts/asyncio]], [[concepts/python-networking]], and [[concepts/matplotlib]] through the Jupyter stack.

Key takeaway: **pinning every package, including pip itself, and tracking the interpreter version** is the gold standard for reproducible kernel environments.

## Development Environments

For contributors or users running Luria from source:

```bash
git clone https://github.com/brainworkup/luria.git
cd luria
pip install -e ".[dev]"
pre-commit install
```

The `[dev]` extras install testing, linting, and formatting tools alongside the main package.

More generally, development environments often sit beside ordinary project files. The `YC-2026` snapshots combine a Python environment with `README.md`, `application.md`, `CLAUDE.md`, `GRAYMATTER.md`, `pyproject.toml`, editor history files, and temporary state directories. This is a useful example of how environment management supports mixed workflows that combine writing, planning, personal application preparation, and lightweight automation rather than only application code.

## Docker as an Alternative

Docker provides full environment isolation beyond virtual environments:

```dockerfile
FROM python:3.10-slim
RUN pip install luria
```

This is useful for [[concepts/deployment-automation]] and reproducible CI pipelines.

## Environment Variables and Configuration

Environment management extends to runtime configuration. Luria uses a `.env` file for sensitive settings:

- `ANTHROPIC_API_KEY`, `OPENAI_API_KEY` — LLM provider credentials
- `OMLX_BASE_URL` — local LLM inference endpoint
- `NEUROPSYCH_DB_PATH` — database location
- `R_HOME` — R interpreter path for optional R integration

Never commit `.env` to version control. See [[concepts/security-policy]] and [[concepts/local-first-architecture]] for broader security and infrastructure context.

Environment configuration also includes non-secret project files such as `pyproject.toml`, kernelspecs, and tool-specific metadata. The `YC-2026` snapshots illustrate this broader view well: environment management is distributed across interpreter metadata (`pyvenv.cfg`), package layout (`site-packages`), executable shims (`.venv/bin/*`), and project-level configuration files. In practice, even a small personal repository may accumulate environment state across multiple layers, not just one lockfile.

## Troubleshooting

| Problem | Solution |
|---|---|
| `command not found: luria` | Add Python scripts dir to PATH |
| Permission errors | Use `--user` flag or virtual env |
| Version conflicts | Fresh virtual environment + `pip check` |
| R integration failures | Verify `R_HOME` in `.env` |
| Jupyter kernel not found | Confirm `ipykernel` is installed in the active environment |
| Wrong Python version | Recreate the environment with the intended interpreter |
| Environment too large or accidentally tracked | Remove `.venv/` from version control and rebuild from dependency specs |

## Related Concepts

- [[concepts/python-project-structure]] — How Python projects are organized alongside environment config
- [[concepts/uv-workspace-layout]] — uv-specific monorepo and workspace patterns
- [[concepts/local-llm-inference]] — Local model setup that depends on correct environment configuration
- [[concepts/r-python-integration]] — Mixed R/Python environments requiring additional setup
- [[concepts/deployment-automation]] — Automated environment setup in CI/CD
- [[concepts/personal-writing-workflows]] — Writing-centric projects that still benefit from isolated Python tooling
- [[concepts/repository-hygiene]] — Why virtual environments are usually excluded from version control
- [[concepts/application-preparation]] — Repositories used to support reflective, high-stakes writing workflows
- [[summaries/installation]] — Luria-specific installation instructions
- [[summaries/requirements]] — Pinned Jupyter/IPython kernel dependency snapshot
- [[summaries/DEPENDENCIES]] — Additional dependency documentation
- [[summaries/snapshot-2026-04-14T04_21_45_999Z_20260413212147]] — Real project snapshot with a full `.venv` and writing-oriented repository structure
- [[summaries/snapshot-2026-04-14T04_21_45_999Z_20260413212342]] — Expanded filesystem snapshot showing the same pattern in more detail

See also: [[summaries/top_level]]

See also: [[summaries/entry_points]]