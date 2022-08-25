---   
layout: post
title: "What is eating my PGA"
date: 2022-08-25
background: '/img/posts/memory-leak.jpg'
categories: Oracle
--- 

# What is eating my PGA?

ORA-04036: PGA memory used by the instance exceeds PGA_AGGREGATE_LIMIT

## Some relevant terms
`PGA` - The Program Global Area is a private memory region that contains the data and control information for a server process
`PGA_AGGREGATE_LIMIT` specifies a limit on the aggregate PGA memory consumed by the instance. 
`PGA_AGGREGATE_TARGET` specifies the target aggregate PGA memory available to all server processes attached to the instance.

## ORA-04036
We got `ORA-04036: PGA memory used by the instance exceeds PGA_AGGREGATE_LIMIT`
so I tried to find what is consuming my PGA...
Here Are two queries that I found helpful:

## what modules/actions/programs consume the PGA
```sql

  SELECT s.STATUS,
         s.module,
         s.action,
         --s.MACHINE,
         --s.SCHEMANAME,
         --s.PROGRAM
         count(s.sid) Number_of_sessions,
         SUM (ROUND (p.PGA_USED_MEM / 1024 / 1024))      PGA_USED_MB,
         SUM (ROUND (p.PGA_ALLOC_MEM / 1024 / 1024))     PGA_ALLOC_MB
    FROM v$session s, v$process p
   WHERE s.paddr = p.addr
GROUP BY s.STATUS, module, action
         --, s.MACHINE,s.SCHEMANAME,s.PROGRAM
ORDER BY PGA_USED_MB DESC;
```


## Is there any suspected session which consumes a lot of PGA?
```sql
SELECT s.STATUS,
         module,
         action,
         --s.MACHINE,
         --s.SCHEMANAME,
         --s.PROGRAM
         ROUND (p.PGA_USED_MEM / 1024 / 1024)      PGA_USED_MB,
         ROUND (p.PGA_ALLOC_MEM / 1024 / 1024)     PGA_ALLOC_MB,
         s.SID,s.s.SERIAL#

    FROM v$session s, v$process p
   WHERE     s.paddr = p.addr
         --AND status = 'INACTIVE'
         --AND s.PREV_EXEC_START < SYSDATE - 16 / 24
         -- AND module IN ( <list of modules> )
         --AND s.MACHINE IN (<list of machines>)
         --AND s.SCHEMANAME (<list of schemas>)
         --AND s.PROGRAM    (<list of programs>)
ORDER BY PGA_ALLOC_MB DESC;
```
