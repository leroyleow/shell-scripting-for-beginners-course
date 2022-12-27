# Scripts For Power Users

## Creating Syslog entries & Generating an alarm
logger is a shell command and makes entries in the system log

How to do it...
1. We are going to use the logger command to enter file_name into syslog file. Run the following command:
```
$ logger -f file_name
```

2. Now we are going to write a script to create an alarm. Create create_alarm.sh
```
#!/bin/bash
declare -i H
declare -i M
declare -i cur_H
declare -i cur_M
declare -i min_left
declare -i hour_left
echo -e "What time do you Wake Up?"
read H
echo -e "and Minutes?"
read  M
cur_H=`date +%H`
cur_M=`date +%M`
echo "You Selected "
echo "$H:$M"
echo -e "\nIt is Currently $cur_H:$cur_M"
if [  $cur_H -lt $H ]; then
    hour_left=`expr $H - $cur_H`
    echo "$H - $cur_H means You Have: $hour_left hours still"
fi
if [ $cur_H -gt $H ]; then
    hour_left=`expr $cur_H - $H`
    echo -e  "\n$cur_H - $H means you have $hour_left hours left \n"
fi
if [ $cur_H == $H ]; then
    hour_left=0
    echo -e "Taking a nap?\n"
fi
if [ $cur_M -lt $M ]; then
    min_left=`expr $M - $cur_M`
    echo -e "$M -$cur_M you have: $min_left minutes still"
fi
if [ $cur_M -gt $M ]; then
    min_left=`expr $cur_M - $M`
    echo -e "$cur_M - $M you have $min_left minutes left \n"
fi
if [ $cur_M == $M ]; then
    min_left=0
    echo -e "and no minutes\n"
fi

echo -e "Sleeping for $hour_left hours and $min_left minutes \n"
sleep $hour_left\h
sleep $min_left\m
mplayer ~/.alarm/alarm.mp3
```

How it works...
1. The logger command made an entry about your file in the syslog file, which was in the /var/log directory of your system. You can check that file. Navigate to the /var/log directory and run nano syslog and you will find the entry in that file.
2. We created a script to create an alarm. We used the date command for the date and time. We also used the sleep command to block the alarm for that particular time.

## Checking for file integrity and tamering
How to do it...
1. We are going to write a script to check whether a file in a directory has been tampered with. Create an integrity_check.sh script and add the following code to it:
```
#!/bin/bash
E_DIR_NOMATCH=50
E_BAD_DBFILE=51
dbfile=Filerec.md5
# storing records.
set_up_database ()
{
    echo ""$directory"" > "$dbfile"
    # Write directory name to first line of file.
md5sum "$directory"/* >> "$dbfile"
    # Append md5 checksums and filenames.
}
check_database ()
{
    local n=0
    local filename
    local checksum
    if [ ! -r "$dbfile" ]
    then
        echo "Unable to read checksum database file!"
        exit $E_BAD_DBFILE
    fi

    while read rec[n]
    do
        directory_checked="${rec[0]}"
        if [ "$directory_checked" != "$directory" ]
        then
            echo "Directories do not match up!"
            # Tried to use file for a different directory.
            exit $E_DIR_NOMATCH
        fi
        if [ "$n" -gt 0 ]
        then
            filename[n]=$( echo ${rec[$n]} | awk '{ print $2 }' )
            # md5sum writes recs backwards,
            #+ checksum first, then filename.
            checksum[n]=$( md5sum "${filename[n]}" )
            if [ "${rec[n]}" = "${checksum[n]}" ]
            then
                echo "${filename[n]} unchanged."
            else
                echo "${filename[n]} : CHECKSUM ERROR!"
            fi
        fi
        let "n+=1"
        done <"$dbfile" # Read from checksum database file.
}
if [ -z "$1" ]
then
    directory="$PWD" # If not specified,
 else
    directory="$1"
fi
clear
if [ ! -r "$dbfile" ]
then
    echo "Setting up database file, \""$directory"/"$dbfile"\".";
    echo
    set_up_database
fi
check_database
echo
exit 0
```
How it works...
When we run this script, it will create a database file named filerec.md5, which will have data about all the files present in that directory. We'll use those files for reference.

## Encrypting/decrypting files from a script

How to do it...
1. We are going to encrypt and decrypt simple messages, using -base64 scheme. First encrypt the message.
```
$ echo "Welcome to Bash Cookbook" | openssl enc -base64
```

2. To decrypt message, run following command in the terminal
```
$ echo " V2VsY29tZSB0byBCYXNoIENvb2tib29rCg==" | openssl enc -base64 -d
```

3. Now going to encrypt and decrypt files. First, we will encrypt a file. 
```
$ openssl enc -aes-256-cbc -in /etc/services -out enc_services.dat
```

4. Decrypt a file
```
$ openssl enc -aes-256-cbc -d -in enc_services.dat > services.txt
```

How it works...
OpenSSL is a tool used for encrypting and decrypting messages, files, and directories. In the preceding example, we used the -base64 scheme. The -d option is used for decryption.

To encrypt a file, we used the -in option, followed by the file that we want to encrypt, and the -out option instructed OpenSSL to store the file. It then stores the encrypted file by the specified name.

## Creating a config file and using it in tandem with your scripts

How to do it...
Now, we are going to create a script and config file. The extension of the configuration file is .conf. Create a script called sample_script.sh and write this code in it:
```
#!/bin/bash
typeset -A config
config=(
    [username]="student"
    [password]=""
    [hostname]="ubuntu"
)
while read line
do
    if echo $line | grep -F = &>/dev/null
    then
        varname=$(echo "$line" | cut -d '=' -f 1)
        config[$varname]=$(echo "$line" | cut -d '=' -f 2-)
    fi
done < sampleconfig.conf
echo ${config[username]}
echo ${config[password]}
echo ${config[hostname]}
echo ${config[PROMPT_COMMAND]}
```
We will now create a configuration file. Create a file called sampleconfig.conf and write the following code in it:
```
password=training
echo rm -rf /
PROMPT_COMMAND='ls -l'
hostname=ubuntu; echo rm -rf /
```

How it works...
After running the script username, password, and hostname, it will display the command we mentioned in  PROMPT_COMMAND.