# Make a Script Behave Like A Daemon

## Running a program continuously (forever) using looping constructs or recursion

 If we wanted to have scripts execute multiple scripts through a menu, or perform tasks in the background automatically forever without being executed each time by scheduling processes (like cron)? This recipe introduces a few ways for a script to run forever until it is killed or exits.

 * Recursive functions combined with a prompt (for example, the read command) can result in a script that loops based on user input

 * Looping constructs such as for, while, and until can be executed in such a way that a condition is never met and cannot exit

 Using the recursive method, the operation may look like this:

1. The script or program enters a recursive function
2. The recursive function can continue calling itself indefinitely, or, wait for a blocking input (for example, the read command)
3. Based on the input provided by the read command, you could call the same function again
4. Go back to step 1 until exit

Note: Using the sleep command is an excellent way to limit CPU usage when using loops in simple scripts. However, time adds up if you are running a long script!

Example Recursion
```
#!/bin/bash

function recursive_func() {

    echo -n "Press anything to continue loop "
    read input
    recursive_func
}

recursive_func
exit 0
```

Example Loop
```
#!/bin/bash
for( ; ; )do
echo"Shall run for ever"
sleep1
done
exit 0
```

Example While
```
#!/bin/bash
EXIT_PLEASE=0
while : # Notice no conditions?
do
   echo "Pres CTRL+C to stop..."
   sleep 1
   if [ $EXIT_PLEASE != 0 ]; then
      break 
    fi 
done
exit 0
```

## Keeping Scripts Running After Logoff

When a user logs in, a session for that user is created, but when they log off—unless the system owns it, processes and scripts typically get killed or closed.

This recipe is about keeping your scripts and activities running in the background after you log off.

Besides having a terminal open, we need to remember a few concepts:

* When a user logs off, any apps or processes owned by the current user will exit (the shell will send a signal)
* The shell is configurable to not send a shutdown signal to processes
* Applications and scripts use stdin and stdout for the usual operations
* Applications or scripts in the background can be referred to as jobs

One neat way is by using &, which is used this way: $ bash runforver.sh &. Unfortunately, using only this technique, we are back at square one—our binary still dies when we exit. Therefore, we need to use programs such as screen, disown, and sighup.

1. Open a terminal and create the loop_and_print.sh script:
```
#!/bin/bash
EXIT_PLEASE=0
INC=0

until [ ${EXIT_PLEASE} != 0 ] # EXIT_PLEASE is set to 0, until will never be satisfied
do
   echo "Boo $INC" > /dev/null
   INC=$((INC + 1))
   sleep 1
done
exit 0
```

2. Open a terminal and run the following commands:
```
$ bash loop_and_print.sh &
$ ps aux | grep loop_and_print.sh # Take note of the PID - write it down
```

3. Next, log off, then log in and run the following command in a new terminal. [Logging back on and running the ps command will produce zero results. This is because the script we put into the background using & has been sent a signal to shutdown or die. ]:
```
$ ps aux | grep loop_and_print.sh # Take note of the PID - write it down
```

4. Can you find the process running? Next, run the following command. [Again, we run the loop_and_print.sh script; command puts it into the background, and disown removes the the background process(es) from the known list of jobs. This disconnects the script and all output from any terminal]:
```
$ bash loop_and_print.sh & # note the PID againg
$ disown
```

5. Next, log off, then log in and run the following command in a new terminal:
```
$ ps aux | grep loop_and_print.sh # Take note of the PID - write it down
```

6. Next, run the following command. [The nohup command is similar to the disown command, except that it explicitly disconnects the script from the current shell. ]:
```
$ nohup bash loop_and_print.sh &
```
 
7. Next, log off, then log in and run the following command in a new terminal:
```
$ ps aux | grep loop_and_print.sh # Take note of the PID - write it down
```

## Sanitizing user input and for repeatable results

Example - Sanitize file path
```
#!/bin/bash
FILE_NAME=$1

# first, strip underscores
FILE_NAME_CLEAN=${FILE_NAME//_/}

FILE_NAME_CLEAN=$(sed 's/..//g' <<< ${FILE_NAME_CLEAN})

# next, replace spaces with underscores
FILE_NAME_CLEAN=${FILE_NAME_CLEAN// /_}

# now, clean out anything that's not alphanumeric or an underscore
FILE_NAME_CLEAN=${FILE_NAME_CLEAN//[^a-zA-Z0-9_.]/}

# here you should check to see if the file exists before running the command
ls "${FILE_NAME_CLEAN}"
```

Example - Sanitize Email
```
#!/bin/bash

EMAIL=$1
echo "${EMAIL}" | grep '^[a-zA-Z0-9._]*@[a-zA-Z0-9]*\.[a-zA-Z0-9]*
RES=$?
if [ $RES -ne 1 ]; then
    echo "${EMAIL} is valid"
else
    echo "${EMAIL} is NOT valid"
fi >/dev/null
RES=$?
if [ $RES -ne 1 ]; then
    echo "${EMAIL} is valid"
else
    echo "${EMAIL} is NOT valid"
fi
```

Example - Sanitize IP
```
#!/bin/bash

IP_ADDR=$1
IFS=.
if echo "$IP_ADDR" | { read octet1 octet2 octet3 octet4 extra;
  [[ "$octet1" == *[[:digit:]]* ]] && 
  test "$octet1" -ge 0 && test "$octet1" -le 255 &&
  [[ "$octet2" == *[[:digit:]]* ]] && 
  test "$octet2" -ge 0 && test "$octet2" -le 255 &&
  [[ "$octet3" == *[[:digit:]]* ]] && 
  test "$octet3" -ge 0 && test "$octet3" -le 255 &&
  [[ "$octet4" == *[[:digit:]]* ]] && 
  test "$octet4" -ge 0 && test "$octet4" -le 255 &&
  test -z "$extra" 2> /dev/null; }; then
  echo "${IP_ADDR} is valid"
else
    echo "${IP_ADDR} is NOT valid"
fi
```

## Making a simple multi-level user menu using select

Getting ready
Select is already a part of the Bash shell, but it has a few less than obvious points. Select relies on three variables:

* PS3: The prompt that's echoed to the user before the menu is created
* REPLY: The index of the item selected from the array
* opt: The value of the item selected from the array—not the index

Technically, opt is not mandatory, but it is the value of the element being iterated by Select in our example.

```
#!/bin/bash
TITLE="Select file menu"
PROMPT="Pick a task:"
OPTIONS=("list" "delete" "modify" "create")

function list_files() {
  PS3="Choose a file from the list or type \"back\" to go back to the main: "
  select OPT in *; do 
    if [[ $REPLY -le ${#OPT[@]} ]]; then
      if [[ "$REPLY" == "back" ]]; then 
        return
      fi
      echo "$OPT was selected"
    else
      list_files
    fi
  done
}

function main_menu() {
  echo "${TITLE}"
  PS3="${PROMPT} "
  select OPT in "${OPTIONS[@]}" "quit"; do 
    case "$REPLY" in
      1 ) 
        # List
        list_files
        main_menu # Recursive call to regenerate the menu
      ;;
      2 ) 
        echo "not used"
      ;;
      3 ) 
        echo "not used"
      ;;
      4 )
        echo "not used"
      ;;
      $(( ${#OPTIONS[@]}+1 )) ) echo "Exiting!"; break;;
      *) echo "Invalid option. Try another one.";continue;;
    esac
  done
}

main_menu # Enter recursive loop
```

How it works...
1. Creating the select_menu.sh script was trivial, but besides the use of select, some of the concepts should look familiar: functions, return, case statements, and recursion. 
2. The script enters the menu by calling the main_menu function and then proceeds to use select to generate a menu from the ${OPTIONS} array. The hard-coded variable named PS3 will output the prompt before the menu, and $REPLY contains the index of the item selected.
3. Pressing 1 and pressing Enter will cause select to walk through the items and then execute the list_files function. This function creates a submenu by using select for the second time to list all of the files in the directory. Selecting any directory will return a $OPT was selected message, but if back is entered, then the script will return from this function and call main_menu from within itself (recursion). At this point, you may select any items in the main menu.

## Generating and trapping signals for cleanup

In Linux, Ctrl + C equates to SIGINT (program interrupt), which typically exits a program. It can be stopped, and other functionality such as cleanup can be executed. Ctrl + Z or SIGTSTP (keyboard stop) typically tells a program to be suspended and pushed to the background (more about jobs in a later section), but it can also be blocked—just like SIGINT. 

Getting Ready
Besides using the keyboard within a program, we can also send signals to programs using the kill command. The most common signals you may use are SIGHUP (1), SIGINT (2), SIGKILL(9), SIGTERM(15), SIGSTOP(17,18,23), SIGSEGV(12), and SIGUSR1(10)/SIGUSR2(12).
```
$ kill -s SIGUSR1 <processID>
$ kill -9 <processID>
$ kill -9 `pidof myprogram.sh`
```
Note: The process id can be found using ps | grep X

How to do it...
mytrap.sh
```
#!/bin/bash

function setup() {
  trap "cleanup" SIGINT SIGTERM
  echo "PID of script is $$"
}

function cleanup() {
  echo "cleaning up"
  exit 1
}

setup

# Loop forever with a noop (:)
while :
do
  sleep 1
done
```

How it works
1. The mytrap.sh script leverages functions and the trap call. Inside of the setup function, we set the function to be called by the trap command. Therefore, when Ctrl + C is called, the cleanup function is executed.
2. Running the script will cause the script to run forever after printing out the PID of the script.
3. Pressing regular keys such as Enter will not have an effect on the program.
4. Pressing Ctrl+C will echo cleanup on the console and the script will exit using the exit command.

## Using temporary file and lock files in your program

It's usually temporary (it resides in /tmp). We can also use the <b>mktemp</b> command to create lock files

Getting Ready

We briefly mentioned that temporary files can reside inside the /tmp directory. Often, /tmp is home to short lived files such as lock files or information that can be volatile (destroyed on a power event without any detriment to the system). It is also usually RAM-based, which can offer performance benefits as well, especially if used as part of an inter-process communication system.

How to do it ...
1. create mylock.sh
```
#!/bin.bash

LOCKFILE="/tmp/mylock"

function setup() { 
  # $$ will provide the process ID
  TMPFILE="${LOCKFILE}.$$"
  echo "$$" > "${TMPFILE}"

  # Now we use hard links for atomic file operations 
  if ln "${TMPFILE}" "${LOCKFILE}" 2>&- ; then
      echo "Created tmp lock"
  else
      echo "Locked by" $(<$LOCKFILE)
      rm "${TMPFILE}"
      exit 1
  fi
  trap "rm ${TMPFILE} ${LOCKFILE}" SIGINT SIGTERM SIGKILL
}

setup

echo "Door was left unlocked"

exit 0
```
2. Execute the script with the $ bash mylock.sh script and review the console's output.
3. Next, we know that the script is looking for a particular lock file. What happens when we create a lock file and then re-run the script?
```
$ echo "1000" > /tmp/mylock
$ bash mylock.sh
$ rm /tmp/mylock
$ bash mylock.sh
```

How it works...
1. The mylock.sh script reuses a couple of concepts that we are already familiar with: traps and symbolic links. We know that if a trap is called or rather, it catches a particular signal, it can clean up a lock file (as is the case in this script). Symbolic links are used since they can survive atomic operations over network file systems. If a file is present at the LOCKFILE location, then a lock is present. If the LOCKFILE is absent, the doors are open.
2. When we run mylock.sh, we will get the following because no lock file exists yet—including any temporary ones:
3. Since the preceding script exited correctly, the SIGKILL signal was handled and the temporary lockfile was removed. In this case, we want to create our own lockfiles that bypass this mechanism. Create a lockfile with a faux PID of 1000; running the script will return Locked by 1000, and upon deleting the lockfile, the regular behavior will occur once more (doors are unlocked). 

## Leveraging timeout when waiting for command comletion

Use case examples are :-
* Where commands take variable lengths of time to complete (for example, pinging a network host)
* Where tasks or commands can be executed in such a way that the master script waits for the success or failure of several multiple operations

It important thing to note is that timeout/wait requires a process, or even a subshell so that it can be monitored (by the Process ID or PID). In this recipe, we will demonstrate the use of waiting for a subshell with the timeout command (which was added into the coreutils package 7.0) and how to do so using trap and kill (for alarms/timers).

Getting ready
important thing to note is that timeout/wait requires a process, or even a subshell so that it can be monitored (by the Process ID or PID). In this recipe, we will demonstrate the use of waiting for a subshell with the timeout command (which was added into the coreutils package 7.0) and how to do so using trap and kill (for alarms/timers).
* \$\$: Which returns the PID of the current script
* \$?: Which returns the PID of the last job that was sent to the background
* \$@ :Which returns the array of input variables (for example, $!, $2)

How to do it...
We begin this recipe knowing that there is a command called timeout available to the Bash shell. However, it falls short of being able to provide the functionality of timeouts in functions within a script itself. Using trap, kill, and signals, we can set timers or alarms (ALRM) to perform clean exits from runaway functions or commands. Let's begin:

1. mytimeout.sh
```
#!/bin/bash

SUBPID=0

function func_timer() {
  trap "clean_up" SIGALRM
  sleep $1& wait
  kill -s SIGALRM $$
}

function clean_up() {
  trap - ALRM
  kill -s SIGALRM $SUBPID
  kill $! 2>/dev/null
}

# Call function for timer & notice we record the job's PID
func_timer $1& SUBPID=$!

# Shift the parameters to ignore $0 and $1
shift 

# Setup the trap for signals SIGALRM and SIGINT
trap "clean_up" ALRM INT

# Execute the command passed and then wait for a signal
"$@" & wait $!

# kill the running subpid and wait before exit
kill -ALRM $SUBPID 
wait $SUBPID 
exit 0
```

2. Using a command with variable times (ping), we can test mytimeout.sh using the first parameter to mytimeout.sh as the timeout variable!
```
$ bash mytimeout.sh 1 ping -c 10 google.ca
$ bash mytimeout.sh 10 ping -c 10 google.ca
```

How it works...
1. In step 1, we create the mytimeout.sh script. It uses a few of our new primitives such as $! to monitor the PID of the function we sent to execute in the background as a job (or subshell, in this case). We arm the timer and then carry on with the execution of the script. Then, we use shift to literally shift the parameters passed to our script to ignore $1 (or the timeout variable). Finally, we watch for SIGALRM and perform a cleanup if necessary.
2. In step 2, mytimeout.sh is executed twice using the ping command, which is targeting google.ca. In the first instance, we use a timeout of 1 second, and in the second instance, we use a timeout of 10 seconds. Ping, in both cases, will perform 10 pings (for example, one ping there and back to whatever host is answering ICMP requests for the DNS entry for google.ca). The first instance will execute early, and the second allows 10 pings to execute cleanly and exit:

## Executing your script on startup

The high overview of Linux boot squence:-
1. The Linux kernel is loaded and mounts the root filesystem.
2. The rootfile system contains a shell at a particular path (the init level).
3. Then, the systemd works its way through a series of services to start (the run level).
If a service or script is added, it will likely be added at the run level. It can also be started, stopped, reloaded, and restarted from the command line at any time as well. When the system is booting, it merely uses the start functionality provided by the init.d or system.d script.

Script Requirement
* Scripts or services need to be enabled for specific system startup levels.
* Scripts to be started and/or stopped can be configured to be started in a specific order.
* Directives for starting (at a minimum), stopping, restarting and/or reloading are executed based on a parameter when executing one of these actions (or blocks). Any number of commands can also be executed when calling start, for example.

Getting reading
Before diving into the "how to do it" section, we are going to create a template init script for posterity and awareness should you run into them. It will be called myscript and it will run myscript.sh. At a minimum, a system.d compatible script looks like the following:
```
#!/bin/sh
### BEGIN INIT INFO
# Provides: myscript
# Required-Start: $local_fs
# Required-Stop: $local_fs
# Default-Start: 2 3 4 5
# Default-Stop: 0 1 6
# Short-Description: Start script or daemons example
### END INIT INFO

PATH=/sbin:/bin:/usr/sbin:/usr/bin
DAEMON_NAME=myscript.sh
DESC=myscript

DAEMON=/usr/sbin/$DAEMON_NAME

case $1 in
  start)
  log_daemon_msg "Starting $DESC"
  $DAEMON_NAME
  log_end_msg 0
  ;;
  stop)
  log_daemon_msg "Stopping $DESC"
  killall $DAEMON_NAME
  log_end_msg 0
  ;;
  restart|force-reload)
  $0 stop
  sleep 1
  $0 start
  ;;
  status)
  status_of_proc "$DAEMON" "$DESC" && exit 0 || exit $?
  ;;
  *)
  N=/etc/init.d/$DESC
  echo "Usage: $N {start|stop|restart|force-reload|status}" >&2
  exit 1
  ;;
esac

exit 0

# vim:noet
```

How to do it...
1. myscript.sh
```
#!/bin/bash

for (( ; ; ))
do
   sleep 1
done
```

2. Next, let's add the correct permissions to the script so that we can create a systemd service using it. Notice the use of the sudo command—enter your password where appropriate:
```
$ sudo cp myscript.sh /usr/bin
$ sudo chmod a+x /usr/bin/myscript.sh
```

3. Now that we have something to execute on start, we need to create a service configuration file to describe our service; we used vi in this example and sudo (note down its location):
```
$ sudo vi /etc/systemd/system/myscript.service 
[Unit]
Description=myscript

[Service]
ExecStart=/usr/bin/myscript.sh
ExecStop=killall myscript.sh

[Install]
WantedBy=multi-user.target
```

4. To enable the myscript service, run the following command:
```
$ sudo systemctl enable myscript # disable instead of enable will disable the service
```

5. To start and verify the presence of the process, run the following command:
```
$ sudo systemctl start myscript
$ sudo systemctl status myscript
```
6. You may reboot the system to see our service in action on startup.

How it works...

1. In step 1, we create a trivial looping program to be ran at system startup called myscript.sh.
2. In step 2, we copy the script to the /usr/bin directory and add permissions using the chmod command for everyone to be able to execute the script (chmod a+x myscript.sh). Notice the use of sudo permissions to create a file in this directory and to apply permissions.
3. In the third step, we create the service configuration file, which describes a service unit for systemd. It goes by the name of myscript and within the [Service] directive, the two most important parameters are present: ExecStart and ExecStop. Notice that the start and stop sections look similar to the SysV/init.d approach.
 
4. Next, we use the systemctl command to enable myscript. Conversely, it can be used in the following way to disable myscript: $systemctl disable myscript.
5. Then, we use systemctl to start myscript and verify the status of our script. You should get a similar output to the following (notice that we double checked the presence using ps):
```
$ sudo systemctl status myscript
myscript.service - myscript
   Loaded: loaded (/etc/systemd/system/myscript.service; enabled; vendor preset:
   Active: active (running) since Tue 2017-12-26 14:28:51 EST; 6min ago
 Main PID: 17966 (myscript.sh)
   CGroup: /system.slice/myscript.service
           ├─17966 /bin/bash /usr/bin/myscript.sh
           └─18600 sleep 1

Dec 26 14:28:51 moon systemd[1]: Started myscript.
$ ps aux | grep myscript
root 17966 0.0 0.0 20992 3324 ? Ss 14:28 0:00 /bin/bash /usr/bin/myscript.sh
rbrash 18608 0.0 0.0 14228 1016 pts/20 S+ 14:35 0:00 grep --color=auto myscript
```
6. On reboot, if enable was set, our script will be running as expected.