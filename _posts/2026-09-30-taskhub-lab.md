---
layout: post
title: "TaskHub: A Lab for AI Agents and Oracle"
author: "Sagiv Barhoom"
date: 2026-09-30
categories: ORACLE,Codex,AI
---

This post introduces the TaskHub lab: what I built, why I built it, and what I want to learn about AI agents working with Oracle Database.

Over the last few days, I built TaskHub, a small application for managing lists and tasks. It is a test system for Codex, Oracle, and MCP.

Instead of copying generated SQL into a database tool and returning the output to the model, I connected Codex to Oracle through SQLcl MCP. In this lab, Codex can inspect database state, apply changes, run tests, and examine the results.

## Why TaskHub

The application is deliberately small. I can follow the whole design while still testing real concerns: a data model, PL/SQL business logic, security, application identity, maintenance, and an APEX UI.

Each development task includes a contract, constraints, acceptance criteria, and tests. The aim is to check both the implementation and the way the agent works.

## A local environment I can rebuild

Oracle Database runs in a Docker container. ORDS runs in a separate container and provides the web access path to APEX. Codex uses SQLcl MCP on the host to work with Oracle.

![Codex connects through SQLcl MCP; ORDS provides a separate web access path to the same Oracle database.](/img/posts/taskhub-lab/lab-environment.png)

Docker gives me an isolated environment that is easier to recreate. MCP does not require Docker. I chose it to keep the lab manageable.

I covered the database setup in [Running Oracle Database 26ai with Docker](https://www.sagiv-barhoom.me/oracle/2026/04/05/oracle_26ai_docker_setup.html).

A full rebuild from a clean environment is a separate validation step. Recreating a container with its existing data volume does not prove that the system can be rebuilt from scratch.

## Three schemas, one database

The system uses three schemas in the same database and PDB.

![TASKHUB_OWNER owns the data, TASKHUB_API implements business operations, and TASKHUB_APP is the restricted application principal.](/img/posts/taskhub-lab/schema-responsibilities.png)

`TASKHUB_OWNER` owns the tables. `TASKHUB_API` holds business logic and access checks. `TASKHUB_APP` uses the permitted views and packages.

These are logical divisions of responsibility and permissions. They are not separate servers or databases. Development connections may have broader grants than the application principal.

## What actually stops an unauthorized action?

My main question is what changes when Codex can act through MCP.

![The lab workflow moves from requirements through inspection, changes, tests, result review, and revision.](/img/posts/taskhub-lab/agent-workflow.png)

That leads to a security question: **if the agent attempts an action I did not intend to allow, which mechanism rejects it?**

I use *security boundary* to mean an enforced restriction I can rely on. Tool restrictions, the selected database identity, Oracle grants, and API checks each need to be examined.

[SQLcl command restrictions](https://docs.oracle.com/en/database/oracle/sql-developer-command-line/25.3/sqcug/configuring-restrict-levels-sqlcl-mcp-server.html) can limit the tool's commands. I do not treat MCP alone as the database authorization boundary.

Missing a direct table grant can prevent direct access. It does not rule out access through an authorized definer's-rights package. [Oracle's execution-rights model](https://docs.oracle.com/en/database/oracle/oracle-database/26/adfns/security.html) makes the package's permissions and checks part of the design.

## Questions for the rest of the series

- What can Codex develop, test, and correct through SQLcl MCP?
- How does SQLcl MCP differ from ORDS MCP?
- How should database identity relate to application identity?
- When should an agent use filtered views and API packages instead of direct table access?
- Which operations should remain outside its authority?

TaskHub gives me a small system in which to test these questions and keep evidence of the results.
