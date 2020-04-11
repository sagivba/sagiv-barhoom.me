---
layout: post
title:  "Analyzing archive files using Oracle log miner"
author: "Sagiv Barhoom"
date:   2020-03-15
categories: ORACLE 
---
# Analyzing archive files using Oracle log miner

## Where are the archive files located?
```sql
SELECT *
FROM v$parameter p
WHERE p.name LIKE '%log_archive_dest_%'
ORDER BY 1;
```


## Find the rate of arc files generation per day/minute
```bash
[oracle@webDB arc_dir]$ cd /your/arc/files/dir/
[oracle@webDB arc_dir]$ # per day
[oracle@webDB arc_dir]$ ls -ltr *.arc | cut -c 40-47 | uniq -c | sort -n  | tail -15
      8  Apr  4
    241  Apr  6
    382  Apr  5
[oracle@webDB arc_dir]$ # per minute    (till the 52'th char)            
[oracle@webDB arc_dir]$ ls -ltr *.arc | cut -c 40-50 | uniq -c | sort -n  | tail -7
     16  Apr  5 04:
     17  Apr  5 17:
     19  Apr  5 19:
     20  Apr  6 03:
     24  Apr  5 16:
    137  Apr  5 00:
    139  Apr  6 00:


```

Here we see that we have a lot of writing activity on 00:00-01:00  on both:  Apr  5'th and  Apr  6'th

## Analize the files using ```logmngr```
```bash
[oracle@webDB arc_dir]$ ls -ltr $PWD/arch_*.* |grep -P 'Apr  [56] 00:' |  cut -c 54-200 | perl -i -pe "s/^/$postfix/;"| perl -i -pe "s/$/$suffix/"
exec DBMS_LOGMNR.ADD_LOGFILE('/oracle_dir/arc_dir/arch_1_859056_620585656.arc');
exec DBMS_LOGMNR.ADD_LOGFILE('/oracle_dir/arc_dir/arch_1_859057_620585656.arc');
exec DBMS_LOGMNR.ADD_LOGFILE('/oracle_dir/arc_dir/arch_1_859058_620585656.arc');
exec DBMS_LOGMNR.ADD_LOGFILE('/oracle_dir/arc_dir/arch_1_859059_620585656.arc');
exec DBMS_LOGMNR.ADD_LOGFILE('/oracle_dir/arc_dir/arch_1_859060_620585656.arc');
exec DBMS_LOGMNR.ADD_LOGFILE('/oracle_dir/arc_dir/arch_1_859061_620585656.arc');
exec DBMS_LOGMNR.ADD_LOGFILE('/oracle_dir/arc_dir/arch_1_859062_620585656.arc');
exec DBMS_LOGMNR.ADD_LOGFILE('/oracle_dir/arc_dir/arch_1_859063_620585656.arc');
exec DBMS_LOGMNR.ADD_LOGFILE('/oracle_dir/arc_dir/arch_1_859064_620585656.arc');
exec DBMS_LOGMNR.ADD_LOGFILE('/oracle_dir/arc_dir/arch_1_859065_620585656.arc');
...
``` 

## Run the avove lines to add the log files
```sql
exec DBMS_LOGMNR.ADD_LOGFILE('/oracle_dir/arc_dir/arch_1_859056_620585656.arc');
exec DBMS_LOGMNR.ADD_LOGFILE('/oracle_dir/arc_dir/arch_1_859057_620585656.arc');
exec DBMS_LOGMNR.ADD_LOGFILE('/oracle_dir/arc_dir/arch_1_859058_620585656.arc');
exec DBMS_LOGMNR.ADD_LOGFILE('/oracle_dir/arc_dir/arch_1_859059_620585656.arc');
exec DBMS_LOGMNR.ADD_LOGFILE('/oracle_dir/arc_dir/arch_1_859060_620585656.arc');


```
## Start oracle log miner 
```sql
exec DBMS_LOGMNR.START_LOGMNR(options =>dbms_logmnr.dict_from_online_catalog);
```

## End oracle log miner session
```sql
exec DBMS_LOGMNR.END_LOGMNR();
```

## Start analizing...

Which actions are most common:
```sql
SELECT COUNT (*) n, lc.seg_owner, lc.table_name,  operation
FROM  v$logmnr_contents lc
GROUP BY lc.seg_owner, lc.table_name, operation
ORDER BY 1 DESC ;
```

Lets take a look at first 25 lines:
```
N          seg_owner  table_name                 operation
--------------------------------------------------------------
587146                                           INTERNAL
174896     USER1      CUTOMERS_TBL               UNSUPPORTED
132175     USER1      CUTOMERS_TBL               UPDATE
67346                                            START
67204                                            COMMIT
28043      USER2      REQUESTS_TBL               UNSUPPORTED
22502      AMSTERDAM  SESSIONS_TBL               UPDATE
9864       SYS        SUM$                       UNSUPPORTED
9834       SYS        SUM$                       UPDATE
6556       SYS        SNAP$                      UPDATE
5637       USER2      SMSES_TBL                  UNSUPPORTED
4206       USER2      MEET_SNP                   DELETE
4206       USER1      MEET_SNP                   INSERT
3283       SYS        SNAP_REFTIME$              UPDATE
3281       SYS        SUMDETAIL$                 UPDATE
2262       SYS        SEQ$                       UNSUPPORTED
2074       USER3      MEET_2_TBL                 INSERT
1795       SYS        WRH$_EVENT_HISTOGRAM       INSERT
1281       SYS        WRH$_SYSMETRIC_HISTORY     INSERT
729        USER1      SESSION_STATUS_TBL         UPDATE
695        SYS        MON_MODS$                  DELETE
679        SYS        WRH$_SYSSTAT               INSERT
582        SYS        WRH$_LATCH                 INSERT
541        SYS        MON_MODS_ALL$              UNSUPPORTED
501        SYS        JOB$                       UPDATE
```

Lets tke a look at the actions of CUTOMERS_TBL:

```sql
SELECT username,
       table_name, 
       to_char(timestamp,'mm/dd/yy hh24:mi:ss') timestamp,
	seg_type_name, seg_name, table_space, 
	session# SID, serial# ,operation
	--,substr(sql_redo,1,400) as sql_
FROM  v$logmnr_contents
WHERE table_name = 'CUTOMERS_TBL';
```


