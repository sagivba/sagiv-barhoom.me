---
layout: post
title: "Moving from Codex Web to Codex CLI with Git Worktree and Docker Compose"
author: "Sagiv Barhoom"
date: 2026-05-08
categories: codex
background: '/img/posts/codex-cli-git-worktree-docker-compose.png'
---

# Moving from Codex Web to Codex CLI with Git Worktree and Docker Compose

Until now, I mostly worked with Codex through the Web version.
The workflow was clear: write a spec, break it into small tasks, define success criteria, send a task to Codex, get a branch in GitHub, review it, merge it, and then pull the code into the local environment.

That worked well, but it had one obvious limitation: Codex worked in one environment, while local testing happened in another.

With Codex CLI, the goal is not to replace the workflow. The goal is to move it into a more controlled local setup. I still want small tasks, separate branches, success criteria, and tests. The difference is that Codex now works inside my local development environment instead of only through a remote branch.

The move is roughly from this:

```text
Codex Web -> remote branch -> merge -> local pull -> manual testing
```

To this:

```text
Codex CLI -> local branch -> Dev tests -> push -> QA tests -> merge to main
```

Because of that, Codex should not work in the same directory where I do manual testing. I want two local working areas:

```text
Dev -> Codex CLI work, code changes, and quick tests
QA  -> manual testing before merging to main
```

Production is not part of this flow. It runs on a separate server, so it is not relevant to the local development cycle.

## Table of contents

- [1. Why not work in the same directory](#1-why-not-work-in-the-same-directory)
- [2. Why Git Worktree](#2-why-git-worktree)
- [3. Creating a Dev environment for Codex](#3-creating-a-dev-environment-for-codex)
- [4. Separating Docker Compose environments](#4-separating-docker-compose-environments)
- [5. Workflow with Codex CLI](#5-workflow-with-codex-cli)
- [6. Task structure for Codex](#6-task-structure-for-codex)
- [7. Quick tests and full tests](#7-quick-tests-and-full-tests)
- [8. A parameterized test script](#8-a-parameterized-test-script)
- [9. QA testing before merge](#9-qa-testing-before-merge)
- [10. After merging to main](#10-after-merging-to-main)
- [11. Rolling this out gradually](#11-rolling-this-out-gradually)
## 1. Why not work in the same directory

It is possible to run Codex CLI in the same directory where manual testing is done, but that creates an unhealthy mix of responsibilities.

While working, Codex may leave the directory in an intermediate state:

- Modified files that have not been committed yet.
- Temporary files.
- Test output files.
- Local configuration changes.
- A dirty Git working tree.

If manual testing happens in that same directory, it becomes harder to know what is actually being tested: a clean version of the branch, or an intermediate result from the development process.

Separating Dev from QA solves that. Codex gets its own workspace, and manual testing gets a separate one.

## 2. Why Git Worktree

One way to solve this is to use two separate clones:

```text
~/src/dev/my_project
~/src/qa/my_project
```

This is easy to understand, but less convenient to maintain. It duplicates the repository, requires managing remotes in more than one place, uses more disk space, and increases the chance of mixing up directories.

A cleaner option is `git worktree`.

`git worktree` allows multiple working directories to share the same repository. Each directory can be checked out to a different branch, while all of them use the same Git object database.

For example:

```text
~/src/dev/my_project -> Dev for Codex CLI
~/src/qa/my_project  -> QA and manual testing
```

This gives the same practical separation as an additional clone, without duplicating the whole repository.

## 3. Creating a Dev environment for Codex

Assume the existing repository is here:

```bash
~/src/qa/my_project
```

Keep it as the QA environment, and create another worktree for Codex:

```bash
cd ~/src/qa/my_project && git worktree add ../../dev/my_project main
```

This command first moves into the existing repository, then creates a second working directory from the `main` branch.

Because the command runs from:

```text
~/src/qa/my_project
```

The relative path:

```text
../../dev/my_project
```

Points to:

```text
~/src/dev/my_project
```

The result:

```text
~/src/qa/my_project  -> QA, manual testing
~/src/dev/my_project -> Dev, Codex CLI work
```

From this point on, Codex works only under:

```bash
~/src/dev/my_project
```

Manual testing happens under:

```bash
~/src/qa/my_project
```

## 4. Separating Docker Compose environments

In this project, I do not use a local virtualenv. Running the app and running the tests both happen through Docker Compose, so the Dev and QA separation should also exist at the Compose level.

The simple approach is to use the same compose file with a different project name.

In Dev:

```bash
docker compose -p my_project_dev up -d --build
```

In QA:

```bash
docker compose -p my_project_qa up -d --build
```

The `-p` flag tells Docker Compose to create containers, networks, and volumes under a different project name. This keeps the two environments from colliding with each other.

If there are real differences between Dev and QA later, override files can be added:

```text
docker-compose.yml
docker-compose.dev.yml
docker-compose.qa.yml
```

`docker-compose.yml` is the shared base file. It contains the common service definitions, build settings, networks, volumes, and default configuration used by both environments.

The override files are only needed when Dev and QA should differ:

```text
docker-compose.dev.yml -> Dev-specific changes
docker-compose.qa.yml  -> QA-specific changes
```

For example, Dev can be started like this:

```bash
docker compose -p my_project_dev -f docker-compose.yml -f docker-compose.dev.yml up -d --build
```

And QA like this:

```bash
docker compose -p my_project_qa -f docker-compose.yml -f docker-compose.qa.yml up -d --build
```

This is useful when the environments need different ports, environment variables, databases, volumes, mock services, or logging settings.

At the beginning, if there is no real difference between the environments, using the same compose file with a different project name is enough.

## 5. Workflow with Codex CLI

Start in the Dev environment:

```bash
cd ~/src/dev/my_project
```

Update `main`:

```bash
git checkout main && git pull
```

Create a branch for the task:

```bash
git checkout -b codex-cli/task-name
```

I prefer using a fixed prefix for branches created for Codex CLI work:

```text
codex-cli/...
```

That makes it easy to identify where the branch came from and what it is for.

Then start the Dev environment:

```bash
docker compose -p my_project_dev up -d --build
```

And run Codex CLI:

```bash
codex --sandbox workspace-write --ask-for-approval on-request
```

While learning the workflow, I prefer not to give Codex too much freedom. It should be able to edit project files, but broader actions should still require approval.

## 6. Task structure for Codex

Codex works better when the task is small and well-defined. For example:

```text
Implement the following task in a minimal and focused way.

Task:
...

Success criteria:
1. ...
2. ...
3. ...

Testing:
Use Docker Compose.
For a quick check, run the short test command.
If it fails, provide the full test command for debugging.

Constraints:
- Use Python unittest, not pytest.
- Do not change unrelated files.
- Do not reorganize code unless explicitly required.
- Keep the change small and reviewable.
```

This structure matters more than the tool itself. Codex behaves better when the task is bounded, the success criteria are clear, and the constraints on the change are explicit.

## 7. Quick tests and full tests

During normal development, I do not always need to see the full `unittest` output. When everything passes, a short answer is enough.

Long test output creates noise. It makes it harder to see whether the test passed, harder to find the important line, and easier for Codex to flood the screen with information that is not useful.

So it is useful to keep two test modes:

```text
quick -> short result for normal development
full  -> full output for debugging
```

Codex should usually run:

```bash
scripts/test.sh quick
```

When debugging, it can run:

```bash
scripts/test.sh full
```

The same script can run against Dev or QA by changing `COMPOSE_PROJECT_NAME`.

## 8. A parameterized test script

Create one script:

```text
scripts/test.sh
```

With this content:

```bash
#!/usr/bin/env bash
set -o pipefail

MODE="${1:-quick}"
PROJECT_NAME="${COMPOSE_PROJECT_NAME:-my_project_dev}"
SERVICE_NAME="${APP_SERVICE_NAME:-app}"
OUTPUT_FILE="/tmp/my_project_unittest_output.log"

case "$MODE" in
  quick)
    if docker compose -p "$PROJECT_NAME" exec "$SERVICE_NAME" python -m unittest discover -s code/_Tests -p "test_*.py" > "$OUTPUT_FILE" 2>&1; then
      echo "TESTS PASSED"
    else
      echo "TESTS FAILED"
      echo "For full output, run:"
      echo "COMPOSE_PROJECT_NAME=$PROJECT_NAME APP_SERVICE_NAME=$SERVICE_NAME scripts/test.sh full"
      echo "Or inspect the captured output with:"
      echo "cat $OUTPUT_FILE"
      exit 1
    fi
    ;;

  full)
    docker compose -p "$PROJECT_NAME" exec "$SERVICE_NAME" python -m unittest discover -s code/_Tests -p "test_*.py" -v
    ;;

  *)
    echo "Usage: $0 [quick|full]"
    exit 1
    ;;
esac
```

Make it executable:

```bash
chmod +x scripts/test.sh
```

Run a quick test in Dev:

```bash
scripts/test.sh quick
```

Run the full test in Dev:

```bash
scripts/test.sh full
```

Run a quick test in QA:

```bash
COMPOSE_PROJECT_NAME=my_project_qa scripts/test.sh quick
```

Run the full test in QA:

```bash
COMPOSE_PROJECT_NAME=my_project_qa scripts/test.sh full
```

This gives Codex one stable interface for tests, while still keeping Dev and QA separated.

## 9. QA testing before merge

After Codex finishes the work in Dev, and the quick tests pass, push the branch:

```bash
git push -u origin codex-cli/task-name
```

Move to the QA environment:

```bash
cd ~/src/qa/my_project && git fetch && git checkout codex-cli/task-name && git pull
```

Start QA:

```bash
docker compose -p my_project_qa up -d --build
```

Run the quick test:

```bash
COMPOSE_PROJECT_NAME=my_project_qa scripts/test.sh quick
```

If needed, run the full test:

```bash
COMPOSE_PROJECT_NAME=my_project_qa scripts/test.sh full
```

After that, do the manual testing.
Only if the manual tests pass, continue to merge into `main`.

## 10. After merging to main

After the merge, update the QA environment back from `main`:

```bash
cd ~/src/qa/my_project && git checkout main && git pull && docker compose -p my_project_qa up -d --build
```

It is also possible to run the quick test again:

```bash
COMPOSE_PROJECT_NAME=my_project_qa scripts/test.sh quick
```

This keeps the QA environment aligned with `main`.

## 11. Rolling this out gradually

Start with a very small task, not a real feature.

For example:

- Add `AGENTS.md` to the project.
- Add `scripts/test.sh`.
- Add a short README note about running tests through Docker Compose.

Only after the workflow itself feels stable should Codex CLI get real development tasks.
