# Lec 1, Admin Stuff + Data Encapsulation, May 1st

### Midterm: June 20. 2018 7:30pm - 9:20pm

- Compiler Construction; Kenneth Loudon
- Modern Compiler Java, Andrew W.
- office hour: Wed 11 - 12:30

## What is a sequential program?

- not concurrent/parallel
- single threaded

### Goal: 
- understand haw sequential programs work
- Describe relationship between high level program language and machine level data representation.
	- Different abstractions


graph:

	C++      Scala
	C Racket Java   -----------
		|  complier           |  
		v  (A5- A11)          |
	Assembly language         | complier (CS444)
		(A3)
		|  assembler          |
		v    (A3)             |
	Machine language <--------
          (A1)

You will write programs that read programs  + output programs.

## Data Representations

- bit: single digit in binary 0/1
- byte: sequence of 8 bits
- word: machine dependent/specific sequence of bits
	- In 32-bit machine, a word is 32 bits/ 4 bytes
- nibble: 4 bits

**Example**: String 100011

- signed binary: -3
- one million & element
- unsigned binary: 67
- 2's compliment number: -61
- ASCII: C
- Garbage value

### Decimal Representations (powers of 10)

1 * 10^6 + 0 * 10^5 + ...

### Binary Representations (powers of 2)

1 * 2^6 + ...

### Converting from decimal to binary

- 67 = 64 + 2 + 1
- Divide it by 2

### Hexadecimal representations (powers of 16)

0 1 2 3 4 5 6 7 8 9 A/a B/b C/c D/d E/e F/f

**Possible 32 bits machine instruction**

1100 1010 1101 0000 0100 1011 0010 0001

12 10 13 0 4 11 2 1

0xCAD04B21 (??????? 0 or O)

### Numeric Ranges
- Given n bits, what is the range of numbers we can represent?
- In general, for a n-bit the unsigned binary range is 0 - 2^n-1

### Represent negative numbers
Reserve a bit to represent the **sign** of use the rest to represent the **magnitude**

- Sign of magnitude:
- 0: positive
- 1: negative

##### Problems:
- two representation for 0
- Requires different (separate) circuits to do addition/subtraction
	- Would much rational use the same circuit

#### 2's Complement (More popular)

Idea: Use Congruence to represent negative numbers

- n: number of bits
- N = 2^n

To represent -b, we will use N-b

Each addition & subtraction will use the **same** circuit

2. a - b
3. a + (-b)
4. a + (N - b)   //congruent N
5. a + ((N-1)-b) + 1
6. a + flips the bits of b + 1


representation of -b

1. b 
2. flips all digits of b
3. add 1




# Lec2 Data Representations + Machine Languages

##### Converting out of a 2's complement value
###### Approach 1
- a negative value will have implement subtraction set to 1

###### Approach 2
1. treat numbers as unsigned binary (obtain decimal value)
2. if the first bit is 0, done
3. if not, subtract 2^n: N

##### Comparison of two numbers
Requires signed/unsigned comparison

#### ASCII Encode
- 7 bits encoding

#### Unicode


## Machine Languages
- set of machine instructions
- **Machine instructions**: sequence of bits that have exactly one meaning on a particular architecture
### MIPS 32/64
- 32 bits instruction
#### MIPS Processor
##### Control Unit
- fetch/execute instructions
- timing unit
- communicating with peripheral
##### ALU
- operations/comparisons

##### Memory
- Register->Fast
- RAM->Slower

##### 32 general Purpose Registers
- 32 bit registers
- Need 5 bits to mention any register (2^5=32)
##### Special
- $0 always holds the value 0
- $30  
- $31      -->both addresses to RAM


#### RAM
- slower
- a lot more memory: 2^9 bytes
- visualize as a byte array

Often we use addresses that are multiples of 4

	     --------
	0x00 |32bits|
	     --------
	0x04 |      |


Operations can only be performed on values in registers

- Might need to load data from RAM
-               store data back to RAM

---------------------------------

**Q**: A program has a code section of other data sections. How does the processor know which part of the memory is code?

**A**: It does not.

**Q**: How does the processor execute a program?

**A**: 
#### Special Register PC (Program Counter) 

- 32 bits
- Contains the address of the *next* instruction to execute

#### Loader
A program called the loader loads the program in memory of sets PC to the start address of where the code is

Conventions: program is loaded at start address 0, then PC <- 0

#### Fetch / Execute Cycle (in hardware)
- Implemented in hardware

PC <- 0

	loop {
		IR<-MEM[PC]; //fetches 32 bit into instruction register
		PC = PC + 4;
		decode/execute
	}  
(infinite loop)

How does a program terminate?

- The loader places the "return address" in $31
- To Stop, a program must jump to this return address


### Machine Instructions
- 18 instructions
- 32 bits

#### Add instruction

	$d = $s + $t    /   add $d, $s, $t
	000000 sssss ttttt ddddd 00000 100000 (op code)

Add value in 5 with value in 7, store the result in $3

	add $3, $5, $7
	000000 00101 00111 00011 00000 100000
	        $5    $7    $3
    In Hex: 0x00A71820

Copy value stored in $3 to $8

	add $8, $3, $0
	000000 00011 00000 01000 00000 100000
	In Hex: 0x00604020
Clear a register
	
	add $3, $0, $0  // $3 <- 0

#### Jump `jr` Instruction



	jr $s
	000000 sssss 00000 00000 00000 001000
	                               opcode

- Sets PC to $s: Treat the value in $s as an address
- Recall that PC is set to the next instruction to execute

To exit:
	
	jr $31
	000000 11111 00000 00000 00000 001000
	To Hex: 0x03E00008

#### Sub Instruction

	sub $d, $s, $t
	$d = $s - $t
	000000 sssss ttttt ddddd 00000 100010

Negate a number 
	
	sub $1, $0, $1