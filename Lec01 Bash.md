# Lec 2 Bash Script Command#

### Input redirection < ###

allows directing input into a program from a file

	$cat < input.txt

**Differences between using file as arg & input redir**
	
	$cat myfile.txt
	     --argument--

cat is sent the string myfile.txt

*cat* opens the file & read the content

	$cat < mefile.txt

The *shell* opens myfile.txt & makes the content available to `cat`

	$wc myfile.txt
	> lines words letters myfile.txt
	
	$wc < mtfile.txt
	> lines words letters
	# wc has no way to know the file name
 

## Streams ##

Every Linux process has 3 streams allowed

	standard input                  standard output
	 (stdin)            ---------     (stdout)
	-----------------> | process | ---------------->
	default:keyboard    ---------    default:screen
                            |        use out redir (>)
    use input redir (<)     |        to change the file 
	to change the file		|
                            |-------> standard error (stderr)
                                       default:screen
 									   use 2> to change the file

example:

	$mycmd arg < in.txt
	$mycmd > out.txt
	# overwirte existing file (creating new at first execution)
	$mycmd 2> err.txt

	$mycmd >> out.txt
	# append to existing file (creating new at first execution)
	$mycmd 2>&1 out.txt
	# put stdout & stderr in same file
	# &1 is where stdout is redirecter now


##Wildcard matching (Global Pattern)##

	*.txt

It's the ability of *shell* (not command)

How it works?

- when the shell sees a global pattern, it finds all files that match the pattern & replace the pattern with file names
- when using global pattern, it passes *several* arguments (instead of one) to the command

example:

	$rm -rf *.cc
	$echo *.txt
	# 1.txt 2.txt 3.txt

	$echo "*.txt"
	$echo '*.txt'
	# *.txt
	# both prevent global pattern from expanding

## Pipe ##

####connect the stdout of a program to the stdin of another program####


	$head -20 sample.txt | wc -w
                  --> temp -->

print a duplicate-free list of word1.txt and word2.txt (one word per line)

	$cat word*.txt | sort | uniq
	# uniq remove duplicate adjent to each other

####use the output of a program as an argument of another program####

use `$(----)` to embed command

	$echo "Today is $(date)"   
	# one argument
	#$echo Today is $(date)

	$echo 'Today is $(date)'
	#Single quote prevents embedding command from expanding

## `egrep`: Global Regular Expression Print##
- search within text file

	usage: $egrep pattern files
				  (regex)


- the output is  *lines*  that match pattern
- By default, it is *case-sensitive*
- And matches all results in prefix, suffix or in middle

i.e

	$egrep "cs246|CS246" index.html

	|     or in regex

- single quote supports global pattern & regex from expanding
- double quote only supports global pattern from expanding

	
Some regex:
	
	"cs246|CS246" == "(cs|CS)246" 
	"(c|C)(s|S)246" == "[cC][sS]246"

	[   ] : choose one char from the set of char
	[^  ] : choose any one char not in the set
	  |   : choose one string
	
	  ?   : 0 or 1 of the proceeding expression
	"cs ?246" == "cs246|cs 246"
	"(cs)?246" == "246|cs246"
	
	  *   : 0 or more of the proceeding pattern
	"(cs)*246" == "246|cs246|cscs246..."

	  +   : 1 or more occurance of the proceeding pattern
	  .   : any one char
	  .*  : any number of any char

# Lec 3, Shell / Shell Script  Sep 14th#

- Use `^` to indicate that the pattern must match for the start of the line
- Use `$` to indicate that the pattern must match for the end of the line
- Use `\` to escape char that has special meaning

**exercises**

print all words that start with "e" and have 5 char

	$egrep "^e....$" <filename>

print words of even length

	$egrep "^(..)*$" <filename>

print words with exactly one "a"
	
	$egrep "^[^a]*a[^a]*$" <filename> 

##File permission##

	$ls -l

	-rw-r-xr--    1       Carey  staff  123 Sep 1st ... <filename>
	|    \    # of links  owner  group  sign  time last modified
    file  permissions           the file belongs to
    type

    rw- r-x r--
     1   2   3
1. owner's permission
2. all users in the file's group (exclude owner)
3. all others

.

	  r    w    x
	read write execute

meaning of execute

- x for non-directory file : can attempt to execute a file
- x for directory file: can navigate to that dir (cd)


Owner is the only one that has right to change permissions

	chmod        <mode>       <file>
	         /     |     \
 	 ownership   operator  permission
	  class
	 u - owner    + add         r
	 g - group	  - remove      w
	 o - others	  = set exacly  x
	 a - all

	$chmod g-r <file>
	$chmod o=rm
	$chmod a+x

## Shell variable##

	x=1 
	# create a variable called x with the string value 1

	$echo "${x}" 
	$echo $x
	# display the value of x
	
	${PATH}
	# list of directories seperated by:
	# where command are stored
	# when runing a command, shell will search command in those dir

	# change PATH variable
	PATH=${PATH}:newdir

##Shell Scripts##

A text file containing a sequence of command executed as a program

	#!/bin/bash      ---shebang: "I am bash script"

To run bash script "basic" in current dir,

	$./basic	
	# you can put "." in PATH

### command line argument###

	$./myscript  arg1  arg2
	    $0        $1    $2   $3
	  (not argu)            (empty string)
Given a script, $1 refers to arg1 and so on.


.......

something here

# LEC 4 Shell Scripts Command, C++, Sep 19th#

##Shell Loops##

####while loop####
Print number 1 to $1

*lectures/shell/scripts/count*

	#!/bin/bash

	usage () {
	  echo "Usage:  $0 limit" 1>&2
	  echo "  where limit is at least 1" 1>&2
	  exit 1
	}

	if [ $# -ne 1 ]; then
	  usage
	fi

	if [ $1 -lt 1 ]; then
	  usage
	fi

	x=1  
	while [ $x -le $1 ]; do
	  echo $x
	  x=$((x + 1))
	done

Notes:

	$((...))  treat it as math expression

	x=$(x + 1)
	 =$x + 1
	 =1 + 1 (string)

----
#### for loop####
Rename all files ending in .c to ending in .cc

*lectures/shell/scripts/renameC*

	#!/bin/bash
	# Renames all .C files to .cc

	for name in *.C; do
	  mv ${name} ${name%C}cc
	done

Notes:

	${name%C} if name ends in C, it will be removed; otherwise, nothing happens
	${z#C} if z begins with C, it will be removed

---
Count the number of times of word $1 appears in file $2

*lectures/shell/scripts/countWords*

	#!/bin/bash
	# countWords word file
	#  Prints the number of times word occurs in file

	x=0
	for word in $(cat "$2"); do
	  if [ $word == $1 ]; then
	    x=$((x + 1))
	  fi
	done
	echo $x

-----
### argument for function vs argument for command ###

	#!/bin/bash
	# Returns the date of the next payday (last Friday of the month)
	# Examples:
	# payday (no arguments) -- gives this month's payday
	# payday June 2016 -- gives payday in June 2016

	if [ $# -ne 0 -o $# -ne 2 ]; then
	 exit 1;
	fi

	answer () {
	  if [ $2 ]; then
	     preamble=${2}
	  else
	     preamble="This month"
	  fi
	  if [ $1 -eq 31 ]; then
	    echo "${preamble}'s payday is on the 31st."
	  else
 	   echo "${preamble}'s payday is on the ${1}th."
	  fi
	}

	answer $(cal $1 $2 | awk '{print $6}' | grep "[0-9]" | tail -1) $1
	               year
            ------------------$1 for anwser-----------------  --$2 for anwser---

#### awk ####

	awk 'pattern {action}' input-file > output-file

If the pattern is omitted, the action is applied to all line. 

	awk '{ print $5 }' table1.txt > output1.txt


	
