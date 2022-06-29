- [Types of permissions](#types-of-permissions)
- [Modes](#modes)
- [Chmod (change mode)](#chmod-change-mode)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>


# Types of permissions
* read
* write
* execute

# Modes
List of numerical values for different modes for owner, group user, everyone
|mode|desc|
|-|-|
0	|000 -> --- -> None
1	|001 -> --x -> Execute Only
2	|010 -> -w- -> Write only
3	|011 -> -wx -> Write and Execute
4	|100 -> r-- -> Read only
5	|101 -> r-x -> Read and Execute
6	|110 -> rw- -> Read and Write
7	|111 -> rwx -> Read, Write & Execute

# Chmod (change mode)
change the read/write permissions of files and directories. 

A mode can either be represented symbolically with unique alphabets or by the permutations of three numbers (octal numbers)

can also add references in the command to indicate the users on which the specified permissions are being applied. Given below is the syntax to chmod command.

```
chmod [reference] [option] [mode] [filename]
```

operator|desc|
|-|-|
\+	|Adds the specified permissions on the files
\-	|Removes the specified perfmissions on the files
\=	|Add ONLY specified permissions on the file and remove the remaining permissions/modes

reference|desc|
|-|-|
u	|User who owns the file(s)
g	|Users in the file group
o	|Users which are neither owner of the file nor in the file group
a	|All users. Same as ugo

|option|desc|
|-|-|
-f	|suppress error messages
-v	|outputs the file being processed
-c	|to output warnings before making any changes
-R	|Means Recursive. This option is used to change directories recursively

```
chmod u=rw,g=wx,o=r file.txt

// wrx to everyone
chmod 777 file.txt

// remove execute from everyone
chmod a-x file.txt

// add readwrite to users
chmod o+rx file.txt
```