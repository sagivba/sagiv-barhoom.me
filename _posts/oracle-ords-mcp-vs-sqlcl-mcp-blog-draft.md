---
title: "Oracle ORDS MCP vs SQLcl MCP: Two Different Ways to Give AI Access to Oracle Database"
date: 2026-09-27
draft: true
tags:
  - oracle
  - ords
  - sqlcl
  - mcp
  - ai
  - database
---

# Oracle ORDS MCP vs SQLcl MCP: Two Different Ways to Give AI Access to Oracle Database

Oracle now offers more than one way to expose Oracle Database capabilities through the Model Context Protocol (MCP).

Two of the most interesting options for developers and DBAs are:

- **Oracle SQLcl MCP Server**
- **Oracle REST Data Services (ORDS) MCP Server**

At first glance they look similar. Both allow an MCP-capable AI client to inspect database metadata and execute SQL or PL/SQL.

Architecturally, however, they solve different problems.

This post is a learning-oriented comparison based on a lab in which I am experimenting with Oracle Database, Codex, SQLcl and MCP. The goal is not to declare one product "better", but to understand what each MCP server exposes, where it runs, what security boundary it creates, and when each approach is a better fit.

> **Version context**
>
> - SQLcl MCP Server was introduced in **SQLcl 25.2**.
> - ORDS MCP was introduced in **ORDS 26.2**.
> - ORDS MCP is an ORDS feature, not an Oracle Database 26ai-only feature. ORDS 26.2 requires a currently supported Oracle Database release, so it can also be used with supported 19c deployments.
> - Select AI is **not required** for either SQLcl MCP or ORDS MCP.
> - Autonomous AI Database has a separate managed MCP Server that integrates with the Select AI Agent framework.

<!-- IMAGE: architecture-overview.png -->
<!-- Suggested visual: AI Client -> SQLcl MCP -> local saved connection -> DB
                     versus
                     AI Client -> HTTPS/OAuth -> ORDS MCP -> JDBC pool -> DB -->

## 1. The Core Architectural Difference

The simplest mental model is:

**SQLcl MCP gives the AI a database client.**

**ORDS MCP exposes database capabilities as a centrally managed server-side service.**

### SQLcl MCP

The SQLcl MCP Server runs alongside SQLcl and uses SQLcl's local connection store.

Conceptually:

```text
AI Client
   |
   | MCP
   v
SQLcl MCP Server
   |
   | named / saved connection
   v
SQLcl / JDBC
   |
   | Oracle Net
   v
Oracle Database
```

The connection definitions are managed locally by SQLcl.

The MCP server can therefore expose not only SQL execution, but also SQLcl-specific functionality.

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
   | authorized MCP database pool
   v
JDBC Connection Pool
   |
   | Oracle Net
   v
Oracle Database
```

The MCP client does not open and close database connections itself. ORDS manages the database pool.

Each MCP-enabled ORDS database target is backed by a **direct ORDS database pool** configured by an administrator.

That distinction explains most of the differences in the available tools.

---

# 2. Tool Comparison

## SQLcl MCP Tools

Oracle currently documents the following SQLcl MCP tools:

| Tool | Purpose | What it means operationally |
|---|---|---|
| `list-connections` | Lists saved Oracle Database connections on the machine | The AI can discover locally configured SQLcl connections |
| `connect` | Connects to a named connection | The AI explicitly chooses and opens a database connection |
| `disconnect` | Terminates the active database connection | Connection lifecycle is visible to the MCP client |
| `run-sql` | Executes SQL and PL/SQL | Standard database execution |
| `run-sqlcl` | Executes SQLcl-specific commands and extensions | The AI can use SQLcl functionality in addition to SQL |
| `schema-information` | Returns metadata for the connected schema | Structured schema discovery |

The important extra capability is `run-sqlcl`.

For example, SQLcl understands commands and features that are not Oracle SQL statements themselves.

This is why the SQLcl MCP Server is more than a simple SQL execution endpoint.

## ORDS MCP Tools

ORDS 26.2 exposes a smaller and intentionally server-oriented set of tools:

| Tool | Purpose | What it means operationally |
|---|---|---|
| `database_list` | Lists MCP database targets authorized for the caller | The AI discovers server-side pools it is allowed to use |
| `schema_information` | Returns schema metadata | Structured database discovery without manually querying dictionary views |
| `sql_run` | Executes SQL or PL/SQL | SQL execution through the selected ORDS database pool |

There is no `connect` or `disconnect` tool because the MCP client does not own the physical JDBC connection.

There is also no equivalent of `run-sqlcl`, because ORDS is not running SQLcl.

---

# 3. Why ORDS Does Not Need `connect` and `disconnect`

With SQLcl MCP, the AI is interacting with a database client that maintains a connection.

With ORDS MCP, the client chooses an authorized database target and ORDS obtains a connection from the corresponding JDBC pool.

Therefore the lifecycle looks more like this:

```text
MCP request
    |
    v
ORDS
    |
    v
Acquire JDBC connection from pool
    |
    v
Execute operation
    |
    v
Return connection to pool
```

The physical database session is an implementation detail of the pool rather than something the MCP client manages directly.

This also means that application design should not assume that two independent MCP calls will necessarily execute on the same physical database session.

Session-dependent workflows should therefore be treated carefully.

---

# 4. `run-sql` vs `sql_run`

At the database level, the overlap is substantial.

Both can be used to execute SQL and PL/SQL, subject to the privileges of the database identity being used.

Typical operations include:

| Operation | SQLcl `run-sql` | ORDS `sql_run` |
|---|---:|---:|
| `SELECT` | Yes | Yes |
| `INSERT` / `UPDATE` / `DELETE` / `MERGE` | Yes, if permitted | Yes, if permitted |
| PL/SQL blocks | Yes | Yes |
| Procedure/function calls | Yes | Yes |
| DDL | Yes, if permitted | Yes, if permitted |
| SQLcl commands such as `SET`, `SPOOL`, `DDL`, etc. | No, use `run-sqlcl` | No |

Oracle explicitly warns that ORDS `sql_run` **can modify database state** and must not be treated as read-only.

The real security boundary is therefore not MCP itself.

It is the combination of:

- MCP authorization
- Database identity
- Database privileges
- Roles
- Object grants
- Database security policies
- Network and runtime controls

A least-privileged database account remains essential.

---

# 5. Identity: Database User vs Human Caller

This is one of the most interesting differences.

Suppose several developers use the same ORDS MCP database pool:

```text
Sagiv  ----Ronen  -----+--> ORDS MCP --> TASKHUB_API --> Oracle
Naama  ----/
```

At the database authentication level, the connection may still be:

```text
TASKHUB_API
```

However, ORDS also propagates information about the authenticated OAuth caller into the database session through the `CLIENTCONTEXT` application context.

Examples documented by Oracle include:

```text
OAUTH_PRINCIPAL
OAUTH_SUB
OAUTH_ISSUER
OAUTH_MODE
REQUEST_ECID
```

This creates two useful identities:

```text
Technical database identity:
TASKHUB_API

Authenticated external identity:
sagiv
```

That makes it possible to use one controlled database principal while still retaining end-user traceability.

## Can SQLcl MCP also identify individual developers?

Yes, but the model is different.

A common SQLcl pattern is:

```text
Sagiv -> Sagiv's saved connection -> database identity
Ronen -> Ronen's saved connection -> database identity
```

or to deliberately set `CLIENT_IDENTIFIER`, use proxy authentication, or implement another identity convention.

So the correct distinction is **not**:

> ORDS has auditing and SQLcl does not.

Both products provide MCP monitoring capabilities.

The difference is that OAuth caller identity is a native part of the ORDS MCP request model.

---

# 6. Auditing and Monitoring

Both SQLcl MCP and ORDS MCP provide auditing/monitoring mechanisms.

Oracle documents MCP activity logging in:

```sql
DBTOOLS$MCP_LOG
```

ORDS MCP also populates database session information such as `MODULE` and `ACTION`, and supports correlation using identity attributes and an ECID.

This is useful when investigating questions such as:

- Which MCP client initiated the operation?
- Which tool was called?
- Which model or client workflow was involved?
- Which SQL was executed?
- Which authenticated user initiated the ORDS request?
- Which ORDS request corresponds to a database session?

For production-oriented deployments, this traceability can be a significant advantage.

---

# 7. When SQLcl MCP Is the Better Fit

SQLcl MCP is especially attractive when the AI is acting as a **developer or DBA assistant on a controlled workstation or lab environment**.

Typical scenarios:

| Scenario | Why SQLcl MCP fits well |
|---|---|
| Local development | SQLcl and the repository already exist on the same workstation |
| Database development | Easy SQL, PL/SQL and schema inspection |
| DBA workflows | SQLcl commands and extensions are available |
| Lab / POC work | Minimal server infrastructure |
| One developer or a small trusted team | Local connection management is simple |
| Repository-driven development | Codex can inspect files and then verify changes through SQLcl MCP |
| Need SQLcl-specific commands | `run-sqlcl` exposes SQLcl functionality |

### Advantages

- Simple local architecture
- Direct connection model
- Access to SQLcl-specific commands
- Natural fit for DBA and database-development work
- Saved connections are easy to switch between
- No ORDS MCP infrastructure is required

### Disadvantages

- Connection definitions are local to the machine
- Credentials/connectivity must be managed on each participating workstation
- Harder to provide one centrally governed MCP endpoint for many users
- Identity and authorization conventions may need to be designed separately
- Broader tool access can increase risk if SQLcl and database privileges are too permissive

---

# 8. When ORDS MCP Is the Better Fit

ORDS MCP becomes more interesting when MCP is treated as a **shared application or enterprise service**.

Typical scenarios:

| Scenario | Why ORDS MCP fits well |
|---|---|
| Many developers or AI clients | One centrally managed endpoint |
| Remote clients | HTTPS access rather than a locally installed SQLcl environment |
| Central authentication | OAuth 2.0 / JWT |
| Per-user authorization | Pools can be restricted using MCP scopes or roles |
| Shared technical DB identity | OAuth identity can still be propagated into the database session |
| Central governance | Pools, authentication and MCP exposure are managed server-side |
| Central auditing | Easier correlation between HTTP identity, ORDS request and DB activity |
| Enterprise infrastructure | Fits naturally with an existing ORDS deployment |

### Advantages

- Central MCP endpoint
- Server-side connection pooling
- OAuth/JWT authentication
- Pool-specific scopes or roles
- Caller identity propagation
- Centralized governance
- No database credentials need to be stored by every MCP client
- Good fit for multiple users and remote clients

### Disadvantages

- More infrastructure than a local SQLcl MCP setup
- Requires ORDS 26.2 or later
- Requires deliberate OAuth/JWT configuration
- Requires dedicated MCP-enabled direct database pools
- No SQLcl-specific commands
- Session state should not be assumed across independent requests
- A poorly privileged pool account can expose significant database capability to an LLM

---

# 9. Decision Matrix

| Requirement | SQLcl MCP | ORDS MCP |
|---|:---:|:---:|
| Local developer workstation | **Strong fit** | Possible, but usually unnecessary |
| DBA-style interaction | **Strong fit** | Limited to exposed DB operations |
| SQL and PL/SQL | Yes | Yes |
| SQLcl commands | **Yes** | No |
| Explicit connect/disconnect | **Yes** | No |
| Local saved connections | **Yes** | No |
| Central server endpoint | No | **Yes** |
| HTTPS-based remote access | Not the core model | **Yes** |
| OAuth/JWT | Not the core connection model | **Yes** |
| Central pool management | No | **Yes** |
| Per-pool MCP authorization | No | **Yes** |
| OAuth caller identity propagated to DB | No | **Yes** |
| Structured schema metadata | Yes | Yes |
| MCP activity monitoring | Yes | Yes |
| Best fit for one developer / lab | **Yes** | Sometimes |
| Best fit for many users / enterprise service | Possible, but awkward | **Yes** |

---

# 10. A Useful Mental Model

The most useful distinction from this experiment is not the tool list itself.

It is the trust boundary.

## SQLcl MCP

```text
AI
 |
 v
Developer / DBA tool
 |
 v
Database connection
 |
 v
Oracle
```

The AI is effectively being given a database client.

## ORDS MCP

```text
AI
 |
 v
Authenticated network service
 |
 v
Authorization policy
 |
 v
Managed connection pool
 |
 v
Oracle
```

The AI is being given access to a centrally governed database capability.

Neither model is universally better.

They are optimized for different operational contexts.

---

# 11. Where Select AI Fits

It is easy to confuse these technologies because they all involve Oracle, AI and MCP.

They should be separated conceptually.

### SQLcl MCP

An MCP server around SQLcl.

### ORDS MCP

An MCP server built into ORDS 26.2, exposing database discovery, schema metadata and SQL/PLSQL execution.

### Autonomous AI Database MCP Server

A different managed MCP service for Autonomous AI Database.

That service integrates with Oracle's **Select AI Agent framework** and can expose built-in and custom Select AI Agent tools.

Therefore:

> **Select AI is not a prerequisite for SQLcl MCP or ORDS MCP.**

It belongs to a different, richer managed-agent architecture in Autonomous AI Database.

---

# 12. Does ORDS MCP Require Oracle AI Database 26ai?

No.

ORDS MCP is a feature of **ORDS 26.2**, not a feature that requires the target database itself to be 26ai.

ORDS 26.2 requires a currently supported Oracle Database version according to Oracle's Lifetime Support Policy.

That means a supported Oracle Database 19c environment can also be used as an ORDS MCP target.

The important distinction is:

```text
ORDS version       -> determines whether ORDS MCP exists
Database version   -> must be supported by that ORDS release
```

This is useful for organizations that run large Oracle 19c estates but still want to experiment with MCP without first migrating the database to 26ai.

---

# 13. Security Observation

The most important security principle is the same for both implementations:

> Never give an AI agent more database privilege than the task actually requires.

For ORDS MCP, Oracle explicitly recommends a least-privilege database account for the MCP connection pool.

A useful pattern is to create a dedicated database identity for the agent rather than connecting MCP directly as the schema owner.

For example:

```text
Canonical owner:
TASKHUB_OWNER

Controlled AI/API identity:
TASKHUB_API

Restricted runtime identity:
TASKHUB_APP
```

The MCP identity should receive only the grants required for the intended workflow.

The same principle applies to SQLcl MCP saved connections.

MCP does not replace Oracle Database security. It exposes whatever capabilities the connected database identity already has.

---

# 14. Where the Two Approaches Overlap

The core overlap is actually small and easy to remember:

```text
                  SQLcl MCP        ORDS MCP
                      \              /
                       \            /
                     Schema metadata
                           +
                     SQL / PL/SQL
```

Everything around that core is different.

SQLcl MCP adds:

```text
Connection lifecycle
SQLcl commands
Local developer tooling
```

ORDS MCP adds:

```text
HTTP service boundary
OAuth identity
Central authorization
Server-side pools
Central governance
```

---

# 15. What I Want to Test Next in the Lab

This comparison raises several useful POC questions.

1. How much information does `schema_information` return compared with SQLcl MCP?
2. How do `BRIEF` and `DETAILED` schema-information modes differ in ORDS?
3. What exact identity values reach `CLIENTCONTEXT`?
4. How does ORDS MCP authorization behave with different `mcp.scope` and `mcp.role` configurations?
5. What happens when two developers use the same MCP pool simultaneously?
6. How useful is `DBTOOLS$MCP_LOG` for reconstructing an AI-driven change?
7. How should a least-privilege MCP database principal be designed for a real development workflow?
8. Which TaskHub operations should be exposed through raw SQL, and which should only be reachable through controlled PL/SQL APIs?

These are more interesting questions than simply asking whether ORDS MCP can execute SQL.

The real question is how Oracle is changing the boundary between **AI clients, developer tools, application infrastructure and database security**.

---

# Sources

Primary Oracle documentation used for this draft:

- Oracle REST Data Services 26.2, **Using ORDS Model Context Protocol (MCP)**  
  https://docs.oracle.com/en/database/oracle/oracle-rest-data-services/26.2/orddg/using-ords-model-context-protocol-mcp.html

- Oracle REST Data Services 26.2, **Changes in Release 26.2**  
  https://docs.oracle.com/en/database/oracle/oracle-rest-data-services/26.2/orddg/changes-release-26.2-oracle-rest-data-services-developers-guide.html

- Oracle REST Data Services 26.2, **Installation Checklist / System Requirements**  
  https://docs.oracle.com/en/database/oracle/oracle-rest-data-services/26.2/ordig/installing-REST-data-services.html

- Oracle SQLcl 25.4, **About the SQLcl MCP Server Tools**  
  https://docs.oracle.com/en/database/oracle/sql-developer-command-line/25.4/sqcug/sqlcl-mcp-server-tools.html

- Oracle SQLcl 25.2, **Changes in Release 25.2**  
  https://docs.oracle.com/en/database/oracle/sql-developer-command-line/25.2/sqcug/changes-release-25.2-oracle-sqlcl.html

- Oracle Autonomous AI Database, **Autonomous AI Database MCP Server**  
  https://docs.oracle.com/en-us/iaas/autonomous-database-serverless/doc/mcp-server.html

- Oracle Autonomous AI Database, **About MCP Server**  
  https://docs.oracle.com/en/cloud/paas/autonomous-database/serverless/adbsb/about-mcp-server.html

---

## Editorial Notes Before Publishing

This is intentionally a **working technical draft**, not the final blog post.

Before publishing, I would still:

- verify the exact ORDS MCP behavior against a real ORDS 26.2 lab;
- capture screenshots or tool-discovery output from both MCP servers;
- add one concrete side-by-side TaskHub example;
- add the architecture infographic near Section 1;
- add the tools infographic near Section 2;
- replace the current lab-question section with findings once the ORDS MCP experiment is complete.
