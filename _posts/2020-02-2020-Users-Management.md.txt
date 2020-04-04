---   
layout: post
title: "Mangeing users and profiles Oracle DB"
subtitle: "Simple usage an examples - of user and profiles management"
date: 2020 02 17 10:45:13  0400
background: '/img/posts/06.jpg'
---   

# mangeing users and profiles
## users
### list users
``` sql
SELECT *
FROM dba_users u
WHERE u.username LIKE 'LAMDANRUN';
```
### unlock user
``` sql
ALTER USER iwebrun ACCOUNT UNLOCK;
```

### change user password 
``` sql
ALTER USER LAMDANRUN IDENTIFIED BY strongpassword;
```
## profiles
### list profiles
#### only profiles
```sql
SELECT * FROM dba_profiles dp
```
#### list profiles and users and resource_name
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


### change user profile 
#### prevent lock

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

