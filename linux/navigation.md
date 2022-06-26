# 1. Default on opening terminal
When opening a terminal, by default it opens to your "home" folder which is typically "current directory"

# 2. Path types
## 2.1 Absolute path
Start from the root

eg.

/home/Documents/file.txt

## 2.2 Relative path
Start from present working directory (pwd).

In unix,

. => current dir

.. => parent dir


# 3. Symbolic links
## 3.1 inode
files in linux contains no meta data...

Inode (index node) is a link that points to the actual content or data. It has its meta data. to see the inode and file pairing, type
```
ls -i

281474976847791 bin/
```
show what inode shows
```
ls -l

drwxr-xr-x 1 Yang 197121       0 Nov  7  2020 bin/
```
to see details
```
stat filename
```

A file in the file system actually points to this inode instead of pointing directly to data.

file -> inode -> actual data

## 3.2 Symbolic link vs hardlink


create symbolic link (Basically a shortcut). This file has different inode numbers than its original file link. they point to the original file as a short cut. 

```
ln -s <source_name> <link_name>
```
if the original file is deleted, the soft link files are useless.
```
                    soft link
file1--->inode 200 -----------> inode 100 <--- original file
file2--->inode 300 ----------/ 
```

Contrary to Symbolic Link, there is Hard Link (different name for the same file) which is the exact copy of the orginal file. This has the same inode number. This means that the original file and its hard link point to the inode in the directory. 

create hard link
```
ln <source_name> <link_name>
```
if we delete the original file, the inode 100 will still exist to point to the data
```
file1 ---> inode 100 <--- original file
file2 --/
```
# 4 Check current directory
```
pwd
```

## 4.1 Print all locations containing an executable
for example, if the executable was pwd
```
type -a pwd
``` 
results in
```
pwd is a shell builtin
pwd is /usr/bin/pwd
pwd is /bin/pwd
pwd is /usr/bin/pwd
```

## 4.2 cd options
|options|descriptions|
|--|--|
/	|To change the current directory to Root directory
-L	|This mode lets cd move directly to the directory the link is pointing to
-P	|This mode is opposite of -L. It uses physical directories and does not follow symbolic links
-e	|This option is only used to show an error in case the cd command fails to determine the directory

## 4.3 Display files in a directory
```
ls [option] file_name
```
|option|description|
|--|--|
-a	|List all the files, even those starting with .
-C	|List files column-wise
-i	|To show the index number of each file
-s	|To show the size of each file
-1	| To show one file per line
-r  | Show in reverse
-l  | show inode details like permissions, owner, inode number, hard links, 

# 5. Creating directory
```
mkdir [option] dirname
```
|option|description|
|-|-|
-p |	Means Parent or Path. It is used when we want to create a directory within a directory that doesn’t already exist
-m|	Means Mode. It is used to specify and control permission modes

```
mkdir my_dir_1 my_dir_2 my_dir_3
```

# 6. Removing directory
## 6.1 Remove empty directories
```
rmdir [option] [dir_name]
```
option|description
|--|--|
-p	|This option tells rmdir to remove nested directories if they become empty after deleting previous directory
-v or verbose	|Verbose mode outputs after every directory operation that you perform

## 6.2 removing directory with files in it
```
rm -rf [name]
```