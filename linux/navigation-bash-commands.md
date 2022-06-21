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
Inode is a link that points to the actual content or data. 

A file in the file system actually points to this inode instead of pointing directly to data.

file -> inode -> actual data

## 3.2 Symbolic link vs hardlink


create symbolic link (Basically a shortcut)
```
ln -s <source_name> <link_name>
```

Contrary to Symbolic Link, there is Hard Link which is the exact copy of the orginal file. This means that the original file and its hard link point to the inode in the directory.  (?????)

create hard link
```
ln <source_name> <link_name>
```