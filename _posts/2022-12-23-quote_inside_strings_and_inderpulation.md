---
layout: post
title:  "Quote inside strings and inderpulation examples"
author: "Sagiv Barhoom"
date:   2022-12-23
categories: ORACLE,Bash,Perl,PowerShell 
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
The full explaination is [here](https://www.gnu.org/software/bash/manual/html_node/Double-Quotes.html)
The behavior is very similar in Perl and PowerShell
```bash
echo "don't walk $HOME"                     # --> don't walk /home/sagiv
echo 'don\'t walk $HOME'                    # --> don't walk $HOME
echo "don\'t walk ${HOME}_2"                # --> don't walk /home/sagiv_2
```

## In Perl (and inrepilation)
```perl
print ("don't walk $ENV{HOME}\n");          # --> don't walk /home/sagiv
print (qq[don't walk $ENV{HOME}_2\n]);      # --> don't walk /home/sagiv_2
print ( q[don't walk $ENV{HOME}_2\n]);      # --> don't walk /$ENV{HOME}_2\n <--no new line here
printf("don't walk %s\n",$ENV{HOME});       # --> don't walk /home/sagiv
```
 ## In PowerShell (and inrepilation)
```powershell
Write-Output "don't go away!"                    # --> don't go away!
Write-Output '"give me one million dollars..!!"' # --> "give me one million dollars..!!"
Write-Output "don't walk $home"                  # --> don't walk /home/sagiv
Write-Output 'don\'t walk $home'                 # --> don't walk $home
```




