---
layout: post
title:  "Using the cut command"
author: "Sagiv Barhoom"
date:   2020-04-06
categories: Linux 
background: '/img/posts/tailor_scissors.jpg'
---
# The ```cut``` command  - usage and examples
The ```cut``` command can be used for extracting portion of text from a file by selecting columns.
Here are some useful examples:
## For each line extract characters 5-7:
```bash
cut -c 5-7 file.txt
```
First useful example - (LOCAL=NO) processes are the processes of connections using SQL*net (localhost or remote machines) and are not using MTS (Multi-Threaded server)
so if we want to list all the pids of processes of connections using SQL*net:
```
ps -ef | grep 'LOCAL=NO' | cut -c 10-15 | wc -l
```

Another example, show the number of files per day in the listed directory:

```bash
$ ls -lt  | cut -c 42-47| uniq -c | sort -n
1
1 Aug 15
1 Jan 10
1 Nov 19
1 Oct 26
1 Oct 27
2 Nov  9
7 Jun 27
```
Explanation:
ls -lt returns lines which look like this:
```
-rw-r--r--    1 Sagiv    UsersGrp       400 Dec 14 19:19 1.txt
```
The lines are sorted by date, so ```cut``` returns only the month and day.
The command ```uniq -c``` counts the lines of each date, and the ```sort -n``` sorts it by number.


## Parse CSV file and create file with the second,third,fourth and sixth colomns.
```bash
cut -f 3-4,6 -d ',' file.csv
```

Here ```-d``` can be any separator (one character).

For example, list users with "User ID Info":
```bash
cut -f1,5 -d':' /etc/passwd  | sort | grep -P ':\w'
 adm:adm
 avahi:Avahi mDNS/DNS-SD Stack
 bin:bin
 colord:User for colord
 daemon:daemon
 dbus:System message bus
 ftp:FTP User
 games:games
 geoclue:User for geoclue
 gluster:GlusterFS daemons
 halt:halt
```
