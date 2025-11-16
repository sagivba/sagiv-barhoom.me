---
layout: post
title:  "SQLcl for Apex APPS Deployment"
author: "Sagiv Barhoom"
date:   2024-11-19
categories: ORACLE 
background: '/img/posts/pivot.jpg'
---

# SQLcl for Apex APPS Deployment
Using SQLcl to Migrate Oracle APEX Applications Between DEV, TEST, and PROD.

SQLcl provides a clean, scriptable way to export and import APEX applications across environments.

## Prerequisites

### Which user should you connect with?
Connect using the **parsing schema** of the APEX application.

If your application runs under `DEV_USER`, connect using:

```
sql dev_user/dev_pass@DEVDB
```

If using a CI/CD user, set:

```sql
ALTER SESSION SET CURRENT_SCHEMA = DEV_USER;
```

### Requirements
- SQLcl installed
- Access to DEV / TEST / PROD
- Workspace name (e.g., `wrkspace`)

---

## 1. Listing APEX Applications

```sql
APEX LIST -verbose
```

---

## 2. Exporting the APEX Application

### Export into a single file
```bash
APEX EXPORT -appid 100 -workspace wrkspace -dir /path/to/export_dir
```

### Export in split format (recommended for Git)
```bash
APEX EXPORT -appid 100 -workspace wrkspace -dir /path/to/export_dir -format split
```

### Full example
```bash
sql dev_user/dev_pass@DEVDB

APEX EXPORT -appid 100 -workspace wrkspace -dir /path/to/export_dir -format split
```

---

## 3. Pre-import Script (Optional)

```bash
sql test_user/test_pass@TESTDB @00_get_ready.sql
```

---

## 4. Importing to the Target Environment

```bash
sql test_user/test_pass@TESTDB

APEX IMPORT -dir /path/to/export_dir -appid 100
```

---

## PowerShell Automation Example

```powershell
$env:ErrorActionPreference = 'Stop'
$env:DEV_CONN = 'dev_user/dev_pass@DEVDB'
$env:TEST_CONN = 'test_user/test_pass@TESTDB'
$env:WORKSPACE = 'wrkspace'

sql -Command {
  APEX EXPORT -appid 100 -workspace $env:WORKSPACE -dir ./apex_app -format split
} $env:DEV_CONN

sql -Command {
  @00_get_ready.sql
} $env:TEST_CONN

sql -Command {
  APEX IMPORT -dir ./apex_app -appid 100
} $env:TEST_CONN
```

---

## Bash Automation Example

```bash
#!/bin/bash
sql dev_user/dev_pass@DEVDB <<EOF
APEX EXPORT -appid 100 -workspace wrkspace -dir ./apex_app -format split
EOF

sql test_user/test_pass@TESTDB @00_get_ready.sql

sql test_user/test_pass@TESTDB <<EOF
APEX IMPORT -dir ./apex_app -appid 100
EOF
```

This ensures reliable and repeatable APEX migrations.
