---
layout: post
title:  VSCode + WSL + Conda Setup (with unittest + Tasks)"
author: "Sagiv Barhoom"
date:   2026-02-13
categories: python 
---

# VSCode + WSL + Conda Setup (with unittest + Tasks)

This post documents a reproducible setup for developing a Python project inside **WSL** using **VS Code**, 
with a dedicated **Conda** environment, **unittest** discovery, and convenient **VSCode Tasks / Debug** configs.
This setup is for  developing  `drafts-checker.py` - Stage-Based Preparation Tool for Publishing Recipes
- https://github.com/sagivba/family_recipes/blob/main/code/drafts-checker.py
- https://sagivba.github.io/family_recipes/

## What `.vscode/` is
`.vscode/` is a **project-local configuration folder** for Visual Studio Code. 
Files here apply only to the current repository/workspace (not global machine settings). 
It’s the place to store reproducible editor/run/debug/test settings for the project.

### Files we create inside `.vscode/`
- **`.vscode/settings.json`**: Workspace settings (interpreter, test discovery, env files, terminal behavior)
- **`.vscode/launch.json`**: Debug configurations (what runs when you press **F5**)
- **`.vscode/tasks.json`**: Reusable command shortcuts (**Tasks: Run Task**) such as tests, checkers, docker compose

## 0) Prerequisites
- Windows with **WSL2** (Ubuntu)
- **VS Code** installed on Windows
- **Miniconda/Conda** installed inside WSL (paths like `/home/<user>/miniconda3/...`)

## 1) Install the WSL Remote Extension in VS Code
In VS Code:
- Extensions -> install **WSL** (Microsoft) / **Remote - WSL**

Then open the project **as a WSL workspace** (not Windows paths).

## 2) Open the Project from WSL (recommended)
From a WSL terminal:

```bash
cd ~/src/family_recipes
code .
```

This ensures VS Code runs the workspace in the WSL context (so it can see Linux interpreters and Linux conda envs).

## 3) Create and Activate a Dedicated Conda Environment
Inside WSL:

```bash
conda create -n family_recipes python=3.12 -y
conda activate family_recipes
which python
python -c "import sys; print(sys.executable)"
```

Expected:
- `which python` and `sys.executable` point to:
  `/home/<user>/miniconda3/envs/family_recipes/bin/python`

## 4) Install Dependencies (Conda + pip)
Some dependencies are best installed via conda, while others are typically installed via pip.

### Install via conda (examples)
```bash
conda activate family_recipes
conda install -y click pyyaml
python -c "import click, yaml; print('ok')"
```

### Install via pip (OpenAI + dotenv)
```bash
conda activate family_recipes
python -m pip install -U pip
pip install openai python-dotenv
python -c "import openai; from dotenv import load_dotenv; print('ok')"
```

## 5) Run Unit Tests from CLI (unittest discover)
From the repo root:

```bash
python -m unittest discover -s code/_Tests -p "test_*.py" -v
```

## 6) Set Project Environment Variables (PYTHONPATH)
Create a repo-level `.env` file (used by VS Code Python tooling):

**`<repo>/.env`**
```bash
PYTHONPATH=code
```

## 7) Configure VS Code (Workspace Settings)
Create:

**`<repo>/.vscode/settings.json`**
```json
{
  "python.defaultInterpreterPath": "/home/sagivba-adm/miniconda3/envs/family_recipes/bin/python",
  "python.testing.unittestEnabled": true,
  "python.testing.pytestEnabled": false,
  "python.testing.unittestArgs": [ "-v", "-s", "code/_Tests", "-p", "test_*.py" ],
  "python.envFile": "${workspaceFolder}/.env",
  "python.terminal.activateEnvironment": true
}
```

Notes:
- The file must contain **one JSON object** (not multiple `{...}{...}` blocks).
- After editing, run: `Developer: Reload Window`.

## 8) Configure Debug (unittest)
Create:

**`<repo>/.vscode/launch.json`** (example)
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "unittest: all (discover)",
      "type": "python",
      "request": "launch",
      "module": "unittest",
      "args": ["discover", "-s", "code/_Tests", "-p", "test_*.py", "-v"],
      "console": "integratedTerminal",
      "justMyCode": true,
      "envFile": "${workspaceFolder}/.env"
    }
  ]
}
```

## 9) Configure Tasks (Run Common Commands in One Click)
Maintain:

**`<repo>/.vscode/tasks.json`**

Key points:
- Use `${command:python.interpreterPath}` so Tasks always run with the interpreter selected in VS Code (your conda env).
- Create tasks for:
  - Docker compose (dev start/stop/logs)
  - `unittest discover`
  - `drafts-checker.py` (with and without `--use-ai`)
  - A “CI quick” meta-task that runs tests + checker in sequence

## 10) OpenAI API Key (for `--use-ai` mode)
To run the pipeline with AI features, export your key:

```bash
export OPENAI_API_KEY="YOUR_KEY_HERE"
```

Persist it (one common approach) by adding to `~/.bashrc`:

```bash
echo 'export OPENAI_API_KEY="YOUR_KEY_HERE"' >> ~/.bashrc
source ~/.bashrc
```

Verify:

```bash
echo "$OPENAI_API_KEY" | head -c 8; echo
```

## 11) Sanity Check (VS Code + WSL + Env)
Inside VS Code terminal:

```bash
conda activate family_recipes
which python
python -c "import sys; print(sys.executable)"
python -m unittest discover -s code/_Tests -p "test_*.py" -v
```

Expected:
- Python path points to the conda env
- Tests run and pass
