# CS246 midterm review  session, Oct 22nd#

### format ###

- trivial
- programming


### Bash program ###

Write a bash scripts which:

- Take a program & args on command line
- Run the program in Valgrind
   1. If no program was supplied, print usage msg and exit with state 2
   2. If there are errors, print msg to stderr and exit 1
   3. Else exit 0

code:

    #!/bin/bash

    if[ $# -le 1 ];then
       echo "Usage: $0 [program]" 1>&2
                                #   /dev/stderr
       exit 2
    fi
    
    ERRFILE=$(mktemp)

    #   $@ are all program args, starting at $1
    Valgrind "$@"  2> "$ERRFILE"

    egrep "0 errors from 0 contexts" "$ERRFILE" &>/dev/null 
    #   get rid of output of egrep output

    if [ $? -ne 0 ];then
        echo "$1 had memory errors" 1>&2
        rm "$ERRFILE"
        exit 1
    fi

    rm "$ERRFILE"
    exit 0
    

### Write the Big 5 for ###

    class BinTree{
     int data;
     BinTree *left,*right;
     BinTree *parent = nullptr;
    };
     public: ...

**Dtor**

    ~BinTree(){
      delete left;
      delete right;
      //Don't delete parent -> infinite loop
    }

**Copy ctor**

    BinTree(const BinTree & other):
      data{other.data},
      left{other.left ? new BinTree{*other.left}:nullptr},
      right{other.right ? new BinTree{*other.right}:nullptr}{
       if (left) left->parent = this;
       if (right) right->parent = this;    //updateParents()
    }
    
**Copy Assignment**

Why don't use MIL here? Because it is not a ctor.

    BinTree & operator=(const BinTree & other) {
     if (this == &other) return *this;

     delete left;
     delete right;

     data = other.data;
     left = other.left?new BinTree{*other.left}:nullptr;
     right = other.right.....;
     updateParents();

     return *this;
    }

    // or

    BinTree &operator=(const BinTree &other){
      BinTree temp {other};
      swap(temp) 
      return *this;     
    }   

    void swap(BinTree &other){
      std::swap(data,other.data);
      std::swap(left,other.left);
      std::swap(right.other.right);
      updateParents();
      other.updateParents();
    }


**Move Ctor**

    BinTree(BinTree &&other):
     data{other.data},
     left{other.left},
     right{other.right}{
      other.left = other.right = nullptr;
      updateParents();
    }
    

**Move assignment**

    BinTree &operator=(BinTree &&other){ //reference to rvalue is lvalue
     swap(other);
     return *this;
    }
