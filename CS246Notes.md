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

​	



# Lec 4 cont'd #

## Testing ##

something here

## C++ ##

Bjarne Shoushrup (80's)

- Simula 67 - first OO langauge
- C with classes

c++99 -> c++03 -> c++11 -> c++14 -> c++17
         (cs246)

**something here**

- in c++, `main` function has `int` return type (in c, can return `void`)
- `main` function allows not write `return` statement (return 0 by default)
- `stdio.h` is available in c++ (but not use in CS246)


## Compile ##

	$g++-5 -std=c++14 hello.cc -o myprogram
	
	// same as
	$g++14 hello.cc -o myprogram

run it:

	./myprogram


### pass command-line argument to c++

	#include <iostream>
	using namespace std;
	
	int main(int argc, char *argv[]) {
	  cout << "Number of arguments: " << argc - 1 << endl;
	  cout << "Arguments: " << endl;
	  for (int i = 1; i < argc; ++i) {
	    string theArg = argv[i]; 
	    cout << " " << i << ": " << theArg << endl;
	  }
	}
	
	// argv[0] is program name


# Lec5, Input/Output control

*c++/intro/plus.cc*

    #include <iostream>
    using namespace std;
    
    int main () {
      int x, y;
      cin >> x >> y;
      cout << x + y << endl;
    }

- cin ignores white space by default

  - `cin >> skipws;`  skip whitespace
  - `cin >> noskipws;` include whitespace 
  - from the `<iostream>` header. 
- If read fails, an int value is set to **0** (c++17)
- If it is too big, it is set as`int_max` 
- If it is too small, `int_min`
- Id read fails due to EOF, int is set to 0

i.e.

	> 3
	> x = 3
	> 3 (ctrl+D)
	> x = 0

We should check whether read fails or succeed

- if a read fails, `cin.fail()` is true
- Additionally, if a read fails due to EOF, `cin.eof()` is also true

------

Read int from stdin, echo them one per line to stdout. Exit if read fails.

*lectures/c++/io/readInts.cc*

	int main() {
	  int i;
	  while (true) {
	    cin >> i;
	    if (cin.fail()) break;   
	    cout << i << endl;
	  }
	}

----

c++ defines an automatic/implicit conversion from `istream` variables to `bool`

i.e. cin can used as a bool. cin is true if last read succeeded

*lectures/c++/io/readInts2.cc*

	int main() {
	  int i;
	  while (true) {
	    cin >> i;
	    if (!cin) break;   //  cin.fail()
	    cout << i << endl;
	  }
	}

------

### operator overloading ###

	cin >> i;
	   input operator
	21 >> 3;
	   right bit shift operator
	     10101 -> 10  
	     21/2^3 = 2

- any operator can be given special meaning determined by the types of operands
- If `>>` is used as input operator, the expression produce the LHS (i.e. `cin`)

**cascading reads**

	cin >> x >> y >> z;
	   cin   >> y >> z;
	        cin   >> z;
	             cin  ;

If read x fails, the all subsequent read (y,z) will also fail.

*readInts3.cc*

	int main() {
	  int i;
	  while (true) {
	    if (!(cin >> i)) break;
	    cout << i << endl;
	  }
	}


*readInts4.cc*

	int main() {
	  int i;
	  while (cin >> i) {
	    cout << i << endl;
	  }
	}

*readInts5.cc*

Read all int i. Echo to stdout until EOF, ignore non-int input


	int main () {
	  int i;
	  while (true) {
	    if (!(cin >> i)) {
	      if (cin.eof()) break;
	      else { // if cin fail due to bad input
	        cin.clear();    // acknowledge fail, lower the fail flag
	        cin.ignore();   // discard one char
	      }                 
	      // if not using, once fail, all subsequnt fail -> infinite loop
	    }
	    else {
	      cout << i << endl;
	    }
	  }
	}

When an input failure occurs and cin.fail() returns true, the input buffer (cin) is placed in an "error state", and further input processing will not work unless you "clear the error state" by calling cin.clear()

i.e.

	> h1a
	> 1

## Read/Write Strings ##

	#include <string>
	
	int main() {
		string s;
		cin >> s;
		cout << s << endl;
	}

- stdin is different from arguments ( " has no special meaning)

i.e.

	> "Hello World"
	> "Hello

- cin will read from the first non-whitespace char until it hits a whitespace char

## Format Specifies

In c ......

In c++, we use **I/O manipulators** to format I/O

	#include <iomanip>
	
	int x = 95;
	cout << x; //print decimal
	cout << hex << x; //print in hex (all the following cout)
	cout << dec << x; //print decimal
	
	float price = 2.00;
	cout << fixed << showpoint << setprecision(2) << price << endl;

- `showpoint` format flag : decimal point is always written   
- `setprecision(2)`: as many digits as necessary are written to match the precision set for the stream (if any).
- `boolalpha` format flag : bool values are inserted/extracted by their textual representation (true or false), instead of integral values (1 or 0).
- `skipws`, `noskipwsignore`: white space or not

Read an entire line

	getline(cin,x); //read until the next /n char

### The stream abstraction can be used to read/write to other source of data (i.e. file/string) ###

	#include <fstream>
	
	int main () {
	  ifstream file{"suite.txt"}; //open the file for reading
	  // type   var intializating
	  // ifstream - to read from file
	  // ofstream - to write to file
	  string s;
	  while (file >> s) {
	    cout << s << endl;
	  }
	}

The file is *automatically* closed when the stack allocated variable goes out of scope.


# Lec6 


*io/buildString.cc*

	#include <sstream>
	int main () {
	  ostringstream ss;   // write to string
	  int lo {1}, hi {100};
	  ss << "Enter a # between " << lo << " and " << hi;
	  string s {ss.str()};  //convert stream to string
	  cout << s << endl;
	}

*io/getNum.cc*

ensure that the input is an integer

	int main () {
	  int n;
	  while (true) {
	    cout << "Enter a number:" << endl;
	    string s;
	    cin >> s;   //succeed as long as user type something
	    istringstream ss{s};
	    if (ss >> n) break;
	    cout << "not a number";
	  }
	  cout << "You entered " << n << endl;
	}


### String

c string are null-terminated character array

- manage your own memory
- error prone i.e. overwrite null terminator

c++ string type

- header: `<string>`
- does own memory management
- no worry about null terminator either

i.e.

	string s;
	string s = "hello";
	// c++ str   c-style str
	//  <---one direction---
	// c-style string can be converted to c++ string
	// c++ str cannot be converted to C-style string

c: functions on string: strcpy,strcat,strlen

c++: string can use operator

- ==; !=; <; >; <=; >=
- + (cat)
- length()
- []: array notation to access individual char in c++ string: `str[0]`

review:

	"a" < "b"g
	"a" < "ab"
	"A" < "a"  //Since A has ASCII value 65; a has a higher ASCII value

## Default Argument

	void printFile (string name = "file.txt"){
	 ifstream fs{name};
	 string s;
	 while (fs >> s){
	  cout << s << endl;
	 }
	}

Function parameters in c++ can be given default value:

	printFile();
	printFile("other.txt");

- cannot follow a para with a default value with a para doesn't have a default value

  `void foo(int x = 5, string str);   //ILLEGAL!!`
  `void foo (<with default>,<without default>)` is invalid

- if you are going to leave something not in a function call, it has to be last parameter, if they have default value.  **default value as last argument**

i.e.

	void foo(int x = 5, string str = "hello");
	
	foo(10,"a");
	foo(10);
	foo();
	
	//foo("str");    X
	//foo(,"str");   X

## Function overloading

	void foo (int,string);
	void foo (int);
	void foo ();

- In c, it is illegal for two functions to have the same name

- In c++, functions can have the same name as long as the differs in the number and/or types of **parameters**

  `int negate (int b) { return -b; }`
  `bool negate (bool x) { return !x; }`
  `Illigal in c, legal in c++`

- It is not enough for function to only differ on the return type

**c++ operators are implemented as functions**

	operator>>(cin,x); //cin >> x
	operator>>(x,3);   //int x; x >> 3

**c++ compiler are able to figure out which function to call base on parameter**

	void foo (int x);
	//void foo (int x, int y = 10);  ILLEGAL, because
	foo(42);   //ambiguous call

## struct ##

In C,

	struct Node{
		int data;
		struct Node *next;
	};
	struct Node n = {5,null};

In c++, 

	struct Node{
	 int data;
	 Node *next;
	}
	
	Node n{5,nullptr};

Why the following code is ILLEGAL?

	struct Node{
	 int data;
	 Node next;
	}

You can NEVER allocate a variable of type Node. Try to figure out the size of Node will cause **infinite loop**

### Parameter Passing (Review)

pass by value

	void inc (int n) {
		n = n + 1;
	}
	
	int x = 5;
	inc (x);
	cout << x << endl; // print 5, pass by value (copy of x)

pass by pointer

	void inc (int *n) 	{
		*n = *n + 1;
	}
	int c (&x);
	// print 6	 


**Passing parameter to input stream**

	cin >> x;
	operator >> (cin,x); 
	// How do we change x?
	// seems no "&", but actually x is passed by reference
	
	//scanf("%d",&x);

## Lvalue Reference ##

	int y = 10;
	int &z = y;

z is a **lvalue reference** to y

lvalue reference is like a **constant pointer** with automatically dereference.

	z = 15; // not *z = 15
	int *p = &z; // p ends up point to y

- z mimics y
- z is another name / alias for y

### Things you cannot do with reference

- You cannot leave them uninitialized

  `int &z;  //ILLEGAL!`

- Must be initialized to a lvalue (something that has an **address**, refers to an actual storage location)

ILLEGAL

	//int &z = 3;
	//int &z = x+y; 
	        -- Rvalue --

- you cannot create a pointer to a reference
- you cannot create a reference to a reference
- you cannot create a array of reference

pass by reference 

	void inc (int &n) {
		n = n + 1;
	}
	int x = 5;
	inc(x); // x is 6


# Lec 7, Short Topic, Preprocessor, Sep 28th

Why does `cin >> x` works?

	operator>>(cin,x); // x is a reference
	istream &operator>> (istream &in, int &n){...}

**Para1 `in` is also passed be reference**

- changes made to `in`, need to be visible outside (i.e. `cin.clear()`)
- stream cannot by copied  [ReadMore](https://stackoverflow.com/questions/6010864/why-copying-stringstream-is-not-allowed "Reason")

i.e.

	int main(){
	   std::stringstream s1("This is my string.");
	   std::stringstream s2 = s1; // error, copying not allowed
	}


### Passing by value vs. Passing by reference

	struct ReallyBig{...}
	void f (ReallyBig rb){...}
	
	ReallyBig myrb{...};
	f (myrb); // create a copy of myrb, which is inefficient

----

In c/c++, we could pass a pointer to avoid the copy

In c++, we can pass by reference

	void g (ReallyBig &rb) {...}
	g(myrb); //no copy is made

- changes made to rb are actually changes to myrb

---

	void h (const ReallyBig &rb) {...}

- efficient, no copy
- rb cannot be used to change the thing it refers to

Advice: prefer to pass by reference to constant for anything bigger than an int

---

	void foo (int &n) {...}
	
	//foo(5);
	//foo(y+y);
	//illegal because referance must be initialized to lvalue


	void bar (const int &n) {...} //promise not change n refer to, just use it
	bar(5);
	bar(y+y);


## Dynamic Memory

In c, we use malloc/free

In c++, malloc/free is forbidden. Instead, we use new/delete

	struct Node{
	 int data;
	 Node *next;
	}
	
	Node *np = new Node;
	...
	delete np; 
	//np must be a POINTER returned by NEW

### Stack vs Heap Memory ###

sth here (review of 136)

### Allocate arrays

	Node *myarr = new Node[10];
	...
	delete [] myarr;

### Return by value

	Node getANode(){
	 Node n;
	 return n; // a copy is created
	}

Tempting to do:

	Node *getANode(){
	 Node n;
	 return &n;
	}

After function called, `&n` is a **dangling pointer** pointing to memory no longer to use.

#### Advice: 

- a function should *never return a pointer/reference to a stack allocated variable* that it created. 
- It can return a ptr/reference to a heap allocated variable.

i.e.

	Node *getANode(){
	 Node *np = new Node;
	 return np;
	}

## Operator Overloading ##

c++ allows us to give meaning to operator for types we create

	struct Vec{
	 int x; int y;
	}
	
	Vec operator+(const Vec &v1, const Vec &v2){
	 Vec v{v1.x+v2.x, v1.y+v2.y};
	 return v;
	}
	
	Vec operator*(const int &k, const Vec &v){}
	
	Vec operator*(const Vec &v, const int &k){
	 return k*v;
	}

### Overloading << & >>###

	sturct Grade {
	 int theGrade;
	}
	
	Grage g;
	cin >> g;
	cout << g; //won't compile
	
	istream& operator>> (istream &in, Grade &g){
	 in >> g.theGrade;
	 // some code here
	 return in;
	}
	
	ostream& operator<< (ostream &out, Grade &g){
	 out << g.theGrade << "%";
	 return out;
	}

passing two reference & return by reference

## Preprocessor

- It runs before the compiler

- modifies the code before the compiler sees it

  `#include <...>`

- processor directive copy & paste the included header file

To see processor output

	$g++14 -E -P

Search & replace occurance of VAR by value

	#define VAR VALUE
	
	#define MAX 100
	...
	int array[MAX];  //made obselete by const
	// MAX is a processor const

### Conditional Compilation ###

sth here

define preprocessor variable during compilation


	g++14 -E -P -DVAR=VALUE <filename>

preprocess.cc

	// compile with -E flag to see how preprocessor worked
	#define MAX 10
	#define IOS 1
	#define BBOS 2
	
	//#define OS BBOS // or IOS
	
	int main() {
		int x[MAX];
	   #if OS == BBOS
	    long long int publicKey; // suppressed if OS != BBOS
	   #elif OS == IOS
	    short int publicKey;
	   #endif
	}
	
	$g++14 -DOS=IOS	





# Lec 8, Preprocessor, c++ class, Oct 3rd#

	#define VAR
	// value is the empty string
	
	#ifdef VAR
	//some code here
	#endif
	
	#ifndef VAR
	//some code here
	#endif

To enable debug without changing cc file

*preprocess/debug.cc*

	g++14 -DDEBUG debug.cc
	g++14 debug.cc

## Separate Compilation

It is good to divide a program into 

- header file(.h): user defined information, forward declaration for functions
- implemented file(.cc) : definitions, implementation of functions

*c++/separate/example1*

### How to compile?

	g++14 main.cc vector.cc

or

	g++14 *.cc

- Never compile header file
  - header file are meant to be included
- Never include a cc file
  - cc file are meant to be given to the compiler

~~~~~~~~~~~~~~~~~separete than merge~~~~~~~~~~~~~~~~~~~~~~~~
**Compile but not link**

	g++14 main.cc
	g++14 vector.cc

It won't work because by default g++ will try to compiler & linl

To compile but not link

	$g++14 -c main.cc
	$g++14 -c vector.cc

	> main.o vector.o
	
In this case, you don't need to re-compile tons of file

Produce **object file**

- binary
- indicate provided & required
- link together

	`$g++14 main.o vector.o -o myprogram`

### Sharing a global var in different files###

*abc.h*

	int global; //declaration & definition

	// don't want definition so can be used globally
	// don't want memory allocated
 
	extern int global; // just declaration

abc.cc

	int global; //declaration && def

If global is defined in multiple .cc file, those file *won't link*

###Problem of multiple include of vector.h###

- causes compilation error

Advice:

- Every header file should have an **include guard**

./example4  

	#ifndef _VECTOR_H_
	#define _VECTOR_H_

	struct Vec {
	  int x;
	  int y;
	};

	Vec operator+(const Vec &v1, const Vec &v2);

	#endif

- Never put `using namespace std;`  in a head file

instead, using the full name
std::string std::istream


## C++ class ##

(in next lecture) 
~~~~~~~~~~~~~~~~~









# Lec 8, c++ class #

Key idea of OO programming:

- **we can put functions inside a struct**

i.e.

	some code here



# Lec 9, c++ class #

    Student *p = new Student{60,70,80};
     .
     .
    delete p;

why use constructors?

- can write any code in constructor body

- sanity check

- default arguments

- overloading

   

example:

    struct Student {
     int assi, mt, final;
     Student (int assi = 0; int mt = 0; int final = 0) { 
       this->assi = assi < 0? 0: assi; //default argument
       this->mt = mt < 0?0: mt;
       this->final = final < 0? 0: final;
       }
    
    }
    
    Student s0 {60,70}; // final = 0;
    Student s1 {60}; // mt = final = 0;
    Student s2 {}; // assi = mt = final = 0;
    Student s3 ;  // assi = mt = final = 0;
    Student s (); X

Every class come with a default constructor (a **zero parameter constructor**)

- free
- calls default ctors on any fields that are *objects*

example:

    Sturct A {
     int x; 
     Student y;
     Vec *z;
    }
    
    A myA;

- field x -- uninitialized (not an object)
- field y -- default ctors called
- field z -- uninitialized (pointer to objects)

The default ctor goes away as soon as you write any ctor.

    Struct Vec {
     int x;
     int y;
    }
    Vec v; // it works
    
    Struct Vec {
     int x; int y;
     Vec (int x, int y) {
      this->x = x;
      this->y = y;}
    }
    Vec v; // it does NOT work

## Initializing fields that are *const* or *reference* ##

    int m;
    struct MyStruct {
     const int x = 10;
     int & y = m;
    }

It is  in class initialization (can never be changed)

Field must be fully constructed before ctor body runs.

#### Steps that happens when an object is created: ####


- space is allocated
- field initialization: default ctor runs for fields that are objects
- ctor body runs


We can hijack step2 using **Member Initialization List (MIL)**  

    struct Student {
     const int id;
     int assi,mt,final;
     Student(int id, int assi, int mt, int final)
       : id{id}, assi{assi}, mt{mt}, final{final} {} //first id is this.id, second is paramater id
    };

A MIL can only be used in a ctor.

1. MIL is not restricted to reference & const
2. In  a MIL, you don't need to use `this` to distinguish field/parameter with same name
3. A MIL always initialized in *declaration orders*
4. Initialization by MIL is more efficient than by ctor body

Via MIL, the object will be constructed and initialized in one operation. Via assignment (construction body), the fields will be first initialized with default ctor and then reassigned (via assignment operator) with actual values.

const and reference

- these must be *initialized* with a valid value, i.e. `int &x`  is illegal
- once they are constructed, *reassigning* them is illegal.
- Thus, they can not be *assigned*, they can only be *initialized*.

-----

if fields are initialized in class & MIL, MIL take precedence.

    struct Vec {
     int x = 10;
     int y = 20;
     Vec (int x, int y): x {x}, y{y} {}
    };

---

## Copy Constructor ##

    Student billy {60,70,80};
    Student bobby = billy; // Student bobby {60,70,80};

Copy ctor is called to construct an object is a copy of another.

Every class come with a default copy ctor.

A class comes with 

- default ctor
- copy ctor
- destructor
- copy assignment operator
- move ctor
- move assignment operator


copy ctor:

    struct Student {
     int assi,mt,final;
     Student (const Student &other)   
       :assi{other.assi},mt{other.mt},final{other.final} {}
    };    
    //passing a const reference

It makes a copy. If billy changes, bobby doesn't change. (the int field)

    struct Node {
     int data;
     Node *next;
     Node (const Node &others):
      data{other.data},next{other.next}{}
    };
    
    Node *np = new Node{1,new Node{2, New Node{3,nullptr}}};
    Node m {*np}; // calls copy ctor
    Node *newp = new Node{*np}; // calls copy ctor again

It copies the address of next node, not the node. It is **shallow copy**.
![Shallow Copy](https://i.imgur.com/HLAzRgh.jpg)

We typically want a **deep copy** where an object is not continuous in memory.(i.e. object contains heap allocated memory)

    struct Node {
     Node (const Node &other):
      data{other.data},
      //next{new Node{*other.next}}{}    NEVER DO IT!
      next{other.next? new Node{*other.next}:nullptr}{}
    };

#### A copy ctor is called when ####

1. constructing an object of a copy of another 
2. passing by value
3. returning by value

Therefore, the parameter to a copy ctor is passing by *reference*. Passing by value will cause an infinite loop and keeping calling ctor.


## Constructors with one parameter ##

This will create an **implicit conversion** of data type on passing to functions.

    struct Node{
     int data;
     Node *next;
     Node(int data)
     : data{data}, next{nullptr} {}
    }
    Node n{4};
    Node n = 4; // LEGAL.

The compiler is smart enough to understand that we are passing thing to a
Node

    void foo(Node n) { ... }
    foo(4); // LEGAL.

Hence, single parameter ctors create implicit automatic conversions.

    string s = "hello";

The string class has a one parameter ctor with type `char*` //???

We can disallow implicit conversions by adding `explicit` keyword to the ctor

    struct Node{
     int data;
     Node *next;
     explicit Node(int data)
      : data{data}, next{nullptr} {}
     }

So, in this case

	Node n{123}; //compile
	Node n=123; // Won't compile
	foo(4); // Won't compile


# Lec10 Separate Compilation of Class, Big five (Continued)  10/12/2017 



## From before

- .h (header file) -> type definition & function headers

- .cc (implementation file) -> implementation of functions


*c++/classes/separate/node {.h/.cc}*

Node.h

    #ifndef NODE_H 
    # define NODE_H 
     struct Node {
       int data;
       Node *next;
       Node (int, Node *);   //constructor
       Node (const Node &);  //copy
       int getData(); //method
     }
    #endif

Node.cc

    #include "node.h"
     Node:: Node (int data, Node *next): data{data}, next {next} {}
     //if you don't put "Node::", complier will treat it as a function
     int Node:: getData() {return data;} 

*`::`      Scope resolution operation, before the method name*


## Destructors ##

**When is an object destroyed?**

- Stack allocated -when goes out of scope

- Heap allocated -when delete is called on a pointer to the object


Destructors are special methods that run when objects are destroyed.

Each class comes with a free destructor.

**Steps to destroy an object:**
(reversion of steps of ctor runs)

1. dtor body runs
2. dtors run for fields that are objects
        - reverse declaration of ctors
3. space is deallocated

**Default destructor** 

Default destructor calls dtors on fields that are objects

The default dtor may not do what we want

    Node *np = new Node {1, New Node {2, New Node {3, nullptr}}};
    
    delete np; // calls dtor on Node1
               // reclaim space of Node1
               // Node2 & Node3 leaks


    Node:: ~Node() { // destructor, modify step1
        delete next; // "delete" will call dtor
                     // It's always save to delete a nullptr, so no need using "if (next!=nullptr)"
    }
    // this recursively calls the dtor which ends up deallocating the entire list

A class can only have *one dtor*

- No overloading

## Copy Assignment Operator ##


    Student billy {60, 70 ,80};
    Student bobby {billy}; //copy ctor
    Student jane; // default ctor
    jane = billy; // copy assignmnet operator
      //operator = (jane, billy);  X
      //jane.operator = (billy);   V

The copy assignment operator is  used to update *an existing object* as a copy of another.

----------------------------

Sometimes the free assignment operator may not do what you want

- for the same reasons as the copy ctor.

-------------------

The assignment operator must always be implemented as a *method*.


​           

    Node &Node:: operator = (const Node &others) { // we don't want to make an unecessary copy of list, so &Node (return a reference)
      
      if (this == &others) return *this; // self assignment


      data = others.data;
      // we are updating an existing linked list, next may already be pointed  heap memory;
    
      delete next;
      next = other.next ? new Node {*other.next} : nullptr;
      return *this;
      }


1. pass a reference to const
2. return by reference
3. check if this == others
4. delete the memory allocated to this first


**An assignment is a statement and an expression.**

Expressions do "return" a value, though they may be cast to (void). Statements don't evaluate to anything and only have side effects.

If the operator is an expression / return lvalue, we should return reference to avoid unnecessary copy.

    a = b = c = 0;  // Method cascading, a copy is made each time
            --c--
        b =   0
        --b--
    a =   0
    -- a --
       0       

**Self Assignment**

    Node n {1, new Node {2, nullptr}};
    n = n; //n.operator = (n);

  Without self assignment check, call assignment operator will cause dangling pointer! 

-----

What if new fails because heap is full?

- `new` will throw an exception.

- `next` has been deleted but is not reassigned -> dangling pointer

- solution: delay the `delele` till after new has succeeded.

**Exception Safe Code**

    Node &Node::operator = (const Node &others) {
       if (this == &others) return *this;
       Node *temp = next;
       next = other.next? new Node {*other.next}: nullptr; // if new fails, it will exit the method 
       delete.temp;
       data = other.data;
       return *this;
    }

##Copy of Swap Idiom ##


    #include <utility> //for swap
    
    void Node::swap(Node &others) {
      using std::swap;
      swap(data,others.data);
      swap(next,others.next); 
    }
    
    Node &Node::Operator = (const Node &others){
      Node temp {others}; //deep copy ctor
      swap (temp);
      return *this;     
    }   //stack allocated memory will pop up automatically.

------

Most operators can be implemented as functions or methods.

**Method overloading** (instead of operator overloading)

    Struct Vec {
      int x;
      int y;
    };
    
    Vec Vec::operator+(const Vec &v) const { // not change the field of the object
      Vec toReturn {x+v.x, y+v.x};
      return toReturn;
    }
    
    v1.operator+(v2);
    // This method does not change the fields of the object on which it is called. This is a const method


​    

# Lec 11 Operators as methods, Big 5 conclusion #

cont'd

    vec v2 = v + v1;
    // Vec 2 = v.operator+(v1);
    
    const Vec v3 {5,6};
    //Had operator+ not been declared const, v3.operator(v2); would not compile
    //Const object can only call const method
    
    Vec Vec::operator*(const int k) const
    {
      return Vec{x*k,y*k};
    }
    Vec v4 = v3.operator*(5); 
             // v3*5; 
    
    Vec operator*(const int k, const &v) {
     return v*k;
    }
    // no need to put const here since it is NOT a member function
    
    Vec v5 = 5 * v4; 
    // 5.operator*(v4); this can NEVER be implemented

Can I/O operators be implemented as method?   
Yes we can, but no, we shouldn't

    ostream &Vec::operator<<(ostream &out)
    {
      out << x << " " << y;
      return out;
    }
    
    v << cout;

It's weird. Even worse, think about cascading.

    v1 << (v << cout);
          --- cout ---

-----------------

Following operators must be implemented as methods

operator = 

operator []  // array, A3Q2

operator ->  // treated it as a pointer to a object

operator ()  // pretend as function

    Vec v;
    v();

operator T() //???

------------

## Big 5 ##

Move ctor

Move assignment operator

*classes/rvalue/node.cc*

    #include <iostream>
    using namespace std;
    
    struct Node {
     int data;
     Node *next;
     Node (int data, Node *next): data{data}, next{next} {
      cout << "Basic ctor" << endl;
     }
     Node (const Node &other) : data{other.data},
      next{other.next ? new Node {*other.next} : nullptr} {
      cout << "Copy ctor" << endl;
     }
     ~Node() { delete next; }
    };
    
    Node plusOne(Node n) {
     for (Node *p {&n}; p ; p = p->next) {
     ++p->data; // syntac sugar :)
     }
     return n;
    }
    // be careful with passing by value, we use it here to demonstrate move ctor
    
    ostream &operator<<(ostream &out, const Node &n) {
     out << n.data;
     if (n.next) out << ' ' << *n.next;
     return out;
    }
    
    int main() {
     Node n {1, new Node {2, nullptr}};
     // 2 Basic ctor 
    
     Node n2 {plusOne(n)}; 
     // copying a node from a node
    
     // plusOne pass argument by value, it calls copy ctor(deep copy)     2 copt ctor
     // after for loop
            n: 2->3
     // plusOne return lvalue, not nvalue, copt ctor use nvalue
     // the complier needs to treat the return form plusOne as a lvalue
     // temp <- n      2 copy ctor
     // temp is rvalue. If copy ctor didn't use const, it won't compile
     // Node n2{temp}      2 copy ctor


     // 6 Copy ctor


     cout << n << endl << n2 << endl;
    }

Compile

    g++14 node.cc
    ./a.out

-> 2 basic ctor

-> 4 copy ctor

The compiler does **Elision (optimization)**

    g++14 -fno-elide-constructor node.cc    
    ./a.out  

-> 6 copy ctor

    Node n2{temp};

`temp` is a temporary created by compiler so that it can be used to construct `n2`

`temp` will be destroyed (call dtor) as soon as `n2` has been constructed

We would like to write a constructor that steals instead of  copying

  - but only from temporary objects


C++14  has the ability to refer to temporary values.

- rvalue --- temporary value

- rvalue reference -- reference to a temporary

- `Node &` -- lvalue reference

- `Node &&` -- rvalue reference

Now we can build our move ctor

![Move Ctor: Share vs Steal ](https://i.imgur.com/Uzlc7Cf.jpg)

    Node::Node (Node &&other) //other is temp
     : data {other.data}, next {other.next} {
       other.next = nullptr;  // steal, no share!
     }


## Move assignment operator ##

![Move Assignment: Swap](https://i.imgur.com/Qrpx012.jpg)

    Node m {1, new Node {2, new Node {3, nullptr}}};
    m = plusOne(m);
        --rvalue--
    
    Node &Node::operator=(Node &&other){
     swap (other);
     return *this;
    }

Or

	BinTree &operator=(BinTree &&other) {
	  if (this == &other) return *this;
	
	  //Destructor part
	  delete left;
	  delete right;
	
	  //Move ctor part
	  data = other.data;
	  left = other.left;
	  right = other.right;
	  updateParents();
	
	  //steal part, not sharing
	  other.left = other.right = nullptr;
	
	  return *this;
	}


Why don't use swap in Move ctor?

Because this.next is not initialized, it is garbage value, try to destroy it cause crash.

## Rule of 5 ##

If you find yourself implementing any one of copy ctor, cope operator = , dtor, move ctor, move operator = , then usually you need all of them.

Copy/Move Elision

- in certain solutions the compiler is  allowed (though not required) to omit calls to copy ctor or move ctor

c++/elision


# Lec 12 short topic: arrays of objects, Invariants of Encapsulation #

    struct Vec {
      int x,y;
      Vec (int x, int y): x{x},y{y} {}
    };
    
    Vec vecs[3]; //stack allocated array of Vec objects
    Vec *vec1 = new Vec[3]; //heap allocated array of Vec objects

It **won't compile** because default ctor - 0 parameter ctor is no longer there

To fix this problem

- provide a 0 parameter ctor (default ctor)

- For stack array, explicitly call the available ctor

  `Vec vecs[3] = {Vec{1,2}, Vec{2,3}, Vec{3,4}};`

- Create an array of ptrs to objects

  1. on stack, `Vec *vecs2[3]`
  2. `Vec **vec3 = new Vec*[3];`

  then, you can do

         vec3[0] = new Vec{1,2};
         .
         .
         for(int i = 0; i < sizeOfArray;++i)
           delete vec3[i];
         delete [] vec3;

------

###--------------------- continuing - c++ class --------------------------



end# Lec 12 short topic: arrays of objects, Invariants of Encapsulation #

cont'd

## Invariants  of Encapsulation ##

    sturct Node {
     int data;
     Node *next;
     Node(int data, Node *next):data{data},next{next}{}
     ~Node(){delete next;}
    }
    
    int main(){
    Node n {1,new Node{2,nullptr}};
    Node n2 {3,nullptr};
    Node n3 {4,&n2};
    }

program will crash!

 When deleting `n3`, we call delete on `&n2`, which is a *stack address*

### Invariants ###

**Invariants**: a statement/assumption that is supposed to be true for the proper functioning of  a class

i.e.

- Node invariant: next is either `nullptr` or points to the heap
- Stack invariant: Last in First Out

It is hard to reason about the correctness of a program if you cannot guarantee invariants.

### Encapsulation ###

- a class is a block box
- seal away/ hide implementation
- provide access through an exposed interface

c++ provides **visibility modifiers**

    struct Vec {
     Vec (int x, int y): x{x},y{y}; //default visivility is public
     private: 
        int x,y; //hidden to the outside world
     public:
       Vec operator+(const Vec &v){
        return {x+v.x,y+y.x};
       }
    }
      
    int main(){
     Vec v{1,2};
     Vec v1 = v + v;
             --rvalue-- //call copy ctor. By default, copy ctor is public.
     // cout << v.x << v.y;      NOT compile
    }

**Advice:**

- field should be private
- helper function should be private

C++ introduced a new keyword: **class keyword**

    Class Vec{ //default visibility is private
     int x,y;
     public:
       Vec(int x, int y)
       Vec operator+ ()
    };

- struct: default public 
- class: default private 



### Guaranteeing the Node Invariant ###

Create a wrapper class for  the Nodes of linked list

(Node invariant: next is either `nullptr` or points to the heap)

*list.h*

    Class List {
      struct Node; // private nested class
      Node *theList = nullptr;
     public:
      void addToFront(int n);
      int ith(int i);
      ~List();
    };
    
    // everything you get for free is public by default

*list.cc*

    struct List::Node {
     int data;  
     Node *next;
     Node(int data, Node *next) : ....
     ~Node(){delete next;}
    }


    List:: ~List() { delete theList;}
    
    List:: addToFront(int n) {
             theList = new Node{n, theList};
    }
    
    int List:: ith (int i) {
     //assumption ith element exists
     Node *current = theList;
     for (int j = 0; j < i && current; current=current->next,++j);
     return current->data;
    }

Using encapsulation, we have guaranteed the node invariant.

What is the cost?

- How do we now 'traverse/iterate the list'?
- What is the efficiency of *printing* the list?  **O(n^2)**

How do we provide **O(n)** traversal by still using encapsulation?

- We want someway which **abstracts a pointer** into our linked list

## Design Pattern ##

(4 guys wrote a book)

Previously defined programming problem with previous developed good solutions.

### Iterates Design Pattern ###

Create another "iterates" class that acts as an abstraction of  a ptr (`p`) into the list. We want to do something like:

    //array is an int array
    for (int *p = array; p!=array + arraySize;++p){
      ~~~~~~~~~~~~~*p~~~~~~~~~~~
    }

List iterates will contain 

- `!=`, prefix `++`, `*` unary
- a way to create an Iterate that indicated the state the list, and one indicate the end

Now,

    Class List {
     struct Node;       // No one outside List class can access node
     Node *theList =  nullptr;


     public:            // But we have access to iterator outside List
      class Iterator {
       Node *current;
    
       public:
        Iterator (Node *current): current{current}{}  
    
        int &operator*(){
         return current->data;
        }               
    	//return as reference, so that we can change the value
    
        Iterator &operator ++(){
         current = current->next;
         return *this; 
        }          
    	// return as reference, since we want update i (instead of a new value) & get updated value
    	// int i = 0; int n = ++i;    int x = i++;                
    	//               -1-  -1-        -1-  -2-
    
        bool operater==(const Iterator &others) const {
         return curr == other.curr;
        }
    
        bool operatre!=(const Iterator &others) const {
         return !(*this==others); //call operator==
        }
      }; //done Iterator Class


      Iterator begin(){
       return Iterator{theList};  // stack created var
      } //if return by reference, it will create a dangling pointer after stack pops up
    
      Iterator end(){
       return Iterator{nullptr};
      }
    }; //done List class

# Lecture 13: Iteration Design Pattern, Friendship, Static, UML#

cont'd.

    int main(){
     List l;
     l.addToFront(1);
     l.addToFront(2);
     l.addToFront(3);
     for(List::Iterator it = l.begin(); it!=l.end(); ++it){
      cout << *it << endl;
     } 
    }

In c++11, there is **automatic type deduction**

	auto x = y; //define x to be the same type as y
	auto it = l.begin(); // it is defined to be return type of begin()

c++11 also supports **Range-based for loops**

- built-in support for the Iteration Pattern

i.e.

    for (auto n : l) {  
     cout << n << endl;
    }
    // n is passing by value!!
    //changes to n do not effect the data in the linked list
    
    // In the for loop
    // n = 5; -> won't change n
    // ++n; -> change n (return by reference)

A range-based for loops can be used for a class, `MyClass` if

1. `MyClass` has `begin` & `end` methods that returns some iterator object
2. the iterator class must implement `!=`, prefix `++`, unary `*`

------

### end of midterm

--------

## Friendship ##

Motivation: Iterator ctor is public.

We would like it private => so that users use `begin()` & `end()` won't access it

However, if we made it private, then `begin` & `end` would not be able to call `Iterator()` either!! 


#### solution ####

- we make Iterator ctor private
- But then Iterator class says list is my friend
- a friend has access to all your private parts => don't make to many friends!

code:

    class List{
     struct Node;
     public:
     class Iterator {
      Iterator (Node *curr) ~~~~~~~~~~~ // this method is private
      public:
       ~~~~~~
       ~~~~~~
      friend class List;   // something new!
     };
    }


## Accessors (getters) /Mutator (Setters) ##

Advice: fields should be private

    class Vec{
     int x,y;
     public:
     int getX() cosnt {return x;}
     int getY() const {return y;}
     void setX(int x) {this->x = x;}
     void setY(int y) {this->y = y;}
    }

Suppose

- `Vec` has private field
- No accesser & mutator
- Do want output operator (a stand alone function)      
- Solution: class `Vec` can make output `operator<<` a friend

i.e.

    class Vec {
     int x, y;
     ~~~~~~~
     ~~~~~~~
     friend ostream &operator<<(ostream &, const Vec&);
    };
    
    ostream &operator<<(ostream &){
     out << v.x;
     return out;
    }

### Static keyword ##

A **static field** is associated with a class, but not any object of the class.

 - there is one memory location for each static field, not each object

i.e.

    class Student{
     public:
     static int numInstance;
    
     Student(int assi, int mt, int final):
      assi{assi},{
       ++numInstance;
      }
    }

Question: where do we initialize numInstance?

c++ Rule: *static field* must be initialized *external* to the file that declare it

student.cc

    int Student:: numInstance = 0

main.cc

	int main(){
	 Student billy{60,70,80};
	 Student bobby {billy};
	 cout << Student:: numInstance << endl;
	}

#### static member function

We can also have static member functions

- these functions can be called without  an  object of this class

student.cc

	static void Student::printNumInstance(){
	 cout << Student:: numInstance << endl;
	}

- static member functions do not have `this` pointer.
- static member functions can only access static field & call static member function

## System Modeling

Design before you implement:

1. what are the main class
2. what are the relationships between classes

### UML: Unified Modeling Language

Represent a class in UML

	----------------
	Vec
	----------------
	-x:Integer        good ideas to use general types rather than language specific
	-y:Integer
	----------------
	+getX():Integer
	+setX(Integer)
	----------------
	
	- private
	+ public

#### Relationship 1: Composition

	class Vec{
	 int x,y;
	 public:
	 Vec (int x, int y):x{x},y{y}{}
	};
	
	class Basic {
	 Vec v1, Vec v2;
	}
	
	Basic b; // won't compile

- b is being default constructed
- in step 2, v1 & v2 are default constructed -> NO `Vec()` available!

Solution:

- solution 1: implemented default Vec() in Vec
- solution 2: call non-default Vec ctor in the Basis MIL

i.e.

	class Basic{
	 Vec v1, v2;
	 public:
	 Basic():v1{0,1},v2{0,1}{}
	};
	
	Basic b; // will compile



# Lec14, System Modelling, UML, Relationship

Last time

	class Vec{
	 int x,y;
	 public:
	 Vec (int x, int y):x{x},y{y}{}
	};
	
	class Basic {
	 Vec v1, Vec v2;
	 public:
	 Basic():v1{0,1},v2{0,1}{}
	}
	
	Basic b; // will compile


Embedding objects of a class (`Vec`) within another class (Basis) is called **composition**

- Basic "OWNS A" Vec
- In A3Q3, Polynomial OWNS Rational (no need to put Vec ctor inside Basic)

A typically "owns A" B is that B has no identity outside A

- A is copied, means B is copied
- A is destroyed, B is destroyed

Graphic:

	---------               -------------  
	|basic	|				| Vec		|
	--------  	 	  v1,v2	|------------
	|		|◆------------>	|-x:integer	|
	|		|		    2	|-y:integer	|
	---------				-------------

Car OWNS A carparts

VS

Carparts in a catalog

We say a catalog HAS A carpart

## Relationship 2: Aggregation

A "HAS A" B is

- B has an identity of its own
- copying A does not copy B (maybe a shallow copy)
- destroying A does not destroy B

i.e.

	class Catalog {
	 Part *parts[100];
	}

Graphic:

	---------               -------------  
	|Catalog|				| Part		|
	--------  	 	  parts	|------------
	|		|◇------------->|			|
	|		|		   0..*	|			|
	---------				-------------

Indicator & meaning

	Indicator	Meaning
	0..*		Zero or more
	*	        Zero or more
	1..*		One or more
	3			Three only
	0..5		Zero to Five


OWN or HAS?

	class B {
	....
	}
	
	class A {
	B b; // OWNS
	B *b;  // it could be either HAS or OWNS
	}

cont'd





# Lec14, System Modelling, UML, Relationship #


### Relationship 1: Composition ###

### Relationship 2: Aggregation ###

### Relationship 3: Inheritance ###

![Imgur](https://i.imgur.com/UBOHTv9.png)

It is hard to create collections of those different types of Books

- array of `void*`
- use a `union` type 
  - `union [name]  { member-list };`
  - [Read more](https://msdn.microsoft.com/en-us/library/5dxy4b7b.aspx)

--------

Observe:

- A textbook **IS A** Book with a topic field
- A comic IS A type of "special" Book with a hero field

UML:

![Imgur](https://i.imgur.com/vrwcIF1.png)

Code:

	class Book{
	 string title,author;
	 int munPages;
	 public:
	 Book(string title, string author, int numPage):....MIL....
	};
	
	class Text : public Book {
	 string topic;
	 public:
	 ....
	};
	
	class comic : public Book{
	......
	};


Some terms of Book & Text:

- Base - Derived
- Superclass - Subclass
- Parent - Child

**Properties:**

- Derived class inherit all members from the Base class
- Anything you can do with a Book object, you can do it with a Text/Comic
- ( Anything you can do with cin (istream), you can do it with ifstream/istringstream )


### Inheriting Private members ###

You inherit everything 

- title, author, numPages are private in Book, but you still inherit them

Text objects have space the hold a title, author & numPages

- but since those fields were private , Text cannot access them


How do you even initialize a Text object?

Following WON'T compile:

	Text:: Text(string title, string author, int numPages, string topic)
		: title{title}, author{author}, numPages{numPages}, topic{topic}{}

**Problems:**

- title, author, numPages is **private** in Book
- **MIL** is only allowed to refer to **its own fields** (fields **it declare**
- Problems happen during the different steps for object construction

#### Steps that happens when an object is created: ####

1. space is allocated
2. **constructed the superclass part of the object (default ctor for superclass)**
3. subclass field initialization: default ctor runs for fields that are objects
4. ctor body runs

Step 2 will fail because Book does not have a default ctor

Let's call a *non-default ctor for Book* to hijack step 2

	Text:: Text(string title, string author, int numPages, string topic)
		: Book{title,author,numPages}, topic{topic}{}

## Protected Visibility ##

Protected members are accessible by the class & its subclasses
	

	class Book{
		protected:   
		string title,author;
		int munPages;
		public:
		....
	};
	
	class Text: public Book {
		...
		public:
		void addAuthor(string auth){
		 author += auth; //can access inherited field as it is protected
		}
	};
	
	int main(){
		Text t{.....};
		// t.author = .....;    won't compile
		t.addAuthor(....);  // works
	}

Private is better than protected

- comes down to *invariant*
- subclass have full access to anything that is protected -> cannot guarantee variant anymore
  (you can do sth bad by create a subclass and modify the field)

#### Advice: ####

*Keep field private, write protected accessors/mutators/methods*

	class Book{
		string title, author;
		int numPages;
		protected:
		void addAuthor(string auth){
			if (!(something bad))
				author += auth;
		}
	}


## Method Overriding ##

Different types of Books have different measures to consider it heavy: 

book: numPages > 200 pages | Text: > 500 pages | Comic: > 30 pages

	class Book{
	......
		public:
		int getNumPages() const {
			return numPages;
		}
		
		bool isHeavy() const{
			return getNumPages() > 200;
		}
	};
	
	class comic : public Book{
	....
		public:
		bool isHeavy() const{
			return getNumPages() > 30;
		}   // replace the implementation in Book by overriding
	};
	
	Comic cb{..,..,40,..};
	cd.isHeavy(); 
	//Comic:isHeavy() runs -> true
	
	Book b = Comic{..,..,40,"Batman"};
	// legal as a Comic IS A Book
	b.isHeavy(); 
	// Book: isHeavy() runs -> false


Placing  a subclass object in the space for a superclass object cause **object slicing**

-------------------------------

# Lec 15, Method Overriding, Virtual, Pure Virtual, Abstract #

Last time:

- Subclass inherit *all* members from the Superclass
- Sometimes a subclass might want to replace the inherit behavior (method overriding)


### Different criteria for determining whether a Book `isHeavy` ###

	class Book{
		protected:
		int numPages;
		public:
		bool isHeavy() const {
			return numPages > 200;
		}
	};
	
	class Comic: public Book{
		public:
		bool isHeavy() const {
			return numPages > 30;
		}
	};
	
	Book b{...,...,190};
	b.isHeavy(); //Book::isHeavy() called, return false
	
	Comic c{...,...,40,...};
	c.isHeavy(); //Comic:isHeavy() called, return true
	
	Book b = Comic{...,...,40,...}; //it is legal because a Comic is a Book
		// it calls Book:: copy ctor
	b.isHeavy(); //Book::isHeavy() called, return false

- Subclass objects are **never smaller** than Superclass objects.
- When a subclass is placed in a space for a superclass object, the object is **sliced**

### Using Superclass pointless ###

	Comic c{...,...,40,...};
	Comic *cp{&c};
	cp->isHeavy(); // Comic:: isHeavy()
	Book *bp {&c}; // legal
	bp->isHeavy(); // Book:: siHeavy()

![Demonstration](https://i.imgur.com/z9VBN7K.jpg)

By default, a c++ class compiler looks at the **declared type** of a pointer / reference to determine which method to call (not the type of object it points/refers to)

It is called **Static Dispatch** in compilation

	Book *collection[20]; // it is legal but they are all types of book

------

We would like to call methods superclass pointer, and Comic is the most "specialized" method to executed.

In c++, we can use the `virtual` keyword

	class Book {
	~~~~~~~~
	public:
	virtual bool isHeavy() const {~~~}
	};
	
	class Comic : public Book{
	public:
	bool isHeavy() const override{~~~}  
		// the method in the subclass is virtual automatically 
		// override is introduced in c++11, it will check superclass for the method
	};

- if you forget to put both `override` and `const`, it is method *OVERLOADING instead of method OVERRIDING*
- if you forget to put `override` and make a typo (i.e. isheavy), there are 2 methods in Comic and it is so dangerous!

usage:

	Comic c{...,...,40,...};
	Comic *cp{&c};
	Book *bp {&c}; 
	Book &r{c};
	
	cp->isHeavy(); // Comic:: isHeavy()
	bp->isHeavy(); // Comic:: isHeavy()
	r.isHeavy()// Comic:: isHeavy()

- For a `virtual` method, the decision on which method is called is made at runtime based on the *runtime type* of the object.
- `Virtual` calls do have a *runtime cost*.

It is **Dynamic Dispatch** in compilation (it is Java's default behaviour)

	Book *collection[20];
	for (int i = 0; i < 20; ++i){
		cout << collection[i]->isHeavy();
	}
	
	// if you forget `*`, it will calls method in Book instead of the most specialized method

Collection is a **Polymorphic array** (

Polymorphism(many forms): The ability to accommodate multiple types under a single abstraction

## Destructors in the presence of inheritance ##

*c++/inheritance/example5*

	class X {
	  int *x;
	 public:
	  X(int n): x{new int [n]} {}
	  //~X() { delete [] x; }
	  virtual ~X() { delete [] x; }
	};
	
	class Y: public X {
	  int *y;
	 public:
	  Y(int n, int m): X{n}, y{new int [m]} {} // X{n} hijack step2 in ctor
	  ~Y() { delete [] y; }
	};


Destruction is the inverse of construction

1. dtor body runs
2. subclass field destroyed: dtors run for fields that are objects
3. destructed the superclass part of the object (**dtor of superclass run**)
4. space is reclaimed

A subclass dtors will **always**, **automatically** call the superclass dtors.

	// Run with valgrind
	int main () {
	  X *xp = new Y{5, 10};
	  delete xp;  
	   //if dtor it is not virtual, will call just ~X() -> memory leaks for field y
	}

### Advice: 

- If a class may have children now as in the future, it should **make its dtor `virtual` (avoid memory leak)**
- If you want an abstract class but you don't have a good P.V. method, make dtor P.V.
- **All dtor must be implemented**, even P.V. dtor
  - Subclass need to call superclass dtor


### `final`

- a virtual function cannot be overridden in a derived class
- Or, a derived class cannot be inherited from.

If a subclass must not be "superclass", then declare is `final`

	class Y final: public X{
		~~~~~~~~~~
	}
	
	class Z : public Y {~~~}  //won't compile!


## Contextual keywords ##

c++11 has "contextual keywords"

- names that act as keywords in certain place

i.e. `= 0`
	

	class Student{
	~~~~~~~~
	public:
		//virtual int fee();
		virtual ine fee() = 0; //Student:: fee() is a PURE Virtual method
	};
	
	class Regular: public Student{
	~~~~~~~~~
	public:
		int fee() override {~~~~}
	};


	class Coop: public Student{
	~~~~~~~~~
	public:
		int fee() override {~~~~}
	};

We would like `Student::fee()` to be UN-implemented, but the linkers would give an error

Solution: make the method **Pure virtual**

# Lecture 16: Pure Virtual, Templates, Exceptions #

	class Student{
		virtual int fee() = 0;
	};
	class Regular: public Student{
		int fee(){~~~}
	};
	class Coop: public Student{
		int fee(){~~~~}
	};

### Pure Virtual Method ###

- method without an implementation

- you CANNOT create objects of type `Student`

  `Student s; //won't compile`

- A class with at least one P.V. Method is an **abstract class**

- Subclass of abstract classed must implement all inherited P.V. method to be considered **concrete**

- P.V. method does not need to have a input (typically)

**Why create an abstract class?**

- Put the common fields/methods
- Polymorphism

**Concrete classes**: any class without pure virtual method 

By default, if you inherit an abstract class, you are also abstract. Unless you implement all P.V. methods of base class

**UML Notes**

- Virtual / Pure Virtual --- *italics*
- Abstract class --- *class name in italics*
- static --- underline   (i.e. `numInstance`)
- protected --- #

## C++ templates ##

What if we want different types of contents?

**C++ template class** are parameterized on type variables.

	template <typename T>
	class Stack {
		T *contents;
		int size;
		int cap;
	public:
		Stack();
		void push (T x) {~~~}
		void pop () {~~~}
		T top() const {~~~}
	};

The types are provided when creating objects of that template class

	Stack<int> sInts;
	sInts.push(5);
	
	Stack<string> sStrings;
	sStrings.push("hello");

you can put several typenames

	template <typename T1, typename T2, ....>

[Template inheritance](https://blog.feabhas.com/2014/06/template-inheritance/)


*list*

	template <typename T> class List {
	
		struct Node {
			T data;
			Node *next;		
		};
	
		Node *theList = nullptr;
	
	public:
		class Iterator {
			Node *curr;
			Iterator(Node *curr) ~~~~~~~~~~~~~
			public:
			T &operator*() {~~~~~~~~~~~}
			~~~~~~~~
			friend class List<T>;  // only type <T> is a friend
		};
	
		T ith(int i){~~~~~~~~~~~~~}
		void addToFront(T &n){~~~~~~} // since we don't know the type of n, we don't want to make a copy of it if it is big
	};


​	

	List<int> l1;
	int x = 5;
	l1.addToFront(x);
	
	List<List<int>> l2;
	l2.addToFront(l1);
	
	for (List<int>::Iterator it = l1.begin(); it != l1.end(); ++it){
		~~~~~~
	}
	
	for (auto n: l2){
		for (auto m: n){
			cout << m << endl;
		}
	}


## Standard Template Library (STL) ##

### vector ###

 This a template class which automatically re-sizes a **heap allocated array** as needed (dynamic length arrays)


	#include <vector>
	using namespace std;
	.
	.
	.
	vector<int>  v{4,5};   // [4,5]
	
	// you don't know the size/cap, you only know the content
		
	v.emplace_back(6); // [4,5,6]
	v.emplace_back(7); // [4,5,6,7]
	v.pop_back(); //remove last element
	
	vector<int> v1(4,5); // [5,5,5,5]
	vector<int> v2; // empty vector
	
	for (int i = 0; i < v.size(); ++i){
		cout << v[i] << endl;
	}
	// i is unchecked, if i is out of range, behaviour is undefinde
	
	for (vector<int>::Iterator it = v.begin(); it != v.end(); ++it){
		cout << *it << endl;
	}
	
	for (auto n: v){
		cout << n << endl;
	}
	
	for(vector<int>::reverse-iterator it = v.rbegin(); it !=v.rend(); ++it) 
	// rend(): the one before the first element
	// you cannot using auto for reverse iterator!!!!


Many STL classes have methods that expect Iterator as a parameter

	v.erase(v.begin()); // all other element is shifted left
						// each pointer may not point to what it originally point to

When you make changes to a vector, internal element might move

- this means all existing **iterators** becomes invalid
- `erase` return a new Iterator that points to the new location of the element that followed the last element erased by the function call. This is the container end `.end()` if the operation erased the last element in the sequence.

i.e
	

	it = v.erase(v.begin()+3); //remove the 4th element
	v.erase(v.end()-1); //mimic the behaviour of popping

### Vector Tutorial ###

-------

- Purpose

  - Syntax
    - Note

- ---

- Create vertor of type T	

  - `vector<T> v;`		
    - calls the default ctor of vector

- Put element at the back	

  - `v.emplace_back(x);`
    - puts a **copy** of x

- Iterator				

  - `for (int i=0;i < v.size();++i)`
    - array style
  - `for (vector<T>:Iterator it = v.begin();it != v.end();++it)`
  - `for (auto n: v)`
    - iterator

- Access the ith element

  - `v[i]`
    - Unchecked (doesn't check if it is out range)
  - `v.at(i)`
    - Checked (it if is not in the range, an exception raised)

## Exception ##

`v[i]` unchecked access

if i not in range. behavior undefined

`v.at(i)` checked access

If i is NOT in range, vector::at will throw an out-of-range exception

	mainline code			library code
	
	vector<int> v;		|
	~					|
	~					|
	v.at(i);			|  // but i is not in range
						|  // it throws an exception

The one that can detect and solve the problem is not the same one


*c++/exceptions/rangeError.cc*

By default, if an exception is not caught, program terminates

*c++/exceptions/rangeErrorCaught.cc*

	try {~~~~~}
	catch (~~~~){~~~~~~}
	catch (~~~~){~~~~~~}
	
	// at least catch will be provided, but only one will be executed

# Lec 17 Exceptions, Design Patterns #

*callchain.cc*

	void f () {
	  cout << "Start f" << endl;
	  throw (out_of_range("f"));  // stack unwinding
	  cout << "Finish f" << endl;   
	}
	
	void g() {
	  cout << "Start g" << endl;    f ();    cout << "Finish g" << endl;
	}
	
	void h() {
	  cout << "Start h" << endl;	g ();    cout << "Finish h" << endl;
	}
	
	int main () {
	  cout << "Start main" << endl;
	  try {
	    h();
	  }
	  catch (out_of_range) { cerr << "Range error" << endl; }
	  cout << "Finish main" << endl;
	}

output:

	Start main
	Start h
	Start g
	Start f
	Range error
	Finish main

**Stack unwinding** (popping up function) in stack frame: in search of a handler for a  particular exception

A scope is exited by: 

- using return XXX
- reaching the end of the scope 
- throwing an exception

-------

	throw out-of-range{"bla"}; // constructor call

Error recovering can be done in stage

	try {
		...
	} catch (SomeExp S) {
		// do portial recovery
		// want to throw an exception
		throw SomeOtherExp{~~~~~~~~~};
		// Or
		throw S;
		// Or
		throw;
	}

- you can't recover from a serious error, so you signal error and quit
- you try to make application work without the code that raised an exception
- you try to fix an error, and call the code that **thrown an exception again**

`throw s;` 

- if  the caught exception was actually a sub type of SomeExp, the exception object is sliced
- we are throwing the sliced object

vs 

`throw;`

- throws the original exception that was caught

### C++ exceptions ###

- All c++ library exception inherit for the `exception` class
- user defined exception do NOT have to inherit from `exception`
- This requires a special syntax for catching *all* kinds of exceptions

syntac `...`

	try {
		~~~~~~~~
	} catch (...) {
		~~~~~~~~
		~~~~~~~~
	}

**C++ allows throwing anything: int, bool, etc**

*recfact.cc* vs *exfact.cc*

*recfib.cc* vs *exfib.cc*

	void fact(int n) {
	  if (n == 0) throw 1;
	  try {
	    fact(n-1);    
	  }
	  catch (int m) {
	    throw (n * m);
	  }
	}
	
	int main() {
	  int n;
	  while (cin >> n) {
	    try {
	      fact(n);
	    }
	    catch (int m) {
	      cout << m << endl;
	    }
	  }
	}


Exceptions are meant to be exceptional.

**Good Practice:**

Create your own exception classes as reuse ones in the c++ library

	class BadInput{};
		~~~~~~~~
		int n;
		try {
			if (!(cin>>n)) throw BadInput{};
		} catch (BadInput &) { 
			cerr << "Bad Input. Using default" << endl;
			n = 0;
		}

We caught exception by reference `&`

- avoid making extra copy
- **No slicing occurs** (as a reference is simply a pointer)

**Advice:** catch exceptions by reference

`bad_alloc` : new fails

`length_error` : attempt to resize vector fails

### exception thrown by destructor ###

By default, if a dtor throws an exception, the program terminated right away (`std::terminate` is called)

- no option to do error recovering

If you want to, you can change the default, 

	~Node() noexcept(false) {
		~~~~~~~~
	}
	// DON'T DO THIS!

- Suppose the stack is unwinding by an exception
- Suppose a dtor is executing, the dtor throws an exception
- we now have 2 live exception
  - program will terminate (`std::terminate`)
- Make sure dtors do not throw exception



## Design Pattern ##

### Observer Pattern ###

- Publish/Subscribe system 
- Publish/Subject - generates data
- Subscribe/Observer - interested in the data

General OO design Strategy

- Create abstract base class to provide the interface
- Use ptrs of this Base class to dynamically call methods
  - **subclasses can be supposed in/out to change behaviour**

![Imgur](https://i.imgur.com/e2IOHIu.jpg)

- Note that the `Subject` class does not actually generate any data
- `Subject` simply provides functionality common in all subjects
- It should be an abstract class
- We need a P.V. method

Convention: when there is no Pure Virtual methods, but we want the class Abstract

- make the destructor Pure Virtual
  - subclass already have a built-in dtor 
  - a parent class must always implement a dtor
    - after all, subclass dtor automatically call the parent ator
    - So let's implement it!
- A pure virtual method is allowed to not have a implementation.
- A pure virtual method must be implemented by a subclass for it to be concrete.

*SE/Observer*

1. Subject’s state is updated()
2. Subject::notifyObservers() -> calls each observer’s notify();
3. Each observer calls ConcreteSubject::getState() to react accordingly

*subject.h*

	#include <vector>
	#include "observer.h"
	
	class Subject {
	  std::vector<Observer*> observers;
	
	 public:
	  Subject();
	  void attach(Observer *o);  // emplace_back(o)
	  void detach(Observer *o);  // if (*it == o) {observers.erase(it);}
	  void notifyObservers(); // for (auto ob : observers) ob->notify();
	  virtual ~Subject()=0;
	};

*observer.h*

	class Observer {
	 public:
	  virtual void notify() = 0;
	  virtual ~Observer(); // Make dtor Virtual / P.V., so that we don't have memory leak
	};


*horserace.h*


	class HorseRace: public Subject {
	  std::fstream in;
	  std::string lastWinner;
	
	 public:
	  HorseRace(std::string source);
	  ~HorseRace();
	
	  bool runRace(); // Returns true if a race (read in) was successfully run.
	
	  std::string getState(); // Return lastWinner
	};

*bettor.h*

	class Bettor: public Observer {
	  HorseRace *subject;
	  const std::string name;
	  const std::string myHorse;
	
	 public:
	  Bettor(HorseRace *hr, std::string name, std::string horse);
	  void notify() override;   // call subject->getState();
	  ~Bettor();
	};


*main.cc*

	int main(int argc, char **argv) {
	  string raceData = "race.txt";
	  if (argc > 1) {
	   raceData = argv[1];
	  }
	
	  HorseRace hr{raceData};
	
	  Bettor Larry{&hr, "Larry", "RunsLikeACow"};
	  Bettor Moe{&hr, "Moe", "Molasses"};
	  Bettor Curly{&hr, "Curly", "TurtlePower"};
	
	  int count = 0;
	  Bettor *Shemp;
	
	  while(hr.runRace()) {
	    if (count == 2)
	      Shemp = new Bettor{&hr, "Shemp", "GreasedLightning"};
	    if (count == 5) delete Shemp;
	    hr.notifyObservers();
	    ++count;
	  }
	}	

Output:

	Winner: Molasses
	Larry loses.
	Moe wins!
	Curly loses.
	Winner: RunsLikeACow
	Larry wins!
	Moe loses.
	Curly loses.
	// etc.

# Lec 18 Observer Pattern, Decorator Pattern, C++ casting  #

## Observer Pattern ##

Horse Racing

Horse Race -> Subject

Bettor -> Observer


	// A4q4: each grid is both a subject and an observer


## Decorator Pattern ##

Suppose we have an existing object

- add features / functionality to the object


![Imgur](https://i.imgur.com/10g6iCM.jpg)

- Decorator IS A component
- Decorator HAS A component

code:

	AbsWindow *w = new BasicWindow;
	w = new Scroll(w);
	w = new Menu(w);
	
	// AbsWindow *w = new Menu (new Scroll (new BasicWindow));

- Do I need make decorator a abstract class?
- Yes. When there is in common, you should put them into a abstract class

*SE/Decoration*

*component*

	class Pizza {
	 public:
	  virtual float price() = 0;
	  virtual std::string description() = 0;
	  virtual ~Pizza() = default;
	};

*concrete component*

	class CrustAndSauce: public Pizza {
	 public:
	  float price() override  { return 5.99; };
	  string description() override  { return "Pizza"; };
	};

*decoration*

	class Decorator: public Pizza {
	 protected:
	  Pizza *component;
	 public:
	  Decorator(Pizza *component) : component{component} {};
	  virtual ~Decorator() { delete component; };
	};

*concrete decoration*

	class Topping: public Decorator {
	  string theTopping;
	  const float thePrice;
	 public:
	  Topping(std::string topping, Pizza *component) :
		 Decorator{component}, theTopping{topping}, thePrice{0.75} {};
	  float price() override { return component->price() + thePrice; };
	  string description() override  {
		 return component->description() + " with " + theTopping;
	  }
	};


## c++ cast ##

cont'd on next lecture



# Lec19 c++ cast, big 5 revisited #

## C++ Casting ##

C Style Casts:

	Node n;
	int *p =  (int*) &n;
			//c style cast
	
	// the behaviour is based on complier (put what field on top, how it manage memory)

C++ style Casts: 4 types

- these are implemented as template function

### static_cast ###

"sensible" casts where the behaviour is well defined

	void foo(int x) {~~~~~~~}
	void foo(donble x){~~~~~~~}
	double a = ~~~~~~~~~
	foo(a); // foo(double) is called

What if  i want to call `foo(int)` using `a`?

	foo (static_cast<int>(a));  // foo(int) is called

--------

	Book *bp = new Text{~~~~~~~~~~~};
		// suppose Text implements getTopic() method
	bp->getTopic(); // won't compile

We need a `text` pointer. 

If we are sure that `bp` does indeed point to a `text` object, we can use `static_cast`


	Text *tp = static_cast<Text*>(bp);
	                  // -- IS A --
	tp->getTopic();

If your assumption is not correct, the code still compiles even executes. The behavior is undefined.

### reinterpret_cast ###

- anything & everything is allowed
- "weird" conversion 

i.e.

	Student s;
	Turtle *t = reinterpret_cast<Turtle*>(&s);

Reinterpret cast relies on low-leveled compiler specific knowledge of how objects are organized in memory (c style)

It can modify the private field of another class

### const_cast ###

Cast to remove / add `const` from a value

	void g (int *p){~~~~~~~~~~~~}
	~~~~
	const int *q = ~~~~~~; // cannot use q to change *q
	// g(q); // won't compile
	
	g(const_cast<int *>(q)); // will compile

### dynamic_cast ###

Safely converts pointers and references to classes up, down, and sideways along the inheritance hierarchy.

	vector<Book *> books;
	~~~~~~~
	~~~~~~~
	Book *bp = books[i];
	// We want to downcast polymorphic type.

Incorrect:
	

	Text *tb = static_cast<Text*>(bp);
	tp->getTipic();
	// the behaviour is undefined if bp is not pointing a Text object

We can use **dynamic_cast** to tentatively try the type conversion & see if it succeeds. (return `nullptr`  if fails)

	Text *tp = dynamic_cast<Text*>(bp);
	// if it succeeds, then tp is a valid ptr to the Text object
	// if it fails, then tp is set to nullptr
	
	if(tp) cout << tp->getTopic();
	else cout << "Not a Textbook";

Requirement:

To use dynamic_cast, the class hierarchy must have **at least one virtual method**

- thats okay as we make dtors virtual

---------------

#### RTTI (Runtime Type Information) ###

	void whatIsIt(Book *bp){
		if (dynamic_cast<Text *>(bp))
			cout << "A Text";
		else if (dynamic_cast<Comic *>(bp))
			cout << "A Comic";
		else 
			cout << "A Book";
	}

This code is **highly coupled** to the class hierarchy. NEVER DO IT!

A better approach is to use **virtual method**.

----------------------------

#### dynamic_cast works on references

	Book &rb = ~~~~~~~~;
	Text &tr = dynamic_cast<Text &>(rb);

If `rb` is actually a reference to Text, then `tr` is a valid reference to this object.

What if it is not?

- cannot assign a reference to be nullptr
- `bad_cast` exception is thrown

## Big 5 revisted (inheritance) ##

### dtor ###

- always make the dtor in Base class virtual
- always implement dtor in Base class

### copy ctor ###

	class Text: public Book {
		~~~~~~~~~
	};
	
	Text a{~~~~~};
	Text b = a; // copy ctor is called

The free copy ctor for Text is called.

- first calls the free copy ctor for `Book`
- construct the subclass field (topic)

How does it look like?

	Text::Text(const Text &other):
		Book{other}, topic{other.topic} {}

### Assignment operator ###

	Text &Text::operator=(const Text &other){
		Book::operator=(other);
		// this->Book::operator=(other);
		topic = other.topic;
		return *this;
	}

### Move ctor ###

	Text::Text(Text &&other):
		Book{other}, topic{other.topic} {}

- other is an rvalue reference
- other is a variable, therefore making it a lvalue
- It is not efficient when passing by parameter, you call the copy ctor
- We need a way to say that treat other as if it were an rvalue
- The `std::library` provide `std::move`, a function that treats other as an rvalue

Correct Version:

	Text::Text(Text &&other):
		Book{std::move(other)},
		topic{std::move(other.topic)} {}

### Assignment Move Operator ###

	Text &Text::operator= (Text &&other) {
		Book::operator=(std::move(other));
		topic = std::move(other.topic);
		return *this;
	}

NOTE: There are not deep copy in the above example

------------------------------------

### Assignment Operator in detail ###

	Text b1{"B1", "Nomair", 200, "CS"};
	Text b2{"B2", "Brad", 100, "Physics"};
	Book *pb1 = &b1;
	Book *pb2 = &b2;
	*pb1 = *pb2; // object assignment

- The complier looks at the declared type of `pb1` and decides to call `Book::operator=`  (static dispatch)
- Only the `Book` part is assigned. 
- `b1` becomes `"B2", "Brad", 100, "CS"`

**--> Partial Assignment Problem** 

Two ways to fix this:

Approach 1:

- Lets prevent assignment through base class ptr
  - Make `Book::operator=` private? Subclass won't have access either
  - Make `Book::operator=` protected? This prevent actual book from ever being assigned
    - Advice: Base class should be *abstract*

![Imgur](https://i.imgur.com/mfruMS5.jpg?1)


	class AbstractBook {
		string title, author;
		int numPage;
		protected:
		AbstractBook &operator= (const AbstractBook &other) {~~~~~}
		public:
		virtual: ~AbstractBook() = 0; // make class abstact, need implemented
	}
	
	Text a{~~~};
	Text b{~~~};
	AbstactBook *pa = &a;
	AbstractBook *pb = &b;
	*pa = *pb;  // won't compile as AbstractBook::operator= is protected



Approach 2:

- Let's make `operator=` virtual

code:

	class Book{
		~~~~~~~~
		public:
		virtual Book &operator=(const Book &other) {~~~~~~}
	};
	
	class Text: public Book {
		public:
		Text &operator= (const Text &other) override {~~~~~~} // won't compile!!
	};

It is not a valid override!

- to be a valid override, the parameter type must be same (no requirement on return type)
- but `Text &operator= (const Book &other) override` allows the rhs to be any type of Book, not just Text
- Mixed assignment problem
- Let's allow assignment only when the rhs is a Text

i.e.

	Text &Text::operator = (const Book &other) override {
		Text &tother = dynamic_cast<Text &>(other);
		Book:operator=(tother);	
		topic=tother.topic;
		return *this;
	}
	
	Text a{~~~~~};
	Text b{~~~~~};
	Comic c{~~~~~};
	Book *pa = &a;
	Book *pb = &b;
	Book *pc = &c;
	
	a = c; // compile, throw bad_cast exception
	*pa = *pc; // compile, bad_cast 
	a = b; // succeeded (checking for run type instead of static type)
	*pa = *pb; // succeeded


# Lec 20 Design Problem #

## Iterator: Design Pattern (Revisited) ##

Abstract Iterator

	template<typename T> class AbsIterator {
	public:
		virtual AbsIterator &operator++() = 0; // PV method
		virtual T &operator*() const = 0; // PV method
		virtual bool operator==(const AbsIterator<T> &other) const = 0;
		bool operator!=(const AbsIterator<T> &other) const {
			return!(*this == other);
		}
		virtual ~AbsIterator() = default; 
		// use the default compiler given version of the big 5 method
	}

List Iterator

	class List{
		struct Node;	
		Node *theList = nullptr;
	public:
		class Iterator: public AbsIterator<int>{
			Node *curr;
			~~
			public:
			Iterator &operator++() override {~~} // it's save to return a Iterator, because it is a AbsIter
			int &operator++() override {~~}
			bool operator==(const AbsIterator<T> &other) const override{
				const Iterator &it = dynamic_cast<const Iterator &>(other);
				return curr == it.curr;
		}; //end Iterator
	~~~~~~~~
	~~~~~~~~			
	}; //end List

Set Iterator

	class set {
	~~~~~~
	public:
		class Iterator: public AbsIterator {
		~~~~~~~~
		};
	};

- We can now code that iterates over different data structure but using operations declared by the `AbsIter` class

example:

	template <typename Fn>
	void foreach(AbsIter &start, const AbsIter &end, Fn f) {
		while (start != end){
			f(*start);
			++start;    // since you are updating start, you can not pass it by const
		}
	}
	
	void addFive(int &x) { x = x + 5; }
	
	List l;
	~~~~~
	~~~~~
	List::Iterator begin = l.begin(); // pass begin by lvalue since we want to modify it
	foreach(begin, l.end(), addFive); 

Why use absIterator? - Once `foreach` is implemented, you can use it for all class

## Factory Method Pattern ##

	Player *p = ~~;
	Level *l = ~~;
	Enemy *e = nullptr;
	while(=->notDead()){
		// generate enemy
		e = l->createEnemy();
		// attack player
	}

Since the type of enemy to generate depends on what level we are at, it make sense that concrete level classes generate enemy

	class Level{
	public:
		virtual Enemy* createEnemy() = 0;
	};
	
	class Normal : public Level{
	public:
		Enemy *createEnemy(){ // more turtle }	
	};
	
	Class Enemy{
		// Level is my friend
	}

Benefit:

- if you want a new level, you only need to create a new concrete level class, without changing the main game while loop (it is easier to change the game)

#### It is also called **Virtual Constructor Pattern** ####

- make the ctor private
- use factory method to *restrict* the kind of objects that are create
  - `addToFront` is a factory of Nodes
  - `begin` / `end` are Iterator factories

![Imgur](https://i.imgur.com/zMXm1b2.jpg)

## Template Method Pattern ##

- Subclass are meant to provide implementation for some methods
- Other methods must remain the same

i.e.

	class Turtle{
	public:
		void draw() {	// non-virtual method
		 drawHead();	// template
		 drawShell();   // template
		 drawFeet();	// template
		} 
	private:
		void drawHead() {~~~~}
		void drawFeet() {~~~~}
		virtual void drawShell() = 0;
	};
	
	class RealTurtle: public Turtle {
		void drawShell() {~~~~~}
	}

- subclass cannot draw own head/feet even if they implement it
- we cannot draw only one part of the turtle

This is an example of the **NVI Idiom (Non-virtual Interface)**

Consider any public virtual method (Interface are public)

- public: part of the interface  -> pre/post conditions
- virtual: invitation to subclasses to change the behaviour
- there is contradiction

NVI Idiom

- all public methods must be non-virtual
- all virtual methods must be private (expect for the dtor)
  - you cannot stack allocated an object if you make the dtor private, because dtor is supposed to automatically called when out of stack

Digital Media Interface Without NVI

	class DigitalMedia{
	public:
		virtual void play()  = 0;
		virtual ~DigitalMedia();
	}

With NVI

	class DigitalMedia{
		publc:
		void play() { 
			copyright();
			playAd();
			doPlay();
		}	
		private:
		virtual void doPlay() = 0;
	};
	
	// music player
	// video player

Without NVI 

- it is difficult to check copyright / play Ad without change the public interface.
- You have to go through the concrete class

## Map Pattern ##

	#include <map>
	
	std::map<string, int> m;
	m["abc"] = 123;	
	m["def"] = -123;
	cout << m["def"] << endl;
	
	m.erase("abc");
	
	// if ( m["xyz"] == 0 )
	// if the key is not found, the key-value pair is added 
	if (m.count("abc")) 
	return 0;  // if not found
	return 1;  // if found
	
	for (auto &p : m){
		cout << p.first << p.second ;
			// key			// value
	}
		// p :  std::pair<string,int>	
		// struct
		// field are public
	
	Order of iterator is sorted key orders


# Lec 21 Visitor Pattern, Compilation Dependencies #

![Imgur](https://i.imgur.com/AEO0t99.png)

	Player *p = ~~;
	Level *l = ~~;
	Enemy *e = nullptr;
	
	virtual void Enemy::strike (Rock &) = 0;
	virtual void Enemy::strike (Stick &) = 0;
	
	Enemy *e = l->createEnemy();
	Weapon *w = p->chooseWeapon();
	
	e->strike(*w);
	// the code won't compile since compiler does not know Weapon's actual type

- c++ does not dynamically dispatch on argument types. (type of `*w` is unknown here)

## Visitor Design Pattern ##

Combination of method **overriding** & method **overloading**


	class Enemy {
	public:
		virtual void strike(Weapon &) = 0;
	};
	
	class Turtle: public Enemy {
	public:
		void strike(Weapon &w) override {
			w.useOn(*this); 
			// it this code got executed, the compiler would know *this is a  Turtle,certainly
		}
	};
	
	class Bullet: public Enemy {
	public:
		void strike(Weapon &w) override {
			w.useOn(*this);
		}		
	};
	
	class Weapon {
	public:
		virtual void useOn(Turtle &) = 0;
		virtual void useOn(Bullet &) = 0;
		// method overloading
	};
	
	class Rock: public Weapon {
	public:
		void useOn(Turtle &t) override { } // Rock used on turtle
		void useOn(Bullet &t) override { } // Rock used on bullet
	};


### Double Dispatch ###

	virtual void Enemy::strike (Rock &) = 0;
	virtual void Enemy::strike (Stick &) = 0;
	
	Enemy *e = l->createEnemy();	// Turtle
	Weapon *w = p->chooseWeapon();	// Rock
	
	e->strike(*w);
	
	// type of *e to use strike()? --- Enemy::strike() is virtual / P.V.
	// type of *w as augument of strike()? --- Weapon::useOn() is virtual
	// type of *e as argument of useOn()? --- useOn(*this)

- VDP can be used to add functionality that is different based on the type of the object but without making changes to the class hierarchy 
  - as long as the class hierarchy has been set up to accept visitor


**Book Example**

	class book {
	public:
		virtual void accept (BookVisitor &v) {
			v.visit(*this); // compiler know *this is Book during execuation
		}
	};
	
	class Text : public Book {
	public:
		void accept (BookVisitor &v) override {
			v.visit(*this);
		}
	};
	
	class Comic : public Book {
	public:
		void accept (BookVisitor &v) override {
			v.visit(*this);
		}
	};
	
	class BookVisitor {	
	public:
		virtual void visit (Book &) = 0;
		virtual void visit (Text &) = 0;
		virtual void visit (Comic &) = 0;
	};


**Catalog my collection of books**

- Books by author
- Text by topic
- Comic by hero

`map<string,int>`

- Without visitor method, you need to create a virtual method catalog, and implemented it in each subclass

VDP:

	class CatalogVisitor: public BookVisitor {
		map<string,int> myCatalog; 
	public:
		void visit(Book &b) override {
			++myCatalog[b.getAuthor()]; // if author doesn't exist in map, a default key-value pair of 0 will be created
		}
		
		void visit(Text &t) override {	
			++myCatalog[t.getTopic()]; 
		}
	
		void visit(Comic &c) override {	
			++myCatalog[c.getHero()]; 
		}
	
	};

- `visit` creates/updates key-value pairs in map
- `accept` will call `visit`

![Imgur](https://i.imgur.com/1OJMJwn.jpg)

**SE/visitor won't compile because of cycle of includes**

- `Book` want to include `bookVisitor`
- `bookVisitor` want to include `Book`

![Imgur](https://i.imgur.com/ifMOXfR.jpg)

## Compilation Dependency ##

- An method causes a compilation dependency

**Advice:**

- Prefer to **forward declare** a class whenever possible instead of include
- know a class exists, but not know the details
  - i.e. `class XYZ;`
  - Assertion that `XYZ` is a type that exists
- Benefits
  - avoid compilation dependency cycle
  - by avoiding includes, compiler needs to see less code
  - reduces frequency of compilation
    - i.e. a.cc includes b.h, every time b.h changes, a.cc needs recompilation

**When can you do forward declaration?**

File *a.h*

	class A {~~~~~};

File *b.h*
	

	#include "a.h"
	// because B inherits from class A
	
	class B : public A {~~~~~~};

File *c.h*

	#include "a.h"
	// when constructing C, you needs to know the size of C -> you need to know the size of A -> include A
	
	class C {
		A a;
	};

File *d.h*

	class A;
	// forward declaration
	
	class D {
		A *myA;
	}

File *d.cc*

	#include "a.h"
	// include it in cc file if you need the detail of A during implementaion
	
	~~~~~~~
	myA->foo();
	~~~~~~~

File *e.h*

	class A;
	// forward declaration
	
	class E {
		A foo(A a); 
	};


### Reducing Dependencies ###

*window.h*

	#include <X11/Xlib.h>
	class Xwindow {
	  Display *d;
	  Window w;
	  int s;
	  GC gc;
	  ~~~~~~~~~~~~
	
	 public:
	  void DrawRectangle(int x, int y, int width, int height, int colour=Black);
	};


*graphicsdisplay.cc*

	#include "window.h"
	~~~~~~
	
	w->drawRectangle(~~);

- every time window.h changes, graphicsdisplay must recompile
- even if the change is only to a private member

**Advice:**

- Remove all private members in to external implementation class
- Replace these with a Pointer to the Implementation
- It is called **pImpl Idiom**

### pImpl Idiom

File *windowimpl.h*

	#include<Xlib>
	struct XwindowImpl {	
	  Display *d;
	  Window w;
	  int s;
	  GC gc;
	};

File *window.h*

	class XWindowImpl; 	// forward declaration
	
	class Xwindow {
		XwindowImpl *pImpl;	 
		~~~~~
	};


File *window.cc*

	#include "window.h"
	#include "windowimpl.h"
	
	Xwindow:: Xwindow(): pImpl {new XwindowImpl} {} //MIL
	~~~~~~~~~~
	// replace d in method with pImpl->d;

*graphicsdisplay.cc*

	#include "window.h"
	// No compilation dependency on XWindowImpl.h

- Only *window.cc* needs to recompile (no need to recompile *graphicsdisplay.cc*) if you change XWindow’s implementation.

![Imgur](https://i.imgur.com/kaypNW4.jpg)


We can generate the pImpl idiom to accommodate multiple implementation

## Bridge pattern

This is an extension of the "pImplementation" idiom with subclasses to abstract alternate implementations. It is often found in graphical applications, like when we switch the window style or a back-end engine.

The bridge pattern makes it easy to swap out implementations. It separates an abstraction(`Window`) from its implementation(`WIndowImpl`) so we can change either separately.

It is based on **Single Responsibility principle**, and establish "bridge" between different responsibility

![Imgur](https://i.imgur.com/QtIG3lE.jpg)





# Lec 22: Design Quality, MVC, Exception Safety #


## Coupling ##

- How much do modules depend on each other
  - Low coupling: modules communicate through function calls with basic parameter types 
  - passing custom types
  - sharing global data
  - High coupling: modules access implementation details
    - friend, public field
- Goal: low coupling

## Cohesion ##

- how related are elements within a module
  - Low cohesion: arbitrary grouping of unrelated elements
  - `<utility> std::move, std::pair, std::swap`
  - High cohesion: module achieve one task
- Goal: high cohesion

## Decouple the interface ##

**Advice:** primary classed should not print things

	class ChessBoard { //game state
		~~~~~~~~~~~
		cout << "Your turn";
	};

This is poor design as `ChessBoard` class is coupled with `cout`

What if you want to make it an web game and output to internet?

-----------------------

Better: ChessBoard could state stream objects to read/write

	class ChessBoard {
		istream &in;
		ostream &out;
	
		out << "Your turn!";
	};


This is poor design as `ChessBoard` class is coupled with *streams*

What if you want to make a graph interface? (graph is not a stream)

------------

ChessBoard should not be communicating at all. 

**Single Responsibility principle:**

a class should only have one reason to change

- game state / communication are 2 reasons

## Model-View-Controller Design Pattern (MVC) ##

![Imgur](https://i.imgur.com/PP4sWS4.jpg?1)
(It is not UML)

- Model: maintain data (game state)
- View: communicate to user
- Controller: facilitate communication between model & view

Two kinds of relationship

- Model / View can be in a subject / observer relationship
- Model <---observer---> Controller <---observer---> View

i.e.

Controller:

- whose turn?
- valid input?

View: graphics/ text display

## Exception Safety ##

### Motivation ###

	void f() {
		MyClass *p = new MyClass;
		MyClass mc;
		g(); // if g throw an exception, delete p will never be called -> memory leak
		delete p;
	};

- An exception should not cause memory leaks, dangling pointers, broken class invariants
- C++ Guarantee: During stack unwinding, stack allocated data is destroyed (dtor run; memory reclaimed ) 
  - why not pointer? Because pointer doesn't mean "OWNS"
  - `mc` is reclaimed but `p` leaked

safe code:

	void f() {
		MyClass *p = new MyClass;
		try {
			Myclass mc;
			g();
		} catch (...) { // catch everthing
			delete p;
			throw;
		}
		delete p;		
	}	

It is tedious / error prone

--------------

We would like a way to guarantee that some piece of code executes irrespective of whether an exception occurred or not

- java ---> `finally`
- Scheme ---> `dynamic_wind`
- C++ does not have such a feature
  - does not need it 

Solution:

- Maximize stack usage

What to do when we need to heap allocated?

#### RAII: Resource Acquisition IS Initialization ####

- resources should be wrapped within a stack allocated object whose job is to free the resource
  - i.e.   `{ ifstream f{"file.txt"}; ~~~~~~~~}`
  - constructor opens/acquires the file (resource)
  - f goes out of scope, destructor closes/frees the file (resource)
- Heap memory is a resource; use RAII
- Every heap allocated object should be wrapped within a stack allocated object whose job is to free the heap object

what to use?

	std::unique_ptr<T>

- ctor of `unique_ptr<T>` takes a `T*`
- dtor of `unique_ptr<T>` deletes the ptr
- A `unique_ptr<T>` object can be treated as a heap allocated ptr ( operator`*`, operator`->` are overloaded)

Better code:

	void f() {
		std::unique_ptr<MyClass> p{new MyClass};
		// not recommanded; better way: 
		// auto p = std::make_unique<MyClass>(); 
		// (): args for the ctor
		MyClass mc;
		g();
	}

*c++/unique_ptr/example1*

What if 2 ptr points to same heap memory?

	std::unique_ptr<MyClass> p{new MyClass};
	std::unique_ptr<MyClass> q = p; // won't compile

- this would have caused a double free
- Copy ctor / copy assignment operator for `unique_ptr` are disabled

*unique_ptr/basicimpl.cc* : an basic idea, not used in real life

- `unique_ptr<T>` cannot be copied but can be moved, so that you can return it in move assignment
- `std::share_ptr` allows multiple ptrs to the same heap allocated object

share_ptr:

	void g() {
		auto p = std::shared_ptr<MyClass>();
		~~~~~~~~
		if (~~~~~~) {
			auto q = p; //share_ptr ctor
			~~~~~~~
		} // dtor for q runs
		~~~~~~~~
	} // dtor for p runs


`share_ptr` uses reference counting

- keep track of how many ptrs point to the object
- dtor only deallocates when reference count is 0

# Lec 23: Exception Safety,  Template Functions

### RAII: Resource Acquisition IS Initialization

- use `shatr_ptr` & `unique_ptr` to manage heap memory
- you can cast `share_ptr` objects
  - `static_pointer_cast`
    - i.e.
    - `std::shared_ptr<base> sp0(new derived);`  
    - `std::shared_ptr<derived> sp1 = std::static_pointer_cast<derived>(sp0);` 
  - `const_pointer_cast`
  - `dynamic_pointer_cast`


## Exception Safety

#### 3 levels of exception safety

- **Basic Guarantee** : If an exception occurs, program is in a valid but *unspecified state*
  - no memory leaks, no dangling pointer, no broke class invariant 
- **Strong Guarantee** : If an exception occurs, during the execution of a function f, it is *as f was never called*
- **No-throw guarantee** : Function never throws an exception, always achieves its goal

Example:

	class A {~~~~~~~};
	class B {~~~~~~~};
	
	class C {
		A a;
		B b;
		public:
		void f() {
			a.m1(); // strong guarantee
			b.m2(); // strong guarantee
		}
	};

- f() is not no-throw
- Does f() have a strong guarantee?
  - if m1 throws, it is as if m1 was never called  ==>  f() was never called
  - if m2 throws, it is as if m2 was never called, m1 might have made some changes to the program state ==>  f() does NOT provide a strong guarantee 
- To provide a strong guarantee, changes made by m1 must be undone
  - often undone things is not possible e.g. printing
- Assume m1 & m2 only have *local side-effects* (m1 only changes field in a) 
- Can we rewrite code to provide a strong guarantee?
- **Idea**: instead of trying to undo things, let's "do" the things on copies. If we succeed, swap the copies with the assigned

#### copy and swap Idiom: 

##### `std::swap()`

execute copy then swap:

	C::f(){
		A atemp{a};
		B btemp{b}; // copy ctor
		atemp.m1();
		btemp.m2();
	
		a = atemp; // swap assignment
		b = btemp;
	}

But what if assignment of B throws an exception ==> a is changed, b is not

Since ptr assignment never throws,  let's create ptrs to copies of objects of swap ptr ==> will use **pImpl Idiom**

	struct CImpl {A a; B b;}
	class C {
		unique_ptr<CImpl> pImpl;
	public:
		void f() {
			unique_ptr<CImpl> temp{new CImpl{*pImpl}};
			                            // CImpl object
			                    // copy ctor for CImpl
			// auto temp = make_unique<CImpl>(*pImpl);
			temp->a.m1();
			temp->b.m2();
			swap(pImpl,temp);
			// std::swap on smart ptr will swap the heap ptrs within these object
		}
	};

#### Exception safety in the STL <vector>

- wraps a heap allocated array
- uses RAII
- dtor deallocates the array

i.e.

	vector<MyClass> v;
	// when v goes out of scope, elements of v are destroyed, then the heap array is destroyed
	
	vector<MyClass *> v1;
	// when v1 goes out of scope, delete is NOT called on elements of v1, heap array is deleted
	
	vector<Observer *> observer;
	
	// A programmer has to manually delete if the elements are indeed owned by the vector	
	for (auto &p : v1) delete p;
	
	// But we don't want to manage memory!
	vector<share_ptr<MyClass>> v2;
	// when v2 goes out of scope, elements are destroyed (dtor run for share_ptr objects)
	// The dtor will decrease the reference count, if ref count is 0, object is deallocated

- `vector<T>::emplace_back` gives a strong guarantee
  - adding an element to a not yet full array (size < capacity)
  - size == capacity
    - create a larger heap array
    - copy from old array to new array // inefficient
    - deacllocate old array // delete is no throw
- Instead of copy, we can move elements to the new array ==> more efficient
  - if during a move, an exception occurs, the old array has been changed ==> no strong guarantee 
- emplace_back will use move operations only if they guarantee that they are no throw. Otherwise, the slower but safer copy operator are used (why is safer???? Recall a2q5, how you resize it??)
- If T's move constructor is not noexcept, vector will use the throwing move constructor. If it throws, strong guarantee is waived and the effects are unspecified.

i.e

	class MyClass {	
	~~~~~~~
		MyClass(MyClass &&other) noexcept {
		~~~~~~
		}
	};

- Always write **noexcept** (especially for the **move** ctor & move assignment) guarantees that other code can rely on the info

# Lec 24, Template Functions, How virtual works

### Long time ago

	template <typename Func>
	void foreach (AbsIter &start, AbsIter &finish, Funcf) {
		while (start != finish) {
			f(*start);
			++start;
		}
	}
	
	// f : must be a function
	// paramter type of f must match return type of AbsIter::operator* (i.e. int)

Generalized:

	template<typename Iter, typename Func>
	void foreach (Iter start, Iter finish, Func f) { 
		// not pass by reference -> allows rvalue as argument
		// same
	}
	
	void f(int n) {
		cout << n << endl;
	}
	
	int a[] = {1,2,3,4,5};
	
	forEach (a , a + 5, f); // prints the array

Another example:

	template <typename T> T min (T x, T y) {
		return x < y ? x : y;
	}
	
	int x = 1, y = 2;
	int z = min (x, y); // z = 1
			// type T is infered to be int by compiler
	// Equivalent 
	// int z = min<int> (x,y);
	
	char w = min('a', 'b');
	double d = min (1.0, 2.0);

- c++ will attempt **automatic type deduction** for *template arguments* in a template function
  - not for template classes
- function `min` can be called on any type `T` that provides `operator <`

## `STL: algorithm`

#### `for_each`

#### `find`

	template<typename Iter, typename T>
	Iter find (Iter first; Iter last; const T &val) {
		// return an iter to the first occurance if val in [first, last)
		while (first! = last) {
			if (*first==val) return first;
			++first;
		}
		// return last if not find
		return last;
	}

#### `count`

	// returns the number of occurance of val
	template<typename Iter, typename T>
	int count (Iter first; Iter last; const T &val) {
		int ret = 0;
		while (first!=last) {
			if (*first == val) ++ret;
			++first;
		}
		return ret;
	}

#### `copy`

	// copy one container range [first, last) to another starting at result
	template <typename InIter, typename OutIter>
	OutIter copy (InIter first, InIter last, OutIter result) {
		while (first!=last) {
			*result = *first;
			++result; 
			++first;
		}
		return result;
	}
	// Note: the function does not allocate new memory, so output container must "have enough" available


	vector<int> v{1,2,3,4,5,6,7};
	vector<int> w(4); // guarantee capacity = 4 or higher
	copy(v.begin()+1, begin()+5, w.begin()); // [2,3,4,5]

#### `transform`

	template <typename InIter, typename OutIter, typaname Func>
	OutIter transform (InIter first, InIter last, OutIter result, Func f) {
		while(first != last) {
			*result = f(*first);
			++first;
			++result;
		}
		return result;
	}
	
	// Note: 
	// not allocate new memory
	// must implement != & * & ++ for InIter, * & ++ for OutIter
	// f should be something that can be called as function
	// paramter type of f ; return type of f 

example:

	int add1(int n) {return n + 1;} 
	vector<int> n {2,3,5,7,11};
	vector<int> m(n.size());
	transform (n.begin(), n.end(), m.begin(), add1);
		// see back_inserter of how we can avoid the "space must be available" requirment

#### `back_inserter`

	vector<int> v {1, 2, 3, 4, 5};
	vector<int> w;
	copy(v.begin(), v.end(), w.begin()); // WRONG!
	// Wrong because copy doesn't allocate any space, and w doesn't have any space allocated to it
	// SEGFAULT

Why can’t copy allocate space?  

- Copy doesn’t even know that w is a vector - it only has an iterator to an item in w.

`back_inserter`: an iterator whose operator= inserts a new item
	

	#include <iterator>
	copy(v.begin(), v.end(), back_inserter(w));
	// back_inserter creates iterators
	// adds v to the end of w, allocateing space

#### Function Objects

	class Plus1{
	public:
		int operator()(int n) {return n + 1;}
	};
	
	Plus1 p;
	int x = p(4); // x = 5
	
	transform(n.begin(), n.end(), m.begin(), p};

A function object is "an object that can be  called as if it was a function" (functor)

	class IncreasingPlus {
		int m = 0;
		public:
		int operator()(int n) {return n+m++;}
	};

Function object are useful as they can **maintain state**

	vector<int> v (5,0); // [0,0,0,0,0]
	vector<int> w (v.size());
	
	transform(v.begin(), v.end(), w.begin(), IncreasingPlus{}); //ctor of IP
	// m is increased by 1 each time
	// w[0,1,2,3,4]

## Lambda

how many ints in a vector are even?

	vector<int> v {~~~~~~~~~~~~~~~};
	bool even(int n) {return n % 2 == 0;}
	int x = count_if(v.begin(), v.end(), even);
	// equivalent
	int x = count_if(v.begin(), v.end(), [](int n) {return n % 2 == 0;});
	                             // capture  parameter  lambda body


## Virtual Table

	struct Vec {
		int x, y;
		void f() {~~~~~~~~} // non virtual
	};
	
	Vec v;
	sizeof(v); // 8 bytes (2 integer)

- Methods are NOT stored within objects

------------------------------------

	Book *bp = (~~~condition~~~) ? new Comic{~~~} : new Book {~~~~~~~};
	bp->isHeavy(); // Comic::isHeavy() or Book::isHeavy()?
	
	struct Vec2 {	
		int x, y;
		virtual void f(){~~~~~~~}
	};
	
	Vec2 w;
	sizeof(w); // 16 bytes (a pointer has been adeed)

- For every class that contains at least 1 virtual method, a virtual table is created

  - Vtable: a lookup table of functions used to resolve functions in a dynamic/late binding manner. 
  - VTable: a static array that the compiler sets up at compile time.
  - Every class that uses virtual functions (or is derived from a class that has virtual functions) is given its own vtable
  - The compiler adds a hidden pointer to the class, which is the vptr

- Virtual contains 

  -  an entry to indicate the type of the class
  -  function pointer for each (that points to the most-derived function accessible by that class.)
  -  virtual method in the class
     ![Imgur](https://i.imgur.com/XAIm6Df.jpg)

- Calling `bp->isHeavy()` (all happens at runtime);

  1. follow ptr to object

  - follow vptr to get the vtable
    - lookup `isHeavy()` funcion ptr in the table
  - follow the function ptr (no need to know the actual object is, only know address)
    - slower than static dispatch

- Declaring at least one virtual function adds a vptr to the object. Therefore, classes with no virtual functions produce smaller objects (lower cost) than if some functions were virtual

Question:

- Why does dynamic casting (dynamic_cast) only work on classes with at least one virtual function? 
- dynamic_cast needs to know what objects we actually have, and the compiler needs the vtable to figure out the type (A class that declares or inherits a virtual function is called a polymorphic class)

