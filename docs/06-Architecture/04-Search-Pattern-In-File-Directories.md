# Search Pattern In Files and Directories

### Examples With Grep

- <b>Note</b> Notice the use of ~/_. This refers to home directory and _ allows us to specify anything from that point on.
- xargs reads standard input and then runs the command you give it on each line of the input
- Grep and many other utilities can be case-sensitive. To use grep in a way that's case insensitive, use the -i flag

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
