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