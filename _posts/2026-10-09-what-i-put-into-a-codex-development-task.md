---
layout: post
title: "What I Put Into a Codex Development Task"
author: "Sagiv Barhoom"
date: 2026-10-09
categories: ORACLE
background: '/img/posts/what-i-put-into-a-codex-development-task/task-contract.png'
---


This post is part of my [TaskHub series](https://github.com/sagivba/sagiv-barhoom.me/blob/gh-pages/_posts/2026-09-30-taskhub-lab.md). In the [previous post](https://github.com/sagivba/sagiv-barhoom.me/blob/gh-pages/_posts/2026-10-09-specification-to-verified-oracle-implementation.md), I described how I move from a system specification to a verified Oracle implementation with Codex. Here I focus on how I structure one development task before handing it to Codex.

I will use **Task 50 — Tasks / Subtasks API** as a running example. It covers creating, updating, and soft-deleting tasks and subtasks, including hierarchy rules, ownership checks, and task status behavior.

## Why I Give Codex a Structured Task

`Implement task management.` leaves too much open to interpretation. I want a bounded unit of work that defines the change, protects earlier decisions, and makes completion testable.

The structure below is what I currently use. It developed during the TaskHub lab. I do not present it as a universal or optimal format.

## The Eight Sections of My Task

I usually divide each task into eight sections. Their size varies with the work involved.

![Eight parts of a Codex task, grouped into requested change, boundaries, and proof](/img/posts/what-i-put-into-a-codex-development-task/task-contract.png)

## Objective — What I Want to Achieve

### The principle

The objective is one clear statement of the result I want. It provides direction without repeating the full specification.

### Task 50 example

Implement task and subtask operations in the existing application API.

## Context — What Already Exists

### The principle

The task builds on earlier work. I identify the relevant components and decisions so Codex does not redesign something that is already in place.

### Task 50 example

The task depends on established parts of TaskHub:

- The existing **data model** for tasks and lists.
- The **application context** and ownership checks.
- The **read views** and existing API conventions.
- The behavior already implemented and validated in previous tasks.

## Scope — What Belongs to This Task

### The principle

The scope lists the operations and behavior I want to change now. Related ideas do not automatically become part of the task.

### Task 50 example

The relevant public operations are:

- `CREATE_TASK` — create a task or subtask.
- `UPDATE_TASK` — update properties, state, or parent relationship.
- `DELETE_TASK` — soft-delete a task and its subtree.

## Constraints — What Must Remain True

### The principle

Constraints define rules that the implementation must satisfy. A function that appears to work is not acceptable if it breaks one of these rules.

### Task 50 example

- Task hierarchy depth must not exceed **three levels**.
- Priority must be in the range `1..3`.
- Setting `DONE` must set `COMPLETED_AT`; leaving `DONE` must clear it.
- Deleting a task must **soft-delete its subtree**.
- Ownership must be enforced without exposing another user's data.
- The **caller controls the transaction**; the API must not commit on its behalf.

## Acceptance Criteria — What Counts as Success

### The principle

Acceptance Criteria state what must be true before I can consider the implementation successful. They are the target conditions, not the test output.

### Task 50 example

- The required procedures exist and compile successfully.
- Valid task and subtask operations work as specified.
- Invalid operations are rejected correctly.
- Ownership and non-disclosure rules hold.
- Existing public APIs remain compatible and runtime grants stay restricted.

## Tests — How I Check the Implementation

### The principle

Tests are part of the task, not something I invent after the code has been written. I specify both allowed behavior and the failures the API must prevent.

### Task 50 example

- **Positive tests:** create, update, re-parent, and soft-delete valid tasks.
- **Negative tests:** reject invalid priority values and excessive hierarchy depth.
- **Security tests:** verify ownership enforcement and non-disclosure.
- **Regression tests:** confirm earlier API behavior still works.

## Do Not Change — What Codex Must Preserve

### The principle

This section protects decisions already made. It is different from behavioral constraints: it defines changes the agent must not make while solving the current problem.

### Task 50 example

- Do not redesign completed tasks or change established architecture without approval.
- Do not change existing public API contracts unless explicitly required.
- Do not create database users or use `SYS` or `SYSTEM`.
- Preserve the existing security model and restricted grants.
- Do not expand the implementation beyond Task 50.

Some of these rules are specific to my lab. They are not universal Oracle development rules.

## Evidence — What I Want to Inspect

### The principle

When Codex reports that a task is complete, I do not want to rely only on a statement that everything passed. I want to see **what was actually checked and what the results were**.

This is what I mean by *Evidence*: inspectable results that support the claim that a task was completed successfully.

The distinction matters:

- **Acceptance Criteria** define what must be true.
- **Tests** define how I will check it.
- **Evidence** records what was checked and what happened.

Evidence makes the result reviewable. It also helps me investigate failures or verify a previous checkpoint later. A PASS statement, without supporting results, is not enough.

### Task 50 example

For Task 50, the evidence I would expect includes:

- Object-validity and compilation checks, including relevant `USER_ERRORS` queries.
- Test output for `CREATE_TASK`, `UPDATE_TASK`, and `DELETE_TASK`.
- Negative-case output showing the expected rejection of invalid operations.
- Ownership and non-disclosure test results.
- Grant and metadata checks, together with an updated task status.

### From Constraint to Evidence

A small example shows the relationship. The diagram describes an illustrative verification path, not a captured test log.

![Constraint for three hierarchy levels leads to a fourth-level test and expected rejection evidence](/img/posts/what-i-put-into-a-codex-development-task/constraint-test-evidence-v2.png)

## Definition of Done — When I Close the Task

For me, a task is not done when the source files exist. Three conditions must come together: the change is implemented, the required checks pass, and the evidence is available for review.

### The Completion Check

![Implemented, verified, and documented work converge on human review before task closure](/img/posts/what-i-put-into-a-codex-development-task/definition-of-done-v2.png)

Codex can execute checks and collect results, but **I decide whether the task is accepted**. A failed check or unclear result means more work, not automatic closure.

This is a checkpoint for the defined task, not proof that the entire application is defect-free. The next post will focus on how I make Codex demonstrate that its Oracle changes worked.
