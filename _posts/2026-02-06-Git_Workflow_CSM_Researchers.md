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
[https://github.com/continuous-symmetry-measure/pdbprep](https://github.com/continuous-symmetry-measure/pdbprep)

 

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

 

## When to Commit, Push, and Tag (Applies to All Cases)

### When to commit

* After completing a clear, meaningful step in your work
* Each commit should represent one coherent research checkpoint
* Think of commits as entries in a lab notebook

### When to push

* At the end of a working session
* When you want a reliable backup on GitHub
* When others in the group may need your updated work

### When to tag

* **Only when releasing an official version of the project**
* Tags should correspond to explicit versions such as `v1.0`, `v1.1`, `v2.0`
* Use tags when results, calculations, or reports depend on that exact version
* Do not use tags for routine progress or intermediate steps

 

## Creating the Repository on GitHub (Before the First Push)

Before you can use `git push`, a repository must already exist on GitHub. This is a one-time setup step.

In the context of the CSM group, this usually means:

* The repository is created under the **continuous-symmetry-measure** organization
* Or, in some cases, under your personal GitHub account

### Creating a new repository on GitHub

1. Go to GitHub in your browser
2. Navigate to the organization (or your personal account)
3. Click **New repository**
4. Choose a repository name (for example: `pdbprep`)
5. Do **not** initialize it with files (no README, no `.gitignore`)
6. Click **Create repository**

GitHub will now show you the repository URL, which you will use locally.

### Connecting your local project to GitHub

After running `git init` locally and creating at least one commit, you connect your project to GitHub:

```
git remote add origin https://github.com/continuous-symmetry-measure/pdbprep.git
```

From this point on, `git push` knows *where* to send your commits.

 

## Core Git Commands - Step-by-Step

The following sections walk through the basic Git commands used in all cases above. Each command corresponds to a concrete action in your day-to-day research workflow.

## 1. `git init` - Initialize a Repository

This turns a regular working directory (for example, one containing structure files and a README) into a Git-tracked project. 
From this point on, Git observes changes-even if they are scientific data files rather than code.


## 2. `git status` - Check Repository State
This is your primary visibility tool. It tells you what has changed since the last recorded step. 
For our work, it is useful to run this often, even if you are not yet ready to commit.

## 3. Make Changes - Do the Research Work

At this stage, you work as usual:

* Update or preprocess structure files
* Add or remove intermediate results
* Edit short documentation files

Git detects changes automatically, but nothing is saved to history yet.

## 4. `git add` - Select a Coherent Step

Examples:

Add a single file, entire directory, ormultiple files using a wildcard

```
git add structure_01.pdb structures/ *.pdb
```

Add everything under the current directory:

```
git add .
```

Staging means selecting **which changes define the next research step**. 
A good commit represents *one clear stage of work*, not a mixture of unrelated experiments.

## 5. `git commit` - Record a Research Checkpoint

```
git commit -m "Describe the completed step"
```

A commit is a deliberate checkpoint. The message should help *future you* or another group member understand what changed and why. Think of commits as entries in a lab notebook.

## 6. `git push` - Save and Continue

```
git push # or 
git push origin main
```

This step stores your checkpoints in the shared GitHub repository. 
It serves as backup and continuity for the group-not public publication.

## 7. `git tag` - Mark Released Versions

```
git tag --list
git tag v1.0
git push origin --tags
```

A tag marks a **released version** of the project. 
I think it should be used sparingly, only for versions you explicitly want to name and reference.

Use tags when:

* You decide to release a version of the project
* Results, calculations, or reports depend on this exact state
* You want a clear, immutable reference such as `v1.0`, `v1.1`, `v2.0`

Do **not** use tags for routine progress or intermediate steps.

Tags do not replace commits or pushes - they sit *on top of them* and signal an official release.

 

## How to Read This as a CSM Researcher

You do not need to memorize commands. Focus on the rhythm:

* Work normally
* Pause at natural milestones
* Save a clear step
* Move on

If you can answer “what changed since the last step?”, Git is already working for you.
