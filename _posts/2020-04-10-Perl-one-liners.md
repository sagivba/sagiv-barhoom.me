---   
layout: post
title: "Perl one-liners"
date:   2020-04-10
background: "/img/posts/camel-perl.jpg"
categories: Perl
---  
# Perl one-liners

## Useful options for Perl one-liners
* ```-e```  It allows you to provide the program as an argument rather than in a file.
* ```-p ``` Places a printing loop around your command so that it acts on each line of standard input.
* ```-n ``` Places a non-printing loop around your command.
* ```-i ``` Modifies your input file in-place (making a backup of the original). Handy to modify files without the {copy, delete-original, rename} process.
* ```-w ``` Activates some warnings. Any good Perl coder will use this.
* ```-d ``` Runs under the Perl debugger. For debugging your Perl code, obviously.
* ```-l Octal_number ``` - Sets ```$\``` () to the character represented by that number and also turns on auto-chomping.
* ```-M Module::Name``` Use "Module::Name"



# Some snipset
1. Replace "this" with "that" in a list of files
   ```bash
   perl -pi -e 's/this/that/g' file1 file2 file3
   ```

2. Replace "this" with "those" in a list of files, but only on lines that match also "these".
```bash
perl -pi -e 's/this/those/g if /these/' file
```
  ```

3. Find all repeated lines in a file
   ```bash
   perl -ne 'print if $a{$_}++' file
   ```

4. Like the command :```unix2dos``` (conversion from Unix to Windows format). 
   The option ```-i.bak``` will create backup 
   You can convert Unix file back to Windows format too:
   ```bash
   perl -i.bak -pe 's/\n/\r\n/' filename 
   ```
   
5. Print range of lines in file (e.g lines 21-37)
   ```bash
   perl -ne 'print if 21 .. 37'
   ```

6. Print balance of quotes in each line (useful for finding missing quotes in Perl and other scripts)
   ```bash
   perl -ne '$q=($_=~tr/"//); print"$.\t$q\t$_";' filename
   ```
   
7. Print only the lines between "START" and "END"
   ```bash
   perl -i.old -ne 'print if /^START$/ .. /^END$/' foo.txt 
   ```
   
8. Compress consecutive empty row instances to a single instance
    ```bash
    perl -00 -pe '' file
    ```
    
9. Encod/Decode Base64
    ```bash
    perl -MMIME::Base64 -0777 -ne 'print encode_base64($_)' file
    perl -MMIME::Base64 -le 'print decode_base64("string")'
    ```
    Wait there is more :-)
    - For URL escape - use ```URL::escape::uri_escape("$string")``` and ```URL::escape::uri_unescape("$esc_string")```
    - For HTML encode - use ```HTML::Entities::encode_entities("<string>")``` and ```decode_entities("&lt;string&gt;")```


## How does it work?
### The option ```-p``` vs ```-n```
While running:
```bash
perl -n -e 'some code' file1
```
Perl will translate it into the following code:
```perl
while (<>) {
        some code
}
```


### The option ```-l```
This option enables automatic line-ending processing.
It has two effects:
1. It automatically chops the line terminator when used with ```-n``` or ```-p```
2. Assigns ```$\``` to have the value of octal number so that any print statements will have that line terminator added back on.
   If octal number is omitted, sets ```$\``` to the current value of ```$/```. F
For instance, to trim lines to 80 columns:
```bash
perl -lpe 'substr($_, 80) = ""'
```
*Note*: The assignment ```$\ = $/``` is done when the switch ```-l``` is processed,
        so the input record separator can be different than the output record separator if the ```-l`` switch is followed by a ```-0`` switch:
        ```bash
        find / -print0 | perl -ln0e 'print "found $_" if -p'
        ```
        This sets $\ to newline and then sets $/ to the null character.

#### The variables: ```$/``` and  ```$\```
*  ```$/``` - The input record separator, newline by default. This influences Perl's idea of what a "line" is.
Works like awk's RS variable, including treating empty lines as a terminator if set to the null string (an empty line cannot contain any spaces or tabs).

* ```$\``` - The output record separator for the print operator. If defined, this value is printed after the last of print's arguments. Default is undef.
You cannot call output_record_separator() on a handle, only as a static method. See IO::Handle.
You set ```$\``` instead of adding "\n" at the end of the print. Also, it's just like ```$/``` , but it's what you get "back" from Perl.                                                                                                                                                                                                           ✔
