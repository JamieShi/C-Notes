#Lec10 Separate Compilation of Class, Big five (Continued)  10/12/2017 




##From before

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

*::      Scope resolution operation, before the method name*

 
## Destructors 

**When is an object destroyed?**

- Stack allocated -when goes out of scope

- Heap allocated -when delete is called on a pointer to the object


Destructors are special methods that run when objects are destroyed.

Each class comes with a free destructor.

**Steps to destroy an object:**

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
                     // It's always save to delete a nullptr
    }
    // this recursively calls the dtor which ends up deallocating the entire list

A class can only have one dtor

- No overloading

##Copy Assignment Operator

    
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



An assignment is a statement and an expression.

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
       data - other.data;
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
    
    Vec Vec::operator+(const Vec &v) const { // not change the fied of the object
      Vec toReturn {x+v.x, y+v.x};
      return toReturn;
    }
    
    v1.operator+(v2);
    // This method does not change the fields of the object on which it is called. This is a const method