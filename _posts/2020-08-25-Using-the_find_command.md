---
layout: post
title:  "Using the find command"
author: "Sagiv Barhoom"
date:   2020-08-25
categories: Linux 
background: '/img/posts/tailor_scissors.jpg'
---

# Some useful snippets for the ```find``` command

## Search files by time limitations
``` bash
find /search/dir -cmin -60                              # change time - older than 60 minutes
find /search/dir -cmin +60                              # change time - younger than 60 minutes
find /search/dir -amin -60                              # access time
find /search/dir -newer file_from-2020.08.20-08:00.tmp  # newer then  file_from-2020.08.20-08:00.tmp
```
from [stackoverflow](https://stackoverflow.com/questions/543946/find-mtime-files-older-than-1-hour):
* ```-mtime N``` means files whose age A in days satisfies N ≤ A < N+1. In other words, -mtime N selects files that were last modified between N and N+1 days ago.
* ```-mtime -N``` means files whose age A satisfies A < N, i.e. files modified less than N days ago. 
* ```-mtime +N``` means files whose age A satisfies N+1 ≤ A, i.e. files modified at least N+1 days ago.

For example, -mtime 1 selects files that were modified between 1 and 2 days ago. -mtime +1 selects files that were modified at least 2 days ago. To get files modified at least 1 day ago, use -mtime +0.

## Search file by its name
```bash
find /search/dir -name file.txt                         # find by name
find /search/dir -iname file.txt                        # find by name ignoring case
find /search/dir -type d -name my_dir                   # find directories using the dir name
find ./test -not -name "*.php"                          # revert match
```
## Search file by its permissions and ownership
```bash
find /search/dir -type f -perm 0717 -print              # find files with 717 permissions
find /search/dir -perm /u=r                             #  find read-only files
find /search/dir -user  sagiv                           #  find all files whose owner is Sagiv 
```

## Search file by its size
```bash
find /search/dir -type f -empty                         # find all empty files
find /search/dir -size +50M -size -100M                 # find file whos size between 50MB and 100MB
find . -type f -exec ls -s {} \; | sort -n -r | head -5 # find largest and smallest files
```

## Limit the search by the depth of the directory tree
```bash
find . -maxdepth 3 -name "*.txt"
find . -mindepth 3 -name "*.txt"
```
## Other useful examples
```bash
find . -type f  -iname *.fmb -perm 0644 -print -exec chmod 664 {} \; # find fmb files with 644 permitions and change it to 664
```

## ```{} ';'``` vs  ```{} '+'```
```bash
$ find /etc/rc* -exec echo Arg: {} ';'
Arg: /etc/rc.common
Arg: /etc/rc.common~previous
Arg: /etc/rc.local
Arg: /etc/rc.netboot

$ find /etc/rc* -exec echo Arg: {} '+'
Arg: /etc/rc.common /etc/rc.common~previous /etc/rc.local /etc/rc.netboot
```



