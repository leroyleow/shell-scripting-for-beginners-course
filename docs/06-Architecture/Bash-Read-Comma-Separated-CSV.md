# Bash Read Comma Separated CSV File

A comma-separated values (CSV) file is a delimited text file that uses a comma to separate values. A CSV file stores tabular data in plain text format. Each line of the file is a data record. You can use while shell loop to read comma-separated cvs file. IFS variable will set cvs separated to , (comma). The read command will read each line and store data into each field.

The syntax is as follows phrase a CSV file named input.csv:
```
while IFS=, read -r field1 field2
do
    echo "$field1 and $field2"
done < input.csv
```

Here is how sample data.csv file look:
```
Vivek Gite,10/10/1972,111-555,8888-444,Yes
Ram Kumar,12/01/1980,222-6666,8888-333,No
Tom Jerry,01/01/1970,333-7777,8888-424,Yes
```

# How to parse a CSV file in bash
```
#!/bin/bash
# Purpose: Read Comma Separated CSV File
# Author: Vivek Gite under GPL v2.0+
# ------------------------------------------
INPUT=data.csv
OLDIFS=$IFS
IFS=','
[ ! -f $INPUT ] && { echo "$INPUT file not found"; exit 99; }
while read flname dob ssn tel status
do
	echo "Name : $flname"
	echo "DOB : $dob"
	echo "SSN : $ssn"
	echo "Telephone : $tel"
	echo "Status : $status"
done < $INPUT
IFS=$OLDIFS
```

# Dealing with missing data/value or field

Unfortunately, there is no easy solution for missing values in a given field. We can detect if a value is missing using the if statement or case statement. For instance:
```
#!/bin/bash
missing=false
while IFS=, read -r field1 field2
do
	if [ "$field1" == "" ]
	then
		echo "field1 is empty or no value set"
		missing=true
   	elif [ "$field2" == "" ]
	then
		echo "field2 is empty or no value set"
		missing=true
	else
		echo "$field1 and $field2"
	fi
done < input.csv
if [ $missing ]
then
	echo "WARNING: Missing values in a CSV file. Please use the proper format. Operation failed."
	exit 1
else
	echo "CSV file read successfully."
fi
```

# Use Awk to Parse Csv
```
awk -F',' 'BEGIN{ print "<table border=1>"; print "<tr><th>Name</th><th>Bounce</th><th>Department</th></tr>"}
           { print "<tr><td>" $1 "</td><td>" $2 "</td><td>" $3 "</td></tr>" } 
           END{ print "</table>"}' input.csv
```

Output
```
<table border=1>
<tr><th>Name</th><th>Bounce</th><th>Department</th></tr>
<tr><td>Vivek Gite</td><td>1000</td><td>IT</td></tr>
<tr><td>Marelna W</td><td>200</td><td>IT</td></tr>
<tr><td>B Fox</td><td>25</td><td>Dev</td></tr>
</table>
```