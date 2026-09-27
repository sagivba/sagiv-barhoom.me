---
layout: post
title: "Using SQLcl as an MCP Server in Codex for Oracle Database"
author: Sagiv Barhoom
date: 2026-09-27
categories: ORACLE
background: /img/posts/codex-sqlcl-mcp-and-oracle/codex-sqlcl-mcp-flow.png
---

I am currently experimenting with MCP as part of a small Oracle development lab. The first goal is simple: let Codex work with an Oracle Database through SQLcl, while keeping the connection path and execution boundaries explicit.

## Goal of this post

This post covers three things:

- What MCP is in practical terms
- How to configure an MCP server in Codex
- How to run Oracle SQLcl as that MCP server

The setup shown here is based on the configuration used in my current `taskhub-oracle-codex-lab` project.

![MCP setup flow from SQLcl installation to Codex](/img/posts/codex-sqlcl-mcp-and-oracle/mcp-setup-flow-sqlcl-to-codex.png)

## What is MCP?

MCP, or Model Context Protocol, defines a standard way for an AI client to discover and call tools exposed by an external server.

In this setup:

- Codex is the MCP client.
- SQLcl runs the MCP server.
- SQLcl exposes database-related tools.
- SQLcl connects to Oracle Database using saved connections.

Instead of giving the model a database password in a prompt, Codex calls tools exposed by SQLcl. SQLcl then performs the database operation using a connection that already exists in its local connection store.

![Codex to Oracle Database through SQLcl MCP](/img/posts/codex-sqlcl-mcp-flow.png)

Oracle documents SQLcl MCP tools for listing saved connections, connecting and disconnecting, inspecting schema information, running SQL and PL/SQL, and running SQLcl-specific commands.

Official documentation:

- [About the SQLcl MCP Server](https://docs.oracle.com/en/database/oracle/sql-developer-command-line/25.4/sqcug/sqlcl-mcp-server.html)
- [About the SQLcl MCP Server Tools](https://docs.oracle.com/en/database/oracle/sql-developer-command-line/25.4/sqcug/sqlcl-mcp-server-tools.html)

## Where Codex stores MCP configuration

Codex can define MCP servers in:

```text
~/.codex/config.toml
```

This is the user-level Codex configuration file. Codex uses it for settings such as model defaults, approvals, sandbox behavior, trusted projects, and MCP server definitions; project-specific overrides can also be placed in a repository-level `.codex/config.toml`.

OpenAI documentation:

- [Codex configuration](https://developers.openai.com/codex/config-basic)
- [Docs MCP](https://developers.openai.com/learn/docs-mcp)

The same MCP configuration is shared between the Codex CLI and IDE integration.

For example, a remote MCP server can be configured with a URL:

```toml
[mcp_servers.openaiDeveloperDocs]
url = "https://developers.openai.com/mcp"
```

A local MCP server is different. Codex starts a local process and communicates with it over stdio.

That is the model used for SQLcl in this lab.

## The SQLcl MCP configuration used in my lab

The project itself is marked as trusted in my Codex configuration:

```toml
[projects."/home/sagivba-adm/src/TaskHub_Codex_MCP_Starter/taskhub-oracle-codex-lab"]
trust_level = "trusted"
```

The SQLcl MCP server is configured separately:

```toml
[mcp_servers.sqlcl-taskhub-lab]
command = "/home/sagivba-adm/.local/bin/sql"
args = ["-R", "1", "-mcp"]

[mcp_servers.sqlcl-taskhub-lab.tools.connections_list]
approval_mode = "approve"

[mcp_servers.sqlcl-taskhub-lab.tools.connect]
approval_mode = "approve"

[mcp_servers.sqlcl-taskhub-lab.tools.schema_information]
approval_mode = "approve"

[mcp_servers.sqlcl-taskhub-lab.tools.sql_run]
approval_mode = "approve"

[mcp_servers.sqlcl-taskhub-lab.tools.sqlcl_run]
approval_mode = "approve"

[mcp_servers.sqlcl-taskhub-lab.tools.disconnect]
approval_mode = "approve"
```

There are three parts here that are worth looking at separately.

### Starting SQLcl in MCP mode

The command is:

```toml
command = "/home/sagivba-adm/.local/bin/sql"
args = ["-R", "1", "-mcp"]
```

The `-mcp` argument starts SQLcl as an MCP server.

The `-R` option controls how much of SQLcl is available to the MCP server.

For a new MCP setup, I recommend starting with the most restrictive level and relaxing it only when a specific requirement justifies it.

**`R4` is the safest starting point** and is also the default for the SQLcl MCP server when no `-R` option is specified. It blocks host commands, script execution, configuration changes, and other potentially sensitive SQLcl operations.

**`R1` is less restrictive.** It still blocks operating-system-level commands such as `HOST`, but allows SQL scripts to be executed with commands such as `@` and `@@`. This is the level I currently use in this lab because the Agent needs to work with project SQL scripts while still being prevented from executing host commands.

**`R0` removes the SQLcl restrictions entirely**, including restrictions on host and script execution. It should therefore be used only when full SQLcl functionality is genuinely required and the surrounding environment is sufficiently isolated and controlled.

A useful way to think about the three levels is:

```text
R4  -> Most restrictive. Recommended starting point.
R1  -> Allows script execution, but blocks host/OS commands.
R0  -> Unrestricted. Allows all commands.
```

Start with `R4`, verify what the Agent can accomplish, and move to `R1` or eventually `R0` only when a required operation is actually blocked.

Oracle documentation:

- [Configuring Restrict Levels for the SQLcl MCP Server](https://docs.oracle.com/en/database/oracle/sql-developer-command-line/25.4/sqcug/configuring-restrict-levels-sqlcl-mcp-server.html)

### Requiring approval for MCP tools

The configuration also sets:

```toml
approval_mode = "approve"
```

for every SQLcl MCP tool enabled in this lab.

For example:

```toml
[mcp_servers.sqlcl-taskhub-lab.tools.sql_run]
approval_mode = "approve"
```

I currently use approval for:

```text
connections_list
connect
schema_information
sql_run
sqlcl_run
disconnect
```

This is intentional. While experimenting with an Agent that can reach a database, I prefer to see the requested operation before it is executed rather than immediately moving to unattended execution.

## SQLcl Connection Manager

SQLcl does not require Codex to receive a username and password directly.

Instead, SQLcl MCP relies on saved SQLcl connections.

The local SQLcl connection store is under:

```text
~/.dbtools
```

This directory is used by SQLcl for its local connection store and related SQL Developer/SQLcl configuration data. For MCP specifically, SQLcl reads the preconfigured saved connections from this store, and those connections can be managed with the `CONNECT` and `CONNMGR` commands.

Oracle documentation:

- [Preparing Your Environment for SQLcl MCP](https://docs.oracle.com/en/database/oracle/sql-developer-command-line/25.2/sqcug/preparing-your-environment.html)
- [Managing Stored Connections with CONNMGR](https://docs.oracle.com/en/database/oracle/sql-developer-command-line/26.1/sqcug/connmgr.html)

Connections can be managed from SQLcl using `CONNMGR`.

For example:

```sql
connmgr list
```

and:

```sql
connmgr show <connection-name>
```

For a saved connection to be usable by an MCP client, Oracle requires its password to be saved. A connection can be created with `-save` and `-savepwd`, for example:

```sql
conn -save agent-dev -savepwd agent_user/password@//dbhost:1521/service
```

Oracle states that saved passwords are stored securely rather than as plain text.

This is better than putting database credentials in prompts, project files, or the Codex MCP configuration.

It should not, however, be treated as a replacement for workstation security. The connection store exists on the machine, and the Agent can use a saved connection without interactively entering its password.

I also would not assume that copying `~/.dbtools` to another machine is harmless. The Oracle documentation describes secure credential storage, but I have not found a documented guarantee that copying the connection store can never result in credential reuse. That is something worth testing separately rather than making assumptions about it.

## Listing and opening connections

The first useful MCP operation is to discover which SQLcl connections are available and then open one of them.

![Listing and opening SQLcl saved connections through Codex](/img/posts/codex-sqlcl-mcp-and-oracle/codex-sqlcl-connection-flow.png)

At a high level, the discovery flow is:

```text
Codex
  |
  | connections_list
  v
SQLcl MCP Server
  |
  | saved connections
  v
SQLcl Connection Manager
```

Once a connection is selected, Codex asks SQLcl MCP to open it:

```text
Codex
  |
  | connect
  v
SQLcl MCP Server
  |
  | selected saved connection
  v
Oracle Database
```

Once connected, Codex can request operations through SQLcl MCP, such as schema inspection or SQL execution.

Oracle documents the corresponding SQLcl capabilities as:

- `list-connections`
- `connect`
- `disconnect`
- `schema-information`
- `run-sql`
- `run-sqlcl`

In my Codex configuration, the tool-specific approval entries appear as:

```text
connections_list
connect
disconnect
schema_information
sql_run
sqlcl_run
```

The important point is that Codex is calling a defined MCP tool rather than receiving an unrestricted database credential and inventing its own connection mechanism.

## Verify the MCP server from Codex

After updating `~/.codex/config.toml`, the configured MCP servers can be checked with:

```bash
codex mcp list
```

A useful first test is intentionally simple: ask Codex to show the Oracle connections available through SQLcl.

That lets us verify the MCP path before allowing any SQL execution.

## Non-production does not mean disposable

It is easy to say that an AI Agent should not have access to production.

That is true, but it is not enough.

A shared DEV or integration environment can still contain:

- Years of accumulated test data
- Difficult-to-reproduce test scenarios
- Dependencies on other systems
- Large and complex schemas
- Masked or specially prepared data
- Environments that are refreshed only every few months
- Environments that are expensive or time-consuming to rebuild

An Agent deleting a central table or modifying a large amount of data can still create a serious incident even when the database is not production.

For this reason, I prefer to begin with the most restrictive SQLcl configuration that still supports the task, keep explicit tool approvals enabled while experimenting, and expand capabilities only when the lab actually requires them.
