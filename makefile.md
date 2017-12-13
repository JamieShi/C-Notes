# TUT5 Oct 18th #


## Separate Compilation ##

If b.cc changed, only need to do 

    g++14 b.cc -c
    g++14 main.o b.o c.o -o main

*tutorial/tut5/make/example.*

makeFile 

    # target: dependencies
    # <TAB> build command
    
    main: main.o book.o textbook.o comic.o # (main depends on these)
    	g++-5 -std=c++14 main.o book.o textbook.o comic.o - o main
    # NOTE: aliases (e.g. g++14) does not work here

    book: book.h book.cc
		 g++-5 -std=c++14 -c book.cc

	main.o: main.cc book.h
        g++-5 -std=c++14 -c main.cc


    clean:
    rm *.o main
 
    .PHONY: clean
    # PHONY dependency - not part of automated compilation
	# (to avoid a file called clean if present)

If book.cc or book.h (dependencies) has last modified time later than book.o (target) rebuild

Tree Structure:

    main
     |
     |
    book.o
     |    \
     |     \
    book.h book.cc

how to use Makefile

    make clean
    make
    
    make textbook.o

    make clean && make


Can we do it better?

Yes!

    
    g++ -MMD textbook.cc -c
    # - MMD compiler will fetch dependency atumotically
	# This creates iter.o and iter.d, and iter.d contains:
	iter.o: iter.cc list.h node.h

    -Wall -Werror
    # -W warning

final version

	CXX = g++-5
	CXXFLAGS = -std=c++14 -Wall -MMD
	EXEC = myprogram
	OBJECTS = main.o list.o iter.o node.o
	DEPENDS = ${OBJECTS:.o=.d}
	
	${EXEC}: ${OBJECTS}
        ${CXX} ${CXXFLAGS} ${OBJECTS} -o ${EXEC}

	-include ${DEPENDS}

	.PHONY: clean

	clean:
        rm ${OBJECTS} ${EXEC} ${DEPENDS}


## lvalue & rvalue ##

- **lvalue** is any entity that has a **memory address**
- **rvalue** is anything that not a lvalue

example:

    int x = 5;
    int &rx = x;
  
    //int &ry = 5; 
    //  lavlue  rvalue 

    int &&rry = 5;
    // rry is rvalue reference, but it is a lvalue
    // int &&rrx = x; 

    int x = 5;
    std.move(x)
    //casts x to rvalue
  


## Why we need move? ##

    Node inc(Node n, int k) {
     for (Node *m = &n; m = nullptr;m->value+=k; m = m.next)
     {}
      return n; 
    }

    Node {inc(*n1,5)}; //unecessary called of copy ctor