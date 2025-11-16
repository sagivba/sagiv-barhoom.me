---
layout: post
title:  "SQLcl for Apex APPS Deployment"
author: "Sagiv Barhoom"
date:   2024-11-19
categories: ORACLE 
background: '/img/posts/pivot.jpg'
---


# SQLcl for Apex APPS Deployment
Using SQLcl to Migrate Oracle APEX Applications Between DEV, TEST, and PROD

When working with Oracle APEX across multiple environments (DEV, TEST, PROD), 
SQLcl offers a simple and scriptable way to export and import applications and pages.

## Prerequisites:

### Which user to connect with?
You should connect using the parsing schema user of the APEX application (i.e., the schema assigned to the app). 
It does **not** have to be SYS or SYSTEM.

If your APEX application uses `DEV_USER` as the schema it runs under, connect with that user.

If you're using a generic user (like a CI/CD user), and your APEX schema is `DEV_USER`, you may also run:
```sql
ALTER SESSION SET CURRENT_SCHEMA = DEV_USER;
```
This allows SQLcl to execute commands as if you were connected directly to `DEV_USER`.

However, it is recommended to connect as the actual schema owner when exporting/importing APEX apps for consistency and access.

- Oracle SQLcl installed on your machine ([Download SQLcl](https://www.oracle.com/database/sqldeveloper/technologies/sqlcl/download/))
- Access to the source and target databases, and a process that saves export to structured file(s) to be run in sequence manually or via a wrapper script
- Workspace name known (e.g., `wrkspace`)

### 1. Exporting the APEX Application
#### listing APPEX APPS
connect with SQLcl to the DB and run
```sql
APEX LIST -verbose
```
Use SQLcl's APEX command to export the full application from the source environment.

There are two options for exporting:

- **With `-FORMAT split`**: Recommended for development and version control. 
The export will be broken into multiple logical files 
(e.g., pages, shared components, plugins) allowing selective review, versioning, and better diff tracking.

  ```bash
  APEX EXPORT -APP 100 -WORKSPACE wrkspace -DIR /path/to/export_dir -FORMAT split
  ```

- **Without `-FORMAT split`**: Generates a single consolidated SQL file. 
Use this when simplicity or minimal file handling is preferred, such as quick migrations or basic backups.

  ```bash
  APEX EXPORT -APP 100 -WORKSPACE wrkspace -DIR /path/to/export_dir
  ```

Choose `split` when you want modular exports for review or CI/CD (I do not use I dont use editionble DB ). 
Skip it for simple one-file handling.

```bash
sql -user dev_user/dev_pass@DEVDB

APEX EXPORT -APP 100 -WORKSPACE wrkspace -DIR /path/to/export_dir -FORMAT split
```

This creates a directory with the application split into readable SQL files for versioning or manual editing.

### 2. Pre-import Script (Optional)

Before importing, you may want to run preparatory scripts:

```bash
sql test_user/test_pass@TESTDB @00_get_ready.sql
```

This script may include cleanup, setting environment variables, or backing up existing data.

### 3. Importing to the Target Environment

Navigate to the export directory and run:

```bash
sql test_user/test_pass@TESTDB

APEX IMPORT -DIR /path/to/export_dir -APPID 100
```

### Notes:

- Make sure the target workspace (`wrkspace`) exists in the TEST/PROD environment.
- Use `-APPID` to keep the same ID across environments.

### Automation Tip:

Wrap the steps above in a PowerShell script to streamline your migrations. 
The script should stop execution on error to ensure safe deployment. 
To improve readability, use `-Command` blocks instead of nested double quotes. You can also pass the workspace name using environment variables. 
Example:
```powershell
$env:ErrorActionPreference = 'Stop'
$env:DEV_CONN = 'dev_user/dev_pass@DEVDB'
$env:TEST_CONN = 'test_user/test_pass@TESTDB'
$env:WORKSPACE = 'wrkspace'

sql -Command {
  APEX EXPORT -APP 100 -WORKSPACE $env:WORKSPACE -DIR ./apex_app -FORMAT split
} $env:DEV_CONN

sql -Command {
  @00_get_ready.sql
} $env:TEST_CONN

sql -Command {
  APEX IMPORT -DIR ./apex_app -APPID 100
} $env:TEST_CONN
```

This approach ensures each step completes successfully before continuing, helping to catch failures early and prevent partial deployments.

```bash
#!/bin/bash
sql dev_user/dev_pass@DEVDB <<EOF
APEX EXPORT -APP 100 -WORKSPACE wrkspace -DIR ./apex_app -FORMAT split
EOF

sql test_user/test_pass@TESTDB @00_get_ready.sql

sql test_user/test_pass@TESTDB <<EOF
APEX IMPORT -DIR ./apex_app -APPID 100
EOF
```

This makes your migration process consistent and repeatable across environments.
