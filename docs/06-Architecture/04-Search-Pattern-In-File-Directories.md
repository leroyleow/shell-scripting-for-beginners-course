# Search Pattern In Files and Directories

### Examples With Grep
* <b>Note</b> Notice the use of ~/*. This refers to home directory and * allows us to specify anything from that point on. 
* xargs reads standard input and then runs the command you give it on each line of the input
* Grep and many other utilities can be case-sensitive. To use grep in a way that's case insensitive, use the -i flag

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