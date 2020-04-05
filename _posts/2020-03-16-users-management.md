---   
layout: post
title: "Mangeing users and profiles -  Oracle DB"
subtitle: "Simple usage an examples - of user and profiles management"
date: 2020-03-16
background: '/img/posts/06.jpg'
---   

# Managing users and profiles on Oracle DB
## Users:
### List all  users
``` sql
SELECT *
FROM dba_users u
WHERE u.username LIKE 'LAMDANRUN';
```
### Unlock user
``` sql
ALTER USER iwebrun ACCOUNT UNLOCK;
```

### Change user password 
``` sql
ALTER USER LAMDANRUN IDENTIFIED BY strongpassword;
```
## Profiles
## Before you start 
Remember  the parameter ```RESOURCE_LIMIT``` determines whether resource limits are enforced in database profiles.
chek this parameter value and only then handle re profiles when needed.
tou can chec the value by using ```show parameter RESOURCE_LIMIT``` in sqlplus. 
### List profiles
#### List only profiles
```sql
SELECT * FROM dba_profiles dp
```
#### List profiles and users and resource_name
```sql
  SELECT du.username,
         du.profile,
         dp.resource_name,
         dp.resource_type,
         dp.LIMIT
    FROM dba_profiles dp INNER JOIN dba_users du ON dp.profile = du.profile
   WHERE     dp.resource_type = 'KERNEL'
         AND dp.LIMIT <> 'DEFAULT'
         AND dp.LIMIT <> 'UNLIMITED'
ORDER BY 2, 1
```


### Change user profile 
#### An Example  - prevent lock

``` sql
CREATE PROFILE umlimited_attempts LIMIT
  FAILED_LOGIN_ATTEMPTS UNLIMITED;

ALTER USER LAMDANRUN PROFILE  umlimited_attempts;

SELECT username, profile, account_status
  FROM dba_users
 WHERE username = 'LAMDANRUN';
```

``` sql
  SELECT du.username,
         du.profile,
         dp.resource_name,
         dp.resource_type,
         dp.LIMIT
    FROM dba_profiles dp INNER JOIN dba_users du ON dp.profile = du.profile
   WHERE     dp.resource_type = 'KERNEL'
         AND dp.LIMIT <> 'DEFAULT'
         AND dp.LIMIT <> 'UNLIMITED'
ORDER BY 2, 1
```
``` sql
SELECT DBMS_METADATA.get_ddl ('PROFILE', profile) AS profile_ddl
  FROM (SELECT DISTINCT profile FROM dba_profiles)
 WHERE profile LIKE UPPER ('%TIME_LIMIT%');
```

