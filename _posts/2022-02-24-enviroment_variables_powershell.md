---
layout: post
title:  "Environment variables in powershell in a nutshell"
author: "Sagiv Barhoom"
date:   2022-02-24
categories: PowerShell 
background: '/img/posts/pivot.jpg'
---

# Environment variables in powershell in a nutshell

Getting and setting environment variables using powershell (on windows)
ע

## GET
### Get all environment variables stored in the operating system:
```powershell
Get-ChildItem Env:
```

### Filter only the relevant names
```powershell
 Get-ChildItem Env: | Where-Object {$_.Name -like 'Program*'}
```

### Get environment variable value 
for example  get the `homepath` value
```powershell
  $env:homepath
```

### Get environment variable which represent list (like `PATH`)
```powershell
 $env:path -split ';'
```
---
layout: post
title:  "Enviroment Variables PowerShell"
author: "Sagiv Barhoom"
date:   202-02-24
categories: PowerShell 
background: '/img/posts/pivot.jpg'
---

## Set
### Setting  and appending new variable
```powershell
PS C:\Users\Sagiv>  $env:names = 'sagiv'
PS C:\Users\Sagiv>  $env:names += ';Sagiv Barhoom'
PS C:\Users\Sagiv>  $env:names -split ';'
sagiv
Sagiv Barhoom
```


### Setting environment variable persistently 
On Windows, environment variables can be defined in three scopes:

- Machine (or System) scope
- User scope
- Process scope

Lets set the environment persistently so they will remain even when close the currrent session.
Use `[System.Environment]` class with the `SetEnvironmentVariable` method.

#### user scope

```powershell
 [Environment]::SetEnvironmentVariable('names','Sagiv;Sagiv Barhoom', 'user')
```

#### system scope
*Note:* you need premmitions to edit the registry - so you might wat to run this as admin

```powershell
 [Environment]::SetEnvironmentVariable('names','Sagiv;Sagiv Barhoom', 'Machine')
```

