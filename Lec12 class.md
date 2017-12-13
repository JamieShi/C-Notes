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
-  What is the efficiency of *printing* the list?  **O(n^2)**

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