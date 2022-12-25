# Search Pattern In Files and Directories

### Examples With Grep

- <b>Note</b> Notice the use of ~/_. This refers to home directory and _ allows us to specify anything from that point on.
- xargs reads standard input and then runs the command you give it on each line of the input
- Grep and many other utilities can be case-sensitive. To use grep in a way that's case insensitive, use the -i flag

#### Getting Ready 

This example uses the commands such as grep, ls, mkdir, touch, traceroute, strings, wget, xargs and find
```
$ ~/
$ wget --recursive --no-parent https://www.packtpub.com www.packtpub.com # Takes awhile
$ traceroute packtpub.com > traceroute.txt
$ mkdir -p www.packtpub.com/filedir www.packtpub.com/emptydir
$ touch www.packtpub.com/filedir/empty.txt
$ touch www.packtpub.com/findme.xml; echo "<xml>" www.packtpub.com/findme.xml
```

Script implementation as follows:-
```
#!/bin/bash

# Let's find all the files with the string "Packt"
DIRECTORY="www.packtpub.com/"
SEARCH_TERM="Packt"

# Can we use grep?
grep "${SEARCH_TERM}" ~/* > result1.txt 2&> /dev/null

# Recursive check
grep -r "${SEARCH_TERM}" "${DIRECTORY}" > result2.txt

# What if we want to check for multiple terms?
# e.g. $ grep -e "Packt" -e "Publishing" -r ~/www.packtpub.com/
grep -r -e "${SEARCH_TERM}" -e "Publishing" "${DIRECTORY}" > result3.txt

# What about find?
find "${DIRECTORY}" -type f -print | xargs grep "${SEARCH_TERM}" > result4.txt

# What about find and looking for the string inside of a specific type of content?
find "${DIRECTORY}" -type f -name "*.xml" ! -name "*.css" -print | xargs grep "${SEARCH_TERM}" > result5.txt

# Can this also be achieved with wildcards and subshell?
grep "${SEARCH_TERM}" $(ls -R "${DIRECTORY}"*.{html,txt}) > result6.txt
RES=$?

if [ ${RES} -eq 0 ]; then
  echo "We found results!"
else
  echo "It broke - it shouldn't happen (Packt is everywhere)!"
fi

# Or for bonus points - a personal favorite
history | grep "ls" # This is really handy to find commands you ran yesterday!

# Aaaannnd the lesson is:
echo "We can do a lot with grep!"
exit 0
```

## Using Widcards, Regex & Globbing/Pattern Matching

* A wildcard can be: *, {*.ooh,*.ahh}, /home/*/path/*.txt, [0-10], [!a], ?, [a,p] m
* A regex can be: $, ^, *, [], [!], | (be careful to escape this)

### Example1
```
$ ls -l | grep '[[:lower:]][[:digit:]]' # Notice no result
$ touch z0.test
$ touch a1.test
$ touch A2.test
$ ls -l | grep '[[:lower:]][[:digit:]]'
-rw-rw-r-- 1 rbrash rbrash 0 Nov 15 11:31 z0.test
-rw-rw-r-- 1 rbrash rbrash 0 Nov 15 11:31 a1.test
```

### Example2
```
#!/bin/bash
STR1='123 is a number, ABC is alphabetic & aBC123 is alphanumeric.'

echo "-------------------------------------------------"
# Want to find all of the files beginning with an uppercase character and end with .pdf?
ls * | grep [[:upper:]]*.pdf

echo "-------------------------------------------------"
# Just all of the directories in your current directory?
ls -l [[:upper:]]*

echo "-------------------------------------------------"
# How about all of the files we created with an expansion using the { } brackets?
ls [:lower:].test .

echo "-------------------------------------------------"
# Files with a specific extension OR two?
echo ${STR1} > test.txt
ls *.{test,txt} 

echo "-------------------------------------------------"
# How about looking for specific punctuation and output on the same line
echo "${STR1}" | grep -o [[:punct:]] | xargs echo

echo "-------------------------------------------------"
# How about using groups and single character wildcards (only 5 results)
ls | grep -E "([[:upper:]])([[:digit:]])?.test?" | tail -n 5

exit 0
```

## Striping/altering/sorting/deleting/searching string with Bash only

### Basic Example [Striping]
```
#!/bin/bash
# Index zero of VARIABLE is the char 'M' & is 14 bytes long
VARIABLE="My test string"
# ${VARIABLE:startingPosition:optionalLength}
echo ${VARIABLE:3:4}
```

### Example 2

#### Geting Ready
Let's get ready the envrionment by creating some data sets which mimic common daily problems:

```
$ rm -rf testdata; mkdir -p testdata
$ echo "Bob, Jane, Naz, Sue, Max, Tom$" > testdata/garbage.csv 
$ echo "Zero, Alpha, Beta, Gama, Delta, Foxtrot#" >> testdata/garbage.csv 
$ echo "1000,Bob,Green,Dec,1,1967" > testdata/employees.csv
$ echo "2000,Ron,Brash,Jan,20,1987" >> testdata/employees.csv
$ echo "3000,James,Fairview,Jul,15,1992" >> testdata/employees.csv
```
Using the 2 CSVs, we are going to:

* Remove the extra spaces on the first two lines of garbage.csv
* Remove the last character from each line in garbage.csv
* Change the case of each character to uppercase in the first two lines of garbage.csv
* Replace Bob with Robert in employees.csv
* Insert a # at the beginning of each line in employees.csv
* Remove the exact date of birth column/field in each line of employees.csv

Script implementation as follows:-
* readarray reads lines from the standard input into an array variable: my_array. The -t option will remove the trailing newlines from each line. 
```
#!/bin/bash
GB_CSV="testdata/garbage.csv"
EM_CSV="testdata/employees.csv"
# Let's strip the garbage out of the last lines in the CSV called garbage.csv
# Notice the forloop; there is a caveat

set IFS=,
set oldIFS = $IFS
readarray -t ARR < ${GB_CSV}

# How many rows do we have?
ARRY_ELEM=${#ARR[@]}
echo "We have ${ARRY_ELEM} rows in ${GB_CSV}"

# Strip the garbage - remove space
INC=0
for i in "${ARR[@]}"
do
  res="${i//[^ ]}"
  TMP_CNT="${#res}"
  while [ ${TMP_CNT} -gt 0 ]; do
    i=${i/, /,}
    TMP_CNT=$[$TMP_CNT-1]
  done
  ARR[$INC]=$i
  INC=$[$INC+1]
done

# Remove the last character from each line
INC=0
for i in "${ARR[@]}"
do 
  ARR[$INC]=${i::-1}
  INC=$[$INC+1]
done

# Turn all characters into uppercase
INC=0
for i in "${ARR[@]}"
do
  ARR[$INC]=${i^^}
  printf "%s" "${ARR[$INC]}"
  INC=$[$INC+1]
  echo
done

# In employees.csv update the first field to be prepended with a # character
set IFS=,
set oldIFS = $IFS
readarray -t ARR < ${EM_CSV}

# How many rows do we have?
ARRY_ELEM=${#ARR[@]}

echo;echo "We have ${ARRY_ELEM} rows in ${EM_CSV}"
# Let's add a # at the start of each line
INC=0
for i in "${ARR[@]}"
do
  ARR[$INC]="#${i}"
  printf "%s" "${ARR[$INC]}"
  INC=$[$INC+1]
  echo
done

# Bob had a name change, he wants to go by the name Robert - replace it!
echo
echo "Let's make Bob, Robert!"
INC=0
for i in "${ARR[@]}"
do
  # We need to iterate through Bobs first
  ARR[$INC]=${i/Bob/Robert}
  printf "%s" "${ARR[$INC]}"
  INC=$[$INC+1]
  echo  # due prinf does not print newline therefore we use echo here
done

# Delete the day in the birth column
echo;
echo "Lets remove the column: birthday (1-31)"
INC=0
COLUM_TO_REM=4
for i in "${ARR[@]}"
do

  # Prepare to also parse the ARR element into another ARR for
  # string manipulation
  TMP_CNT=0
  STR=""
  IFS=',' read -ra ELEM_ARR <<< "$i"
  for field in "${ELEM_ARR[@]}"
  do
    # Notice the multiple argument in an if statement
    # AND that we catch the start of it once
    if [ $TMP_CNT -ne 0 ] && [ $TMP_CNT -ne $COLUM_TO_REM ]; then
      STR="${STR},${field}"
    elif [ $TMP_CNT -eq 0 ]
    then
      STR="${STR}${field}"
    fi 
    TMP_CNT=$[$TMP_CNT+1]
  done
  ARR[$INC]=$STR
  echo "${ARR[$INC]}"
  INC=$[$INC+1]
done
```

Note: For more information about ["Bash Read Comma Separated CSV File"](Bash-Read-Comma-Separated-CSV.md). The readarray command parses the input into an array using the IFS and oldIFS variables. It separates the data based on a common delimiter (IFS), and oldIFS can maintain the old values, should they be altered:

### Example 3 
The following script show splitting and string comparison
```
#!/bin/bash

# Let's play with variable arrays first using Bash's equivalent of substr

STR="1234567890asdfghjkl"

echo "first character ${STR:0:1}"
echo "first three characters ${STR:0:3}"

echo "third character onwards ${STR: 3}"
echo "forth to sixth character ${STR: 3: 3}"

echo "last character ${STR: -1}"

# Next, can we compare the alphabeticalness of strings?

STR2="abc"
STR3="bcd"
STR4="Bcd"

if [[ $STR2 < $STR3 ]]; then
 echo "STR2 is less than STR3"
else
 echo "STR3 is greater than STR2"
fi

# Does case have an effect? Yes, b is less than B
if [[ $STR3 < $STR4 ]]; then
 echo "STR3 is less than STR4"
else
 echo "STR4 is greater than STR3"
fi
```

## Using SED and AWK to remove/replace substring
When should we use sed and awk tools ?
* When we care less about the speed that might be gained by using the built-in functionality of Bash
* When more complex features are needed (when programming constructs like multi-dimensional arrays are required or editing streams)

Doing the same with sed and awk using Example 2
```
#!/bin/sh
GB_CSV="testdata/garbage.csv"
EM_CSV="testdata/employees.csv"
# Let's strip the garbage out of the last lines in the CSV called garbage.csv
# Notice the forloop; there is a caveat

set IFS=,
set oldIFS = $IFS
readarray -t ARR < ${GB_CSV}

# How many rows do we have?
ARRY_ELEM=${#ARR[@]}
echo "We have ${ARRY_ELEM} rows in ${GB_CSV}"

# Let's strip the garbage - remove spaces
INC=0
for i in "${ARR[@]}"
do
   : 
  ARR[$INC]=$(echo $i | sed 's/ //g')
  echo "${ARR[$INC]}"
  INC=$[$INC+1]
done

# Remove the last character and make ALL upper case
INC=0
for i in "${ARR[@]}"
do
   : 
  ARR[$INC]=$(echo $i | sed 's/.$//' | sed -e 's/.*/\U&/' )
  echo "${ARR[$INC]}"
  INC=$[$INC+1]
done

set IFS=,
set oldIFS = $IFS
readarray -t ARR < ${EM_CSV}

INC=0
for i in "${ARR[@]}"
do
 : 
 ARR[$INC]=$(sed -e 's/^/#/' <<< $i )
 echo "${ARR[$INC]}"
 INC=$[$INC+1]
done

sed -i 's/Bob/Robert/' ${EM_CSV}
sed -i 's/^/#/' ${EM_CSV} # In place, instead of on the data in the array
cat ${EM_CSV}
# Now lets remove the birthdate field from the files
# Starts to get more complex, but is done without a loop or using cut
awk 'BEGIN { FS=","; OFS="," } {$5="";gsub(",+",",",$0)}1' OFS=, ${EM_CSV}
```
Note: In the final example, we briefly use AWK to show the power of this utility. In this example, we specify the delimiters (FS and OFS) and then we specify the fifth column alongside the gsub sub command in the AWK language to remove the column or field. Begin specifies the rules AWK shall use when parsing input and if there are multiple rules, the order received is the order executed.

Alternatively, we can print the first column or field using awk 'BEGIN { FS=","} { print $1}'  testdata/employees.csv and even the first occurrence by specifying NR==1 like this: awk ' BEGIN { FS=","} NR==1{ print $1}' . Specifying the number or returned records is very useful when using the grep command and copious amounts of matches are returned.

Note

Doing the same with sed and awk using Example 3
```
#!/bin/bash
STR="1234567890asdfghjkl"
echo -n "First character "; sed 's/.//2g' <<< $STR # where N = 2 (N +1)
echo -n "First three characters "; sed 's/.//4g' <<< $STR

echo -n "Third character onwards "; sed -r 's/.{3}//' <<< $STR
echo -n "Forth to sixth character "; sed -r 's/.{3}//;s/.//4g' <<< $STR

echo -n "Last character by itself "; sed 's/.*\(.$\)/\1/' <<< $STR
echo -n "Remove last character only "; sed 's/.$//' <<< $STR
```
Note: [What do <<< mean?](https://unix.stackexchange.com/questions/80362/what-does-mean)

## Using file attributes with conditional logic

* -e: The file exists
* -f: This is a regular file and not a directory or device file
* -s: The file is not empty or zero in size
* -d: This is a directory
* -r: This has read permissions
* -w: This has write permissions
* -x:This has execute permissions
* -O: This is the owner of the file the current user
* -G: This executes the user if they have the same group as yours
* f1 (- nt, -ot, -ef) f2: Refers to if f1 is newer than f2, older than f2, or are hard-linked to the same file

### Example1
Getting data ready
```
$ cd ~/
$ mkdir -p fileops
$ touch fileops/empty.txt
$ echo "abcd1234!!" > fileops/string.txt
$ echo "yieldswordinthestone" > fileops/swordinthestone.txt
$ touch fileops/read.txt fileops/write.txt fileops/exec.txt fileops/all.txt
$ chmod 111 fileops/exec.txt; chmod 222 fileops/write.txt; chmod 444 fileops/read.txt; fileops/all.txt;chmod 777 fileops/all.txt
$ sudo useradd bob
$ echo "s the name" > fileops/bobs.txt
$ sudo chown bob.bob fileops/bobs.txt
```

Scripts implementation
```
#!/bin/bash
FILE_TO_TEST=""

function permissions() {

  echo -e "\nWhat are our permissions on this $2?\n"
  if [ -r $1 ]; then 
    echo -e "[R] Read" 
  fi
  if [ -w $1 ]; then 
    echo -e     "[W] Write" 
  fi
  if [ -x $1 ]; then 
    echo -e "[X] Exec" 
  fi
}

function file_attributes() {

  if [ ! -s $1 ]; then
    echo "\"$1\" is empty" 
  else 
    FSIZE=$(stat --printf="%s" $1 2> /dev/null)
    RES=$?
    if [ $RES -eq 1 ]; then
      return
    else
      echo "\"$1\" file size is: ${FSIZE}\""
    fi
  fi

  if [ ! -O $1 ]; then
    echo -e "${USER} is not the owner of \"$1\"\n"
  fi
  if [ ! -G $1 ]; then
    echo -e "${USER} is not among the owning group(s) for \"$1\"\n"
  fi

  permissions $1 "file"

}

```

How it works
* Files and directories can be owned. This means that they can have an owner (user) and groups associated with their ownership. For this, we can use the chown and chgrp commands.
* Files and directories can have different permissions applied to them. This means that they may be executable, readable, writable, and/or everything. For this, we can use the chmod command and the appropriate permission setting.
* Files and directories can also be empty.
* The read command, which is used to wait for user input and read it into a variable. It is also useful for "pause" functionality in scripts.
* Recursive functions. Notice that inside of the script unless it exits or the user presses ctl + C, the script keeps calling a particular function. This is recursion and it will continue unless stopped or a limit is applied.

Output
```
# fileops/bobs.txt

"fileops/bobs.txt" file size is: 11"
rbrash is not the owner of "fileops/bobs.txt"

rbrash is not among the owning group(s) for "fileops/bobs.txt"

What are our permissions on this file?

[R] Read

What is the complete path of the file you want to inspect?
 # fileops/write.txt

"fileops/write.txt" is empty

What are our permissions on this file?

[W] Write

What is the complete path of the file you want to inspect?
 # fileops/exec.txt

"fileops/exec.txt" is empty

What are our permissions on this file?

{X] Exec

What is the complete path of the file you want to inspect?
 # fileops/all.txt

"fileops/all.txt" is empty

What are our permissions on this file?

[R] Read
[W] Write
{X] Exec

What is the complete path of the file you want to inspect?
 # fileops

Directory "fileops" has children:

all.txt
bobs.txt
empty.txt
exec.txt
read.txt
string.txt
swordinthestone.txt
write.txt

What are our permissions on this directory?

[R] Read
[W] Write
{X] Exec

What is the complete path of the file you want to inspect?
 # thisDoesNotExist.txt

Error: "thisDoesNotExist.txt" does not exist!
$
```

## Searching for file by name and/or extension
* locate (also a sibling of the updatedb command): Used to find files more efficiently using an index of files. The file index can be updated using the following command:
```
$ sudo updatedb
```
* find: Used to find files with specific attributes, extensions, and even names within a specific directory, [Note: Please be aware that your platform may not support all of GNU find's features. This may be the case with limited shells for embedding, resource constraints, or security reasons.]
### Getting Ready
```
$ sudo apt-get install locate manpages manpages-posix
$ sudo updatedb
$ git clone https://github.com/PacktPublishing/Linux-Device-Drivers-Development.git Linux-Device-Drivers-Development # Another Packt title
$ mkdir -p ~/emptydir/makesure
```
[Note:If a file is not found using the locate command, the database might be simply out of date and needs to be re-ran. It is possible that updatedb is also not indexing partitions such as those contained on removable media (USB sticks) ]

Understand locate command
```
$ locate stdio.h
$ sudo touch /usr/filethatlocatedoesntknow.txt /usr/filethatlocatedoesntknow2.txt
$ sudo sh -c 'echo "My dear Watson ol\'boy" > /usr/filethatlocatedoesntknow.txt'
$ locate filethatlocatedoes
$ sudo updatedb
$ locate filethatlocatedoesntknow
```

Understand find command
```
$ sudo find ${HOME} -name ".*" -ls
$ sudo find / -type d -name ".git"
$ find ${HOME} -type f \( -name "*.sh" -o -name "*.txt" \)
```

Chain find commands together with && and ultimately perform an exec instead of piping the output to another process
```
$ find . -type d -name ".git" && find . -name ".gitignore" && find . -name ".gitmodules"
$ sudo find / -type f -exec grep -Hi 'My dear Watson ol boy' {} +
```

one of common uses to find is to delete file using either the built-in -delete flag or by using exec combined with rm -rm
```
$ find ~/emptydir -type d -empty -delete
$ find Linux-Device-Drivers-Development -name ".git*" -exec rm -rf {} \;
```
At a minimum, the find command is executed this way: find  {START_SEARCH_HERE} {OPTIONAL_PARAMETERS ...}. We can chain the find commands together using this format: $ cmd 1 && cmd2 && cmd3 && .... This guarantees that if the proceeding command evaluates to true, then the next will execute and so on.