---
layout: post
title:  "vim macros"
author: "Sagiv Barhoom"
date:   2021-02-20
categories: Linux 
background: '/img/posts/be-linux.jpg.jpg'
---

# vim  macros
vim macro allows you to record a sequence of commands that you use to perform a given task. You then execute that macro many times to repeat the same job in an automated way.

## Recording a macro and using it
1. Start record thw macro by: ```[ESC] q <a letter>``` (vim will inform you "recording @<theletter you chose>)"
2. Record your commands and keystrokes while doing the edits required to solve the problem on the first segment (word/line/paragraph...)
3. Save the command sequence to the register you chose by - ```[ESC] q```
4. Apply the macro to do the same on the remaining segments - ```[ESC] @<the letter you chose>


## Example

Wrapping paragraph of PL/SQL code with ```BEGIN END;```  lines;
Before:
```sql
DBMS_OUTPUT.PUT_LINE(SYSDATE); 
```
After:
```sql
BEGIN
DBMS_OUTPUT.PUT_LINE(SYSDATE);
END;
```


### Start record the macro by: ```[ESC] q s``` (the letter for the macro will be `s`)
Place the cursor on the paragraph.
Start recording by ```[ESC] q s``` (vim will inform you"recording @s"

### Record your commands and keystrokes
Type the following sequence: ```{oBEGIN[ESC]}OEND;[ENTER][ESC]```.
let's break it down...
#### Add BEGIN where the paragraph starts
1. So we want to move to aline before the current paragraph ```{``` will get us a line before  the paragraph
2. We want to add anew line: ```o``` will add a new line and will get us to insert mode in that line.
3. Now let type at that line ```BEGIN```
#### Add END; where the paragraph ends
4. Let us move to the end of the paragraph:
        * Type ```[ESC]``` for command mode
        * Type ```}``` to move to the end of the paragraph
5. Add a new line above the current line using ```O``` this will change the mode to insert mode
6. Type ```END;``` and <ENTER>
6. Return to command mode by ```[ESC]```

### Save the command sequence to the register you chose.
on command mode type ```[ESC] q```

## Apply the macro

Place  your cursor on another paragraph
On command mode type ```@s```

Applying the macro multiple times  (for example 7 times) can be done by typing  ```7@s```.
