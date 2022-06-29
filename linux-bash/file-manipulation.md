- [1. Creating file](#1-creating-file)
- [2. Word count](#2-word-count)
- [3. Concatenation](#3-concatenation)
  - [3.1 concatenation](#31-concatenation)
  - [3.2 read new files](#32-read-new-files)
  - [3.3 new file creation](#33-new-file-creation)
- [4. Remove file](#4-remove-file)
- [5. Move file to directory OR rename file](#5-move-file-to-directory-or-rename-file)
- [6. Copy file](#6-copy-file)
- [7. Ziping file](#7-ziping-file)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>


# 1. Creating file
```
touch [option] [file_name(s)]
```
|option|description|
|-|-|
-am, -a, -m|A is for Access Time and M is for Modification Time. You can change both together by using this option
-r	|Means reference. It makes a reference to another file’s timestamps and use them for your file
-B	|Means Back. This options changes time by going a few seconds back, specified by the user
-F|	Means Forward. This options changes time by going a few seconds forward, specified by the user
-d or -t	|These two options are used if the user wants to add his/her own access time in a specific format

# 2. Word count
number of lines, number of words, number of characters
```
wc [option] [file]

wc file.txt
6  9 65 file.txt

```
|option|description|
|-|-|
-m	|Print the character counts.
-c	|Print the byte counts.
-l	|Print the newline counts.
-w	|Print the word counts.
-L	|Print the length of the longest line.

# 3. Concatenation 
operator|description
|-|-|
\>	|Redirection Operator. Redirects the contents of one file to another
\>>	|Append Operator. Appends the content of one file at the end of the other file. Used to prevent overwritting issues.
\|	|Piping Operator. Pipes the content of a file if its too large to display

|option|description|
|-|-|
-n	|Display the contents of file with line numbers
-E	|Concatenate ‘$’ at the end of each line of file
-T	|Replaces tab as ^I
-v	|To show non-printable characters on command line
-A	|Combination of v,E and T

3 main purposes:

## 3.1 concatenation
cat [option] [file_name(s)]

## 3.2 read new files
print contents of the file
```
cat [options] filename 
```
To redirect content to another file
```
cat filename > file_name2
```
To filter the content to be displayed with piping
```
cat [file_name] | less
```
## 3.3 new file creation
To create new file or overwrite if file already exists
```
cat > [new_file_name]
```
To preserve previous file if it already exists by appending any new text
```
cat >> [existing_file_name]
```

# 4. Remove file
```
rm [option] [filename(s)]
```
remove directory
```
rm -r directoryname
```
|option|description|
|--|--|
-f	|Means force. This option tells rm to force delete all the specified files without showing any message
-i	|Means interactive. This option prompts the user for confirmation before deleting any file
-r or -R	|Means recursive. It is used to recursively delete diectories by first emptying them and then removing them one by one

# 5. Move file to directory OR rename file
```
mv [options] [source] [destination]
```

to rename file to same directory
```
mv filename1.txt filename2.txt
```

move all files i just created to new location
```
touch file1.txt file2.txt file3.txt
mv * ../new 
```

# 6. Copy file
```
cp [option] [filename] [newname]
```

option|description
|-|-|
-r or -R|	Means Recursive. It is used to copy directories including all its content
-i	|This option is only used to warn the user about overwrite issues
-b	|Used to make backup copies
-f|	Means Force. This option is used to force open the destination files
-u|	Means Update. As the name suggests, it only updates the file if any changes are made in the file
-x	|This option is to indicate cp to stay on the same file system
-s	|By using this option you can only make references rather than deep copying everything

# 7. Ziping file
tar - short for tape archive. converts files to .tar archive.
```
tar [option(s)] [archivename] [filename(s)]

// have to put in the option...
tar -cf helloarchive hello.txt hellocopy.txt
```
option|description
|-|-|
-c	|Creates an archive
-x	|Extracts an archive
-f	|Create archive with the given name
-t	|Display the files in archive
-z	|Creates archive with gzip
-r	|Update/Add the archive file with new files
-v|	Verbose

```
gzip [options] [filename]

// example.. this will g zip but you lose the hello.txt
gzip hello.txt

// to keep the original form, do
gzip -k hello.txt
```
|option|description|
|-|-|
-k	|Compresses the file in its original form
-d	|Decompresses the gzip file
-r|	Compresses the file in a folderwise manner and places them in their respective gzip files
0-9	|Used to set compression levels
-v	|Verbose. Displays information about compression
-l | show details of the compressed file

```
gzip -l hello.txt.gz
```