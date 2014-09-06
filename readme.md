# Description of the c16

## Goals:
 - Simple
 - Consistent
 - Shared logic between similar instructions
 - Make common operations easy and uncommon operations possible

## Features:
 - 16bit word size
 - 8 16bit registers
 - Load/store architecture
 - Word addressable memory
 - Lack of predicates or condition flags
 - MIPS-like conditions
 - Arithmetic instructions with shifts
 - Arithmetic instructions with Immediates
 - Register relative addressing and branching
 - PC relative addressing and branching


## Registers

    R0-R6 - general purpose registers
    N - constant -1 register
    PC - unaddressable program counter


## Instruction Encoding

    Type A:
    | op |f| rd| ra| rb|sh|
    |xxxx|x|xxx|xxx|xxx|xx|
    
    Type B:
    | op |f| rd| ra| imm5|
    |xxxx|x|xxx|xxx|xxxxx|
    
    Type C:
    | op |f| rd|  imm8  |
    |xxxx|x|xxx|xxxxxxxx|
    
    op - 4 bit opcode encodes 16 general opcodes
    f - format bit indicates format of instruction
    rd - destination register
    ra - register a
    rb - register b
    sh - 2 bit shift encoding
      00 - no shift
      01 - left shift 1 bit
      10 - logical right shift 1 bit
      11 - arithmetic right shift 1 bit
    imm5 - sign extended 5 bit immediate or 4 bit shift with arith/logic bit
    imm8 - sign extended 8 bit immediate
    

## Instruction Set

    or - bitwise or
    xor - bitwise xor
    and - bitwise and
    andn - bitwise and with not
    add - integer addition
    sub - integer subtraction
    slt - set if less than
    sltu - set if less than unsigned
    
    mul - multiply
    div - divide
    shl - shift left
    shrl - shift right logical
    shra - shift right arithmetic
    
    ld - load
    st - store
    lea - load effective address
    call - branch and store return address
    breq - branch if zero
    brne - branch if not zero


## Interesting encodings

    or r0,$0,r0     => nop (0x0000)
    brne n,$-1      => halt (0xffff)
    
    and n,imm5,rd   => set imm5,rd
    and ra,$-1,rd   => mov ra,rd
    lea $0,rd       => mov pc,rd
    
    xor ra,$-1,rd   => not ra,rd
    
    call ra+imm5,n  => br ra+imm5
    call imm8,n     => br imm8


## Professor Gheith's Questions

##### How do we do function calls?
The call opcode stores the return address in a specified register and there is 
no special stack register, so function calls are defined entirely by convention. 

Here is an EABI similar to ARM's:
 * Store return address in R6
 * R0-R3 used to pass arguments
 * R0 used to pass pointer to argument list if there are more than 4
 * R4,R5 must be saved by the callee
 * R0 used for return value

##### How many register file read/write ports?
For most instructions there are maximum 2 register reads and 1 register write.
As usual, mul/div are the exceptions and can write to 2 registers in one 
instruction. However, it never makes sense for this to be the same register 
and can be performed in a single cycle.

##### How many memory read/write ports?
As a word-addressable load/store architecture, there is just 1 exclusive, 
word-sized, read/write port. Aside from fetches, only ld and st can access memory.

##### How do we implement arrays / structures / pointers / function pointers?
 * Arrays can be accessed using add with builtin shifts to find 8bit, 16bit, 
   and 32bit addresses in arrays, followed by a ld from register.
 * Structs can be accessed using the ld from register with imm5 offset. 
 * Pointers can be implemented with the register based load instructions
 * Function pointers can be implemented with the register based call intructions

##### How do we implement control structures: if / while / switch?
 * If/While statements can be implemened using the breq/brne instructions to 
   branch to relative addresses on the state of a register. This can be 
   combined with stl/stlu/sub to branch on other conditions.
 * Switch statements can be implemented with a jump table looked up using 
   lea and jumped through using the register based call instruction 
   (without saving the return address).

##### How do you implement the stack?
Much like function calls this is also up to convention, and even the above 
EABI is stack agnostic. Any register can be used to implement a stack using 
manual add and sub instructions, and the register based ld/st with imm5 offset.

##### How easy is it to detect / resolve inter-instruction dependencies?
There are no dependencies on condition codes or implicit operations in 
instructions. The only dependencies are the general purpose registers.

##### How easy is it to identify operations / operands?
The operation and format can be determined from only the first 5 bits 
in the instruction and should simplify decoding. Up to 3 used registers are 
specified in consistent positions through the instruction set.

##### How easy is it to identify branch targets?
All branch/call instructions use a consistent format of 
register + imm5 or pc + imm8. 

##### Implicit vs. explicit operands / coupling?
All operands are explicit for every instruction and no registers 
are implicitly used.

