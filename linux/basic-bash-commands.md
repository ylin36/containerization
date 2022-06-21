# 1. Run terminal
```
ctrl alt t
```
# 2. Hot keys
```
Esc + T: Swap the last two words before the cursor
Ctrl + H: Delete the letter starting at the cursor
Ctrl + W: Delete the word starting at the cursor
TAB: Auto-complete files, directory, command names and much more
Ctrl + R: To see the command history.
Ctrl + U: Clear the line
Ctrl + C: Cancel currently running commands.
Ctrl + L: Clear the screen
Ctrl + T: Swap the last two characters before the cursor
```

# 3. Special characters
|character|description|
|--|--|
|/	|Directory separator, used to separate a string of directory names. Example: /home/projects/file|
|\	|Escape character. If you want to reference a special character, you must “escape” it with a backslash first. |Example: \n means newline; \v means vertical tab; \r means return
|#	|Lines starting with # will not be executed. These lines are comments|
|.	|Current directory. When its the first character in a filename, it can also “hide” files|
|..	|Returns the parent directory|
|~	|Returns user’s home directory|
|~+	|Returns the current working directory. It corresponds to the $PWD internal variable|
|~-	|Returns the previous working directory. It corresponds to the $OLDPWD internal variable|
|*	|Represents 0 or more characters in a filename, or by itself, it matches all files in a directory. Example: file*2019 can return: file2019, file_comp2019, fileMay2019|
|[]	|Can be used to represent a range of values, e.g. [0-9], [A-Z], etc. Example: file[3-5].txt represents file3.txt, file4.txt, file5.txt|
|\|	|Known as “pipe". It redirects the output of the previous command into the input of the next command. Example: ls \| less|
|<	|It redirects a file as an input to a program. Example: more < file.txt|
|>	|In script name >filename it will redirect the output of “script name” to “file filename”. Overwrite filename if it already exists. Example: ls > file.txt|
|>>	|Redirect and append the output of the command to the end of the file. Example: echo "To the end of file" >> file.txt|
|&	|Execute a job in the background and immediately get your shell back. Example: sleep 10 &|
|&&	|“AND logical operator”. It returns (success) only if both the linked test conditions are true. It would run the second command only if the first one ran without errors. Example: let "num = (( 0 && 1 ))"; cd/comp/projs && less messages|
|;	|“Command separator”. Allows you to execute multiple commands in a single line. Example: cd/comp/projs ; less messages|
|?	|This character serves as a single character in a filename. Example: file?.txt can represent file1.txt, file2.txt, file3.txt|

# 4. Exiting terminal
```
exit
```

# 5. General command syntax
```
command_name [-option(s)] [argument(s)]
```

# 6. Manual
In order to display manual page from specific section:

```
man [section-num] keywords(s)
```
|section num|description|
|--|--|
|1	|Programs or shell commands|
|2	|System calls (functions provided by the kernel)|
|3	|Library calls (functions within program libraries)|
|4	|Special files (usually found in /dev)|
|5	|File formats and conventions eg /etc/passwd|
|6	|Games and demonstrations|
|7	|Miscellaneous (including macro packages and conventions), e.g., man, groff|
|8	|System administration commands (usually only for root)|
|9	|Device driver interfaces|

options
```
man [option(s)] keyword(s)
```
|option|description|
|--|--|
|-s	|man  <section num\> <command>	To specifically view a section of a man page.|
|-a	|man -a <command>	To display all manual pages where a command exists.|
|-w	|man -w [command/tool name]	To view the location for man pages.|
|-I	|man -I [command/tool name]	To enable case-sensitivity while searching for man pages|
|-H	|man -H[browser-command] [command/tool name]	To display manual pages in web browser.|
|-f|	man -f [command/tool name]	To lookup for manual pages and display short descriptions as well.|

# 7. System date a time
```
date [+FORMAT]
```

|format|desc|
|--|--|
 |%a	|The abbreviated weekday name (e.g., Mon).|
|%A	|The full weekday name (e.g., Monday).|
|%b	|The abbreviated month name (e.g., Feb).|
|%B	|The full month name (e.g., February).|
|%c	|The date and time (e.g., Fri Feb 6 21:45:39 2018).|
|%d	|Day of month (e.g., 04).|
|%D	|Date; same as %m/%d/%y.|
|%F	|Full date; same as %Y-%m-%d.|

## 7.1 get date
```
date
```
get date in specific format
date '+%Y-%m-%d %H:%M:%S'

## 7.2 set system date
```
date -s "07/01/2018 03:15:00"
```

# 8. Commonly used commands
## 8.1 Echo
```
echo [option(s)] [string(s)]
```
|option|desc|
|--|--|
|-n	|Do not output a trailing newline.|
|-e	|Enable interpretation of backslash escape sequences.|
|-E	|Disable interpretation of backslash escape sequences.|
|–help|	Display a help message and exit.|

## 8.2 Clear
clear the terminal
```
clear
```

## 8.3 Sleep
pause terminal for specified amount of time
```
sleep NUMBER[SUFFIX]

sleep 4h
sleep 5d

// sleep for 5.5s
sleep 5.5
```