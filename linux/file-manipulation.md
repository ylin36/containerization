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

