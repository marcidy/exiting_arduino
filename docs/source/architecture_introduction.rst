.. _architecture_introduction:

=====================================
Introduction to Computer Architecture
=====================================
A computer's architecture refers to how it will interpret 1s and 0s as instructions and data.  Binary code is just a long list of 1s and 0s.  The architecture defines how to interpret the 1s and 0s into instructions.  We'll detail this below, don't worry about understanding any details just yet.  

A binary is literally just a bunch of 1s and 0s, something like

::

    1010001111111110001001001111

An arcitecture will determine how to segrement that list, maybe into something like

::

    1010
    0011
    1111
    1110
    0010
    0100
    1111
    

and further to

::

    10 1 0
    00 1 1
    11 1 1
    11 1 0
    00 0 1
    01 0 0
    11 1 1

The architecture of a processor will take a string of binary instructions and make decisions about what to do.  An important decision is how to split up all the 1s and 0s into meaningful units.  To do that, it requires that the binary be in a specified format.  That format is defined by the people who design the architecture.  IF the binary is in the right format, the processor can execute the binary as a list of instructions.

This will go over how a computer executes binary.  Using the above as an example, a (imaginary for now) processor knows how to interpret the long list of 1s anda 0s in 4 bit chunks, and each 4 bits is further interpretted as a specific instruction.  

When we say different processors have different architectures, it's because they interpret binary differently.  They will chunk binary differently, and also interpret chuncks differently.  Using the same string of 1s and 0s, a different architecture may interprety the same binary like:

::

    1010 00 11
    1111 11 10
    0010 01 00
    1111

And here you see a problem without knowing anything yet about the details, the last line isn't the right length.  Not only that, but we can see that the different segments aren't going to mean the same thing as they are chunked differently.  This is why binary (aka machine code, a binary) for one architecture does not execute (at least not correctly) on another.  At a minimum it expects a certain length, and a we'll see a certain format for each chunk.

The way an architecture interpretats binary is labled an instruction set.  It defines how to chunk the binary and what to do with each chunk.  The same program compiled for different processors takes into account the processors instruction set, as the binary representation of instructions depends on the design of the procesors.

We'll also see how Assembly lanauge created and written, and how it corresponds to the binary the machine executes.  

We'll do that by designing an architecture as we go.  First let's focus on what the hardware can do, the available operations, and give them names, codes, and formats.

Hardware
========
Very briefly, processors are a large number of logic gates hard wired to respond to input in specific ways intended by the designer.  These logic gates can be relays, transistors, vacuum tubes, whatever, anything that takes in (at least) one input and produces an output.  

We'll call transforming from input to output an operation.  For the sake of this example, we'll say operations are hard-wired into the processor, like there's actually a set of logic gates to perform the operations we're about to describe. In many cases that is true, but the real answer is it's far more complex.

There are single input gates like inverters which take a 1 and make it a 0.  

We will represent these as input/output tables:

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

The above is an inverter for the signal A.  The output stands for the "operation", where "~" means "not".  So the output is "not A".

While this is a logic gate, we've decided it's also an operation, and we've created a chip which can do it, so it's also an architecture.  In a very minimal sense.  It takes single bit programs (0, 1) as the input and turns them to single bit output (1, 0).  It doesn't take any *instructions* as input, only data.  Pretty boring.  But we can pretend to program it by following it's rules of allowed inputs and outputs.

Let's show examples for a few more gates, add them as operations to our architecture, and then create instructions to do slightly more complex things like add a way to select which operation to perform.

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

Exclusive or is similar to Or, but it's only 1 if either A or B is 1, but not both.  It's exclusive.


Ok, so now we have 3 more hardware gates that we can use as operations: AND, OR and XOR, each of these take two inputs.  These are 3 more operations that we could run, but we don't yet know how ot tell the processor *which* operation to run.  

These operations take two binary inputs (A, B), each input can only be 1 bit, and produces a single output bit that's either 1 or 0.

Now we have 4 operations which are a little bit the same, and a little bit different.  Invert is single input, while the other 3 are two inputs for a total of 4 operations.  

How do we select which of these operations to run?  We have to represent the selection of the operation, and this isn't encoded in the data bits A or B.  The data bits don't care what operation is run, and have no influence on it.  So we need to represent this information, specifically in binary.

.. _instructions:

Instructions
------------
===========  =====  =======
Instruction  Input  Output
===========  =====  =======
Invert         A      ~A
And          A, B    A*B
Or           A, B    A+B
XOr          A, B   A XOR B
===========  =====  =======

Ok, so now we have 4 operations that we can perform.  They have some similarities: they all produce a single bit as output, and they have some differences:  Invert only takes one input, while And, Or, and XOr take 2.

Quick quiz, hot shot.  You have 4 operations.  How can you represent them in binary (as in unqiuely label) using the smallest amount of space?

There's only one answer:  with 2 bits.  2^2 is 4.  We have 4 things, and 2 values (1 and 0), so we need at least 2 bits, each with 2 values, to represent 4 things. 

Ok, so now we have some information about our operations.  They each need a code, and they each have at least 1 input, some have 2 inputs.

- The most information we need to represent at once is 2 bits, which is the code for each operation
- Some operations take one input, while others take 2, so we need 2 inputs
- The most amout of information about an input we need is 1 bit, since regardless of the number of inputs, any specific input (say A or B) is only 1 bit.

So we need 4 bits total.  2 bits for the operation, and 1 bit for each of the 2 inputs.

But in what order are those bits?  How do we know which bit is an operation code, and which is an input?

We have to make a decision here about the *format* of the instruction (operation code plus inputs to the operation) based on this information.

Instruction Format
------------------
The format of the instruction is the interpretation of each bit in the instruction, and how many total bits are in each instruction.

let's use 'o' to mean a bit in the instruction which designates the op code (code or label of the operation, in binary), and 'i' to mean a bit in the input.  We need 2 bits to cover all our op-codes, and two inputs, each with one bit.

::

    ooii

Let's use brackets to separate these out so it's more clear

::

    [oo][i][i]

So now we can talk about things like the format of the instructions without talking about any specific operation.  We have 2 bit op codes, and 2 1 bit inputs, for an instruction width of 4 bits.


Great job, We've made a strong decision about the format of the machine code now.  We've decided that the op codes are in the first two bits, and inputs are in the next two bits.  We could have made other decisions about the location of the bits:

::

    [i][i][oo]
    [i][oo][i]
    [o][i][i][o]

etc. 

There's nothing fundamentally different about this format vs the one we've chosen.  In fact [i][oo][i] looks a bit like math, eh?  1 + 1?  So it's not like they are foreign or weird.  Just different.

Some formats might make more sense then others.  But the point here is we've made a decision whch determines how a binary instruction like *1101* will be interpretted by our architecture.  The *11* is the op-code, *0* is input 1, and *1* is input 2.  We haven't decided which input is A or B though.  Doesn't seem to matter too much for And, Or, and XOr, but it's crtiical to understand that for Invert.   Invert will only operate on the A input.  So, we have more decisions to make.

What we do know, however, is that the compiled binary for this architecture must be in multiples of 4 bits.  A 17 bit long binary will not run, as there is no defined behavior for the "extra" bits.  When we compile a program, or maybe even try to execute it, we can use this small amount of information to verify something about the binary: it's format isn't wrong.

Architecture Decisions
----------------------
We are designing this architecture, so we get to decide which combination of 1s and 0s represent each instruction.  

::

    00 - Invert
    01 - And
    10 - Or
    11 - XOr

Remember our format:

::

    [oo][i][i]

The "o" bits are the codes for the instructions, also knows as *op codes*, and the "i" bits are the inputs to the instructions, and currently we allow for 2 inputs.  
        
We have one issue to note: the invert instruction only takes one input.  But we have decided the format for all instructions is 4 bits.  What do we do about that extra information in the second input?

Let's say the first input bit is `[input 1]`, and `[input 1]` is always A from the tables describing our operations Invert, And, Or, and XOr.  Likewise, the second input bit is `[input 2]`, which is always B.

We solved the problem by just making a decision when presented with an ambiguity.  

We decided a format specific to the invert instruction which tells the programmer exactly how to use it.  We decided to only use the first input.  

We can easily decide to only use the second input bit.  It's just how we decided it will work, and programmers will have to take note.  If you consider another architecture where everything is exactly the same as the one we have designed so far, except for this decision to use `[input 1]` as A, you can predict how the code would have to change.

For example, "Invert 1" would swiitch from 

::

    0010

to

::

    0001


Ok, so now we can clearly understand how each bit in a binary instruction will map to every operation, and how each binary instruction will change the output.

So, we now can represent our 4 logic operations and their inputs as binary instructions.

We can enumerate every single instruction like so:

===========  ======  =======  =======  =========  ======
Instruction  OpCode  Input A  Input B  Operation  Output
===========  ======  =======  =======  =========  ======
  0000          00       0        0       ~A          1
  0001          00       0        1       ~A          1
  0010          00       1        0       ~A          0
  0011          00       1        1       ~A          0
    
  0100          01       0        0      A*B          0
  0101          01       0        1      A*B          0
  0110          01       1        0      A*B          0
  0111          01       1        1      A*B          1
    
  1000          10       0        0      A+B          0
  1001          10       0        1      A+B          1
  1010          10       1        0      A+B          1
  1011          10       0        1      A+B          1
    
  1100          11       0        0     A XOR B       0
  1101          11       0        1     A XOR B       1
  1110          11       1        0     A XOR B       1
  1111          11       1        1     A XOR B       0
===========  ======  =======  =======  =========  ======

Now all 4 bit instructions, in binary, all have specifically defined results.

We can now write programs, which are sequences of 4 bit binary instructions.  

In order for these to make sense we had to make a number of decisions which aren't related to operations we want to use in our program, but are related to how the machine decodes the binary instruction to *select* the operation, and how the operation understands it's input.

The instructions have a format: the first two bits encode which operation to use, and the next two bits are the inputs to the operations.  The operations themselves have to use the format to decide what inputs are used, and how they are used.  We made all those decisions for this architecture.  They could have been different.

Assembly Language
-----------------
OK, we have op codes for each of our operations, but reading op codes in binary is tedious.  Also, typing those names "Invert", "And", "Or", "XOr" is compilated.   Well, maybe not, but let's make up a simple language to refer to each instruction that uses at most 3 letters.  Later we might decide to use more than 3 letters, but for now lets use these:

=========  ===   =========   =======
Function   ASM   Operation   Op Code
=========  ===   =========   =======
Invert     NOT     ~A          00
And        AND     A*B         01
Or         OR      A+B         10
XOr        XOR    A XOR B      11
=========  ===   =========   =======

Wow, much easier.  Now we have The human readable function that performs a operation, an assembly language instruction for it, and an op code which is the binary representation of the same thing, in a convenient table.  

Assembly language a "human" readable name for the OP code, and will also use register names (covered soon up in :ref:`registers`) instead of addresses.  Assembly is basically reading the binary directly, though, as there as a direct, line for line, op code to ASM correspondance between the assembly and binary.  

So yeah, since we're designing this architecture, we are ALSO deciding on the assembly language used to program it, becuase they are the same thing.   One is binary for processors, one is "human" readable.  For "Humans".  Yeah.


Anyways, going forward, note that the op codes directly translate to an ASM command, and vice-versa.

Simple Program
--------------
We haven't really described a program yet, but it's just a list of the instructions.  With the above information, however, we can write a list of instructions that's a program:

::

    0010
    1110
    1001

So yeah, there's a simple program.  We know exactly what it will do.  Let's write it in assembly.

::

    NOT 1 0
    XOR 1 0
    OR 0 1

Wow, look at that.  We know what the assembly means and what the binary means.  Kind of.  Well, we can look it up in the tables from the beginning.  Hey, we don't want to do that, that's why we're designign a processor!  Where's the result??

.. _registers:

Registers
=========
What we haven't explained is how to get the result at all, let alone how to use the outputs of one instruction as the inputs to another instruction.  This requires registers, which store values.  It also means we need more instructions, as none of the ones we currently have do anything about registers.  

*Oh no, but we don't have any more bits!*  

Well, we'll add more.   We're the designers.  We need registers to make useful programs, and to use registers, we need to be able to store values in them and move values between them.

Another issue is we also need to store the whole program somewhere.  Flashing firmware onto a chip means storing the binary instructions somewhere on the chip.   Executing the instructions means knowing which instruction is first, which is next, and actually, you know, reading the instruction to do something.  

All this requires not just some concept of *memory*, but specific *locations* in memory.  Locations in memory are registers.  Groups of registers are blocks, and a list of all registers in a continuous address range is a register file.  When we talk about on-chip memory, we're talking about all the registers, or possibly some block of them.  Knowing which block you're in means knowing the address of region in memory you are accessing.

What is a register?
-------------------
A register is s location in memory that can store a value.  We don't yet have any way of storing the output value creating by running an instruction, but we do need to store all the instructions in our program somewhere on the chip.

What do we know about our architecture which influences some decisions about registers?  

Our current 4 bit instruction set requires 4 bits for each instruction, and we also need a location for each instruction in the program.  So we can design the size of a register to be at least 4 bits.  

The location identifiers (addresses) must also be binary numbers, since everything in a computer is in binary.  How many bits do we need to represent the location of a register?  Well, how many registers do we need?

To run a program with more than one instruction, we need a list of registers to store each 4 bit instruction.  We also need a way to reference each location in the list of registers so we know which insturction to run.  How do you know which instruction is the first instruction?  How do you know which instruction is the next instruction?

So many questions.  Let's simplify these questions first, ignoring storing a program for now.  But keep in mind that the size of the program we want to store is related to how big the instructions are, and how many we want to be able to keep on the chip at once.  It's 2-dimensional with a *width*: the maximum amount of bits in one register, and a *length*: the number of registers.

Accessing Registers
-------------------
We'll go more into how registers are defined as memory, but as of right now we can think of special locations which have codes for their location.  What kind of codes?  well binary codes, we know that much.  Becuase this is a computer and everything is in binary.  Right?  RIIIIIIIGGGHHHHTTTT??????

Let's create a list of registers, not worrying about saving the program just yet, but to get a grasp on what they are individually.  We'll stick to 2 bit addresses, meaning we can represent up to 4 registers.  For now, each register will store only 2 bits.

=======  =====
Address  Value
=======  =====
00       xx
01       xx
10       xx
11       xx
=======  =====

Ok, there's a list of registers with addresses.  'x' just means we don't know what the value is yet.  But that's all it is.  A list of addresses, and values at each address.  When we say "The value at 00" we get whatever is stored in the register with the address 00.

Now that we have registers, we can name them, and mess with our architecture a bit to be able to use registers.  As mentioned above, naming the registers also helps us write assembly by replacing the addresses with labels.

Let's create 3 registers.  A, B, and Z.  We don't yet know the details, but we can say "Register A is located at 00, Register B is located at 01, and register Z is located at 10".  Whatever that means isn't important just yet, what's important is that we can identify the register by a binary number.  Again, we've just made this decision which impacts our architecture.  Regardless of *why* we chose A to be 00, it suffices to say that we had to make some decision.

===========  =============  =====
Reg Address  Register Name  Value
===========  =============  =====
00                A          xx
01                B          xx
10                Z          xx
===========  =============  =====

Now we can talk about storing a value into A, B, or Z.  Again, 'xx' here means we don't have a value defined yet.

Store
_____

Ok, remember why we started talking about registers?  We wanted to put stuff in them, and also get things from them.  Which means we need operations to do that.

We can instruct the processor to place a value in a register with a new operation, called Store, and the assembly for Store will be STO.  We need to design the operation in terms of our instruction format.

::

    [op code] [input 1] [input 2]

The operation STO will store the *value* of `[input 1]` in register *addressed by* `[input 2]`, and in assembly, Store the value of 1 in register A looks like:

::

    STO 1 00

We don't have a op code for the STO operation yet seince we're out of bits.  Let's increase the number of bits for operations to 4, and set 0100 to STO.

::

    [oooo] [i] [i]

0100 1 00 is Store 1 in register A.  Ahh, but we have a problem.  We need another bit in `[input 2]` to handle the size of the register addresses, which is 2 bits.  Ok, let's add it.


::

    [oooo] [i] [ii]

Note that while we've added bits to the op codes and `[input 2]`, we have not changed the general format of the instruction set.  It's still 

::

    [op code] [input 1] [input 2]

What we have done is *increased the widthi* of the op code and *increased the width* of `[input 2]`.

So, we have changed 2 things, but also added a capability to our architecture.  
    1. We added bits to the op codes so we can handle more operations in our programs
    2. We need 2 bits for input 2 now, 
    3. We have added a new way to interpret inputs, namely as *register addresses*
           
When we write a STO instruction, `[input 2]` can now be a register address, rather than a value.  

`[input 1]` here is still a value.  We are storing a value in a register.  

`[input 2]` can now be a register address, which is not directly a value, but *refers to a value*.  

I can say "The value in Register A" without knowing what is stored in register A.  It's still clear what I mean, even though I don't have the actual value, just the address of register A.  Register addresses are not values, but they are *references* to values.  They point to a value.  They are pointers.  But that's an entirely different discussion.  (or is it?  Can all processors use pointers?  Can we?  What do I mean by use?)

Note, however that there is nothing inherently special about the "00" in the STO instruction.  It's still a "valule" in some sense, it's two binary digits.  

What's special is how we used it.  "00" becomes a register address becuase of the op code that we used, because of the instruction, not becuase there is something special about "00".  By selecting different instructions, the interpretation of the value is different.  And we made that decision.

Put anothe way, for STO 1 00, the *value* "1" is in `[input 1]`, and the *address* of the register A is the *value* "00" in `[input 2]`.

Of course we should add bits to `[input 1]`, but lets see specifically why that's the case.

Move
____
The move instruction will take a value from one register and put it in another register.

::
    
    MOV Z A

Has the effect of taking the value in register Z and putting it in register A.

We'll assign it the op code 1000.  We could have selected 0101, since we're the disigner here, and can do whatever we want, but I'm going to reserve this address for now.  

This instruction doesn't take any *values* as inputs, only register addresses.  STO is responible for putting *values* in registers.  MOV only acts on register addresses as inputs.  Still 2 bit binary inputs, but the inputs are both interpretted as register addresses.

Again, it still has the form

`[op code] [input 1] [input 2]`

But again with the interpretation that `[input 1]` is a register address and so is `[input 2]`, so we need to increase the number of bits allowed in inputs to cover register addresses.

Instruction format update
_________________________
How many bits do we need for input 1 and input 2?  Well, we have 3 registers right now, and we need at least 2 bits for those addresses.  This means we need `[input 1]` and `[input 2]` to handle 2 bits each.  This is another change to the format of our instruction set.  We need 4 bit op codes, and 2 bit inputs.

::

    [oooo][ii][ii]


We don't need to update the op-codes for the other operations right no, other than add '00' to the beginning so they are 4 bits.  Note that we have to change the *behavior* of the other operations, or at least define them better, becuase they were originally designed to only handle 1 bit inputs.  

The intial 4 operations only took *values* as inputs.  But each operation is defined only in terms of 1 bit values, and now we have 2 bit values.

So, just like the invert operation needed to decide how to manage the instruction format, and we decided to ignore `[input 2]`, now we have to decide which bits of the inputs will be used in those single bit operations.  

The right approach is to use operations which can manage all the input bits.  But that raises some other interesting questions, but let's leave those for now.  (What if I only want to AND specific bits??).  Let's just assume we've redefined all the operations to work on all bits of their inputs, in a *bitwise* manner.

Rather than go futher into the details of specific operations here, I'll cover these in a separate section on Logic.  Eventually.  Maybe never.  If I do, I should link to it from here.  Needless to say, you now know the keywords to slam into your favorite search engine.

Saving Results
--------------
Ok, took a while, but we're finally here.

We established a Z register, and we saw that we can move values from it to A and B.  

Z is going to be a very special register, however.  It will always be the result from the last instruction, meaning based on the instruction op code and inputs, it will store the output. 

Back in :ref:`instructions` we saw all the possible inputs and outputs, but we didn't see how to use the output.  Now we always will have access to the output of the last instruction as the *value* in register Z.  We decided that, we selected Z as the register which stores the Output from each instruction.  Z now has a *special function*, and is referred to as a *special funtion register*.  `Who said naming things is hard <https://martinfowler.com/bliki/TwoHardThings.html>`_?

Special Function Registers
==========================
The processor itself controls what's in Z.  And we're going to make it read only.  This is so we don't accidentally change the output from an operation before we save the result of that operation.  

We will not allow putting anything in Z, we will only allow reading from it.  STO 1 Z won't work, and MOV A Z won't work, but MOV Z B will.  

We've defined a set of access controls on the register Z that the processor enforces.  We haven't said how, that's beyond the scope of this guide, but it's just some set of logic operations the processor manages, a bunch of transitors and logic gates which make a stupid decisison.  However, we know also have defined a set of *illegal instructions*.  Z cannot be the second argument in a STO or MOV instruction.  We can communicate this back to whoever is designing a compilier for our architecture.  Never create these instructions!  They will fail!

How big is Z, as in how many bits does it have?  Right now, the output from all our instructions is just one bit.  Z could be only 1 bit.  That seems out of line with the definition of our other registers, but we can just assume that's the case.  We'll have to make sure that we pay attention to where bits are going in any case, since Z can be loaded in to A or B. 

It probably makes sense to make all memory locations the same size.  But we also need to pay attention to where all the bits are going.   We'll leave that alone for now, it's like all other things here:  Make a decision, and make sure all other operations respect that decision.

Programming with Registers
==========================
Let's get back to how to program with our expanded instruction set.

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

But how to I AND the results I stored in A and B?  A and B are both registers, so I need an operation that takes two register addresses as it's inputs.  Right now I only have one instruction which takes two addresses as inputs, MOV.  So while I can store the result of the last operation, and I can access the result, I can't actually *use* the result.

Well, we need more operations, the ones I have are not sufficient.   Good thing we added extra bits to op codes already, and that we have sufficient bits in `[input 1]` and `[input 2]` to manage all our register addresses. So the Instruction format can stay exactly the same: 4 bit op codes, and 2 inputs, each 2 bits.

Right now I have "direct" operations for Invert, And, Or, and XOr.  I need "indirect" versions, which interpret their inputs ad register addresses instead of values.  They are "indirect" because they need to access registeres for their values.

So let's design somme new operations.  We'll need op codes and ASM representations for operations that know to look in registers with addresses `[input 1]` and `[input 2]` instead of using `[input 1]` and `[input 2]` as only values.  We'll name those ASM instructions  AND_R, OR_R, and XOR_R, where the _R let's us know the inputs are register addresses, not values.  We'll conveniently forget about NOT_R for now, fabricate the chip, and let the software guys deal with that problem [1]_.

Now we can do the following:

::
    
    STO 1 A     # Store the value 1 in A
    STO 0 B     # Store the value 0 in B
    AND_R A B   # AND the values in registers A and B.
                # Z becomes the result of 1 AND 0, or 0

We could assign these instructions similar to the previous ones which acted only on values.

=======  =========  ==
Op code  Operation 
=======  =========  ==
 0000      NOT      # Immediate Invert
 0001      AND      # Immediate And
 0010      OR       # Immediate Or
 0011      XOR      # Immediate XOr
 0100      STO      # Store
 0101     AND_R     # Indirect And
 0110     OR_R      # Indirect Or
 0111     XOR_R     # Indirect XOr
 1000      MOV      # Move
=======  =========  ==

What's neat about this scheme, and why I reserved the op codes 0101, 0110, and 0111 above, is that now OP codes are grouped by how they operate.  00xx op codes work on values direcly, and 01xx op codes work on registers (including STO).  

This can do more than just organize a table of instructions, it can inform the processor to get ready for accessing memory.  The first two bits can mean "interpret inputs as addresses" and the last two bits can still mean "operation".  But don't get carried away thinking that's how actual processors work.  You'd have to investigate their architecture to know.

Fast vs Slow Instructions
-------------------------
This is not the only way to solve this problem.  I could make all the instructions ONLY use registers, except for STO which puts values in registers.  Then I could designate register A and register B as `[input 1]` and `[input 2]` for all operations, and the operations always take the values in those registers.

::

    STO 1 A  # Store value 1 in A
    STO 0 B  # Store value 0 in B
    AND      # Use the new AND operation which always perform AND on the value in 
             # A and the value in B.  Z becomes 0, the result of 1 AND 0.


We lose the "fast" instructions which don't requre registers to operate on known values for the benefit of having a smaller instruction set.  

It's a trade off, do you expect to do more "fast" (also called "immediate" instructions) where the values are stored in the program, or do you need to to more chained operations which use the results from previous calcualtions?

In both cases, our processor is capable of doing operations on known values, e.g. "And the values 1 and 0", but the assembly and machine code now always requires to two store instructions, then an AND instruction.  We can no longer do a direct AND 1 0, which is faster since it's one instruction.

Let's keep these two modes in mind, immediate and indirect.

Recall what we're doing here.  We're making a bunch of design decisions about our architecture that determine how a binary string, a list of 1s and 0s, will be interpretted.  Things are getting complicated, and we have had to make a number of decisions which impact the format of our instructions, and therefore the format of the binary code that will execute on our architecture.  These decions also impact how many instructions are required need to perform a task.  


Memory
======
Now we can do something closer to programming, where we can use the results of operations as the inputs to new operations.   Let's look a bit about what it means to store a program in memory.

Memory is a list of registers, usually in a continuous block, where we can write and read based on addresses.

We have 3 special function registers right now, A, B and Z, at addresses 00, 01, and 10.  Let's assume we're going to have more.  So including those and the space we'd need to store the actual program, we obviously need to increase our address space (the number of bits required to address each memory location), so let's do that to 8 bits.  

Program Memory
--------------
If we have 8 bit addresses, we have 2^8 = 256 registers total.  We'll reserve addresses 0-15 for special functions, currently A, B, an Z, and 17-256 can be whatever we want.  We haven't said specifically that only the program will be stored there.  We could make the decision, and say we have 16 sfrs (special function registers), and (256-16) 250 registers reserved for the program, and call that Program Memory.  That's all program memory is, a block of the registers reserved for only the program.  If we did that, then we add more *illegal instructions*, those that attempt to write to program space.  

We could further segment memory as 16 SFRs, 128 bytes (remember 8 bits is a byte, and the program memory is chunk of 8 bit registers) for Program Memory, and the rest (256 - 16 - 128 = 102) bytes for data.  If we don't reserve some memory for data, we'd only have Registers A and B to save results of computations.  If we have memory space, we can use all of it just like using register A and B, with STO and MOV instructions.

In reality, processors have many special function registers to store settings and results.  They all work in the same way, but let's look at ones that are specific to executing a program.

For now, let's assume we've loaded a program into memory.  We'll cover the details of how that's done later.  But you can safely assume it looks a lot like 

::

    STO value register
    

where "value" is the binary number which represents one instruction and 'register' is a register address, which are now 8 bits.  

But we've created some problems by increasing the address space.  We need some way to access it, and all our register operations only handle 2 bit registers.  So clearly we have to go back and update everything.

It should be clear, now, that the instruction format is directly linked to how much memory you can access.  Since we have instructions that can operate on two register addresses, and we need 8 bits for each address, this tells you how at least how wide the registers reallly need to be in memory to handle our instruction format, `[op code] [input 1] [input 2]` or `[oooo][iiiiiiii][iiiiiiii]` in the case of 4 bits of op codes, and op codes which can handle 2 registers as inputs.  Wait, that's much bigger than what we were dealing with before!  That's 20 whole bits!

Well, we could make other decisions, like how instructions are stored.  Maybe they are stored in 2 registers instead of 1 (otherwise knows as 'stored across 2 registers'), Or maybe across 3 regsiters instead.  Up to us really.  But if we need 2 or 3 registers per instruction, then we need more program memory to store the same program, which gives us less space for data, since 8 bit addresses will only yield 256 addresses.  Right now we need 20 bits, which would require 3 addresses, and we'd waste space since 20 isn't equally divisible by 8.

Obviously this is another trade off.

There isn't a strict need to separate program memory.  You might want to for many reasons, but for now lets talk about a processor that has one register file that has the separation only for SFRs.  It knows a little bit about itself, but not much.  One special register we'll need is the start of the program.  Let's use register 16 for that.


Program Execution
=================
We haven't really discussed execution yet, and frankly, we don't really need to in order to define the binary format.  However, it's good to cover it a bit.

As we decided above, our architecture knows to always start the program from register 16, it can happily march down the register file.  It doesn't even need to know, in advance, when to end.  It can end when it runs out of instruction space (either the end of program memory, hard wired in the chip, or just teh value 256 if there's no pre-defined end).  "end" here means "crash" "halt" or "stop", whatever you prefer.

However, we want the ability to loop when we're done.  That means we need some way to tell the processor to go back to the beginning.  We know the beginning is address 16 (that's what we designed).  So we have a number of things we can do, but they all require more instructions.

Automatic
---------
As the processor designers, we could determine that any time the processor reaches the end of program memory, it automatically does something.  Maybe it reboots, clearing memory and starting from scratch.  Maybe it doesn't reboot, but just starts again from the beginning on the program memory.  In this case, the special function registers like A, B, and Z might retain their values, and it might cause our program to execute differently.  Or not.  (this is why initializing variables in your program is always a good idea, you don't want to depend on the chip designer).

While automatic works fine for simple cases, let's give our programmers more control over their execution.  We can let them jump around.  Which can lead them to a world of pain.  Or maybe a House of Pain.  A bouncy House of Pain.

Goto
----
oh, goto.  We love you.  Maybe we'll introduce a new instruction called "goto" which takes a a register addreses as it's input.  We could have called it "jump", but "goto" raises specters of fear and uncertainty in programmers, and we're hardware designers now.

::

    GOTO 16

of course the instruction format is STILL `[op code] [input 1] [input 2]` but the GOTO instruction just ignores `[input 2]`.

What's going to 16 actually mean?  The processor will perform the instruction at register address 16 after this instruction.  GOTO is clearly useful, but it's not the only way to change what the next instruction is, maybe we don't need a new instruction.  After all we have a number of instructions which already operate on register addresses.


Program Counter
---------------
In fact, we can re-use the STO operator, but we need a new special function registers called the Program Counter, or the PC.  

The program counter determines which instruction will execute next.  It points to the address of the next instruction.  If we are ticking away happily, it increases by 1, because that is the location of the next instruction.  However, it's a register just all the other registers.  We use STO value PC to change how our program executes.  Now don't need a special instruction like GOTO, we can re-use the instruction STO at the cost of one special functino register.

So, for example, on reset, the chip will always execute a 

::

    STO 16 PC

instruction.  But what's actually happening?  For that, we need to talk about how the processor actually executes, which is called the Instruction Cycle.


Instruction Cycle
=================
The chip has an instruction cycle, which is a bunch of logic that determines and executes the instructions.  The processor takes the same steps each cycle, where the only differences are the specifics of the instruction being executed.  

It's the stuff that's doing the actual processing.  When it's time to perform the next instruction (a clock cycle, maybe a couple), it will *fetch* the instruction from the memory location in the PC, *decode* the instruction based on the instruction format, and *execute* the instruction.

A simplistic cycle will decode the operation based on the instruction format, decide how to interpret the inputs based on the instruction, execute the instruction, store the result in Z, incremenet the PC by + 1, then repeating this cycle.  

However, what if the PC was written to during the cycle?  Better to increment the PC right after the FETCH.  Why?  Becuase I said so. I mean we said so.  We just designed our instruction pipeline.  Let's see why incrementing after FETCH is required to get the behavior we want when we change the value of the PC during the execution of an instruction.

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

to restart at the beginning of program memory.  The next cycle would first increment the PC to 16, then fetch the instruction at address 16, etc.  Makes more sense to store 16.

But what if I STO 3 PC ??
-------------------------
Unlike Z, the PC isn't read only, and the processor will try to fetch an instruction from the memory address represented by the value stored in the PC.  What happens if I write a value to it that doesn't represent an address in program memory?

SEGFAULT.  Segmentation fault.  Memory Segmentation Fautlt.  Memory Segmentation is the seperation of memory into different areas, and we have 2 areas. Special function registers in addresses 0-15, and "other stuff" like program insturctions above 16.  Great job, you crashed the computer.  Thanks a lot.  I'm not even done designing it yet.  Please, don't do that.


What else can I do with the PC though?
--------------------------------------
Well, if you see that manipulating the PC is like a GOTO, you can move around your program now based on the results of computations.  

This is called branching, and covers things like flow control, conditionals, and functions.  It's a much deeper topic, which I'll cover somewhere other than an introduction to architecture.  

Suffice to say, if you start jumping around, you need to pay attention to stuff.  Lots of stuff.  Heaps of stuff.  Stacks of stuff. Ok memory (maybe you've head of the heap and the stack?), you need to pay attention to what you're doing with and to memory, when you do it, and where you are going to end up when you branch.  

If you write assembly directly, you will know which instructions end up in various addresses just by counting the instructions, so you can hard code these GOTOs.  But that's obviously a recipe for disaster.  Make a change and forget to update an address, and you get  behaviors like jumping into the middle of a different function!  What fun!

When compiling, the compilier fills in all these addresses during the linking step.  See :ref:`compilation_primer` for more on that.

GOTO instruction vs the PC
--------------------------
When deciding to use more memory instead of adding an instruction, we made another trade off.  We used more memory space in order to re-use parts of our existing instruction set.   This is just one example of trade offs made by hardware designers, but since you already know that the sizes of things are also impacted by how many of the things you have, some times it's easier to use more memory, and sometimes it's easier to use up empty spots in the instruction set.  

The nice thing about the PC is that, well, we need it regardless.  That's how the processor knows which instruction is next.  The PC always stores the adddress of the next instruction.  It's another SFR.  The interesting thing about the PC is that it's just another register, and you can save it's value and restore it.  

I'm going to add memory locations to a program now, pretendling like it's loaded in memory.  Remember that we can access any memory address above 16, as well as the SFRs as usual.

::

    16 ADD 1 0   # AND 1 and 0, setting Z to 0
    17 MOV Z A   # Save 0 from Z to A
    18 MOV PC B  # Save the value of PC (the address of the "next" instruction, which is 19) in B
    19 STO 40 PC # Set the address of the next instruction to 40
    20 ...       # whatever
    .. ...
    40 OR 1 0    # OR 1 and 0, setting Z to 1
    41 MOV B PC  # go back to where we were..hmm...


I'm going to use the word "line" here as each "line" is a line in the program, but on the chip, it's also the address in program memory of an instruction.  Keep that in mind.

If we execute the above "in order", we'll get to line 18, which store the value 19 from the PC into B.  Remember, the PC was incremented before executing the instruction on line 18.  And line 18 moves the PC value into B.  Now line 19 is executed which sets the PC to 40, and we've jumped to line 40.

What happens after line 41?

Well, B is 19.  We set the PC to 19.  What does line 19 do?  It sents the PC to 40.  We've created an infinite loop.

There's a harder problem here, in that it takes an instruction to save the PC, and we currently have no way to change the value of the PC.  Of course a real processor can add and subtract, but that's not relevent to the problem, but does offer a way to solve it:  Add 1 to the PC before saving it.  But we can't yet.

There's another lurking problem not in that code.  What if we changed register before restoring the PC?

This makes jumping around difficult.  However, there's a great solution.  Let's section off part of the free memory for something called "the stack", and save the values of the PC, A, and B before jumping.  Instead of writing all those instructions ourselves (which we could, but we still have the problem of restoring the PC properly), let's create an instruction named "Call" which does all that.  It takes a memory address.  We'll pair it with an instruction called RET which takes no inputs.  RET pops the vlaues for A, B, and PC off the stack, but leaves Z as-is.

So now we'd do something like:

::

    16 ADD 1 0   # AND 1 and 0, setting Z to 0
    17 MOV Z A   # Save 0 from Z to A
    19 CALL 40   # Save A, B, and PC to the stack, and jump to line 40
    20 ...       # whatever
    .. ...
    40 OR 1 0    # OR 1 and 0, setting Z to 1
    41 RET       # go back to where we were, restoring A, B and the PC


This is much nicer for a couple reasons.  One is we don't need to mess around with the PC, the processor does it automatically as long as there's room on the stack.  Each time we use CALL, 3 memory addresses are required, so save the values in A, B, and the PC.  When we use RET, we recover those 3 addresses as free space.  So the stack will grow and shink the more we use CALL and RET.

We've created a basic ability to call functions.  We can even name them, and do a "find/replace" on the name to it's final memory address once we are done with the program.  This is part of what the linker keeps track of, the memory addresses to where we'd jump during function calls.  The stack keeps track of to where we'd return.  So it's not that "return" means "return a value", but it means "return to the previous location".

What's interesting is the processor doesn't need much extra functionality to do this automatically.  It needs to know where to start the stack, and where the current stack stops.  These are addressses, aka pointers.  So it needs a new SFR called the stack pointer whose value is the top of the stack.

The processor needs to know which registers to save.  That's easy, it's going to be most of the SFRs, since those determine the current state of the processor.  So we know how big the stack will grow or shrink each time we call a function.  That amount is called a *frame* or *stack frame* and is literally the value of each SFR when the function was called.  Since we designed the processor, we can say "store SFRs 0-15 in order during a call".  How long would that take?

Well, it's 16 SFRs and the processor will just use the MOV instruction to do it.

::

    MOV 0 STACK_POINTER
    INC STACK_POJNTER
    MOV 1 STACK_POINTER
    INC STACK_POINTER
    ...

OK, i snuck an "INC" instruction in there.  "INC" takes a register as it's `[input 1]` and increases the value in that register by 1.  That's it.  We'll need a "DEC" too in a second.  It takes a register as `[input 1]` and decreases it's value by one.  Again, more logical operations.  Remember though, we needed an INC operation for the PC duing the instruction cycle.

So we have 16 MOV instructions and 16 INC instructions.  So it takes 32 instruction cycles to call every function!  But what we get is a very easy way to call functions, and who doesn't love functions?  But this is why calling functions in C/C++ is *more expensive* than writing assembly directly.  I might not need to save every SFR, for example, if I know I'm only going to use A.  I might not need to save any of them at all except tthe PC.

Anyways, now we have included the ability to call functions in our instruction set.  The fuctions are just "addresses to jump to" and look like memory locations.  We even introduced a stack, stack frames, and a stack pointer.

We've also further segmented memory by using the stack, and the interesting thing about teh stack is it's dynamic.  If we call a function *while in another function* we'll have 2 frames on the stack, and so on.  If we run out of stack space, from calling too many functions without returning, we'll run out of memory for the stack, since we won't allow the STACK_POINTER to enter Program Memory.

We can add another SFR called ERRORS with 8 bits.  Each bit can represent a specific type of error that can occur during program execution.  This way we can stop, pop all the frames off the stack one by one, and let the user know an error occurred.  This is why programs can crash without crashing the whole processor.


Putting it together
===================
I mean good luck, the above is a smattering of information about what a computer achitecture is.  Why is this important?  Because different architectures make these decisions differently.  They have different special function registers that mean different things, the op codes for similar instructions are not the same, they don't all implement the same instructions, and the have different numbers of bits available for addressing, operations, values, etc.  I didn't even explain endianness!  Did you know that 1100 can mean either 12 or 3?  Yah it can be read forwards or backwards!  I'm the designiner, i get to decide! AAHAHAHAHAHAHAHA


Whoopse, sorry, got a bit drunk on power there.  But there's no single rule which must be followed here.  Some decisions make processors really good at certain tasks compared to others, theres no one perfect solution.  The only thing to take away from this is that, while we can write C/C++ code for a large (huge) number of architectures, the binary which get spit out of the compilier is different for each architecture.

This is the real beauty of Arduino.  You don't need to care at all, it manages these differences for you, to a large degree.  You tell it what you bought, it figures out the rest.

I haven't really discussed how the op codes map to actual operations.  It's a little complicated, but it all boils down to logic.  For example, you could to all the operations at once, and select the ouput based on a multiplexer.  The op codes merely select the desired output from the multiplexer. Some parts of the processor work this way.  Suffice to say, it doesn't exactly matter here to make my point, but perhaps I'll get more into that in a new section at some point down the road.

But, you should now be able to make sense of an Instruction set.  In fact, instead of messing around with this garbage one I've defined, have at the `ATMega328P Instruction Set <http://ww1.microchip.com/downloads/en/devicedoc/atmel-0856-avr-instruction-set-manual.pdf>`_.  I think I'll add a section on a guided reading of this at some point, but for now, start barrelling through it.  

You'll find it has different instruction formats as part of it's ISA.  Who cares, you can handle that, because you know what an instruction format is.

    .. image:: img/ATMEGA328p-direct-register-ops.png
        :width: 640px
        :alt: Direct register addressing

Hey, look, it's got a STACK and a Stack Pointer (SP).

    .. image:: img/ATMEGA328p-stack.png
        :width: 640px
        :alt: ATMega328P Stack registers

It's got instructions which use two registers, and a wacky addressing scheme in the instruction format:

    .. image:: img/ATMEGA328p-direct-AND.png
        :width: 640px
        :alt: Direct AND instruction from ATMega328P instruction set


What the heck is this:

::

    0010 00rd dddd rrrr

What the heck does that mean?  Why didn't they put all the 'r's and 'd's together?  

The Operands are both 0-31, which is 32 values.  How many bits do you need to represent 32 values?  2^5 = 32, so 5 bits.  

The instruction has two operands which are register addresses, so the register addreseses can be 5 bits, but they split up the bits in the addresses!  Weird!  Maybe there's a reason, maybe they were just drunk!  But you can read this and understand it.  That's what's important here.

Read through some of the instructions themselves.  You know what "Immediate" means.  You know what JMP does, what INC does, now you can see how they work for real.  There's Exclusive OR!  Hot diggity.

Anyways, if you don't understant something, I should probably add a section describing it.  Let me know.  However, one thing to learn about specifications is *how* to read them.  That usually means going over them multiple times.  Information is typically defined rigorously, and only once, but everything is usually defined somewhere in the document.  Like SREG.  Each instruction seems to change some value of some bits in SREG. What's the address of SREG?  (Trick question!  It's in the `ATMega328P Datasheet <http://ww1.microchip.com/downloads/en/DeviceDoc/Atmel-7810-Automotive-Microcontrollers-ATmega328P_Datasheet.pdf>`_!)

Why does the processor need an SREG?  Hey, remember how (1 + 1 = 2), but (1 OR 1 = 0)?  What does "carry" mean?  What does the "Carry" bit mean of the SREG mean when it's set during the ADC instruction?  Come on, you got this.


.. [1] Always get the latest version of the datasheet and instruction set reference for your target architecture and read the errata!
