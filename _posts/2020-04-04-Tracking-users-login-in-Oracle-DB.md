---   
layout: post
title: "Tracking users login in Oracle DB"
date:   2020-04-04
background: "/img/posts/tracking_footprints.jpg"
categories: Oracle
---   
# Tracking users logon

DBA_USERS describes all users of the database.
DBA_AUDIT_SESSION displays all audit trail records concerning CONNECT and DISCONNECT.
So the following query might help undretstand and Track user login:

``` sql
SELECT s.username,
       s.userhost,
       s.terminal,
       s.timestamp,
       s.action_name,
       u.*
  FROM dba_users u LEFT JOIN dba_audit_session s ON u.username = s.username
 WHERE u.account_status = 'LOCKED';
 
```
