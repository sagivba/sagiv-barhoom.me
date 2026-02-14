---
layout: post
title:  "Making a Repo AI-Friendly"
author: "Sagiv Barhoom"
date:   2026-02-14
categories: python 
---
# Making a Repo AI-Friendly
This note describes a simple way to use **Codex (web UI)** to implement changes on top of a **GitHub repo** while keeping diffs small, tests green, and merges boring.

Example repo layout (from this project): a **static site** + a **Python recipes toolchain** (lint/check/pipeline).


## 1) Repo prep (one-time, recommended)

### Add basic docs
Create or update these files:

- `README.MD`  
  Explains what the repo is, how it is structured (site + python tooling), and the shortest path to run/tests.

- `docs/dev-python.md`  
  Developer guide for the Python toolchain: env setup, install deps, run unit tests, common commands.

- `.github/CONTRIBUTING.md`  
  Contribution rules: minimal diffs, no secrets, preferred testing style (unittest), TDD expectation, etc.

Why: Codex (and humans) work better when the repo clearly states how to build and test.


### Add CI (tests on PR)
`.github/workflows/ci.yml` is typically a GitHub Actions workflow file that defines your CI (Continuous Integration) pipeline.
What it usually does:
- Runs automatically on events like push and pull_request
- Checks out your code
- Sets up the runtime (Node/Python/Java/etc.)
- Installs dependencies
- Runs linting, tests, builds, or security scans
- Uploads artifacts (test reports, build outputs) if needed

#### What’s inside (typical structure):
```yml
name: CI
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: echo "Run tests here"
```

Why: CI is your safety net. If CI is green, merge is low-risk.
Common pitfall: missing Python deps in CI. Install from a requirements file (see below).


### Add a simple Makefile (optional, but helpful)
A Makefile in a Python project is a convenience automation file for the make tool. 
It lets you run common project tasks with short, consistent commands (locally and in CI), without remembering long CLI strings.
#### What it’s used for (typical)
- Setup: create venv, install deps
- Quality: format, lint, type-check
- Tests: run unit tests, coverage
- Build/Release: build wheels/sdist, publish
- Housekeeping: clean caches, remove build artifacts

#### example:
```bash
# Declare targets that are not real files. This prevents "make" from skipping them
# if a file with the same name exists in the working directory.
.PHONY: venv install test lint fmt clean

# Virtual environment directory name/path.
VENV := .venv

# Convenience variables pointing to the venv's Python and pip executables.
# Note: This layout matches Linux/macOS. On Windows it would be .venv/Scripts/python.exe, etc.
PY := $(VENV)/bin/python
PIP := $(VENV)/bin/pip

# Create the virtual environment (idempotent: re-running keeps existing venv).
venv:
	python3 -m venv $(VENV)

# Install dependencies. Depends on `venv` to ensure the environment exists first.
# - Upgrades pip inside the venv.
# - Installs packages listed in requirements.txt.
install: venv
	$(PIP) install -U pip
	$(PIP) install -r requirements.txt

# Run tests quietly (-q) using pytest from the venv.
test:
	$(PY) -m pytest -q

# Run lint checks (static analysis) using Ruff on the whole repo.
lint:
	$(PY) -m ruff check .

# Auto-format code using Ruff formatter across the repo.
fmt:
	$(PY) -m ruff format .

# Remove common build/test caches and artifacts.
# - Deletes pytest/ruff caches and Python build outputs.
# - Removes __pycache__ directories recursively.
clean:
	rm -rf .pytest_cache .ruff_cache dist build *.egg-info
	find . -name "__pycache__" -type d -prune -exec rm -rf {} +
```
Why: less typing, less “works on my machine”.


### Track Python deps
Create or update:

`code/requirements.in` (or `requirements.txt`)  
The Python packages needed to run the toolchain and tests. 
 Keep it minimal and accurate.

Example (one per line):
```
openai
click
pyyaml
python-dotenv
```

Why: tools like Codex, CI, and teammates need a single source of truth for dependencies.

