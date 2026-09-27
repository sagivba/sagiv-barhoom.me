---
layout: post
title: "Oracle ORDS MCP vs SQLcl MCP: Two Different Ways to Give AI Access to Oracle Database"
author: Sagiv Barhoom
date: 2026-09-27
categories: ORACLE
background: /img/posts/oracle-ords-mcp-vs-sqlcl-mcp/architecture-overview.png
---

Oracle now offers more than one way to expose Oracle Database capabilities through the Model Context Protocol (MCP).

Two of the most interesting options for developers and DBAs are the **Oracle SQLcl MCP Server** and the **Oracle REST Data Services (ORDS) MCP Server**.

At first glance they look similar. Both allow an MCP-capable AI client to inspect database metadata and execute SQL or PL/SQL. Architecturally, however, they solve different problems.

## Goal of this post

This post is a learning-oriented comparison based on the lab work I am currently doing with Oracle Database, Codex, SQLcl and MCP.

The goal is not to declare one implementation better than the other. I want to understand:

- where each MCP server runs;
- what tools each one exposes;
- how database connections are managed;
- where authentication and authorization happen;
- what identity reaches the database;
- and where each architecture may fit better.

> **Hands-on note:** My practical experience with ORDS MCP is still limited. The architectural and feature comparisons in this post are based on Oracle documentation and the lab work completed so far. I do not yet consider the operational conclusions final.

## The core architectural difference

The simplest mental model is:

> **SQLcl MCP gives the AI a database client.**

> **ORDS MCP exposes database capabilities as a centrally managed server-side service.**

![SQLcl MCP and ORDS MCP architecture comparison](/img/posts/oracle-ords-mcp-vs-sqlcl-mcp/architecture-overview.png)

### SQLcl MCP

The SQLcl MCP Server runs locally with SQLcl and uses SQLcl's saved connection store.

Conceptually:

```text
AI Client
   |
   | MCP / stdio
   v
SQLcl MCP Server
   |
   | saved connection
   v
Oracle Database
```

The connection definitions are managed on the machine running SQLcl.

This model also allows the MCP server to expose SQLcl-specific functionality in addition to SQL and PL/SQL execution.

### ORDS MCP

ORDS MCP is exposed by an ORDS server over its `/mcp` endpoint.

Conceptually:

```text
AI Client
   |
   | HTTPS + OAuth 2.0 Bearer JWT
   v
ORDS MCP
   |
   | authorization
   v
MCP Database Pool
   |
   | JDBC
   v
Oracle Database
```

The MCP client does not own a physical JDBC connection. ORDS manages database connections through server-side pools.

That difference explains most of the behavior we see later in the tool model, authorization model and session lifecycle.

## Version context

There are several Oracle MCP implementations, so it is worth separating them before going further.

| Component | Context |
|---|---|
| SQLcl MCP Server | Introduced in SQLcl 25.2 |
| ORDS MCP | Introduced in ORDS 26.2 |
| Select AI | Not required for SQLcl MCP or ORDS MCP |
| Autonomous AI Database MCP Server | A separate managed MCP service that integrates with the Select AI Agent framework |

ORDS MCP is an **ORDS feature**, not an Oracle AI Database 26ai-only feature. ORDS 26.2 requires a currently supported Oracle Database release.

Based on that requirement, ORDS MCP appears to be usable with supported Oracle Database 19c deployments as well. **I have not tested ORDS MCP with 19c myself yet**, so I currently treat this as a documentation-based conclusion rather than a hands-on finding.

Oracle documentation:

- [Changes in Oracle REST Data Services 26.2](https://docs.oracle.com/en/database/oracle/oracle-rest-data-services/26.2/orddg/changes-release-26.2-oracle-rest-data-services-developers-guide.html)
- [ORDS Installation Checklist and System Requirements](https://docs.oracle.com/en/database/oracle/oracle-rest-data-services/26.2/ordig/installing-REST-data-services.html)
- [Changes in SQLcl 25.2](https://docs.oracle.com/en/database/oracle/sql-developer-command-line/25.2/sqcug/changes-release-25.2-oracle-sqlcl.html)

## Tool comparison

The overlap between the two MCP implementations is useful, but it is smaller than it first appears.

![SQLcl MCP and ORDS MCP tool surface comparison](/img/posts/oracle-ords-mcp-vs-sqlcl-mcp/tool-surface-comparison.png)

### SQLcl MCP tools

Oracle currently documents the following SQLcl MCP capabilities:

| Tool | Purpose |
|---|---|
| `list-connections` | Lists saved Oracle Database connections on the machine |
| `connect` | Connects to a named connection |
| `disconnect` | Terminates the active database connection |
| `run-sql` | Executes SQL and PL/SQL |
| `run-sqlcl` | Executes SQLcl-specific commands and extensions |
| `schema-information` | Returns metadata for the connected schema |

The important extra capability is `run-sqlcl`.

SQLcl understands commands and workflows that are not SQL statements themselves. That makes SQLcl MCP more than a simple SQL execution endpoint.

### ORDS MCP tools

ORDS 26.2 exposes a smaller server-side tool set:

| Tool | Purpose |
|---|---|
| `database_list` | Lists MCP database targets authorized for the caller |
| `schema_information` | Returns schema metadata for an authorized target |
| `sql_run` | Executes SQL or PL/SQL against an authorized target |

There is no `connect` or `disconnect` tool because the MCP client does not own the physical JDBC connection.

There is also no equivalent of `run-sqlcl`, because ORDS is not running SQLcl.

Oracle documentation:

- [SQLcl MCP Server Tools](https://docs.oracle.com/en/database/oracle/sql-developer-command-line/25.4/sqcug/sqlcl-mcp-server-tools.html)
- [Using ORDS Model Context Protocol](https://docs.oracle.com/en/database/oracle/oracle-rest-data-services/26.2/orddg/using-ords-model-context-protocol-mcp.html)

## Why ORDS does not need `connect` and `disconnect`

With SQLcl MCP, the AI is interacting with a database client that maintains an active connection.

With ORDS MCP, the caller selects an authorized database target and ORDS obtains a connection from the corresponding JDBC pool.

The lifecycle is closer to:

```text
MCP request
    |
    v
ORDS
    |
    v
Acquire connection from JDBC pool
    |
    v
Execute operation
    |
    v
Return connection to pool
```

The physical database session is therefore an implementation detail of the pool rather than something the MCP client manages directly.

This also means application logic should not assume that independent MCP requests will necessarily execute on the same physical database session.

Session-dependent workflows should be treated carefully.

## `run-sql` vs `sql_run`

At the database level, the overlap is substantial.

Both can execute SQL and PL/SQL subject to the privileges of the database identity being used.

| Operation | SQLcl `run-sql` | ORDS `sql_run` |
|---|:---:|:---:|
| `SELECT` | Yes | Yes |
| `INSERT` / `UPDATE` / `DELETE` / `MERGE` | Yes, if permitted | Yes, if permitted |
| PL/SQL blocks | Yes | Yes |
| Procedure and function calls | Yes | Yes |
| DDL | Yes, if permitted | Yes, if permitted |
| SQLcl commands | No, use `run-sqlcl` | No |

Oracle explicitly warns that ORDS `sql_run` can modify database state. It must not be treated as a read-only tool.

Oracle also documents `sql_run` as intended for **interactive tool use**, not as a bulk-export or pagination API.

This matters because MCP does not replace the database security model. The real execution boundary still includes:

- the database identity;
- database privileges and roles;
- object grants;
- database security policies;
- MCP authorization;
- and the surrounding network and runtime controls.

## ORDS MCP database pools are dedicated MCP resources

An ORDS MCP database pool is not just a regular ORDS connection pool.

For a pool to be available through MCP, Oracle requires it to:

- connect **directly** to the target database user;
- not connect through an ORDS runtime user such as `ORDS_PUBLIC_USER`;
- and define either `mcp.scope` or `mcp.role`.

Pools configured this way are dedicated to MCP use and are not considered for normal REST URL mapping.

### `mcp.scope`

With scope-based authorization, the pool is configured with a scope value:

```text
mcp.scope = "taskhub:mcp:developer"
```

The caller's JWT must contain that scope before ORDS allows access to the pool.

### `mcp.role`

With role-based authorization, ORDS is configured to read roles from a claim in the JWT:

```text
mcp.security.jwt.profile.role.claim.name = "/roles"
```

The pool can then require a specific role:

```text
mcp.role = "ORDS_MCP_TASKHUB"
```

The caller must have that role in the configured JWT claim.

Oracle documentation:

- [Using ORDS Model Context Protocol](https://docs.oracle.com/en/database/oracle/oracle-rest-data-services/26.2/orddg/using-ords-model-context-protocol-mcp.html)
- [ORDS Configuration Properties](https://docs.oracle.com/en/database/oracle/oracle-rest-data-services/26.2/ordig/about-REST-configuration-files.html)

## ORDS authorization and identity flow

ORDS applies more than one security boundary before a database operation is executed.

![ORDS MCP authorization and database identity flow](/img/posts/oracle-ords-mcp-vs-sqlcl-mcp/ords-authorization-identity-flow.png)

At a high level:

```text
External caller
     |
     | OAuth 2.0 JWT
     v
ORDS MCP global authorization
     |
     | global MCP scope
     v
Pool authorization
     |
     | mcp.scope or mcp.role
     v
ORDS MCP Database Pool
     |
     | database user
     v
Oracle Database
```

The bearer token must first satisfy the ORDS MCP JWT profile and include the global MCP access scope:

```text
urn:oracle:dbtools:ords:mcpserver:all
```

ORDS then checks whether that caller is allowed to access the selected database target. Depending on the configuration, the target can require either a pool-specific scope or a role.

Only after those checks does ORDS execute a database operation using the database user configured for the pool.

This is one of the most important architectural differences from SQLcl MCP: authorization to an ORDS MCP database target can be controlled centrally from the caller's JWT before the database connection is used.

## Database execution identity vs authenticated external caller identity

This distinction is easy to miss.

Assume several developers use the same ORDS MCP pool:

```text
Sagiv  -----\
Ronen  ------+--> ORDS MCP --> TASKHUB_API --> Oracle
Naama  -----/
```

At the database authentication level, the pool may connect as:

```text
TASKHUB_API
```

I refer to this as the **database execution identity**.

Its Oracle Database privileges determine what can actually be selected, modified or executed.

The human or external system that authenticated to ORDS is a different identity. I refer to this as the **authenticated external caller identity**.

ORDS propagates information about that authenticated caller into the database session through the `CLIENTCONTEXT` application context.

Oracle documents attributes including:

```text
OAUTH_PRINCIPAL
OAUTH_SUB
OAUTH_ISSUER
OAUTH_MODE
OAUTH_APP_ROLES
OAUTH_APP_SCOPES
REQUEST_ECID
```

This makes it possible to use one controlled technical database identity while still carrying information about the external caller into the database session.

That is useful for traceability, but it should not be confused with the Oracle Database login itself.

## Auditing and monitoring

Both SQLcl MCP and ORDS MCP provide monitoring capabilities, including MCP activity logging in:

```sql
DBTOOLS$MCP_LOG
```

ORDS MCP adds useful request-side context.

Oracle documents that ORDS populates database session information and propagates the authenticated caller through `CLIENTCONTEXT`. The `REQUEST_ECID` value can also be used to correlate database activity with ORDS logs.

One detail is worth keeping precise: the `MODULE` and `ACTION` conventions documented for ORDS MCP and SQLcl MCP are not identical, so I would not assume that the same fields have the same meaning across both implementations.

For ORDS MCP, Oracle documents:

- `V$SESSION.MODULE` with the LLM name;
- `V$SESSION.ACTION` with the MCP tool name;
- `DBTOOLS$MCP_LOG` for persisted MCP activity;
- `CLIENTCONTEXT` for caller identity and ECID information.

Oracle documentation:

- [Monitoring the ORDS MCP Server](https://docs.oracle.com/en/database/oracle/oracle-rest-data-services/26.2/orddg/using-ords-model-context-protocol-mcp.html)

## Where SQLcl MCP fits well

SQLcl MCP is a natural fit when the AI is acting as a developer or DBA assistant on a controlled workstation or lab environment.

Typical cases include:

| Scenario | Why SQLcl MCP fits |
|---|---|
| Local development | SQLcl and the repository already exist on the workstation |
| Database development | Direct SQL, PL/SQL and schema inspection |
| DBA workflows | SQLcl commands and extensions are available |
| Lab / POC | Minimal server-side MCP infrastructure |
| Repository-driven work | Codex can inspect files and verify changes against the database |
| SQLcl-specific operations | `run-sqlcl` exposes client functionality |

The trade-off is that connection definitions, credentials and local SQLcl configuration live on each participating machine.

## Where ORDS MCP fits well

ORDS MCP becomes more interesting when MCP is treated as a shared service rather than a local developer tool.

Typical cases include:

| Scenario | Why ORDS MCP fits |
|---|---|
| Multiple developers or AI clients | One centrally managed MCP endpoint |
| Remote clients | HTTPS rather than a local SQLcl process |
| Central authentication | OAuth 2.0 Bearer JWT |
| Per-target authorization | `mcp.scope` or `mcp.role` |
| Shared database execution identity | External caller identity can still be propagated |
| Central connection management | Database pools are managed server-side |
| Central monitoring | ORDS requests can be correlated with DB activity |

The trade-off is additional infrastructure and security configuration: ORDS 26.2 or later, an authorization server capable of issuing the required JWTs, dedicated MCP database pools, and deliberate scope or role design.

## A shorter decision matrix

| Requirement | SQLcl MCP | ORDS MCP |
|---|---|---|
| Deployment model | Local client | Central server |
| Transport | Local MCP / stdio | Streamable HTTP over HTTPS |
| Connection ownership | Client-side saved connection | ORDS-managed JDBC pool |
| Authentication model | Local database connection | OAuth 2.0 Bearer JWT + database pool identity |
| Target authorization | Local connection availability and client controls | Global MCP authorization + pool scope/role |
| SQLcl commands | Yes | No |
| External caller identity propagation | Not native to this model | Yes, through `CLIENTCONTEXT` |
| Typical fit | Developer / DBA workstation | Shared or centrally governed service |

## Where Select AI fits

SQLcl MCP, ORDS MCP and the Autonomous AI Database MCP Server should be treated as separate concepts.

### SQLcl MCP

An MCP server around SQLcl.

### ORDS MCP

An MCP server built into ORDS 26.2 that exposes authorized database discovery, schema metadata and SQL/PLSQL execution.

### Autonomous AI Database MCP Server

A separate managed MCP service for Autonomous AI Database.

Oracle documents this service as integrating with the **Select AI Agent framework**, exposing custom and built-in Select AI Agent tools through MCP.

Therefore:

> **Select AI is not a prerequisite for SQLcl MCP or ORDS MCP.**

It belongs to a different managed-agent architecture in Autonomous AI Database.

Oracle documentation:

- [Autonomous AI Database MCP Server](https://docs.oracle.com/en-us/iaas/autonomous-database-serverless/doc/mcp-server.html)

## What I want to test next in the lab

The comparison is useful, but the next step should be practical testing rather than adding more architecture diagrams.

The four things I want to verify first are:

1. **Identity propagation**  
   What values actually reach `CLIENTCONTEXT` when Codex calls ORDS MCP?

2. **Pool authorization**  
   How does access change when different `mcp.scope` and `mcp.role` configurations are used?

3. **Schema discovery**  
   How does ORDS `schema_information` compare with SQLcl MCP in `BRIEF` and `DETAILED` modes?

4. **Audit reconstruction**  
   How useful are `DBTOOLS$MCP_LOG`, `V$SESSION`, `CLIENTCONTEXT` and `REQUEST_ECID` when reconstructing an AI-driven database operation?

Those tests should make it easier to move from a documentation-based architectural comparison to an opinion based on actual ORDS MCP usage.
