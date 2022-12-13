# Program Flag With Getopts

The getopts command is a built in shell command for parsing command line arguments. It is better than the getopt alternative for several reasons.
### Basic Example
Let's create a basic sample.sh script that take a single option <b>a</b>
```
$ ./sample.sh -a
```
Implementation as follows:
```
#!/bin/bash
while getopts "a" opt; do
  case $opt in
     a)
       echo "argument -a called" >&2
       ;;
  esac
done
```
The output of this script will be 
```
> arguement -a called
```
### Example getopts option with parameter
Let's say want the option a  to accept parameter:-
```
$ ./sample.sh -a hello
```
Implementation will be:-
```
#!/bin/bash
while getopts "a:" opt; do
  case $opt in
     a)
       echo "argument -a called with parameter $OPTARG" >&2
       ;;
  esac
done
```
Output of the script will be:
```
argument -a called with parameter hello
```

### Example Working With Multiple Arguments
You can work with any number of arguments in your option string. Let's say you have a script taking two parameters, a and b:
```
$ ./sample.sh -a hello -b goodbye
```
Notice how the argument a takes a parameter hello while the b argument acts as a flag. We could implement this like so:
```
#!/bin/bash
while getopts ":a:b:" opt; do
  case $opt in
     a)
       echo "argument -a called with parameter $OPTARG" >&2
       ;;
     b)
       echo "argument -b called with parameter $OPTARG" >&2
       ;;
     *)
       echo "invalid command: no parameter included with argument $OPTARG"
       ;;
  esac
done
```
Output of this script will be:-
```
argument -a called with parameter hello
argument -b called with parameter goodbye
```

Getopts allows us to grab the program parameter flags for use within our program. There are also primitives, which we have learned as well—conditional logic, while loop, and case/switch statements. Once a script develops into more than a simple utility or provides more than a single function, the more basic Bash constructs will become commonplace.

### Example
```
#!/bin/bash

HELP_STR="usage: $0 [-h] [-f] [-l] [--firstname[=]<value>] [--lastname[=]<value] [--help]"

# Notice hidden variables and other built-in Bash functionality
optspec=":flh-:"
while getopts "$optspec" optchar; do
    case "${optchar}" in
        -)
            case "${OPTARG}" in
                firstname)
                    val="${!OPTIND}"; OPTIND=$(( $OPTIND + 1 ))
                    FIRSTNAME="${val}"
                    ;;
                lastname)
                    val="${!OPTIND}"; OPTIND=$(( $OPTIND + 1 ))
                        LASTNAME="${val}"
                    ;;
                help)
                    val="${!OPTIND}"; OPTIND=$(( $OPTIND + 1 ))
                    ;;
                *)
                    if [ "$OPTERR" = 1 ] && [ "${optspec:0:1}" != ":" ]; then
                        echo "Found an unknown option --${OPTARG}" >&2
                    fi
                    ;;
            esac;;
        f)
                val="${!OPTIND}"; OPTIND=$(( $OPTIND + 1 ))
                FIRSTNAME="${val}"
                ;;
        l)
                val="${!OPTIND}"; OPTIND=$(( $OPTIND + 1 ))
                LASTNAME="${val}"
                ;;
        h)
            echo "${HELP_STR}" >&2
            exit 2
            ;;
        *)
            if [ "$OPTERR" != 1 ] || [ "${optspec:0:1}" = ":" ]; then
                echo "Error parsing short flag: '-${OPTARG}'" >&2
                exit 1
            fi

            ;;
    esac
done

# Do we have even one argument?
if [ -z "$1" ]; then
  echo "${HELP_STR}" >&2
  exit 2
fi

# Sanity check for both Firstname and Lastname
if [ -z "${FIRSTNAME}" ] || [ -z "${LASTNAME}" ]; then
  echo "Both firstname and lastname are required!"
  exit 3
fi

echo "Welcome ${FIRSTNAME} ${LASTNAME}!"

exit 0
```
### Output
```
$ bash flags.sh 
usage: flags.sh [-h] [-f] [-l] [--firstname[=]<value>] [--lastname[=]<value] [--help]
$ bash flags.sh -h
usage: flags.sh [-h] [-f] [-l] [--firstname[=]<value>] [--lastname[=]<value] [--help]
$ bash flags.sh --fname Bob
Both firstname and lastname are required!
rbrash@moon:~$ bash flags.sh --firstname To -l Mater
Welcome To Mater!
```