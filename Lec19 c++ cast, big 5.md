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
	-  `begin` / `end` are Iterator factories

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
-  You have to go through the concrete class

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
		XwindowImpl *pImpl;	 // No compilation dependency on XWindowImpl.h
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

- Only *window.cc* needs to recompile (no need to recompile *graphicsdisplay.cc*) if you change XWindow’s implementation.

![Imgur](https://i.imgur.com/kaypNW4.jpg)


We can generate the pImpl idiom to accommodate multiple implementation

## Bridge pattern

This is an extension of the "pImplementation" idiom with subclasses to abstract alternate implementations. It is often found in graphical applications, like when we switch the window style or a back-end engine.

The bridge pattern makes it easy to swap out implementations. It separates an abstraction(`Window`) from its implementation(`WIndowImpl`) so we can change either separately.

It is based on **Single Responsibility principle**, and establish "bridge" between different responsibility

![Imgur](https://i.imgur.com/QtIG3lE.jpg)

