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
3. delete the memory allocated to this first


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

###--------------------- continuing - c++ class --------------------------###