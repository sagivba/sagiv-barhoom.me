---
layout: post
title:  "SQLcl for Apex APPS Deployment"
author: "Sagiv Barhoom"
date:   2024-11-19
categories: ORACLE 
background: '/img/posts/pivot.jpg'
---

# SQLcl for Apex APPS Deployment
(Version 23.2)
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
- Access to DEV/TEST/PROD
- Workspace name (e.g., `wrkspace`)

## 1. Listing APEX Applications

```sql
APEX LIST
```
## 2. Exporting the APEX Application

### Export into a single file
First, set your export dir:
`SET APEXEXPORTDIR  /path/to/export_dir`
```bash
APEX export-application -applicationid 185
```

## 3. importing - run the script in the destination DB
```sql
@ /path/to/export_dir/f100.sql
```


