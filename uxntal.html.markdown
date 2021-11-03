---
language: uxntal
contributors:
    - ["Devine Lu Linvega", "https://wiki.xxiivv.com"]
filename: learnuxn.tal
---

Uxntal is a stack-machine assembly language targeting the
[Uxn virtual machine](https://wiki.xxiivv.com/site/uxn.html).

Stack machine programming might look at bit odd, as it uses a postfix notation, 
which means that operators are always found at the end of an operation. For 
instance, one would write `3 4 +` instead of `3 + 4`.

The expression written `(5 + 10) * 3` in conventional notation would be
written `10 5 + 3 *` in reverse Polish notation.

```forth
( This is a comment. )

(
  All programming in Unxtal is done by manipulating the stack.

  The following examples represent the state of the stack using comments, for
  example:

    (01 02 03)
            ^--- top of stack (TOS)

  Represents a stack with 3 bytes, the last element pushed in being 03
  (hexadecimal).

  Forth style comments are used to document the stack inputs (pops from the
  stack) and stack outputs (push onto stack) of an operator, representing the
  state of the TOS (Top Of Stack):

  An operator that takes two bytes and returns the sum as a byte:

  ( a b -- a+b )
     |______⋮____ input (pops from stack)
            ⋮………… output (push onto stack)

  Other ways to represent the exapmle above:

  Implicit types:
  ( a b -- c )

  Explicit types:
  ( a^ b^ -- c^ )
  ( a* b* -- (a+b)* )

  In this tutorial all signatures will be fully typed for clarity sake, notice
  that types are implicit most of the time, depending if Short Mode is used,
  however not all Short Mode operators push or pop short values to the stack,
  which is when this convention is most useful.

  An operator that takes no inputs and has no outputs is documented in this way:

  ( -- )

  Symbol | Meaning
  ----------------------------------------------------------------------------
  bool   | The byte 00 (false) or any other 8-bit value (true).
  a      | An 8-bit or 16-bit value, depending upon whether Short Mode is set.
  a^     | An 8-bit value, regardless of whether Short Mode is set.
  a*     | A 16-bit value, regardless of whether Short Mode is set.

  Operator opcodes will be included in brackets along with their description,
  example: [0x01].

  Whitespace is used to delimit tokens and is also significant around brackets
  and braces, but not parenthesis (comments):

  |10 @Console [ &vector $2 &read $1 &pad $5 &write $1 &error $1 ]  ( ok )
  |10 @Console [&vector $2 &read $1 &pad $5 &write $1 &error $1 ]   ( error )
  |10 @Console [ &vector $2 &read $1 &pad $5 &write $1 &error $1]   ( error )

  %DEBUG { ;print-hex JSR2 #0a .Console/write DEO }  ( ok )
  %DEBUG {;print-hex JSR2 #0a .Console/write DEO }   ( error )
  %DEBUG { ;print-hex JSR2 #0a .Console/write DEO}   ( error )

  Uxn has 32 opcodes, each opcode has 3 possible modes

  The modes are:
    [2] The Short Mode consumes two bytes from the stack.
    [k] The Keep Mode does not consume items from the stack.
    [r] The Return Mode makes the operator operate on the return-stack.

  Modes can be combined: 2k, 2r, 2kr, kr.

  Uxntal is case sensitive, built-in operators are uppercase modes are lowercase.

  ADDk  ( ok )
  ADDK  ( error )
  addk  ( error )
)

( ---------------------------------------------------------------------------- )


( OPERATORS )
( Stack )

(   BRK  ( -- )

  Break: Halts the program, further instructions will not be run. [0x00] )
BRK

(   LIT  ( -- a^ )
    LIT2 ( -- a* )

  Literal: Pushes the next value seen in the program onto the stack. [0x00]

  This opcode is technically just BRK but with Keep Mode always set. )
LIT 01 LIT 02  ( 01 02 )
LIT 1 LIT 2    ( 01 02)

( Literal syntax for LIT uses # symbol. )
#1 #2       ( 01 02 )
#01 #02     ( 01 02 )
LIT2 0102   ( 01 02 )
LIT2 01 02  ( 01 02 )
LIT2 1 2    ( 01 02 )
#0102       ( 01 02 )

(   INC  ( a^ -- (a+1)^ )
    INC2 ( a* -- (a+1)* )

  Increment: adds 1 to the value a. )

#1 INC          ( 02 )
LIT 1 INC       ( 02 )
#0001 INC2      ( 00 02 )
LIT2 0001 INC2  ( 00 02 )

(    POP  ( a^ -- )
     POP2 ( a* -- )

   Pop: Removes the topmost value from the stack. [0x02] )

#1234 POP   ( 01 02 03 )
#1234 POP2  ( 01 02 )

(    DUP   ( a^ -- a^ a^ )
     DUP2  ( a* -- a* a* )

   Duplicate: Duplicates the topmost value on the stack. [0x03] )
#1234 DUP   ( 12 34 34 )
#1234 DUP2  ( 12 34 12 34 )

(    NIP   ( a^ b^ -- b^ )
     NIP2  ( a* b* -- b* )

   Nip: Removes the second-topmost value from the stack. [0x04] )
#1234 NIP          ( 34 )
#1234 #5678 NIP2   ( 56 78 )
( Using Keep Mode, same as DUP/DUP2. )
#1234 NIPk         ( 12 34 34 )
#1234 #5678 NIP2k  ( 12 34 56 78 56 78 )

(    SWP   ( a^ b^ -- b^ a^ )
     SWP2  ( a* b* -- b* a* )

   Swap: Swaps the first- and second-topmost values on the stack. [0x05] )
#1234 SWP          ( 34 12 )
#1234 SWPk         ( 12 34 34 12 )
#1234 #5678 SWP2   ( 56 78 12 34 )
#1234 #5678 SWP2k  ( 12 34 56 78 56 78 12 34 )

(    OVR   ( a^ b^ -- a^ b^ a^ )
     OVR2  ( a* b* -- a* b* a* )

   Over: Duplicates the second-topmost value on the stack. [0x06] )
#1234 OVR          ( 12 34 12 )
#1234 OVRk         ( 12 34 12 34 12 )
#1234 #5678 OVR2   ( 12 34 56 78 12 34 )
#1234 #5678 OVR2k  ( 12 34 56 78 12 34 56 78 12 34 )

(    ROT   ( a^ b^ c^ -- b^ c^ a^ )
     ROT2  ( a* b* c* -- b* c* a* )

   Rotate: Rotates the three topmost stack values to the left, wrapping around.
           [0x07] )
#1234 #56 ROT            ( 34 56 12 )
#1234 #56 ROTk           ( 12 34 56 34 56 12 )
#1234 #5678 #9abc ROT2   ( 56 78 9a bc 12 34 )
#1234 #5678 #9abc ROT2k  ( 12 34 56 78 9a bc 56 78 9a bc 12 34 )

( ---------------------------------------------------------------------------- )

( Logic )

(    EQU   ( a^ b^ -- bool^ )
     EQU2  ( a* b* -- bool^ )

   Equals: Pushes 01 to the stack if a==b, 00 otherwise. [0x08] )
#1 #0 EQU          ( 00 )
#0101 EQU          ( 01 )
#0100 EQUk         ( 01 00 00 )
#0001 #0000 EQU2   ( 00 )
#0001 #0000 EQU2k  ( 00 01 00 00 00 )

(    NEQ   ( a^ b^ -- bool^ )
     NEQ2  ( a* b* -- bool^ )

   Not Equals: Pushes 01 to the stack if a!=b, 00 otherwise. [0x09] )
#1 #0 NEQ          ( 01 )
#0101 NEQ          ( 00 )
#0100 NEQk         ( 01 00 01 )
#0001 #0000 NEQ2   ( 01 )
#0001 #0000 NEQ2k  ( 00 01 00 00 01 )

(    GTH   ( a^ b^ -- bool^ )
     GTH2  ( a* b* -- bool^ )

   Greater Than: Pushes 01 if a > b, 00 otherwise. [0x0a] )
#1 #0 GTH          ( 01 )
#0101 GTH          ( 00 )
#0100 GTHk         ( 01 00 01 )
#0001 #0000 GTH2   ( 01)
#0001 #0000 GTH2k  ( 00 01 00 00 01 )

(    LTH   ( a^ b^ -- bool^ )
     LTH2  ( a* b* -- bool^ )

   Greater Than: Pushes 01 if a > b, 00 otherwise. [0x0b] )
#1 #0 LTH          ( 00 )
#0101 LTH          ( 00 )
#0100 LTHk         ( 01 00 00 )
#0001 #0000 LTH2   ( 00 )
#0001 #0000 LTH2k  ( 00 01 00 00 00 )

( ---------------------------------------------------------------------------- )

( Control Flow )

(    JMP   ( a^ b^ -- bool^ )
     JMP2  ( a* b* -- bool^ )

   Jump: Sets the program counter to the address addr. The address is relative normally, and absolute if Short Mode is set. [0x0c] )
#1 #0 LTH          ( 00 )
#0101 LTH          ( 00 )
#0100 LTHk         ( 01 00 00 )
#0001 #0000 LTH2   ( 00 )
#0001 #0000 LTH2k  ( 00 01 00 00 00 )









#12 #34 ADD ( 46 )
#12 #34 ADDk ( 12  34  46 )

( The modes can be combined )

#1234 #5678 ADD2k ( 12  34  56  78  68  ac )

( The arithmetic/bitwise opcodes are:
    ADD SUB MUL DIV
    AND ORA EOR SFT )

( New opcodes can be created using macros )

%MOD2 { DIV2k MUL2 SUB2 }

#1234 #0421 MOD2 ( 01  b0 )


( A short is simply two bytes, each byte can be manipulated )

#1234 SWP ( 34  12 )
#1234 #5678 SWP2 ( 56  78  12  34 )
#1234 #5678 SWP ( 12  34  78  56 )

( Individual bytes of a short can be removed from the stack )

#1234 POP ( 12 )
#1234 NIP ( 34 )

( The stack opcodes are:
    POP DUP NIP SWP OVR ROT )

( ---------------------------------------------------------------------------- )

( To compare values on the stack with each other )

#12 #34 EQU ( 00 )
#12 #12 EQU ( 01 )

( Logic opcodes will put a flag with a value of either 00 or 01 )

#12 #34 LTH 
#78 #56 GTH 
    #0101 EQU2 ( 01 )

( The logic opcodes are:
    EQU NEQ GTH LTH )

( ---------------------------------------------------------------------------- )

( Uxn's accessible memory is as follows: 
    256 bytes of working stack
    256 bytes of return stack
    65536 bytes of memory
    256 bytes of IO memory )

( The addressable memory is between 0000-ffff )

#12 #0200 STA ( stored 12 at 0200 in memory )
#3456 #0201 STA2 ( stored 3456 at 0201 in memory )
#0200 LDA2 ( 12  34 )

( The zero-page can be addressed with a single byte )

#1234 #80 STZ2 ( stored 12 at 0080, and 34 at 0081 )
#80 LDZ2 ( 12  34 )

( Devices are ways for Uxn to communicate with the outside world
    There is a maximum of 16 devices connected to Uxn at once
    Device bytes are called ports, the Console device uses the 10-1f ports
    The console's port 18 is called /write )

%EMIT { #18 DEO }

#31 EMIT ( print "1" to console )

( A label is equal to a position in the program )
@parent ( defines a label "parent" )
    &child ( defines a sublabel "parent/child" )

( Label positions can be pushed on stack )
;parent ( push the absolute position, 2 bytes )
,parent ( push the relative position, 1 byte )
.parent ( push the zero-page position, 1 byte )

( The memory opcodes are:
    LDZ STZ LDR STR
    LDA STA DEI DEO )

( ---------------------------------------------------------------------------- )

( Logic allows to create conditionals )

#12 #34 NEQ ,skip JCN
    #31 EMIT
    @skip

( Logic also allows to create for-loops )

#3a #30
@loop
    DUP EMIT ( print "123456789" to console )
    INC GTHk ,loop JCN
POP2

( Logic also allows to create while-loops )

;word
@while
    LDAk EMIT
    INC2 LDAk ,while JCN
POP2
BRK

@word "vermillion $1

( Subroutines can be jumped to with JSR, and returned from with JMP2r )

;word ,print-word JSR
BRK

@print-word ( word* -- )
    @while
        LDAk EMIT
        INC2 LDAk ,while JCN
    POP2
JMP2r

@word "cerulean

( The jump opcodes are: 
    JMP JCN JSR )
```

## Ready For More?

* [Uxntal Lessons](https://compudanzas.net/uxn_tutorial.html)
* [Uxntal Assembly](https://wiki.xxiivv.com/site/uxntal.html)
* [Uxntal Resources](https://github.com/hundredrabbits/awesome-uxn)
