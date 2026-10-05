---
layout: post
title: "MCP Is Not the Security Boundary: Limiting AI Agent Risk in Oracle Development"
author: "Sagiv Barhoom"
date: 2026-10-05
categories: ORACLE
background: '/img/posts/taskhub-mcp-security/defense-in-depth.png'
---

This post looks at a practical problem from my [TaskHub lab]({% post_url 2026-09-30-taskhub-lab %}): once Codex can do more than generate SQL and can actually inspect Oracle, execute changes, run tests, and examine the results through SQLcl MCP, what limits the damage if the agent gets something wrong?

The point is not to build a complete enterprise security model. TaskHub is a development POC. I want enough freedom to test agentic database development, while keeping the failure radius controlled and making the important boundaries explicit.

## The problem: the agent can now execute

With a normal AI-assisted workflow, there is a clear manual step between generated code and the database:

![Before MCP, the AI generates SQL but a human reviews it before Oracle executes it.](/img/posts/taskhub-mcp-security/traditional-ai-assisted-workflow.png)

With SQLcl MCP, that loop changes. Codex can inspect the current state, execute SQL or PL/SQL, check object status, run tests, and use the results to decide what to do next.

![The TaskHub workflow moves from requirements through inspection, changes, tests, result review, and revision.](/img/posts/taskhub-lab/agent-workflow.png)

That is exactly what I wanted to test, but it changes the risk model.

The question becomes:

> If the agent misunderstands the task, generates the wrong SQL, or performs a broader operation than I intended, what actually limits the damage?

For this post, I use *security boundary* to mean an enforced restriction that still applies when the agent makes a mistake. A prompt can influence behavior. A database privilege can reject an operation. Those are not the same thing.

## First control: keep the development blast radius local

The TaskHub development process runs against a local Oracle Database in Docker. It does not run against a shared organizational development database.

![Codex connects through SQLcl MCP; ORDS provides a separate web access path to the same local Oracle database.](/img/posts/taskhub-lab/lab-environment.png)

This changes the impact of a development mistake. If Codex damages the lab database, other developers are not using the same database and no shared development data or workflow is affected.

Docker is not an Oracle authorization mechanism. It does not decide whether `UPDATE`, `DROP`, or `GRANT` is allowed. Its role here is isolation: a failure remains local.

The lab also has a volume backup/restore path, and a clean rebuild is a separate validation step. I covered the Docker setup and the lab-oriented recovery approach in [Oracle 26AI Docker Setup - Fast Hands-On Guide for Local AI Database]({% post_url 2026-04-05-oracle_26ai_docker_setup %}).

This local isolation lets me take a controlled amount of risk while experimenting with the development workflow.

## Option 1: rely on the prompt

The simplest control is to tell Codex what it must not do.

For example:

```text
Do not use SYS or SYSTEM.
Do not create database users.
Do not change unrelated objects.
Do not perform direct DML against owner tables.
```

This is useful. It defines scope, keeps the task focused, and reduces accidental scope expansion.

### Advantages

- Very easy to add and change.
- Makes the expected workflow explicit.
- Works well for task boundaries and engineering conventions.
- Helps the agent reason about what it should and should not touch.

### Limitation

A prompt is not database enforcement.

If the connected Oracle identity has permission to perform an operation, Oracle does not know that the prompt told Codex not to do it.

So I treat the prompt as policy and workflow guidance, not as the only security control.

## Option 2: reduce the MCP tool surface

There is a deeper control at the MCP layer.

An MCP server does not simply mean "the model can do everything the target system can do." The server exposes a defined set of tools, and the client decides how those tools are made available to the agent.

Oracle SQLcl 26.1 documents MCP tools for connection management, schema inspection, SQL and PL/SQL execution, and SQLcl-specific commands. SQLcl also supports **restrict levels** for the MCP server. By default, the MCP server starts at the most restrictive level, R4, which blocks host commands, script execution, and other sensitive SQLcl operations. See the [Oracle SQLcl 26.1 documentation](https://docs.oracle.com/en/database/oracle/sql-developer-command-line/26.1/).

There is also a client-side approval dimension. In the SQLcl MCP setup I documented earlier, each MCP tool required approval before execution. That keeps a human confirmation point in the loop even though the agent can formulate the action itself. See [Using SQLcl as an MCP Server in Codex for Oracle Database]({% post_url 2026-09-27-codex_sqlcl_mcp_and_oracle %}).

The important distinction is:

```text
MCP / client controls:
What can the agent ask the tool to do?

Oracle controls:
What can the connected database identity actually do?
```

### Advantages

- Reduces the operational surface exposed to the agent.
- Can stop some classes of actions before they reach the database.
- Restrict levels reduce SQLcl-side capabilities such as host or script execution.
- Approval gates can keep a human decision point around sensitive tool calls.

### Limitations

- A broad tool such as general SQL execution is still a broad capability.
- SQLcl restrict level controls SQLcl-side commands; it does not replace Oracle privileges.
- Database-side capabilities still depend on the privileges of the connected Oracle user and other database controls.

This means MCP is part of the control surface, but I do not treat it as the complete database authorization boundary.

## Option 3: give the development agent broad database privileges

The easiest way to remove friction is to connect the agent as a schema owner or another identity with broad development privileges.

For a local POC, this has obvious benefits. The agent can create objects, modify PL/SQL, apply schema changes, run tests, and fix failures without stopping every time a privilege is missing.

### Advantages

- Very little friction during development.
- Lets the POC test real agentic database work rather than a read-only demo.
- Makes migrations and object changes easier to automate.
- The local Docker environment limits the wider impact of mistakes.

### Limitations

- A mistake can affect many objects in the schema.
- It does not test least privilege very well.
- It is not representative of the runtime application's security model.

For TaskHub, broad development access is useful for some development tasks, but it is not the model I want the application itself to rely on.

## Option 4: let Oracle enforce the database identity

The next control moves from behavior to enforcement.

Instead of asking only what I told Codex to do, I can ask a more important question:

> What is the Oracle user behind this connection actually allowed to do?

If that user does not have `UPDATE` on a table, Oracle rejects the operation even if Codex asks for it. The same principle applies to object privileges, system privileges, package execution, and access to other schemas.

### Advantages

- Enforcement happens inside Oracle.
- It does not depend on the agent interpreting the prompt correctly.
- It supports familiar least-privilege and separation-of-duties patterns.
- The privilege model can be tested with security regression tests.

### Trade-off

A development agent often needs different capabilities from a runtime application. If I use one identity for everything, I either give the runtime too much access or make development unnecessarily difficult.

That is why identity separation matters.

## The TaskHub choice: several controls, not one

TaskHub does not rely on one mechanism to solve the whole problem.

![TaskHub defense in depth separates local isolation, agent and MCP controls, Oracle authorization, and application data access.](/img/posts/taskhub-mcp-security/defense-in-depth.png)

The model is intentionally layered:

1. **Local Docker isolation** limits the blast radius of a development mistake.
2. **Task contracts and prompts** define the expected behavior and scope.
3. **MCP and client controls** define which tools are available, how SQLcl commands are restricted, and where approval is required.
4. **Saved connections** select a known Oracle database identity rather than passing ad hoc credentials to the agent.
5. **Oracle privileges** enforce what that identity can actually do.
6. **Application packages and views** define the read and write surface used by the runtime application.

The lab uses named SQLcl saved connections for the application, API, and owner identities. Codex does not use `SYS` or `SYSTEM`, and user creation is outside the agent workflow.

The important point is not the exact number of layers. It is that a failure in one layer should not automatically remove all the others.

## Three schemas, one database

The application model uses three schemas:

![TASKHUB_OWNER owns the data, TASKHUB_API implements business operations, and TASKHUB_APP is the restricted application principal.](/img/posts/taskhub-lab/schema-responsibilities.png)

`TASKHUB_OWNER` owns the tables.

`TASKHUB_API` contains the business API and access checks.

`TASKHUB_APP` is the restricted runtime principal. It receives access to the permitted views and packages rather than direct DML on the owner tables.

These are logical security and responsibility boundaries inside the same Oracle Database and the same PDB. They are not separate servers or physical tiers.

This separation lets the lab test two different questions:

- Can the agent build and modify the application?
- Does the application it builds still enforce the intended runtime boundaries?

## Read access needs a boundary too

It is easy to focus only on `INSERT`, `UPDATE`, and `DELETE`.

An agent or application with broad `SELECT` access can still see data it should not see.

For TaskHub, the runtime schema reads through filtered views such as:

```text
V_TODO_LISTS
V_TODO_LIST_TAGS
V_TASKS
V_TASK_TREE
```

The filtering becomes part of the database interface instead of depending on every consumer remembering the correct predicate.

For this POC, filtered views are simple, visible, and easy to test.

### What I could test beyond the POC: VPD

For a more complex system with several access paths to the same tables, Oracle Virtual Private Database (VPD) is another option. VPD can attach row- or column-level policies directly to database objects and dynamically add predicates based on session or application context. Oracle documents this in [Using Oracle Virtual Private Database to Control Data Access](https://docs.oracle.com/en/database/oracle/oracle-database/26/dbseg/using-oracle-vpd-to-control-data-access.html).

That would move more of the row-level enforcement from a selected view to a policy attached to the protected object itself.

I did not use VPD in TaskHub. The filtered-view approach is enough for the current POC and keeps the model easier to inspect. There is also an edition constraint: the Oracle 26ai `DBMS_RLS` documentation states that the package is available with Enterprise Edition only. The TaskHub lab uses Oracle Database Free, so this is a design extension to evaluate in a different environment, not a missing switch I could simply enable in the current lab. See [`DBMS_RLS`](https://docs.oracle.com/en/database/oracle/oracle-database/26/arpls/DBMS_RLS.html).

## Controls I deliberately left outside this POC

The current lab is not a complete enterprise security design. Several extensions would be worth testing separately:

- **A dedicated build identity**: separate the agent identity used for schema changes from both the data owner and the runtime principal.
- **Unified Auditing**: record and query the database actions performed by the identities used during the agent workflow. Oracle 26ai uses unified auditing as the current auditing model; see [Introduction to Auditing](https://docs.oracle.com/en/database/oracle/oracle-database/26/dbseg/introduction-to-auditing.html).
- **Resource controls**: constrain CPU, TEMP, session duration, or other resources. A statement can be fully authorized and still be operationally expensive.
- **Stronger approval gates**: require explicit human approval before selected DDL or privilege changes.
- **Rollback-aware changes**: make pre-change state, validation, and recovery planning part of the task contract for changes with a larger blast radius.

These are natural next experiments, but adding all of them to the first POC would make it harder to see which control is solving which problem.

## The simple test I keep using

For every control, I ask the same question:

> If the agent ignores the instruction or simply makes a mistake, what still stops or contains the action?

| Layer | What it controls | If the agent makes a mistake |
|---|---|---|
| Local Docker environment | Development blast radius | The damage remains inside the local lab |
| Prompt / task contract | Expected behavior and scope | No hard enforcement by itself |
| MCP / client controls | Tool surface, SQLcl restrictions, approvals | Some actions can be blocked before reaching Oracle |
| Saved connection | Database identity used for the action | Selects the privilege context |
| Oracle privileges | Database authorization | Oracle rejects operations the identity is not allowed to perform |
| API packages / filtered views | Application read/write surface | Business checks and filtering remain in the database interface |

That distinction is the main lesson from this part of the lab.

I do not want an agent that can do nothing. That would defeat the purpose of testing agentic database development. But I also do not want the safety model to depend on the assumption that the agent will always interpret every instruction exactly as intended.

The local Docker environment gives me room to experiment. MCP and approval controls reduce the tool surface. Oracle identities and privileges provide database enforcement. The application API defines the runtime contract.

The next question is identity at the application level: if all end users reach Oracle through the same `TASKHUB_APP` runtime schema, how does the database distinguish Alice from Bob? That leads to Oracle Application Context and the difference between **database identity** and **application identity**.
