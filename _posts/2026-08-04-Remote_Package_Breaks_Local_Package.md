---
layout: post
title: "When a Remote Package Breaks After a Column Change - Even Though the Package Is VALID"
author: "Sagiv Barhoom"
date: 2026-08-04
categories: ORACLE
background: '/img/posts/oracle_dblink.png' 
---

A local package failed to compile after a column was changed in a remote database, even though the related remote PL/SQL units were still marked as `VALID`.

## Architecture

This behavior was observed in an Oracle Database 19c environment.
The system used two databases:

```text
Local Database ---dblink---> Remote Database
```

A local business package called a package in the remote database:

```text
APP.ORDER_PROCESSOR --> MSG.MESSAGE_SERVICE@REMOTE_DB --> MSG.MESSAGE_HEADER
```

The remote package worked with the `MSG.MESSAGE_HEADER` table.

## What Changed

The length of one column was increased in the remote database:

```sql
ALTER TABLE MSG.MESSAGE_HEADER MODIFY subject_text VARCHAR2(1000);
```

After the change, compiling the local package failed with an error similar to:

```text
PLS-00907: cannot load library unit
MSG.MESSAGE_HEADER@REMOTE_DB
(referenced by MSG.MESSAGE_SERVICE@REMOTE_DB)
```

Additional errors such as the following also appeared:

```text
PL/SQL: Statement ignored
```

Those messages pointed to statements that called the remote package, but they were secondary compilation errors rather than the most useful diagnostic message.

## Why It Was Confusing

The remote package specification and package body were both marked as `VALID`:

```text
MESSAGE_SERVICE PACKAGE       VALID
MESSAGE_SERVICE PACKAGE BODY  VALID
```

A regular recompilation also completed without errors:

```sql
ALTER PACKAGE MSG.MESSAGE_SERVICE COMPILE;
ALTER PACKAGE MSG.MESSAGE_SERVICE COMPILE BODY;
```

The local package still failed to compile.

The table itself was available, but its status was not meaningful evidence that all PL/SQL dependency metadata was consistent. 
A `VALID` PL/SQL unit also does not guarantee that every remote dependency load through a database link will succeed.

## The Important Dependency

The remote package specification used `%TYPE` references to columns in the table:

```sql
PROCEDURE send_message(
    p_message_id IN MSG.MESSAGE_HEADER.message_id%TYPE,
    p_subject    IN MSG.MESSAGE_HEADER.subject_text%TYPE
);
```

The important detail is that `%TYPE` appeared in the package specification.

This creates a compile-time dependency between the package specification and the referenced column definition. 
The package spec depended on column-derived metadata through `%TYPE`, 
so the column change required that metadata to be resolved correctly when the package was compiled and referenced remotely.

The base type of `subject_text` remained `VARCHAR2`. 
Therefore, the column length change alone does not prove that the remote PL/SQL signature used by `REMOTE_DEPENDENCIES_MODE=SIGNATURE` changed.

## What Did Not Work

Explicit recompilation did not resolve the issue:

```sql
ALTER PACKAGE MSG.MESSAGE_SERVICE COMPILE;
ALTER PACKAGE MSG.MESSAGE_SERVICE COMPILE BODY;
```

Closing the database link also did not help:

```sql
ALTER SESSION CLOSE DATABASE LINK REMOTE_DB;
```

Starting a new session produced the same result.

`REMOTE_DEPENDENCIES_MODE` was already set to:

```text
SIGNATURE
```

That setting determines how Oracle checks remote PL/SQL dependencies. 
Confirming or changing the setting does not itself recompile or redefine the remote package.

Its value also does not prove that changing the length of a `VARCHAR2` column should have been treated as a signature change. 
In this incident, the setting did not prevent the load failure, but that does not mean it was expected to fix it.

## The Fix

The remote package specification was recreated with `CREATE OR REPLACE`:

```sql
CREATE OR REPLACE PACKAGE MSG.MESSAGE_SERVICE AS
    ...
END;
/
```

The package body was then recreated:

```sql
CREATE OR REPLACE PACKAGE BODY MSG.MESSAGE_SERVICE AS
    ...
END;
/
```

After that, the local package compiled successfully.

When using a deployment tool, verify that it actually executes both `CREATE OR REPLACE` statements.

Also verify whether the package source sent to Oracle differs from the currently stored source. 
Oracle Database 19c documents that `CREATE OR REPLACE` of a PL/SQL object can have no effect when the source text and the stored compilation settings are identical.

A harmless source change, such as adding or modifying a comment, can therefore serve two purposes: 
it can bypass checksum-based deployment logic, and it can ensure that Oracle receives changed source text.

The comment has no runtime meaning and does not change the package logic. 
Its purpose is only to make the source text different so that the package definition is processed again.

```sql
CREATE OR REPLACE PACKAGE MSG.MESSAGE_SERVICE AS

    -- Force package redefinition after dependency incident

    ...
END;
/
```

## Why It Worked

In this incident, redefining the package with `CREATE OR REPLACE` and changed source text resolved the inconsistency, while explicit recompilation did not.

This suggests stale or inconsistent compiled dependency metadata, but the precise internal cause was not established.

No trace was collected that identified a specific stale cache, dictionary entry, signature record, or internal Oracle mechanism. The result should therefore be treated as an observed workaround, not as proof of a general internal behavior.

## Recommended Troubleshooting Order

1. Collect all local compilation errors, not only the first reported message.
2. Identify the complete dependency chain across the database link.
3. Check the remote package specification and package body.
4. Review `ALL_ERRORS` or `DBA_ERRORS` on the remote database:

   ```sql
   SELECT owner,  name, type, line,  position, text
   FROM all_errors
   WHERE owner = 'MSG' AND name = 'MESSAGE_SERVICE'
   ORDER BY sequence;
   ```

5. Check `REMOTE_DEPENDENCIES_MODE` in the same session that performs the local compilation:

   ```sql
   SELECT value
   FROM v$parameter
   WHERE name = 'remote_dependencies_mode';
   ```

6. Recompile the remote package specification and body normally:

   ```sql
   ALTER PACKAGE MSG.MESSAGE_SERVICE COMPILE;
   ALTER PACKAGE MSG.MESSAGE_SERVICE COMPILE BODY;
   ```

7. Start a new session or close the database link, then retry the local compilation.
8. If the error remains, compare the deployed source with the currently stored source, then run `CREATE OR REPLACE` for the remote package specification and package body using changed source text.
9. When possible, record `LAST_DDL_TIME`, preserve the deployed source, and document the exact commands executed before making the change.

