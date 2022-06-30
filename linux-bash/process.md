# 1. PID
Process ID is unique id assigned to the process at the time of creation. 

# 2. PPID
PID of parent process that spawned it is the PPID. 

# 3. PS 
Process status
```
ps [options]
```
|option|desc|
|-|-|
-A	|Select all processes. Identical to -e.
-d	|Select all processes except session leaders.
r	|Restrict the selection to only running processes.
p |pidlist OR -p pidlist OR --pid pidlist	Select by process ID.
--ppid |pidlist	Select by parent process ID.
-s |sesslist OR --sid sesslist	Select by session ID.

```
// every process running on the system
ps -e 

      PID    PPID    PGID     WINPID   TTY         UID    STIME COMMAND
     1911       1    1911      44836  cons0     197609 23:09:42 /usr/bin/bash
     1936    1911    1936      38556  cons0     197609 23:14:31 /usr/bin/ps
```

# 4. Jobs
A group of processes running in series or parallel

```
jobs [-lnprs] [jobspec]
jobs -x command [arguments]
```

```
// display all running jobs
jobs

// dispaly process id in addition to job number 
jobs -l
```

# 5. Sudo (root priviledge)
sudo (superuser do) runs commands with root access in your own user account
```
sudo -V | -h | -l | -L | -v | -k | -K | -s | [ -H ] [-P ] [-S ] [ -b ] | 
     [ -p prompt ] [ -c class|- ] [ -a auth_type ] [-r role ] [-t type ] 
     [ -u username|#uid ] command
```

|option|desc|
|-|-|
-H	| (HOME) option sets the HOME environment variable to the home directory of the target user (root by default) as specified in passwd. By default, sudo does not modify HOME.
-P	| (preserve group vector) option causes sudo to preserve the current user’s group vector unaltered. By default, sudo will initialize the group vector to the list of groups of the target user. The real and effective group IDs, however, are still set to match the target user.
-S	| (stdin) option causes sudo to read the password from standard input instead of the terminal.
-b	| (background) option tells sudo to run the given command in the background.
-h	| (help) option causes sudo to print a usage message and exit.
-l	| (list) option will print out the commands allowed (and forbidden) the user on the current host.
-v	|(validate) option will update the user’s timestamp, prompting for the user’s password if necessary.
-k	|(kill) option to sudo invalidates the user’s timestamp by setting the time on it to the epoch. The next time sudo is run a password will be required. This option does not require a password and was added to allow a user to revoke sudo permissions from a .logout file.
-s	| (shell) option runs the shell specified by the SHELL environment variable if it is set or the shell as specified in the file passwd.

```
// shutdown in 1min (Default)
sudo shutdown

// shutdown in proper way now then reboot
sudo shutdown -r now
```

# 6. Kill a process
kill [signal or option] PID(s)
```
kill 123
```
```
// 9 => sigkill (stronger signal incase prev kill fails)
kill 9 123
```