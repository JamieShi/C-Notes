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
		Myclass mc;
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
- ctor of `unique_ptr<T>` takes `a` T*
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
	- `const_pointer_cast`
	- `dynamic_pointer_cast`


## Exception Safety
#### 3 levels of exception safety

- **Basic Guarantee** : If an exception occurs, program is in a valid but unspecified state
	- no memory leaks, no dangling pointer, no broke class invariant 
- **Strong Guarantee** : If an exception occurs, during the execution of a function f, it is as f was never called
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

code:

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
	// when v1 goes out of scope, delet is NOT called on elements of v1, heap array is deleted

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

i.e

	class MyClass {	
	~~~~~~~
		MyClass(MyClass &&other) noexcept {
		~~~~~~
		}
	};

- Always write noexcept (especially for the move ctor & move assignment) guarantees that other code can rely on the info

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
		// type T is infered to be int
	// Equivalent 
	// int z = min<int> (x,y);
	
	char w = min('a', 'b');
	double d = min (1.0, 2.0);

- c++ will attempt **automatic type deduction** for template arguments in a template function
	- not for template classes
- function `min` can be called on any type `T` that provides `operator <`

## `STL: algorithm`
#### `for_each`
#### `find`
	template<typename Iter, typename T>
	Iter find (Iter first; Iter last; const T &val) {
		// return an iter to the first occurance
		// if val in [first, last)
		// return last if not find
	}
#### `count`
	// returns the number of occurance of val
#### `copy`
	template <typename InIter, typename OutIter>
	OutIter copy (InIter first, InIter last, OutIter result) {
		// *result = *first;
		// ++first;
		// ++result;
		// copy one container range [first, last) to another starting at result
	}
	// Note: the function does not allocate new memory, so output container must "have enough" available


	vector<int> v{1,2,3,4,5,6,7};
	vector<int> w(4); // guarantee 4 or higher
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
	// must implement != / * / ++ for InIter, * / ++ for OutIter
	// f should be something that can be called as function
	// paramter type f; return type of f 
example:

	int add1(int n) {return n + 1;} 
	vector<int> n {2,3,5,7,11};
	vector<int> m(n.size());
	transform (n.begin(), n.end(), m.begin(), add1);
		// see back_inserter to see how we can avoid the "space must be available" requirment



##### Function Objects

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
	// w[0,1,2,3,4]

## Lambda

how many ints in a vector are even?

	vector<int> v {~~~~~~~~~~~~~~~};
	bool even(int n) {return n % 2 == 0;}
	int x = count_if(v.begin(), v.end(), even);
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