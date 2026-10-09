---
layout: post
title: "From Specification to Working Oracle Code: My Codex Workflow"
author: "Sagiv Barhoom"
date: 2026-10-09
categories: ORACLE
background: '/img/posts/specification-to-verified-oracle-implementation/workflow-overview.png'
---

In my TaskHub lab, I use Codex to move from a system specification to a working Oracle implementation. The goal is not just to generate SQL or PL/SQL. I want Codex to inspect the existing system, make changes, run tests, and examine the results.

This post describes **the workflow I currently use**. I am not claiming it is the only or best way to work with an AI coding agent. It evolved while building TaskHub, and it gives me enough structure to understand what changed and when I can move forward.

## Start with the specification, not a build-everything prompt

TaskHub is a small task-management application built for an Oracle and Codex experiment. The application is intentionally simple. The interesting part is the workflow used to build it.

I start with a specification that describes the whole system: its data model, business logic, security, maintenance, and the ORDS and APEX components.

But I do not give Codex that document and ask:

```text
Build TaskHub.
```

A complete specification is useful for describing the target system. It is too broad for the unit of work I want an agent to execute and verify at once.

Instead, I divide the implementation into bounded tasks.

## Break the implementation into tasks

These were the initial TaskHub stages:

```text
Task 00 - Bootstrap
Task 10 - Data Model
Task 20 - Security Context & API Base
Task 30 - Read Isolation
Task 40 - Lists / Tags API
Task 50 - Tasks / Subtasks API
Task 60 - Maintenance & Purge
Task 70 - ORDS + APEX Foundation
```

The numbers are only identifiers. What matters is the progression. Each task covers a limited part of the system and builds on the state established by the previous tasks.

For every stage, I want to be able to understand the requested change, implement it, check the result, and close the stage before starting the next one.

![The TaskHub workflow moves from system specification through bounded tasks and validation](/img/posts/specification-to-verified-oracle-implementation/workflow-overview.png)

This does not mean the specification stops mattering after planning. It remains the reference for the overall design. Codex also works with the existing repository and the context created by earlier stages.

I will cover **how I structure an individual task** in a separate post. Here I want to focus on the larger workflow.

## Give Codex one task at a time

When I start a task, Codex works within an architecture that already exists. It should not redesign the entire application or reopen decisions without a reason.

The repository contains the implementation and documentation from earlier stages. The Oracle database contains the actual state that Codex needs to inspect.

The loop I ask Codex to follow is straightforward:

1. Read the task and inspect the repository.
2. Inspect the relevant Oracle objects and current state.
3. Implement the required change.
4. Execute the change against the lab database.
5. Run the relevant tests and inspect the results.
6. Correct problems, then validate again.

This is where the workflow becomes different from asking an AI model for a code sample.

## MCP changes who performs the next step

In my earlier AI-assisted workflows, a model could generate SQL, PL/SQL, or a suggested fix. I still needed to run it, collect the output, and return any errors to the model.

In TaskHub, Codex connects to Oracle through SQLcl MCP. Within the permissions and restrictions of the lab, it can inspect the database, execute changes, run tests, and use the results to decide what needs to be corrected.

![Traditional human-run code execution compared with the Codex and SQLcl MCP development loop](/img/posts/specification-to-verified-oracle-implementation/development-loop-comparison.png)

MCP does not remove the need for review or database controls. It allows the agent to carry out more of the **inspect → implement → execute → test → correct → validate** loop without requiring me to manually copy every result between tools.

The important change is not that the generated code is better. It is that the agent can work against the real system and check the outcome.

For the connectivity and setup details, see my earlier posts on [SQLcl MCP and Oracle](https://github.com/sagivba/sagiv-barhoom.me/blob/gh-pages/_posts/2026-09-27-codex_sqlcl_mcp_and_oracle.md) and [Codex CLI, Git Worktree, and Docker Compose](https://github.com/sagivba/sagiv-barhoom.me/blob/gh-pages/_posts/2026-05-08-codex-cli-git-worktree-docker-compose.md).

## A validated state is the checkpoint

I do not move to the next task just because Codex has finished generating files.

I want to know that the intended change was applied, the relevant checks were run, and any remaining issue is understood. Only then do I treat that task as closed.

This makes the workflow easier to review. When something fails later, I have a smaller set of changes to investigate. It also helps preserve the decisions made in earlier stages.

A validated stage is not proof that the entire application has no defects. It is a known checkpoint with explicit scope, suitable for continuing the lab.

## What remains my responsibility

Codex performs much of the engineering loop, but it does not own the system design.

I still decide what TaskHub should do, define the architecture, split the work into stages, set boundaries, review the result, and decide when the next task can start.

That division is not absolute. I can ask Codex to investigate a design question or suggest an alternative. But the decision to accept a change remains mine.

For me, the useful pattern is simple: **one bounded task, an execution and testing loop, a reviewed state, then the next task**.

The next post will go one level deeper: how I define an individual Codex task so that the expected behavior, constraints, and validation are clear before implementation begins.
