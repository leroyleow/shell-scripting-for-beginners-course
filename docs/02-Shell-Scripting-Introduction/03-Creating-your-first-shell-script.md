# Creating your first shell script

- Take me to [Video Tutorial](https://kodekloud.com/topic/creating-your-first-shell-script/)

In this section, we will take a look at creating your first shell script.

To Create a shell script name create-and-launch-script.sh

```
$ vi create-and-launch-script.sh
```

![create-script](../../images/create-script.PNG)

## Run script as command

- There are different ways to execute a shell script
  - Execute a shell script with **`bash`** command
    ```
    $ bash create-and-launch-script.sh
    ```
  - Execute a shell script as an **`executable`**
    ```
    $ create-and-launch-script.sh
    ```
    **`Note`**: It is a best practice to not name your script with the **`.sh`** extension when you would like to create an executable of a script.
    ```
    $ create-and-launch-script
    ```

## Configure a script to run as command

- Whenever a command is run at a linux system, the O.S looks at the path configured in the **`$PATH`** environment variable to locate the executable or script for the command.
- If it cannot find the command in the **`$PATH`** then a **`command not found`** error will be thrown.

- To add our script as a command, append the path to the directory containing the script to the end of the $PATH variable.
  ```
  $ export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/home/michael
  ```
- A better way to do this is to append the path to the directory to $PATH variable
  ```
  $ export PATH=$PATH:/home/michael
  ```
- Run the command
  ```
  $ create-and-launch-script
  ```
- To see the location of the command
  ```
  $ which create-and-launch-script
  ```

![run-sh](../../images/run-sh.PNG)

## Executing a script

- For a shell script to work, we must set the correct permissions to the file, if the permissions are not set a "Permission Denied" error will be thrown when you run the script for the first time.

- Inspect the file permissions
  ```
  $ ls -l /home/michael/create-and-launch-script
  ```
- Add executable permissions to a file and then inspect the file to check if it got write permissions.
  ```
  $ chmod +x /home/michael/create-and-launch-script
  $ ls -l /home/michael/create-and-launch-script
  ```
- Now, run the script
  ```
  $ /home/michael/create-and-launch-script
  ```

![exec-sh](../../images/exec-sh.PNG)

## Best Practices

![best](../../images/best.PNG)

## Control Operators & Redirection Operators

Examples of control operators

```
&   &&    ( )   ;   ;;    <newline>   |   ||
```

<b>!</b> is not a control operator but a Reserved Word

### A.1 List terminators

<b>;</b> : Will run one command after another has finished, irrespective of the outcome of the first.

```
command1 ; command2
```

First command1 is run, in the foreground, and once it has finished, command2 will be run.

A newline that isn't in a string literal or after certain keywords is not equivalent to the semicolon operator. A list of ; delimited simple commands is still a list - as in the shell's parser must still continue to read in the simple commands that follow a ; delimited simple command before executing, whereas a newline can delimit an entire command list - or list of lists. The difference is subtle, but complicated: given the shell has no previous imperative for reading in data following a newline, the newline marks a point where the shell can begin to evaluate the simple commands it has already read in, whereas a ; semi-colon does not.

<b>&</b>: This will run a command in the background, allowing you to continue working in the same shell.

```
  command1 & command2
```

Here, command1 is launched in the background and command2 starts running in the foreground immediately, without waiting for command1 to exit.

A newline after command1 is optional.

### A.2 Logical operators

<b>&&</b> : Used to build AND lists, it allows you to run one command only if another exited successfully.

```
  command1 && command2
```

Here, command2 will run after command1 has finished and only if command1 was successful (if its exit code was 0). Both commands are run in the foreground.

This command can also be written

```
   if command1
   then command2
   else false
   fi
```

or simply if command1; then command2; fi if the return status is ignored.

<b>||</b> : Used to build OR lists, it allows you to run one command only if another exited unsuccessfully.

```
  command1 || command2
```

Here, command2 will only run if command1 failed (if it returned an exit status other than 0). Both commands are run in the foreground.

This command can also be written

```
   if command1
   then true
   else command2
   fi
```

or in a shorter way if ! command1; then command2; fi.

Note that && and || are left-associative; see Precedence of the shell logical operators &&, || for more information.

<b>!</b>: This is a reserved word which acts as the “not” operator (but must have a delimiter), used to negate the return status of a command — return 0 if the command returns a nonzero status, return 1 if it returns the status 0. Also a logical NOT for the test utility.

```
 ! command1

 [ ! a = a ]
```

And a true NOT operator inside Arithmetic Expressions:

```
   $ echo $((!0)) $((!23))
   1 0
```

### A.3 Pipe operator

</b>|</b> : The pipe operator, it passes the output of one command as input to another. A command built from the pipe operator is called a pipeline.

```
  command1 | command2
```

Any output printed by command1 is passed as input to command2.

<b>|&</b> : This is a shorthand for 2>&1 | in bash and zsh. It passes both standard output and standard error of one command as input to another.

```
 command1 |& command2
```

### A.4 Other list punctuation
<b>;;</b> is used solely to mark the end of a case statement. Ksh, bash and zsh also support ;& to fall through to the next case and ;;& (not in ATT ksh) to go on and test subsequent cases.

( and ) are used to group commands and launch them in a subshell. { and } also group commands, but do not launch them in a subshell. See this answer for a discussion of the various types of parentheses, brackets and braces in shell syntax. 

#### () vs {}

If you want the side-effects of the command list to affect your current shell, use {...}
If you want to discard any side-effects, use (...)

For example, I might use a subshell if I:

want to alter $IFS for a few commands, but I don't want to alter $IFS globally for the current shell
cd somewhere, but I don't want to change the $PWD for the current shell
It's worthwhile to note that parentheses can be used in a function definition:

normal usage: braces: function body executes in current shell; side-effects remain after function completes
```
$ count_tmp() { cd /tmp; files=(*); echo "${#files[@]}"; }
$ pwd; count_tmp; pwd
/home/jackman
11
/tmp
$ echo "${#files[@]}"
11 
```
unusual usage: parentheses: function body executes in a subshell; side-effects disappear when subshell exits
```
$ cd ; unset files
$ count_tmp() (cd /tmp; files=(*); echo "${#files[@]}")
$ pwd; count_tmp; pwd
/home/jackman
11
/home/jackman
$ echo "${#files[@]}"
0
```

## B. Redirection Operators
POSIX definition of Redirection Operator

In the shell command language, a token that performs a redirection function. It is one of the following symbols:

```
<     >     >|     <<     >>     <&     >&     <<-     <>
```

These allow you to control the input and output of your commands. They can appear anywhere within a simple command or may follow a command. Redirections are processed in the order they appear, from left to right.

<b><</b> : Gives input to a command.

```
 command < file.txt
```

The above will execute command on the contents of file.txt.

<b><></b> : same as above, but the file is open in read+write mode instead of read-only:

```
 command <> file.txt
```

If the file doesn't exist, it will be created.

That operator is rarely used because commands generally only read from their stdin, though it can come handy in a number of specific situations.

<b>></b> : Directs the output of a command into a file.

```
 command > out.txt
```

The above will save the output of command as out.txt. If the file exists, its contents will be overwritten and if it does not exist it will be created.

This operator is also often used to choose whether something should be printed to standard error or standard output:

```
   command >out.txt 2>error.txt
```

In the example above, > will redirect standard output and 2> redirects standard error. Output can also be redirected using 1> but, since this is the default, the 1 is usually omitted and it's written simply as >.

So, to run command on file.txt and save its output in out.txt and any error messages in error.txt you would run:

```
   command < file.txt > out.txt 2> error.txt
```

<b>>|</b> : Does the same as >, but will overwrite the target, even if the shell has been configured to refuse overwriting (with set -C or set -o noclobber).

```
 command >| out.txt
```

If out.txt exists, the output of command will replace its content. If it does not exist it will be created.

<b>>></b> : Does the same as >, except that if the target file exists, the new data are appended.

```
 command >> out.txt
```

If out.txt exists, the output of command will be appended to it, after whatever is already in it. If it does not exist it will be created.

<b>>&</b> : (per POSIX spec) when surrounded by digits (1>&2) or - on the right side (1>&-) either redirects only one file descriptor or closes it (>&-).
> A >& followed by a file descriptor number is a portable way to redirect a file descriptor, and >&- is a portable way to close a file descriptor.

If the right side of this redirection is a file please read the next entry.

>&, &>, >>& and &>> : (read above also) Redirect both standard error and standard output, replacing or appending, respectively.

```
 command &> out.txt
```

Both standard error and standard output of command will be saved in out.txt, overwriting its contents or creating it if it doesn't exist.

```
   command &>> out.txt
```

As above, except that if out.txt exists, the output and error of command will be appended to it.

The &> variant originates in bash, while the >& variant comes from csh (decades earlier). They both conflict with other POSIX shell operators and should not be used in portable sh scripts.

<b><<</b> : A here document. It is often used to print multi-line strings.

```
  command << WORD
      Text
  WORD
```

Here, command will take everything until it finds the next occurrence of WORD, Text in the example above, as input . While WORD is often EoF or variations thereof, it can be any alphanumeric (and not only) string you like. When any part of WORD is quoted or escaped, the text in the here document is treated literally and no expansions are performed (on variables for example). If it is unquoted, variables will be expanded. For more details, see the bash manual.

If you want to pipe the output of command << WORD ... WORD directly into another command or commands, you have to put the pipe on the same line as << WORD, you can't put it after the terminating WORD or on the line following. For example:

```
  command << WORD | command2 | command3...
      Text
  WORD
```

<b><<<</b> : Here strings, similar to here documents, but intended for a single line. These exist only in the Unix port or rc (where it originated), zsh, some implementations of ksh, yash and bash.

```
 command <<< WORD
```

Whatever is given as WORD is expanded and its value is passed as input to command. This is often used to pass the content of variables as input to a command. For example:

```
    $ foo="bar"
    $ sed 's/a/A/' <<< "$foo"
    bAr
    # as a short-cut for the standard:
    $ printf '%s\n' "$foo" | sed 's/a/A/'
    bAr
    # or
    sed 's/a/A/' << EOF
    $foo
    EOF
```

A few other operators (>&-, x>&y x<&y) can be used to close or duplicate file descriptors. For details on them, please see the relevant section of your shell's manual (here for instance for bash).

That only covers the most common operators of Bourne-like shells. Some shells have a few additional redirection operators of their own.

Ksh, bash and zsh also have constructs <(…), >(…) and =(…) (that latter one in zsh only). These are not redirections, but process substitution.
