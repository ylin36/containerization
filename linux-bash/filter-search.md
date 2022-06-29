# 1. Locate (must faster than find)
quick way to search for the locations of files and directories.

uses prebuilt db to do this, so cache could be old

this will list all files found to match

option|description
|-|-|
-q	|To suppress error messages, such as those that might be returned in the event that the user does not have permission to access designated files or directories.
-n	|This option followed by an integer limits the results to a specific number.
-i	|To perform a case-insensitive search.
-V	|To show which version of locate is used, including whether it is locate or slocate.

```
locate file.txt dir

locate "*.txt"

locate -n "*.txt"
```

# 2. Find
very powerful command which is used to search for specific patterns of texts within files and directories.

this searches real time.

uses filters (tests)

```
find [path...] [expression]

// finds file.txt in the folder
find /path/folder -name "file.txt"

// find all files ending in .txt
find /path/folder -name "*.txt"

// delete after finding
find relpath/folder -name "file.txt" -delete
```

test|description
|-|-|
-type f	|Selects files.
-type d	|Selects directories.
-name	|True if the base of the file name (the path with the leading directories removed) matches shell pattern pattern.
-iname	|Search without regard for text case.
-prune	|To ignore a whole directory tree.
-path	|This is the exact same as name, except that it doesn’t only apply on the filename, but the whole path.
-not	|Return only results that do not match the test case.

# 3. Sort
sort a program or file given its input

```
sort [option] filename
```

option|desc
|-|-|
-o	|To write the output to a new file.
-r	|To sort in reverse order.
-n	|To sort a file numerically.
-nr	|To sort a file with numeric data in reverse order.
-k	|To sort a table on the basis of any column number.
-c	|To check if the file given is already sorted or not.
-u	|To sort and remove duplicates

```
sort file.txt

a
b
c
```

```
sort file1.txt > outputfile1.txt
```

# 4. Head
view beginning of the file
```
head [options] [file(s)]
```

option|desc|
|-|-|
-n	|It can be used followed by an integer representing the number of lines to be displayed.
-c	|This option can be used followed by the number of bytes desired.
-q (quiet)	|Never print headers identifying file names.
-v (verbose)	|Always print headers identifying file names.

```
head -n 10 file.txt
```

# 5. Tail
```
tail [options] [file(s)]
```

defaults to last 10

option|desc
|-|-|
-n	|It is followed by an integer indicating the number of lines that are to be printed.
-c	|To print the specific number of bytes. this option precedes the number of bytes.
-q (quiet)	|It causes tail to not print the file name before each set of lines and to eliminate the vertical space between each set of lines when there are multiple input sources.
-v (verbose)	|It causes tail to print the file name even if there is just a single input file.

# 6. Uniq
filter and view multiple repeatable lines
```
uniq [option] [input[output]]
```

option|desc
|-|-|
-c	|Prefix lines with a number showing how many times they occurred.
-d	|Only print duplicated lines.
-u	|Only print unique lines.
-z	|End lines with 0 byte (NULL), instead of a newline.
-w	|Compare no more than N characters in lines.
-i	|To perform case-insensitive comparisons.
-f	|To avoid comparing first N fields of a line before determining uniqueness. (Field is a set of characters delimeted by a white space.)
-s	|To avoid comparing first N characters before determining uniqueness.

```
uniq hellocopy.txt
```

can also use in pipe
```
sort hellocopy.txt | uniq
```

# 7. Regex
|operator|desc|
|-|-|
\?	|The preceding item is optional and matched at most once.
\*	|The preceding item will be matched zero or more times.
\+	|The preceding item will be matched one or more times.
\{n}	|The preceding item is matched exactly n times.
\{n,}	|The preceding item is matched n or more times.
\{n,m}	|The preceding item is matched at least n times, but not more than m times.
\$	|Matches the end of the line.
\^	|Matches the beginning of the line.
\( )	|Allows us to group several characters to behave as one.
\|	|Its the logical OR operation.
\.	|A single character.
\[agd]	|The character is one of those included within the square brackets.
\[^agd]	|The character is not one of those included within the square brackets.
\[a-d]	|The dash within the square brackets operates as a range. Here, it means all characters between a and d, including “a” and “d”.

# 8. grep, egrep, fgrep
## 8.1 grep (global regular expression print)
grep [option(s)] pattern [file(s)]

// the brackets above are optional
```
// can do either
1) cat filename | grep regex

2) grep regex filename
```
|option|desc|
|-|-|
Option	Description
-E |(extended regexp)	Causes grep to behave like egrep.
-F |(fixed strings)	Causes grep to behave like fgrep.
-G |(basic regexp)	Causes grep, egrep, or fgrep to behave like the standard grep utility.
-r	|To search recursively through an entire directory tree (i.e., a directory and all levels of subdirectories within it)
-I	|Process a binary file as if it did not contain matching data.
-c|	To report the number of times that the pattern has been matched for each file and to not display the actual lines.
-n	|To precede each line of output with the number of the line in the text file from which it was obtained.
-v	|It matches only those lines that do not contain the given pattern.
-w	|To select only those lines that contain an entire word or phrase that matches the specified pattern.
-x	|To select only those lines that match exactly the specified pattern.
-l	|To not return the lines containing matches but to only return only the names of the files that contain matches.
-L	|It is the opposite of the -l option (and analogous to the -v option) i.e. it will cause grep to return only the names of files that do not contain the specified pattern.

## 8.2 egrep (extended global regular expression print) [extended regex]
```
grep -E is same as egrep

takes same options too
```

## 8.3 fgrep (fixed strings grep) [no regex]
used to interpret pattern as a list of fixed strings (the whole string is interpreted literally), separated by new lines, Hence, *regular expressions can’t be used*.

```
fgrep -n "Teststring" file.txt
```