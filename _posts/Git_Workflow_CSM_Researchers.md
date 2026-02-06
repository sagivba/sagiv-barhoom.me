# A Practical Git Workflow for CSM Researchers

## Purpose

This post describes a minimal, practical Git workflow tailored for researchers in the Continuous Symmetry Measure (CSM) group. The goal is organized personal work, clear checkpoints, and reliable history - not code publication or open-source workflows.

We assume a repository such as **`pdbprep`**, used to manage scientific files (e.g., molecular structures, processed outputs, and short documentation).

Repository URL:
https://github.com/continuous-symmetry-measure/pdbprep

---

## High-Level Flow Overview

### Case 1: Starting a new project

```
git init
Repeat until you are ready to save a clear step:
  git status -> make changes -> git add -> git commit
git push
git tag   # only for version release
```

You create a new working directory for a fresh research task, initialize it with `git init`, and begin recording your work from the very first files.

When to commit:
- After completing a clear, meaningful step (e.g., first prepared structures, initial cleanup, basic documentation).

When to push:
- At the end of a working session
- When you want a safe backup on GitHub

When to tag:
- Only when releasing a version
- Tags should correspond to explicit versions (v1.0, v1.1, ...)

---

## Creating the Repository on GitHub (Before the First Push)

Before you can use `git push`, a repository must already exist on GitHub. This is a one-time setup step.

Steps:
1. Go to GitHub
2. Navigate to the organization or your personal account
3. Click New repository
4. Choose a repository name (e.g. pdbprep)
5. Do not initialize with README or gitignore
6. Click Create repository

Connect locally:

```
git remote add origin https://github.com/continuous-symmetry-measure/pdbprep.git
```

---

## Core Git Commands - Step-by-Step

### git init
Initialize Git tracking in the directory.

### git status
Check what changed since the last commit.

### git add
Select which changes belong to the next step.

### git commit
Record a clear research checkpoint.

### git push
Save your work to GitHub.

### git tag
Mark a released version only.

---

## How to Read This as a CSM Researcher

Focus on rhythm, not commands:
- Work
- Commit clear steps
- Push regularly
- Tag only released versions
