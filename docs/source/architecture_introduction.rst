.. _architecture_introduction:

=====================================
Introduction to Computer Architecture
=====================================
This will go over how a computer executes binary.  The reason why difference processors have difference instruction sets, and we need to compile the same program for different processors, is becuase the binary representation of instructions depends on the actual silicon of the procesors.

Hardware
========
Very briefly, processors are a large number of logic gates.  These can be relays, transistors, vacuum tubes, whatever, anything that takes in (at least) one input and produces an output.  There are single input gates like inverters which take a 1 and make it a 0.  We will represent these as input/output tables:

Inverter
--------
======  ======
Inputs  Output
------  ------
  A       ~A
======  ======
  0       1
  1       0
======  ======

The above is an inverter for the signal A.  The output stands for the "operation", where "`" means "not".  So the output is "not A".

While this is a logic gate, it's also an architecture.  It takes single bit programs (0, 1) as the input and turns them to single bit output (1, 0).  It doesn't take any instructions as input, only data.  Pretty boring.

Let's show examples for a few more gates before building up to the separation of instructions and data.

And
---
======  ======  ======
    Inputs      Output
--------------  ------
  A       B      A*B
======  ======  ======
  0       0       0
  0       1       0
  1       0       0
  1       1       1
======  ======  ======

A and B is 1 if and only if both A and B is 1.  Note that A and B is represented with multiplication.  Makes sense, A*B = 1 if and only if both A and B is one.

Or
--
======  ======  ======
    Inputs      Output
--------------  ------
  A       B      A+B
======  ======  ======
  0       0       0
  0       1       1
  1       0       1
  1       1       1
======  ======  ======

A or B is 1 if either both A or B is 1.  Note that A and B is represented with addition.  Makes some sense, since 0+0 is 0, 0+1 is 1, and 1+0 is .  1+1 is 2, which breaks the analogy. Just hang on to this for now.

XOr
---
======  ======  ========
    Inputs       Output
--------------  --------
  A       B      A XOR B
======  ======  ========
  0       0         0
  0       1         1
  1       0         1
  1       1         0
======  ======  ========

Exclusive or is similar to Or, but it's only 1 either A or B is 1, but not both.  It's eclusive.


Ok, so now we have 3 gates, AND, OR and XOR that take two inputs.  These are 3 different operations that could run, each program take two binary inputs (A, B) and produces a single output bit that's either 1 or 0.

We have 1 gate which is single input, and 3 gates which are two inputs for a total of 4 operations.  The question is now, how do we select which of these operations to run?  We have to represent the selection of the operation, and this isn't encoded in the data bits.  The data bits don't care what operation is run, and have no influence on it.  So we need to represent this information in an instruction.

Instructions
------------
Quick quiz, hot shot.  You have 4 operations.  How can you represent them in binary?

There's only one answer:  with 2 bits.  The most information we need to represent at once is 2 bits.  We'll get more into that shortly, just keep it in mind.

We are designing this architecture, so we get to decide which combination of 1s and 0s represent reach instruction.  

::

    00 - Invert
    01 - And
    10 - Or
    11 - XOr

We need to include the inputs into the instructions as well, some how.  So we'll create a format for the instruction.

    `[nn][ii]`
        the "n" bits are the codes for the instructions, and the "i" bits are the inputs to the instructions.

We have one issue to note: the invert instruction only takes one input.  But we have decided the format for all instructions is 4 bits.  What do we do?

We also decide the format specific to the invert instruction.  We decide to only use the first input bit. We can easily decide to only use the second input bit.  But we make the decision to use the first input bit.

So, we now can represent our 4 logic operations as instructions.

We can enumerate every single instruction like so:

===========  =======  =======  =========  ======
Instruction  Input A  Input B  Operation  Output
===========  =======  =======  =========  ======
    00          0        0       ~A          1
    00          0        1       ~A          1
    00          1        0       ~A          0
    00          1        1       ~A          0
    
    01          0        0      A*B          0
    01          0        1      A*B          0
    01          1        0      A*B          0
    01          1        1      A*B          1
    
    10          0        0      A+B          0
    10          0        1      A+B          1
    10          1        0      A+B          1
    10          0        1      A+B          1
    
    11          0        0     A XOR B       0
    11          0        1     A XOR B       1
    11          1        0     A XOR B       1
    11          1        1     A XOR B       0
===========  =======  =======  =========  ======

These statements in binary all have specifically defined results.

So we can write programs now which are sequences of binary instructions.  In order for these to make sense we had to make a number of decisions which aren't related to operations, but are related to how the machine decodes the binary instruction to select the operation, input and output.

The instructions have a format: the first two bits encode which operation to use, and the next two bits are the inputs to the operations.  The operations themselves have to use the format to decide what inputs are used, and how they are used.  We made all those decisions.

Assembly Language
-----------------

Saying "INVERT", "AND", "OR", "XOR" is compilated.  Let's make up a language to refer to each instruction that uses at most 3 letters.  Later i might decide to use more than 3 letters, but for now lets use

=========  ===   =========   =======
Function   ASM   Operation   Op Code
=========  ===   =========   =======
Invert     NOT     ~A          00
And        AND     A*B         01
Or         OR      A+B         10
XOr        XOR    A XOR B      11
=========  ===   =========   =======

Wow, much easier.  Now we have The human readable function that performs a logical operation, an assembly language instruction for it, and an op code which is the binary representation of the same thing.  

Assembly language a "human" readable name for the OP code, and will use register names (covered below) instead of addresses.  It's basically reading the binary directly though, as there as a direct, line for line, op code to ASM correspondance between the assembly and binary.  So yeah, since we're designing this architecture, we are ALSO deciding on the assembly language used to program it, becuase they are the same thing.   One is binary for processors, one is "human" readable.  For "Humans".  Yeah. So.

Registers
=========
What we haven't explained is how to use the outputs of one instruction as the inputs to another instruction.  This requires registers, which store values.  It also means we need more instructions.  *Oh no, but we don't have any more bits!*  Well, we'll add more.   We're the designers.  We need registers to make useful programs, and to use registers, we need to be able to store values in them and move values between them.

We also need to store the whole program somewhere.  Flashing firmware onto a chip means storing the binary instructions on the chip.   Executing the instructions means knowing which instruction is first, which is next, and loading the instruction.  All this requires not just memory, but specific locations in memory.  Locations in memory are also registers.

What is a register?
-------------------
A register is something that can store a value.  As an example with our current 4 bit instruction set, we need at least 4 bits for each instruction.  To run a program with more than one instruction, we need a list of registers to store each 4 bit instruction.  We also need a way to reference each location in the list of registers so we know which insturction to run.  How do you know which instruction is the first instruction?  How do you know which instruction is the next instruction?  So many questions.

Accessing Registers
-------------------
We'll go more into how registers are defined as memory, but we can think of special locations which have codes for their location.  What kind of codes?  well binary codes, we know that much.  Becuase this is a computer and everything is in binary.  Right?  RIIIIIIIGGGHHHHTTTT??????

For example, we already have a concept for inputs A and B from the logic gate section, and those operations have an output, which we'll call Z.  We don't yet know the details, but we can say "Register A is located at 0x00, Register B is located at 0x01, and register Z is located at 0x10".  Whatever that means isn't important just yet, what's important is that we can identify the register by a binary number.

===========  =============  =====
Reg Address  Register Name  Value
===========  =============  =====
0x00                A        XX
0x01                B        XX
0x10                Z        XX
===========  =============  =====

Now we can talk about storing a value into A, B, or Z.  XX here means we don't have a value defined yet.

Store
_____
We can instruct the processor to place a value in a register with a new operation, called Store.
Store the value of 1 in register A can be translated to
STO 1 00
We don't have a code for the STO operation yet seince we're out of bits.  Let's increase the number of bits for operations to 4, and set 0100 to STO.

0100 1 00 is Store 1 in register A.

While we've added bits to the op codes, we have not changed the format of the instruction set.  It's still 

`[op code] [input 1] [input 2]`

But, we have changed 2 things.  We need 2 bits for input 2 now, and we have added a different interpretation.  Input 2 is now a register address, rather than a value.  Input 1 here is still a value.  Input 2 requires 2 bits, however, to access all the registers.  

Note the difference here between the value 1 in `[input 1]`, and the address of the register in which we store the value in `[input 2]`.

Of course we should add bits to `[input 1]`, but lets see specifically why that's the case.

Move
____
The move instruction will take a value from one register and put it in another register.

::
    
    MOV Z A

Has the effect of taking the value in register Z and putting it in register A.

We'll assign it the op code 1000.  We could have select 0101, but I'm going to reserve this address for now.  This instruction doesn't take any values as inputs, only register addresses.  Again, it still has the form

`[op code] [input 1] [input 2]`

But again with the interpretation that `[input 1]` is a register address and so is `[input 2]`, so we need to increase the number of bits allowed in inputs to cover register addresses.

Instruction format update
_________________________
How many bits do we need for input 1 and input 2?  Well, we have 3 registers right now, and we need at least 2 bits for those addresses.  This means we need `[input 1]` and `[input 2]` to handle 2 bits each.  This is another change to the format of our instruction set.  We need 4 bit op codes, and 2 bit inputs.
[oooo][ii][ii]

Note that we don't have to change the behavior of the other operations.  They just have '0's added in front of their op codes.  However, just like the invert operation needed to decide how to manage the instruction format, and we decided to ignore `[input 2]`, now we have to decide which bits of the inputs will be used in those operations.  

The right approach is to use operations which can manage all the input bits.  But that raises some other interesting questions, but let's leave those for now.  (What if I only want to AND specific bits??)

Saving Results
--------------
We established a Z register, and we saw that we can move values from it so A and B.  But Z is going to be a very special register.  It will always be the result from the last instruction.  Back in Instructions we saw all the possible inputs and outputs, but we didn't see how to use the output.  The output could have been a GPIO pin, for example, but what's most useful is if we have access to the value in a register.  We select Z as the register which stores the Output from each instruction.

When we do something like:

::

    AND 1 1


the result is 1.  So after that instruction executes, Z will have the value 1.  If we want to store that value, we can put it in register A with a MOV instruction.  So the program becomes

::

    AND 1 1  # Z becomes 1
    MOV Z A  # 1 is moved into A

I could also AND 1 and 0 and store that in B

::

    AND 1 0  # Z becomes 0
    MOV Z B  # 0 is moved into B

But how to I AND A and B?

Well, I need instructions for that.  I need instructions that know to look in registers with addresses `[input 1]` and `[input 2]` instead of using `[input 1]` and `[input 2]` as values.  We'll name those ASM instructions  AND_R, OR_R, and XOR_R, where the _R means "the inputs are registers, not values.  We need to do this becuase there is nothing else which can make the distinction.   We'll conveniently forget about NOT_R for now, fabricate the chip, and let the software guys deal with that problem [1]_.

Now we can do the following:

::
    
    STO 1 A     # Store the value 1 in A
    STO 0 B     # Store the value 0 in B
    AND_R A B   # AND the values in registers A and B.
                # Z becomes the result of 1 AND 0, or 0

We could assign these instructions similar to the previous ones which acted only on values.

=======  =========
Op code  Operation 
=======  =========
 0101     AND_R
 0110     OR_R
 0111     XOR_R
=======  =========

What's neat about this scheme, and why I reserved the op codes 01XX, is that now OP codes are grouped by how they operate.  00XX op codes work on values, and 01XX op codes work on registers.  This can do more than just organize a table of instructions, it can inform the processor to get ready for certain kinds of operations.

The first two bits can mean "interpret inputs ad addresses" and the last two bits can still mean "operation".

This is not the only way to solve this problem.  I could make all the instructions ONLY use registers, except for STO which puts values in registers.  Then I could designate register A and register B as `[input 1]` and `[input 2]` for all operations, and the operations always take the values in those registers.

But this means that performing a logical operation on values will take 3 instructions:

::

    STO 1 A  # Store value 1 in A
    STO 0 B  # Store value 0 in B
    AND      # Use the new AND operation which always perform AND on the value in 
             # A and the value in B.  Z becomes 0, the result of 1 AND 0.



We lose the "fast" insstructions which don't requre registers for the benefit of having a smaller instructionset.  It's a trade off, do you expect to do more "fast" (also called immediate" instructions where the values are stored in the program, or do you need to to more chained operations which use the results from previous calcualtions?

Things are getting complicated, and we have had to make a number of decisions which impact the format of our instructions, and these decions impact how many computations need to occur to perform logical operations.  However, now we can do something closer to programming, where we can use the results of operations as the inputs to new operations. 

Memory
======
Memory is a list of registers, usually in a continuous block.

We have 3 special function registers right now, A, B and Z, at addresses 00, 01, and 10.  Let's assume we're going to more.  We obviously need to increase our address space (the number of bits required to address each memory location), so let's do that to 8 bits.  

If we have 8 bit addresses, we have 2^8 = 256 registers total.  We'll reserve addresses 0-15 for special functions, currently A, B, an Z, and 17-256 can be whatever we want.

In reality, processors have many special function registers to store settings and results.  They all work in the same way, but let's look at ones that are specific to executing a program.

For now, let's assume we've loaded a program into memory.  We'll cover the details of how that's done later.  But you can safely assume it looks a lot like 

STO value register

where "value" is the binary number which represents an instruction and 'register' is a register address, which are now 8 bits.  

This tells you how at least how wide the registers reallly need to be in memory to handle our instruction format, `[op code] [input 1] [input 2]` or `[oooo][iiiiiiii][iiiiiiii]` in the case of 4 bits of op codes, and op codes which can handle 2 registers as inputs.  Wait, that's much bigger than what we were dealing with before!  But that's 20 whole bits!

Well, we could make other decisions, like how instructions are stored.  Maybe they are stored in 2 registers instead of 1.  Or maybe 3 regsiters instead.  Up to us really.

Program Memory
--------------
There isn't a strict need to separate program memory.  You might want to for many reasons, but for now lets talk about a processor that has one register file.  It knows a little bit about itself, but not much.  One special register we'll need is the start of the program.  Let's use register 4 for that.

Once the chips knows to always start the program from register 4, it can happily march down the register file.  It doesn't even need to know, in advance, when to end.  It can end when it runs out of instruction space.  "end" here means "crash" "halt" or "stop", whatever you prefer.

However, we want to loop when we're done.  That means we need some way to tell the processor to go back to the beginning.  We know the beginning is address 16 (that's what we designed).  So we have a number of things we can do, but they all require more instructions.


Automatic
_________
As the processor designers, we could determine that any time the processor reaches the end of program memory, it automatically does something.  Maybe it reboots, clearing memory and starting from scratch.  Maybe it doesn't reboot, but just starts again from the beginning on the program memory.  In this case, the special function registers like A, B, and Z might retain their values, and it might cause our program to execute differently.  Or not.  (this is why initializing variables in your program is always a good idea, you don't want to depend on the chip designer).


Goto
____
oh, goto.  We love you.  Maybe we'll introduce a new instruction called "goto" which takes a a register addreses as it's input.

::

    GOTO 16

of course the instruction format is STILL `[op code] [input 1] [input 2]` but the GOTO instruction just ignores `[input 2]`.
What's going to 16 actually mean?  The processor will perform the instruction at register address 16 after this instruction.  GOTO is clearly useful, but do we really need it to manage this case?

No.  In fact, we can re-use the STO operator, but we need a new special function registers called the Program Counter, or the PC.

Program Counter
_______________
The program counter determines which instruction will execute next.  It points to the address of the next instruction.  If we are ticking away happily, it increases by 1, because that is the location of the next instruction.  However, it's a register just all the other registers.  We use STO value PC to change how our program executes.  Now don't need a special instruction like GOTO, we can re-use the instruction STO at the cost of one special functino register.

So, for example, on reset, the chip will always execute a 

::

    STO 16 PC

instruction.  But what's actually happening?

Instruction Cycle
=================
The chip has an instruction cycle, which is a bunch of logic that determines and executes all the steps it takes.  It's the stuff that's doing the actual processing.  When it's time to perform the next instruction (a clock cycle, maybe a couple), it will fetch the instruction from the memory location in the PC.  A simplistic cycle will decode the operation based on the instruction format, decide how to interpret the inputs based on the instruction, execute the instruction, store the result in Z, and then incremenet the PC by + 1, repeating this cycle.  However, what if the PC was written to during the cycle?  Better to increment the PC right after the FETCH.  Why?  Becuase I said so. I mean we said so.  We just designed our instruction pipeline.

So the steps are 
::

    FETCH
    INCREMENT PC
    DECODE
    EXECUTE

pretty fancy.  It's the "processing" that the processor does.

We want our cycle to work this way because it makes life slightly easier for programmers.  We could, for instance, increment the PC before the FETCH, or after EXECUTE, but then a programmer would have to do something like

::
    
    STO 15 PC

to start at the beginning.  The next cycle would first increment the PC to 16, then fetch the instruction at address 16, etc.  Makes more sense to store 16.

But what if I STO 3 PC ??
-------------------------
SEGFAULT.  Segmentation fault.  Memory Segmentation Fautlt.  Memory Segmentation is the seperation of memory into different areas, and we have 2 areas. Special function registers in addresses 0-15, and "other stuff" like program insturctions above 16.  You crashed the computer.  Thanks a lot, I'm not even done designing it yet.  So please, don't do that.


What else can I do with the PC though?
--------------------------------------
Well, if you see that manipulating the PC is like a GOTO, you can move around your program now based on the results of computations.  This is called branching, and covers things like flow control, conditionals, and functions.  It's a much deeper topic, which I'll cover somewhere other than an introduction to architecture.  Suffice to say, if you start jumping around, you need to pay attention to stuff.  Lots of stuff.  Heaps of stuff.  Stacks of stuff. Ok memory, you need to pay attention to what you're doing with and to memory, and when you do it.  If you write assembly directly, you will know which instructions end up in various addresses just by counting the instructions, so you can hard code these GOTOs.  But that's obviously a recipe for disaster.  Make a change and forget to update an address, and you get  behaviors like jumping into the middle of a different function!  What fun!


Putting it together
===================
I mean good luck, the above is a smattering of information about what a computer achitecture is.  Why is this important?  Because different architectures make these decisions differently.  They have different special function registers that mean different things, the op codes for similar instructions are not the same, they don't all implement the same instructions, and the have different numbers of bits available for addressing, operations, values, etc.  I didn't even explain endianness!  Did you know that 1100 can mean either 12 or 3?  Yah it can be read forwards or backwards!  I'm the designiner, i get to decide! AAHAHAHAHAHAHAHA


Whoopse, sorry, got a bit drunk on power there.  But there's no single rule which must be followed here.  Some decisions make processors really good at certain tasks compared to others, theres no one perfect solution.  The only thing to take away from this is that, while we can write C/C++ code for a large (huge) number of architectures, the binary which get spit out of the compilier is different for each architecture.

This is the real beauty of Arduino.  You don't need to care at all, it manages these differences for you, to a large degree.  You tell it what you bought, it figures out the rest.

I haven't really discussed how the op codes map to actual operations.  It's a little complicated, but it all boils down to logic.  For example, you could to all the operations at once, and select the ouput based on a multiplexer.  The op codes merely select the desired output from the multiplexer. Some parts of the processor work this way.  Suffice to say, it doesn't exactly matter here to make my point, but perhaps I'll get more into that in a new section at some point down the road.

.. [1] Always get the latest version of the datasheet and instruction set reference for your target architecture and read the errata!
