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

**sth here**

### pass argument to c++##
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


# Lec5, Input/Output control

*c++/intro/plus.cc*

    #include <iostream>
	using namespace std;
something here

- cin ignores white space by default

  - `cin >> skipws;`  include whitespace
  - `cin >> noskipws;` ignore whitespace 
  - from the `<iostream>` header. 
- If read fails, an int value is set to **0** (c++17)
- If it is too big, `int_max` will be read
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
	    if (!cin) break;   // !cin == cin.fail()
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
- If `>>` is used as input operator, the expansion produce the LHS (i.e. `cin`)

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
	      else {
	        cin.clear();    //acknowledge fail, lower the fail flag
	        cin.ignore();   //discard one char
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

**something here**

- stdin is different from arguments ( " has no special meaning)

i.e.

	> "Hello World"
	> "Hello
- cin will read from the first non-whitespace char until it hits a whitespace char

##Format Specifies ##
In c ......

In c++, we use **I/O manipulators** to format I/O

	int x = 95;
	cout << x; //print decimal
	cout << hex << x; //print in hex (all the following cout)
	cout << dex << x; //print decimal

`<iomanip>`

	float price = 2.00;
	cout << fixed << showpoint << setprecision(2) << price << endl;

- `showpoint` format flag : decimal point is always written   
- `setprecision(2)`: as many digits as necessary are written to match the precision set for the stream (if any).

- `boolalpha` format flag : bool values are inserted/extracted by their textual representation (true or false), instead of integral values (1 or 0).
- ignore white space or not

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
	  string s {ss.str()};  //convert ss to s
	  cout << s << endl;
	}


*io/getNum.cc*

ensure that the input is an integer

	int main () {
	  int n;
	  while (true) {
	    cout << "Enter a number:" << endl;
	    string s;
	    cin >> s;   //succed as long as user type something
	    istringstream ss{s};
	    if (ss >> n) break;
	    cout << "not a number";
	  }
	  cout << "You entered " << n << endl;
	}


###String###
	
something here

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

	"a" < "b"
	"a" < "ab"
	"A" < "a"  //Since A has ASCII value 65; a has a higher ASCII value

## Default Argument##

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

- if you are going to leave something not in a function call, it has to be last parameter, if they have default value.

i.e.

	void foo(int x = 5, string str = "hello");

	foo(10,"a");
	foo(10);
	foo();

	//foo("str");    X
	//foo(,"str");   X

## Function overloading##

	void foo (int,string);
	void foo (int);
	void foo ();

- In c, it is illegal for two functions to have the same name
- In c++, functions can have the same name as long as the differs in the number and/or types of parameters

	some code here

- It is not enough for function to only differ on the return type


**c++ operators are implemented as functions**

	operator>>(cin,x); //cin >> x
	operator>>(x,3);   //int x; x >> 3

**c++ compiler are able to figure out which function to call base on parameter**

	void foo (int x);
	//void foo (int x, int y = 10);  ILLEGAL
	foo(42);   ambiguous call



## struct ##

In C,

sth here


In c++, 

	struct Node{
	 int data;
	 Node *next;
	}

	Node n{5,nullptr};

The following code is ILLEGAL?

	struct Node{
     int data;
	 Node next;
	}

You can NEVER allocate a variable of type Node. Try to figure out the size of Node will cause **infinite loop**

###Parameter Passing (Review)###

sth here

pass by value vs. pass by parameter

**Passing parameter to input stream**

	cin >> x;
	operator >> (cin,x); // seems no "&", but actually x is passed by reference

	//scanf("%d",&x);

## Lvalue Reference ##

	int y = 10;
	int &z = y;

z is a **lvalue reference** to y

lvalue reference is like a **constant pointer** with automatically dereference.

sth here

Things you cannot do with reference

- You cannot leave them uninitialized

	`int &z;  //ILLEGAL!`
- Must be initialized to a lvalue (something that has an **address**, refers to an actual storage location)

ILLEGAL:

	//int &z = 3;
	//int &z = x+y; 
	        -- Rvalue --
- you cannot create a pointer to a reference
- you cannot create a reference to a reference
- you cannot create a array of reference

some code here


#Lec 7, Short Topic, Preprocessor, Sep 28th#

Why does `cin >> x` works?

	operator>>(cin,x); // x is a reference
	istream &operator>> (istream &in, int &n){...}

**Para1 `in` is also passed be reference**

- changes made to `in`, need to be visible outside (i.e. `cin.clear()`)
- stream cannot by copied  [https://stackoverflow.com/questions/6010864/why-copying-stringstream-is-not-allowed](https://stackoverflow.com/questions/6010864/why-copying-stringstream-is-not-allowed "Reason")

i.e.

	int main(){
	   std::stringstream s1("This is my string.");
	   std::stringstream s2 = s1; // error, copying not allowed
	}


### Passing by value vs. Passing by reference###

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


## Dynamic Memory##

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

### Allocate arrays###

	Node *myarr = new Node[10];
	...
	delete [] myarr;

### Return by value###

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

Advice: 

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

## Preprocessor##

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






	

#Lec 8, Preprocessor, c++ class, Oct 3rd#

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

## Separate Compilation##

It is good to divide a program into 

- header file(.h): user defined information, forward declaration for functions
- implemented file(.cc) : definitions, implementation of functions

*c++/separate/example1*

### How to compile?###

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
	`$g++14 main.o vector.o`

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

instead, ...... sth here


## C++ class ##

(in next .md fileS) 