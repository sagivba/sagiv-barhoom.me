---
layout: post
title:  "SQLPLUS configuration and settings"
author: "Sagiv Barhoom"
date:   2022-08-05
categories: ORACLE 
---
Here are some configurations and settings tips for Oracle SQLPLUS.



## Storing enviroment setiings
* `STORE SET file_name` - To store the current setting of all system variables, enter
* 'START file_name' - restore the stored system variables, enter

* settings
* `set linesize 200` - sets the number of characters to display on each report line
* `set pagesize 2000` - sets the number of lines to display in a report page
* `set long  10000` - Represents the number of characters you want to display from LONG columns (default is 80)
* `set time on` -  displays the current time in command prompt
* `set define off` -  turn variable substitution off - this is usfule when your script contatns '&' as characters.
* `set feadback off` - unset the comment at the end of your listing that tells you how many rows were returned. 
* `alter session set nls_date_format = 'yyyy-mm-dd hh24:mi:ss'`
* `set markup html on spool on` + `spool file.html` - create html file

## Formatimg coloumns
* COLUMN last_name format a20
* COLUMN total format 999,999,999
* COLUMN LAST_NAME        HEADING 'LAST NAME' or COLUMN LAST_NAME        HEADING 'LAST|NAME' for beeking th line



## Make SQL prompt show database name and time
```
column global_name new_value gname
set termout off
select lower(user) || '@' || SUBSTR(global_name, 0, INSTR(global_name, '.')-1)||' sql> ' as global_name from   global_name;
set sqlprompt '&gname'
set termout on
set time on
```
prompt will appear as: 
```
12:21:56 system@MYDEVDB sql>  
```


## Configuration files- ```login``` and ```glogin.sql```
When SQL*Plus first starts, it executes the ```login.sql``` and ```glogin.sql``` scripts from in the working directory or ```SQL_PATH```.
```SQLPATH` is a colon separated list of directories. There is no default value set in UNIX installations.
In Windows, ```SQLPATH``` is defined in a registry entry during installation. 
https://docs.oracle.com/cd/E11882_01/server.112/e16604/ch_two.htm#i1133354

After reading ```glogin.sql```, SQL*PLUS looks for the file ```login.sql``` and to executes it.
If the restriction level is set to 3, the login.sql is not read.
A common login.sql file

Since Oracle 10g, the login.sql is executed after a connect in order to  have a prompt that displays the username.

Take a look at the files:
```bash
$ORACLE_HOME/sqlplus/admin/glogin.sql
$ORACLE_HOME/sqlplus/admin/login.sql
```
