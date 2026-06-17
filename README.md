# medrap-analysis

[![Python 3.12+](https://img.shields.io/badge/-Python_3.12+-blue?logo=python&logoColor=white)](https://www.python.org/downloads/release/python-3100/)
[![Tests](https://github.com/McDermottHealthAI/medrap-analysis/actions/workflows/tests.yaml/badge.svg)](https://github.com/McDermottHealthAI/medrap-analysis/actions/workflows/tests.yaml)
[![Test Coverage](https://codecov.io/github/McDermottHealthAI/medrap-analysis/graph/badge.svg)](https://codecov.io/github/McDermottHealthAI/medrap-analysis)
[![Code Quality](https://github.com/McDermottHealthAI/medrap-analysis/actions/workflows/code-quality-main.yaml/badge.svg)](https://github.com/McDermottHealthAI/medrap-analysis/actions/workflows/code-quality-main.yaml)
[![Contributors](https://img.shields.io/github/contributors/McDermottHealthAI/medrap-analysis.svg)](https://github.com/McDermottHealthAI/medrap-analysis/graphs/contributors)
[![Pull Requests](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](https://github.com/McDermottHealthAI/medrap-analysis/pulls)
[![License](https://img.shields.io/badge/License-MIT-green.svg?labelColor=gray)](https://github.com/McDermottHealthAI/medrap-analysis#license)

Post-hoc LLM judge, demographic analysis, and comorbidity scoring for
[MedRAP](https://github.com/McDermottHealthAI/MedRAP) pipeline outputs.

## Modules

- **`llm_judge`** — LLM-based pairwise comparison judge for evaluating retrieval quality; supports
    OpenAI-compatible endpoints and a `FakeJudge` for offline testing.
- **`demographic_analysis`** — Patient demographic breakdown, keyword/topic analysis, and
    heatmap visualization.
- **`comorbidity`** — Charlson Comorbidity Index scoring from ICD code lookups.

## Installation

```bash
# Core (LLM judge + viz extras recommended)
pip install "medrap-analysis[llm_judge,viz] @ git+https://github.com/McDermottHealthAI/medrap-analysis.git"
```

## Repository Structure

```python
>>> print_directory(
...     Path("."),
...     config=PrintConfig(ignore_regex=(
...         "^(\\.git|.*\\.gitkeep|\\.venv|\\.pytest_cache|.*__pycache__|.*\\.egg-info"
...         "|node_modules|\\.ruff_cache|\\.claude"
...         ")$"
...     ))
... )
├── .github
│   ├── actions
│   │   └── setup
│   │       └── action.yaml
│   ├── dependabot.yml
│   ├── workflows
│   │   ├── code-quality-main.yaml
│   │   ├── code-quality-pr.yaml
│   │   ├── python-build.yaml
│   │   └── tests.yaml
│   └── zizmor.yml
├── .gitignore
├── .pre-commit-config.yaml
├── .python-version
├── AGENTS.md
├── CLAUDE.md
├── CONTRIBUTORS.md
├── LICENSE
├── README.md
├── conftest.py
├── pyproject.toml
├── scripts
│   ├── aggregate_llm_judge_rank_sweep.py
│   ├── aggregate_llm_judge_rank_sweep_slurm.sh
│   ├── extract_and_visualize.py
│   ├── plot_llm_judge_winrates.py
│   ├── run_demographic_heatmap.py
│   ├── run_extraction_pipeline.py
│   ├── run_llm_judge.py
│   ├── run_llm_judge_rank_sweep_array_slurm.sh
│   ├── run_llm_judge_rank_sweep_slurm.sh
│   ├── submit_llm_judge_rank_sweep.sh
│   └── summarize_sweep.py
├── src
│   └── medrap_analysis
│       ├── __init__.py
│       ├── comorbidity.py
│       ├── demographic_analysis.py
│       └── llm_judge.py
├── tests
│   ├── test_comorbidity.py
│   ├── test_demographic_analysis.py
│   ├── test_llm_judge.py
│   ├── test_run_extraction_pipeline.py
│   └── test_run_llm_judge.py
└── uv.lock

```

> [!WARNING]
> Do not commit data or secrets to this repository. Datasets must be stored outside the repo; API
> keys (e.g. `OPENAI_API_KEY`) must be passed via environment variables or a local `.env` file that
> is listed in `.gitignore`.

## Documentation

### Python Build System: `uv`

This project uses [`uv`](https://docs.astral.sh/uv/) as its sole Python package manager. `uv` is a
fast Rust-based tool from [Astral](https://astral.sh) (the same team behind `ruff`) that replaces
`pip`, `pip-tools`, `virtualenv`, and `pyenv` with one tool and a single, consistent workflow.

Why we standardize on `uv` across the lab:

- **Reproducible environments.** `uv.lock` pins exact versions of every transitive dependency, so
    collaborators and CI install the identical environment.
- **Fast.** Dependency resolution and installs run in seconds, not minutes.
- **Modern defaults.** Lockfile-first, `pyproject.toml`-native; `uv` also installs the right Python
    interpreter for you based on `.python-version`, so you don't need a separate Python-version
    manager.
- **One tool, one workflow** across all of our research repos, which keeps onboarding fast.

The shortest path to install `uv` is `curl -LsSf https://astral.sh/uv/install.sh | sh` (Linux/macOS)
or `brew install uv`. See the [install guide](https://docs.astral.sh/uv/getting-started/installation/)
for other platforms, and see [`CONTRIBUTORS.md`](CONTRIBUTORS.md#build-system-uv) for the commands
you'll use day-to-day.

### Linting / Code Style

For linting and code style we use [`ruff`](https://docs.astral.sh/ruff/) following the
[Python Google Style Guide](https://google.github.io/styleguide/pyguide.html). Formatting and lint
fixes happen automatically on commit via [`pre-commit`](https://pre-commit.com/) hooks, configured
in [`.pre-commit-config.yaml`](.pre-commit-config.yaml). See
[`CONTRIBUTORS.md`](CONTRIBUTORS.md#code-style) for install steps and the full list of hooks
(which includes more than just ruff — secret scanning, dependency hygiene, workflow security, etc.).

### Testing

We use [`pytest`](https://docs.pytest.org/en/stable/) plus
[`doctest`](https://docs.python.org/3/library/doctest.html), with
[`pytest-cov`](https://github.com/pytest-dev/pytest-cov) reporting coverage to
[codecov.io](https://about.codecov.io/) for the README badge and PR-level coverage diffs. Run
`uv run pytest -v` to execute the full suite locally; see
[`CONTRIBUTORS.md`](CONTRIBUTORS.md#testing) for the full setup and conventions.

#### Testing Style and Doctests

While conventional wisdom in the software engineering community is to avoid doctests, I disagree. I feel that
doctests are an excellent way (provided appropriate APIs are written, tools are used, and the kinds of tests
included are appropriate) to ensure that code examples in docstrings and markdown documentation remains
accurate and reliable. This is especially important in research code, where the audience may be less
experienced programmers and more likely to copy-paste code examples from documentation. To this end, I
recommend, in general, writing conventional unit tests that validate a function or class's API as doctests
wherever possible. If such a test would be excessively long, complex, or unclear, then it should be written as
a standalone unit test in a `tests/**/test_*.py` file.

> [!NOTE]
> Note that when embedding doctests in markdown files, you must still use the `>>>` and `...` prompts, and you
> must ensure there is a new line separating the final output line from the `\`\`\`\` closing the code block.
> See above for an example.

Note that you can make doctests much easier to write and read (by omitting common setup or import code) by
using a [`conftest.py`](conftest.py) file to define common fixtures and add imports to the
[doctest namespace](https://docs.pytest.org/en/stable/how-to/doctest.html#doctest-namespace-fixture).
See the linked example for how to enable this functionality.

> [!NOTE]
> Note that the linked [`conftest.py`](conftest.py) file is located in the root directory, _not_ the root test
> directory (`tests/`). This is because we want the fixtures and imports to be available to doctests in
> non-test files (e.g., docstrings in the main package and markdown documentation files).

#### Additional Testing Packages

Beyond the default packages, you may also want to use:

- [`pytest-doctestplus`](https://github.com/scientific-python/pytest-doctestplus) for advanced doctest
    support.
- [`hypothesis`](https://hypothesis.readthedocs.io/en/latest/) for property-based testing.
- [`pretty-print-directory`](https://github.com/mmcdermott/pretty-print-directory) for easy
    visualization of directory structures in tests (especially doctests).
- [`yaml_to_disk`](https://github.com/mmcdermott/yaml_to_disk) to easily initialize a temporary directory
    structure from a YAML string in tests (especially doctests).
- [`pytest-codeblocks`](https://github.com/nschloe/pytest-codeblocks) to enable testing shell codeblocks as
    well as python codeblocks in markdown files; however, this would make it more challenging to have
    non-tested codeblocks in markdown files, so there are tradeoffs.

### Additional Files

#### `CONTRIBUTORS.md`

Source of truth for build, test, code-style, and PR-workflow conventions for both human contributors
and AI agents. See [`CONTRIBUTORS.md`](CONTRIBUTORS.md) for day-to-day commands and PR workflow.

#### `AGENTS.md` / `CLAUDE.md`

`AGENTS.md` provides AI coding agents with project conventions. `CLAUDE.md` is a symlink so Claude
Code picks up the same instructions via its native path.

### Repository management

This repository lives within the
[McDermott Health AI Lab GitHub Organization](https://github.com/McDermottHealthAI), which sets
default issue labels and (in time) issue templates that propagate to new repos. Use those labels
on new issues so filings stay searchable and consistent. All changes to `main` go through pull
requests; see the "Pull Request Workflow" section of [`CONTRIBUTORS.md`](CONTRIBUTORS.md) for the
expected flow (closing keywords, CI watching, comment replies, merge-vs-rebase policy). Versioning
follows semantic versioning, managed through `git` tags (e.g., `git tag 0.0.1`) — `setuptools-scm`
reads the tag and stamps the package version automatically.
