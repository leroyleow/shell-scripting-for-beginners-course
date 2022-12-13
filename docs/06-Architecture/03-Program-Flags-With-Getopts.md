# Program Flag With Getopts

In addition to the basic logic, we can see that the code leverages a piece of functionality called getopts. Getopts allows us to grab the program parameter flags for use within our program. There are also primitives, which we have learned as well—conditional logic, while loop, and case/switch statements. Once a script develops into more than a simple utility or provides more than a single function, the more basic Bash constructs will become commonplace.

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