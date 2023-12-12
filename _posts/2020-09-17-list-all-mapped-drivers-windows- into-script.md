---
layout: post
title:  "List and edit all mapped drivers(windows) - into a script"
author: "Sagiv Barhoom"
date:   2020-09-17
categories: Windows 
background: '/img/posts/window-driver.jpg'
---

# `use net` - list and edit all mapped drivers by CLI
This is the easiest way to mapp a new driver or to copy your configuration to another windows machine.

## List all maped drivers:
```
net use > mapped_dirvers.bash
```
## Edit the file ```mapped_dirvers.bash``` into somthig like:
```
net use H: \\work.local\homes\sagivba\home  /persistent:yes
net use N: \\work.local\nets                /persistent:yes
net use S: \\work.local\sharing             /persistent:yes
net use T: \\work.local\sharing\nohal$      /persistent:yes
net use W: \\work.local\sharing\wordtmp$    /persistent:yes
```
