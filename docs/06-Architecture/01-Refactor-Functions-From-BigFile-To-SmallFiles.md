# Refactor-Functions-From-BigFile-To-SmallFiles

This sections talks about the process and pattern to refactor big file with many functions to multiple small files.
This generally implemented with <b>source</b> command. <b>source</b> is a shell built-in command which is used to read and execute the content of a file(generally set of commands), passed as an argument in the current shell script. If any arguments are supplied, they become the positional parameters when filename is executed.  The entries in $PATH are used to find the directory containing FILENAME, however if the file is not present in $PATH it will search the file in the current directory. The source command has no option and the argument is the file only. 

### Syntax
```
source FILENAME [arguments]
```

## Simple-Case
### Problem
To separate the function from main file and to pass parameter from the main file to child file

### Solution
second.sh
```
func1 {
   fun="$1"
   book="$2"
   printf "func=%s,book=%s\n" "$fun" "$book"
}

func2 {
   fun2="$1"
   book2="$2"
   printf "func2=%s,book2=%s\n" "$fun2" "$book2"
}
```

And then call these functions from script first.sh like this:
```
source ./second.sh
func1 love horror
func2 ball mystery
```

#### Output
```
func=love,book=horror
func2=ball,book2=mystery
```

### Important Note
this works only if the first.sh is executed from within the same directory where the first.sh is located. Ie. if the current working path of shell is /foo, the attempt to run command 
```
cd /foo
./bar/second.sh
```
prints error:
```
/foo/bar/second.sh: line 4: func1: command not found
```
That's because the source ./first.sh is relative to current working path, not the path of the script. Hence one solution might be to utilize subshell and run
```
(cd /foo/bar; ./second.sh)
```

## More Generic Solution 
Given /foo/bar/first.sh
```
function func1 {  
   echo "Hello $1"
}
```
and /foo/bar/second.sh
```
#!/bin/bash
source $(dirname "$0")/first.sh
func1 World
```
then
```
cd /foo
./bar/second.sh
```
print 
```
Hello World
```
### How it Works
*
* $0 returns relative or absolute path to the executed script
* dirname returns relative path to directory, where the $0 script exists
* $( dirname "$0" ) the dirname "$0" command returns relative path to directory of executed script, which is then used as argument for source command
* in "second.sh", /first.sh just appends the name of imported shell script
* source loads content of specified file into current shell

## Global vs Local Variables
### Local Bash Variables
Local variables are visible only within the block of code. local is a keyword which is used to declare the local variables. In a function, a local variable has meaning only within that function block.

```
$ cat localvar.sh
#!/bin/bash
pprint()
{
  local lvar="Local content"
  echo -e "Local variable value with in the function"
  echo $lvar
  gvar="Global content changed"
  echo -e "Global variable value with in the function"
  echo $gvar
}

gvar="Global content"
echo -e "Global variable value before calling function"
echo $gvar
echo -e "Local variable value before calling function"
echo $lvar
pprint
echo -e "Global variable value after calling function"
echo $gvar
echo -e "Local variable value after calling function"
echo $lvar
```
In the above output, local variables will have only empty value before and after calling the function. Its scope is only with in the function. It got vanished out of the function, whereas the global variable has the updated value even after the function execution.