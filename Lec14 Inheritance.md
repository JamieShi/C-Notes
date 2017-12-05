# Lec14, System Modelling, UML, Relationship #


### Relationship 1: Composition ###

### Relationship 2: Aggregation ###

### Relationship 3: Inheritance ###

	-------------------    	------------
	| Book            |		|Textbook			Comic                       
	-------------------		--------------
	|-title:String	  |		|-title:			-title:
	|-authors:String  |		|-authors:			-authors:
	|-numPages:Integer|		|-numPages			-numPages:
	|				  |		|-topics:String  	-hero:String
	-------------------		----------------
	|                 |
	-------------------

It is hard to create collections of those different types of Books

- array of `void*`
- use a `union` type

--------
Observe:

- A textbook **IS A** Book with a topic field
- A comic IS A type of "special" Book with a hero field

UML:

			-------
			| Book                                 
			-------------------
			|-title:String			
			|-authors:String	
			|-numPages:Integer						
			---------------
			|
			----------------
					 ^
    	 ____________|______________
		|							|	
	-------
	|Textbook						Comic                       
	-------------------
	|-title:String				-title:
	|-authors:String			-authors:
	|-numPages:String			-numPages:
	|-topics:String  			-hero:String
	---------------


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

Following won't compile:

	Text:: Text(string title, string author, int numPages, string topic)
		: title{title}, author{author}, numPages{numPages}, topic{topic}{}

**Problems:**

1. title, author, numPages is **private** in Book
2. **MIL** is only allowed to refer to **its own fields** (fields it declare)
3. Problems happen during the different steps for object construction

#### Steps that happens when an object is created: ####

- space is allocated
- **constructed the superclass part of the object (default ctor for superclass)**
- subclass field initialization: default ctor runs for fields that are objects
- ctor body runs

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
			return getNumPages()>200;
		}
	};

	class comic : public Book{
	....
		public:
		bool isHeavy() const{
			return getNumPages()>30;
		}   // replace the implementation in Book by overriding
	};

	Comic cb{..,..,40,..};
	cd.isHeavy(); 
	//Comic:isHeavy() runs -> true

	Book b = Comic{..,..,40,"Batman"};
	// legal as a Comic IS A Book
	b.isHeavy(); 
	// Book: isHeavy() runs ->false

Placing  a subclass object in the space for a superclass object cause **object slicing**


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

###Using Superclass pointless###

	Comic c{...,...,40,...};
	Comic *cp{&c};
	cp->isHeavy(); // Comic:: isHeavy()
	Book *bp {&c}; // legal
	bp->isHeavy(); // Book:: siHeavy()

![Demonstration](https://i.imgur.com/z9VBN7K.jpg)

By default, a c++ class compiler looks at the declared type of a pointer / reference to determine which method to call (not the type of object it points/refers to)

It is called **Static Dispatch** in compilation

	Book *collection[20]; // it is legal but they are all types of book

------
We would like to call methods superclass pointer, and hero the most "specialized" method to executed.

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

- if you forget to put both `override` and `const`, it is method OVERLOADING instead of method OVERRIDING
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

##Destructors in the presence of inheritance##

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
	  Y(int n, int m): X{n}, y{new int [m]} {} // X{n} hijack step2
	  ~Y() { delete [] y; }
	};


Destruction is the inverse of construction

1. dtor body runs
2. subclass field destroyed: dtors run for fields that are objects
3. destructed the superclass part of the object (**dtor of superclass run**)
3. space is reclaimed

A subclass dtors will always, *automatically* call the superclass dtors.
 
	// Run with valgrind
	int main () {
	  X *xp = new Y{5, 10};

	  delete xp;  
	   //if dtor it is not virtual, will call just ~X() -> memory leaks for field y
	}

**Advice:**

- If a class may have children now as in the future, it should make its dtor `virtual`
- If you want an abstract class but you don't have a good P.V. method, make dtor P.V.
- All dtor must be implemented, even P.V. dtor
	- Subclass need to call superclass dtor


###final###
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

i.e. `=0`
	
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

We would like `Student::fee` to be UN-implemented, but the linkers would give an error

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
- Subclass of abstract classed must implement all inherited P.V. method to be considered concrete
- P.V. method does not need to have a input (typically)

**Why create an abstract class?**

- Put the common fields/methods
- Polymorphism

**Concrete classes** : any class without pure virtual method 

By default, if you inherit an abstract class, you are also abstract. Unless you implement all P.V. methods of base class

**UML Notes**

- Virtual / Pure Virtual --- *italics*
- Abstract class --- *class name in italics*
- static --- underline
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
	sInts.push(s);

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
		friend theList<T>;  // only type <T> is a friend
	};
	
	T ith(int i){~~~~~~~~~~~~~}
	void addToFront(T &n){~~~~~~} // since we don't know the type of n, we don't want to make a copy of it if it is big
		
	List<int> l1;
	int x = 5;
	l1.addToFront(x);
	
	List<List<int>> l2;
	l2.addToFront(l1);
	
	for (List<int>::Iterator it = l1.begin();it!=l1.end();++it){
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

	for (vector<int>::iterator it = v.begin(); it != v.end(); ++it){
		cout << *it << endl;
	}

	for (auto n: v){
		cout << n << endl;
	}

	for(vector<int>::reverse-iterator it = v.rbegin(); it !=v.rend(); ++it)  //rend(): the one before the first element

	// you cannot using auto for reverse iterator!!!!


Many STL classes have methods that expect Iterator as a parameter

	v.erase(v.begin()); // all other element is shifted left
						// each pointer may not point to what it originally point to

When you make changes to a vector, internal element might move

- this means all existing **iterators** becomes invalid
- `erase` return a new Iterator that ~~~~~~~~~ (look it up!!!)

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
	- `for (int i=0;i<v.size();++i)`
		- array style
	- `for (vector<T>:Iterator it=v.begin();it!=v.end();++it)`
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

stack unwinding (popping up function) in stack frame: in search of a handler for a  particular exception

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

- if  the caught exception was actually a subtype of SomExp, the exception object is sliced
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

By default, if a dtor throws an exception, the prog terminated right away (`std::terminate` is called)

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

## c++ cast ##

cont'd on next lec