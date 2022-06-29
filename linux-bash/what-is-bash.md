- [1. Types of Shell](#1-types-of-shell)
  - [1.1 Bourne Shell](#11-bourne-shell)
  - [1.2 C shell](#12-c-shell)
- [2. Typical example of shell programmping](#2-typical-example-of-shell-programmping)
- [3. Linux](#3-linux)
- [4. Shell classification](#4-shell-classification)
  - [4.1 Commandline shell](#41-commandline-shell)
  - [4.2 Graphical shell](#42-graphical-shell)
- [5. Linux shell](#5-linux-shell)
- [6. Find metadata about shell](#6-find-metadata-about-shell)
- [7. Command use of bash](#7-command-use-of-bash)
- [8. Do not use bash for](#8-do-not-use-bash-for)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>


# 1. Types of Shell
## 1.1 Bourne Shell
default prompt is # character
## 1.2 C shell
default prompt is % character

# 2. Typical example of shell programmping
* Allows to create user accounts
* Writing application startup scripts, especially unattended applications
* Find out what processes are eating up your system resources
* Find out available and free memory
* Find out all logged in users and what they are doing
* Writing system boot scripts
* User administration as per your security policies
* Find out information about local or remote servers
* Creating application package installation tools
* Automation of customized processes

# 3. Linux 
Strickly speaking, linux is a kernel, not an OS.

A kernel provides access to computer hardware and a controlled access to system resources such as:

* Security and firewall
* GNU libraries and utilities
* Other management and installation scripts
* Logged in users
* Running and loading programs to memory
* Networking systems
* Files and data
* Process management
* Device management
* I/O management

# 4. Shell classification
## 4.1 Commandline shell
access by users using the terminal program. or cmd in windows.
eg. bash, ksh, csh
## 4.2 Graphical shell
gui for the user to interact with. 
eg. windows OS, Ubuntu OS


# 5. Linux shell
A shell is an environment provided for the user to interact with the machine.

It is NOT part of linux kernel, but linux kernel uses it to execute programs, create files, etc. Several shell tools are:

bash, csh, ksh, tcsh, zsh, fish


# 6. Find metadata about shell
```
ps $$

      PID    PPID    PGID     WINPID   TTY         UID    STIME COMMAND
     1290       1    1290      12412  cons0     197609 20:04:50 /usr/bin/bash
     1315    1290    1315       7156  cons0     197609 20:29:39 /usr/bin/ps
```

if default shell is not bash, change it with
```
chsh -s /bin/bash
```

get manual
```
man <command>

eg.
man cal
```
get info 
```
info <command>

eg. 
info data
```

# 7. Command use of bash
* File manipulation
* Program execution
* Creating your power tools/utilities
* Automating command input or entry
* Customizing administrative tasks
* Creating simple applications
* Creating customized power utilities
* Printing text

# 8. Do not use bash for
use a program for these
* Need direct access to system hardware or external peripherals
* Need data structures, such as linked lists, graphs or trees
* Need to generate or manipulate graphics or GUIs
* Need native support for multi-dimensional arrays
* Need port or socket I/O
* Extensive file operations (Bash is limited to serial file access, and that only in a clumsy and inefficient line-by-line fashion)
* Where cross-platform portability is required
* Resource-intensive tasks, especially where speed is an important factor (sorting, recursion, hashing, etc.)


