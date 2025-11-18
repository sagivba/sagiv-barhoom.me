---
layout: post
title:  "Oracle Wallet vs SQLcl -savepwd"
author: "Sagiv Barhoom"
date:   2025-10-04
categories: ORACLE 
background: '/img/posts/pivot.jpg'
---


# Oracle Wallet vs SQLcl -savepwd

When you need to store database login credentials securely, two common options are available to DBAs: 
using `Oracle Wallet` or SQLcl's `-savepwd` option. 
`Oracle Wallet` is a secure, encrypted container managed by Oracle, 
while `-savepwd` stores credentials in a user-local encrypted file.

Those methods differ significantly in security and usage.

## Oracle Wallet
Oracle Wallet is a secure container for storing credentials or certificates used to connect to databases. 
It supports auto-login, allowing passwordless connections.

## Features:
- Stores encrypted files such as `ewallet.p12` and `cwallet.sso`.
- Enables passwordless authentication using Oracle's Secure External Password Store (SEPS).
- **Supported across Oracle tools like SQL*Plus,SQcl, RMAN, expdp, etc**.
- If auto-login is enabled, any process with file access can use the wallet.
- Requires strict file and directory permission control.
- Designed for production, automation, and enterprise-level deployments.

## SQLcl `-savepwd`

SQLcl allows saving login credentials using either full connection strings or TNS aliases defined in `tnsnames.ora`, such as:

```bash
# Using a full connection string
sql -save my_conn -savepwd scott/tiger@//dbhost:1521/orclpdb1

# Using a TNS alias
sql -save my_alias -savepwd scott/tiger@ORCL_ALIAS
```

### Features:
- Stores credentials in a `connections.json` file in the user's home directory :
  - `~/.sqlcl` on Linux 
  - `%USERPROFILE%\sqlcl` on Windows).
- Passwords are encrypted **but only decryptable by the same OS user**.
- Requires OS-level access to the user account that created the profile.
- Best suited for development or personal environments, and supported only by SQLcl.
- Not considered secure for enterprise or production use.

## Comparison:

| Feature                | Oracle Wallet            | SQLcl `-savepwd`           |
| ---------------------- | ------------------------ | -------------------------- |
| Security level         | High (strong encryption) | Medium (OS user-dependent) |
| OS user dependency     | No                       | Yes                        |
| Tool support           | All Oracle tools         | Only SQLcl                 |
| Production suitability | Recommended              | Not recommended            |
| Developer convenience  | Requires initial setup   | High                       |

## Summary

Use Oracle Wallet for secure, production-grade automation. 
Use `-savepwd` only for quick, personal, or development use where convenience outweighs strict security.
