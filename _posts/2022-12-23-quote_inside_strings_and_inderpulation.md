---
layout: post
title:  "Quote inside strings and inderpulation examples"
author: "Sagiv Barhoom"
date:   2022-12-23
categories: ORACLE,Bash,Perl 
background: ''
---
# Quote inside strings and inderpulation examples

## quotes in sting using Oracle sql an plsql:
Single quote inside of a quoted string is escaped with second single quote: 'don''t'. But better use Q literals: Q'[don't]'.
```sql
select  'don''t walk away!', q'[don't] stay here' from  dual;
```
The result:
```
don't walk away! don't stay here
```

## In bash (and inrepilation)
```bash
echo "don't walk $HOME"          # --> dont walk /home/sagiv
echo 'don\'t walk $HOME'         # --> dont walk $HOME
echo "don\'t walk ${HOME}_2"     # --> dont walk /home/sagiv_2
```

## In Perl (and inrepilation)
```perl
print ("don't walk $ENV{HOME}\n");          # --> dont walk /home/sagiv
print (qq[don't walk $ENV{HOME}_2\n]);      # --> dont walk /home/sagiv_2
print ( q[don't walk $ENV{HOME}_2\n]);      # --> dont walk /$ENV{HOME}_2\n <--no new line here
printf("don't walk %s\n",$ENV{HOME});       # --> dont walk /home/sagiv
```



