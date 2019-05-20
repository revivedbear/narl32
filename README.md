================================
==========  NARL16  ============
================================

NARL16 is a register & stack based 16-bit ISA.

In every instrucion cycle, the instruction at the location of PC is fetched, decoded and executed. Then the PC increments/changes.


Data
====

* NARL16 supports 16 bit signed/unsigned integers, known as a word.
* Signed values are encoded using two's compliment
* Floating point values are represented using IEE754 half precision.
* Memory mapped 64x64 display


Memory
======

NARL16 is capable of addressing 2^16 memory indexes

The following available registers are:
--------------------------------------

* $pc $sp $ia $cr $r1-12
* program counter, stack pointer, interupt address, cause register, registers 1-12

The memory layout is as follows:
--------------------------------

--------------------------
|   /\ Graph: 0xF000     |
|------------------------|
|  \/ stack:  0xE000     |
|------------------------|
|  /\ heap:   0x7000     |
|------------------------|
|  /\ Static: 0x6000     |
|------------------------|
|  /\ text:   0x2000     |
|------------------------|
|     System Reserved    |


Instructions
============

* Instructions are all 16 bit values using little endian

yyyyyxxxxx000000


Lower -> high bits:
* 6 bit opcode
* 5 bit val x => Always processed as the destination
* 5 bit val y => Always processed as the target

x & y can be a specifier for a register, memory address or an immediate value.

* 0-16 maps to registers
* 17 maps to push
* 18 maps to pop
* 19 maps to peek
* 20 maps to signed immediate
* 21 maps to unsigned immediate
* 22 maps to half precision IEE754 floating point
* 23 maps to memory

The next instruction after (when a numeric value is required) will be a full-word immediate value


Opcode table:
-------------

----------------------------------------------------------------------------------------------
| Value | Name | Description                                                                 |
|-------+------+-----------------------------------------------------------------------------|
|  0x0  | NOP  | Signifies the PC to increment and skip this instruction                     |                    
|-------+------+-----------------------------------------------------------------------------|
|  0x1  | SET  | Sets x to y                                                                 |   
|-------+------+-----------------------------------------------------------------------------|
|  0x2  | ADD  | Sets x to x+y. If there is an overflow set $cr to 0x1 and save PC in stack  |                                                                 
|-------+------+-----------------------------------------------------------------------------|
|  0x3  | SUB  | Sets x to x-y. If there is an underflow set $cr to 0x2 and save PC in stack |        
|-------+------+-----------------------------------------------------------------------------|
|  0x4  | MUL  | Sets x to x*y                                                               |
|-------+------+-----------------------------------------------------------------------------|
|  0x5  | DIV  | Sets x to x/y                                                               |
|-------+------+-----------------------------------------------------------------------------|
|  0x6  | AND  | Sets x to x AND y                                                           |
|-------+------+-----------------------------------------------------------------------------|
|  0x7  | OR   | Sets x to x OR y                                                            |
|-------+------+-----------------------------------------------------------------------------|
|  0x8  | XOR  | Sets x to x XOR y                                                           |
|-------+------+-----------------------------------------------------------------------------|
|  0x9  | NOT  | Sets x to NOT x                                                             |
|-------+------+-----------------------------------------------------------------------------|
|  0xA  | MOD  | Sets x to x mod y                                                           |
|-------+------+-----------------------------------------------------------------------------|
|  0xB  | REM  | Sets x to the remainder of x / y                                            |
|-------+------+-----------------------------------------------------------------------------|
|  0xC  | SRL  | Sets x to x SHIFT RIGHT LOGICAL y                                           |
|-------+------+-----------------------------------------------------------------------------|
|  0xD  | SLL  | Sets x to x SHIFT LEFT LOGCIAL y                                            |
|-------+------+-----------------------------------------------------------------------------|
|  0xE  | SRA  | Sets x to x SHIFT RIGHT ARITHMETIC y                                        |
|-------+------+-----------------------------------------------------------------------------|
|  0xF  | SLA  | Sets x to x SHIFT LEFT ARITHMETIC y                                         |
|-------+------+-----------------------------------------------------------------------------|
|  0xF1 | IEQ  | Perform next instruction only if x is equal to y                            |
|-------+------+-----------------------------------------------------------------------------|
|  0xF2 | INE  | Perform next instruction only if x is not equal to y                        |
|-------+------+-----------------------------------------------------------------------------|
|  0xF3 | IGE  | Perform next instruction only if x is greater than or equal to y            |
|-------+------+-----------------------------------------------------------------------------|
|  0xF3 | IGT  | Perform next instruction only if x is greater than y                        |
|-------+------+-----------------------------------------------------------------------------|
|  0xF4 | ILT  | Perform next instruction only if x is less than y                           |
|-------+------+-----------------------------------------------------------------------------|
|  0xF3 | ILE  | Perform next instruction only if x is less than or equal to y               |
|-------+------+-----------------------------------------------------------------------------|
|  0xF5 | IBS  | Perform next instruction only if x and y are both non zero                  |
|-------+------+-----------------------------------------------------------------------------|
|  0xF6 | INB  | Perform next instruction only if x and y are not both non zero              |
|-------+------+-----------------------------------------------------------------------------|
|  0xFA | JAL  | Push PC to stack and jump to the value in x                                 |
|-------+------+-----------------------------------------------------------------------------|  
|  0xFB | RTN  | Pop the stack and set PC to that address                                    |
|-------+------+-----------------------------------------------------------------------------|
|  0xFC | SYS  | System call the function with arguments on the stack                        |
|-------+------+-----------------------------------------------------------------------------|
|  0xFD | INT  | Call an interrupt, push PC then set it to $ia                               |
----------------------------------------------------------------------------------------------



Flags
=====

In the case of an error, the flag register $fr should be set to indicate the error.

Error codes:
------------

-------------------------------------------------------
| 0x1 | Overflow  | Indicates an arithmetic overflow  |
|-----+-----------+-----------------------------------|
| 0x2 | Underflow | Indicates an arithmetic underflow |
------+-----------+------------------------------------



Graphics
========

* NARL16 has a memory mapped region for graphics memory starting at base address 0xF000
* As