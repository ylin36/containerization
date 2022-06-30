# 1. Types of variables
## 1.1 Shell variables
Combo of local variable required for shell to operate
Accessible only within that shell (not local to parent or children shell)

## 1.2 Local variables
Set at command prompt. Only available in current shell. Cannot be accessed by child processes running in current shell. 

All user defined variables are local

### 1.2.1 assignment
```
// assigned
varName=value
echo $varName
echo "hello $varname"
```

Sometimes the ＄ sign is not enough, so we add curly brackets with ＄ sign to indicate Bash the beginning and ending of variable name, like this: ${name}'s

Uninitialized variables will have value of null
## 1.3 Env variables
Accessible anywhere in shell and child process called in a shell. 

Created by *export* command in bash

## 1.4 Other
System variables set default by system

example|desc
|-|-|
\$USER	|Returns the username
\$HOSTNAME	|Returns the machine name
\$$	|Returns the Script ID
\$0|	Returns the name of the Script
\$1-9	|Returns the first 9 arguments in the Script

# 2. Expansion operator

operators to modify value to make output in diff presentation style
|operator|desc|
|-|-|
\//	|Replace characters, special characters etc.
\%	|Trim the value from end based on any character or delimeter
\#	|Trim the value from start based on any character or delimeter