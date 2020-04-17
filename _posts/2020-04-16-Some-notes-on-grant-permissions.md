---
layout: post
title:  "Some notes on grant permissions"
author: "Sagiv Barhoom"
date:   2020-04-16
categories: ORACLE 
---

# Some notes on grant permissions

### common grants commands
We want the user *intrauser* to have the ability to perform ```SELECT```, ```UPDATE```, ```INSERT```, and ```DELETE``` capabilities on the table ```students_tbl``` in the ```intra_schema`` schema, we might execute the following GRANT statement:
### Grant CRUD example
```sql
GRANT  SELECT, INSERT, UPDATE, DELETE ON  intra_schema.students_tbl  TO intrauser;
```

## ```WITH ADMIN OPTION``` vs ```WITH GRANT OPTION```
Usage:
```sql
GRANT  CREATE INDEX TO intrauser WITH ADMIN OPTION;
GRANT  CREATE INDEX TO intrauser WITH GRANT OPTION;
```

These options hand over the control of the privileges from a single owner to multiple users.
###  ```WITH GRANT OPTION```:
- Only for object privileges, not system privileges.
- Only the person who granted the privilege can revoke the privilege.
**Revoked privileges can "cascade"**, allowing the first grantor to revoke many subsequent grants.

###  ```WITH ADMIN OPTION```:
Only for [system privileges](  #System-privileges ), not [object privileges](#Object-privileges).

#### System privileges vs object privileges
##### System privileges
**System privileges** allow the user to perform functions that deal with managing the database and the server. 
Most of the different types of permissions supported by the database vendors fall under the system privilege category, for example:
1. The ```CREATE USER``` permission, when granted to a database user, allows that database user to create new users in the database.
2. The ```CREATE TABLE``` permission, allows the database user to create tables in their **own** schema. 
This type of privilege is also available for other object types – like stored procedures and indexes.
3. The ```CREATE SESSION``` permission, allows the user to connect to the database.

#### Object privileges
**Object privileges** are privileges given to users so that they can perform certain actions upon certain database objects.Eexamples of object privileges:
Grant  DELETE and/or SELECT from a particular table. 
  This is done using the GRANT clause as seen [above](#Grant-CRUD-example).
