# Linking Commands, Pipes and Input/Output

This section is probably one of the most important in the book because it describes a fundamental and powerful feature on Linux and Unix: the ability to use pipes and redirect input or output. in the Bash scripting world, because pipes and redirection allow you to enhance commands with the functionality of other commands or features. 

## Example 1
Bob, wants to look at his logs in real time (live), but he only wants to find the entries related to the wireless interface. The name of Bob's wireless device can be found using the iwconfig command. The iwconfig command is deprecated now. The following commands also will give you wireless interface information:
```
$ iw dev                # This will give list of wireless interfaces
$ iw dev wlp3s0 link    # This will give detailed information about particular wireless interface
```
Now that Bob knows his wireless card's identifying name (wlp3s0), Bob can search his system's logs. It is usually found within /var/log/messages. Using the tail command and the -F flag, which allows continuously outputting the logs to the console, Bob can now see all the logs for his system. Unfortunately, he would like to filter the logs using grep, such that only logs with the keyword wlp3s0 are visible.
```
$ tail -F /var/log/messages | grep wlp3s0
Nov 10 11:57:13 moon kernel: wlp3s0: authenticate with 18:d6:c7:fa:26:b1
Nov 10 11:57:13 moon kernel: wlp3s0: send auth to 18:d6:c7:fa:26:b1 (try 1/3)
Nov 10 11:57:13 moon kernel: wlp3s0: send auth to 18:d6:c7:fa:26:b1 (try 2/3)
...
```
## stdin, stdout, stderr
```
$ ls /filethatdoesntexist.txt 2> err.txt
$ ls ~/ > stdout.txt
$ ls ~/> everything.txt 2>&1 # Gets stderr and stdout
$ ls ~/>> everything.txt 2>&1 # Gets stderr and stdout
$ cat err.txt
ls: cannot access '/filethatdoesntexist.txt': No such file or directory
$ cat stdout.txt
.... # A whole bunch of files in your home directory
```
### Example
```
#!/bin/sh

# Let's run a command and send all of the output to /dev/null
echo "No output?"
ls ~/fakefile.txt > /dev/null 2>&1

# Retrieve output from a piped command 
echo "part 1"
HISTORY_TEXT=`cat ~/.bashrc | grep HIST`
echo "${HISTORY_TEXT}"

# Output the results to history.config
echo "part 2"
echo "${HISTORY_TEXT}" > "history.config"

# Re-direct history.config as input to the cat command
cat < history.config

# Append a string to history.config
echo "MY_VAR=1" >> history.config

echo "part 3 - using Tee"
# Neato.txt will contain the same information as the console
ls -la ~/fakefile.txt ~/ 2>&1 | tee neato.txt
```
The first is a way of prodcing an error nad instead of pushing errneous output to the cdonsole, it is instead redirected to a special device in Linux /dev/null

In the last command, script redirect both stdout and stderr with a pipe to tee command instead to a file. like ls -la ~/fakefile.txt ~/ > everything.txt 2>&1
