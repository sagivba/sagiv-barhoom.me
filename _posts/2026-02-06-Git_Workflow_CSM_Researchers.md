---
layout: post
title:  "A Practical Git Workflow for CSM Researchers"
author: "Sagiv Barhoom"
date:   2026-02-06
categories: github 
background: '/img/posts/be-linux.jpg.jpg'
---

# A Practical Git Workflow for CSM Researchers

## Purpose

This post describes a minimal, practical Git workflow tailored for researchers in the Continuous Symmetry Measure (CSM) group. The goal is **organized personal work**, clear checkpoints, and reliable history-*not* code publication or open-source workflows.

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

---

### Case 2: Taking an existing project that was not managed with Git

```
cd existing_project_directory
git init
Repeat until you are ready to save a clear step:
  git status -> make changes -> git add -> git commit
git push
git tag   # only for version release
```

You already have a directory with files and results, but no version control. Running `git init` allows you to start managing the project history *from this point onward*.

---

### Case 3: Working on an existing project from GitHub

There are two common situations:

**Clone (group repository):**

```
git clone https://github.com/continuous-symmetry-measure/pdbprep.git
cd pdbprep
Repeat until you are ready to save a clear step:
  git status -> make changes -> git add -> git commit
git push
git tag   # only for version release
```

You clone an existing repository from the group organization and work on it locally.

**Fork (personal copy):**

```
# Fork on GitHub first
git clone https://github.com/<your-username>/pdbprep.git
cd pdbprep
Repeat until you are ready to save a clear step:
  git status -> make changes -> git add -> git commit
git push
git tag   # only for version release
```

You work independently on your own fork.

In all cases, the local workflow (status -> add -> commit -> push) is the same; the difference is where the changes are pushed.

---

## When to Commit, Push, and Tag (Applies to All Cases)

### When to commit
- After completing a clear, meaningful step in your work
- Each commit should represent one coherent research checkpoint
- Think of commits as entries in a lab notebook

### When to push
- At the end of a working session
- When you want a reliable backup on GitHub
- When others in the group may need your updated work

### When to tag
- **Only when releasing an official version of the project**
- Tags should correspond to explicit versions such as `v1.0`, `v1.1`, `v2.0`
- Use tags when results, calculations, or reports depend on that exact version
- Do not use tags for routine progress or intermediate steps

---

## Creating the Repository on GitHub (Before the First Push)

Before you can use `git push`, a repository must already exist on GitHub. This is a one-time setup step.

### Creating a new repository on GitHub

1. Go to GitHub in your browser
2. Navigate to the organization (or your personal account)
3. Click **New repository**
4. Choose a repository name (for example: `pdbprep`)
5. Do **not** initialize it with files (no README, no `.gitignore`)
6. Click **Create repository**

### Connecting your local project to GitHub

```
git remote add origin https://github.com/continuous-symmetry-measure/pdbprep.git
```

---

## Core Git Commands - Step-by-Step

### `git init`
Initialize Git tracking in the directory.

### `git status`
Check what changed since the last commit.

### `git add`

Examples:

Add a single file, a directory, or multiple files using a wildcard:

```
git add structure_01.pdb
git add structures/
git add *.pdb
```

Add everything under the current directory:

```
git add .
```

### `git commit`
Record a clear research checkpoint.

### `git push`
Save your work to GitHub.

### `git tag`
Mark a released version only.

---

## How to Read This as a CSM Researcher

Focus on rhythm:
- Work
- Commit clear steps
- Push regularly
- Tag only released versions
