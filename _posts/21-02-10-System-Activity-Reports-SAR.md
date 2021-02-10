---
layout: post
title:  "System Activity Report - SAR "
author: "Sagiv Barhoom"
date:   2020-02-10
categories: Linux 
background: '/img/posts/be-linux.jpg.jpg'

# System Activity Report (SAR) 
Using the SAR tool, we can look back over a period of time and see how the server has been running.
SAR historic data is stored in ```/var/log/sa``` directory on RedHat based distributions. 
```bash 
ls -ltr  /var/log/sa
```
## Changing sar historical data retention
Change the *HISTORY* parameter setting in ```/etc/sysconfig/sysstat``` or ```/etc/sysstat/sysstat```. 

## Utilities part of Sysstat
- ```sar``` - collects and displays ALL system activities statistics.
- ```sadc``` - stands for “system activity data collector”. This is the sar backend tool that does the data collection.
- ```sa1``` - stores system activities in the binary data files. sa1 depends on sadc for this purpose. sa1 runs from cron.
- ```sa2``` - creates daily summary of the collected statistics. sa2 runs from cron.
- ```sadf``` - can generate sar report in CSV, XML, and various other formats. Use this to integrate sar data with other tools.
- ```iostat``` - generates CPU, I/O statistics
- ```mpstat``` - displays CPU statistics.
- ```pidstat``` - reports statistics based on the process id (PID)
- ```nfsiostat``` - displays NFS I/O statistics.
- ```cifsiostat``` - generates CIFS statistics.


## Usecase for sar usage
We had a peak at one of our systems at one point
The system contains a web server and an oracle DB.
we had to view past performance on the relevant Linux servers.
The incidence occurred on the 9th day of the month between 13:50 and 14:10.
We want to compare the average load, CPU utilization,  memory usage, io, and network at that time frame.

### find the sar file:

Let's find the ```sa``` file, and then take a look at the:* Load average
* CPU
* I/O usage
* Network
* Memory

These are the sq files, and the relevant file is ```/var/log/sa/sa09``` since we want the 9th day of the month.

```bash
[DBA@my_DB ~]$ ls -ltr /var/log/sa
total 8992
-rw-r--r--  1 root root 604662 Sep  3 23:53 sar03
-rw-r--r--  1 root root 418416 Sep  4 23:50 sa04
-rw-r--r--  1 root root 604658 Sep  4 23:53 sar04
-rw-r--r--  1 root root 418416 Sep  5 23:50 sa05
-rw-r--r--  1 root root 604658 Sep  5 23:53 sar05
-rw-r--r--  1 root root 418416 Sep  6 23:50 sa06
-rw-r--r--  1 root root 604663 Sep  6 23:53 sar06
-rw-r--r--  1 root root 418416 Sep  7 23:50 sa07
-rw-r--r--  1 root root 604663 Sep  7 23:53 sar07
-rw-r--r--  1 root root 418416 Sep  8 23:50 sa08
-rw-r--r--  1 root root 604664 Sep  8 23:53 sar08
-rw-r--r--  1 root root 418416 Sep  9 23:50 sa09
-rw-r--r--  1 root root 604664 Sep  9 23:53 sar09
-rw-r--r--  1 root root 418416 Sep 10 23:50 sa10
-rw-r--r--  1 root root 604709 Sep 10 23:53 sar10
-rw-r--r--  1 root root 418416 Sep 11 23:50 sa11
-rw-r--r--  1 root root 605119 Sep 11 23:53 sar11
-rw-r--r--  1 root root 302256 Sep 12 17:10 sa12
```

### Load average 
```
[dba@my_DB ~]$ sar -q 1  -f    /var/log/sa/sa09 -s 08:00:00 -e 12:00:00 | grep -P  'Average|ld'
08:00:01 AM   runq-sz  plist-sz   ldavg-1   ldavg-5  ldavg-15
Average:            3       995      2.65      2.63      2.52
[dba@my_DB ~]$ sar -q 1  -f    /var/log/sa/sa09 -s 13:50:00 -e 14:10:00 | grep -P  'Average|ld'
01:50:01 PM   runq-sz  plist-sz   ldavg-1   ldavg-5  ldavg-15
Average:           12      1376     10.50      9.39      7.47
```

### CPU

```bash 
[dba@my_DB ~]$ sar -f    /var/log/sa/sa09 -s 08:00:00 -e 12:00:00 | grep -P  'Average|CPU'
Linux 2.6.32-431.29.2.el6.x86_64 (my_DB.openu.ac.il)   09/09/2020      _x86_64_        (12 CPU)
08:00:01 AM     CPU     %user     %nice   %system   %iowait    %steal     %idle
Average:        all     19.30      0.00      2.76      2.35      0.00     75.60 
[dba@my_DB ~]$ sar -f    /var/log/sa/sa09 -s 13:50:00 -e 14:10:00 | grep -P  'Average|CPU'
Linux 2.6.32-431.29.2.el6.x86_64 (my_DB.openu.ac.il)   09/09/2020      _x86_64_        (12 CPU)
01:50:01 PM     CPU     %user     %nice   %system   %iowait    %steal     %idle
Average:        all     56.63      0.00     10.64      6.94      0.00     25.79
```
### I/O usage
```bash 
[dba@my_DB ~]$ sar  -b 1 2   -f    /var/log/sa/sa09 -s 08:00:00 -e 12:00:00 | grep -P  'Average|tps'
08:00:01 AM       tps      rtps      wtps   bread/s   bwrtn/s
Average:       459.86    253.00    206.87  12812.10  10352.55
[dba@my_DB ~]$ sar  -b 1 2   -f    /var/log/sa/sa09 -s 13:50:00 -e 14:10:00 | grep -P  'Average|tps'
01:50:01 PM       tps      rtps      wtps   bread/s   bwrtn/s
Average:      3951.81   2983.20    968.60 121630.14   5737.18
```
### Network
```bash
[dba@my_DB ~]$ sar  -n DEV 1 1  -f    /var/log/sa/sa09 -s 08:00:00 -e 12:00:00 | grep -P  'Average|N'
Average:        IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s
Average:           lo     11.52     11.52      3.02      3.02      0.00      0.00      0.00
Average:         eth0   1059.57   1115.38    211.93   1259.58      0.00      0.00      0.00
[dba@my_DB ~]$ sar  -n DEV 1 1  -f    /var/log/sa/sa09 -s 13:50:00 -e 14:10:00 | grep -P  'Average|N'
Average:        IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s
Average:           lo      9.06      9.06      1.78      1.78      0.00      0.00      0.00
Average:         eth0   5917.22   6671.15    833.13  14262.91      0.00      0.00      0.00
```
### memory usage
```bash
[dba@my_DB ~]$ sar -r 1   -f    /var/log/sa/sa09 -s 08:00:00 -e 12:00:00 | grep -P  'Average|mem'
08:00:01 AM kbmemfree kbmemused  %memused kbbuffers  kbcached  kbcommit   %commit
Average:       784101  65189151     98.81    769229  48398725  44962113     54.33
[dba@my_DB ~]$ sar -r 1   -f    /var/log/sa/sa09 -s 13:50:00 -e 14:10:00 | grep -P  'Average|mem'
01:50:01 PM kbmemfree kbmemused  %memused kbbuffers  kbcached  kbcommit   %commit
Average:      1071591  64901661     98.38    541657  37337559  48939920     59.14
```

Now U leave you the clever reader to  conclude from the above data... :-)




