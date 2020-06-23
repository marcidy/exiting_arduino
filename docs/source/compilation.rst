.. _compilation_primer:

=============================
Primer on Program Compilation
=============================
There's a lot of resources on the web about what compiling is and does, however, it's useful to have the same understanding for the purposes of these guides.

Let's get this out of way really quickly, the terminology is confusing.

* Compilation refers to a 3 or 4 step process
* One of the steps in the process is called compiling

Unless someone is talking about the compilation step of the process, compiling generally refers to the whole process.

In the case of Arduino, :ref:`arduino_builder` performs it's own pre-processing step before starting the compilation process, which then also has a pre-processing step.  This should make some sense, the IDE (and the files in which the IDE stores informat) contains a lot of important information needed to compile files.  Please see that section to see what it's doing and why.  Just know below that I'm only talking about the "standard" pre-processing step when compiling C/C++ programs.

Compilation Process
===================
    Pre-processing
        Before trying to translate the code, gather all the information about the files which need to be translated.
    Compilation
        Using the Grammer of the language standard, convert the symbols in the source files into target architecture specific Object code in the target architecture's assembly language
    Assembly
        Gather all the object code during compilation into one cohesive unit and convert into machine code
    Linking
        Find all references in the object code which requie filling in.  It is now possible to find all locations in the code which require knowing specific memory addresses, for example.

Programming Lanauge
===================
Let's quickly review what a pogrammming lanauge is.  We write source code, which is a bunch of text in a file.  Compilation translates this text into a instructions that the target processor can run.   Instead of looking at it as text in a file, let's think about it more generally:  It's a long list of symbols, including spaces, new lines, etc.  Everything in the file is just a long list of single-characters.

Syntax
------
The way the text in the file is written is extremely important.  It's written is a specific language (let's say C++) and even within this language, it's written to confirm to a specific version of the language (a standard like C++11).  The language standard defines what text in the file represents valid syntax.

Syntax is how a thing is presented, as opposed to what a thing means (which is semantics).  In english, we use nouns, verbs, articles and adjectives in certain orders which are "correct".  This includes punctuation which break up meangingful units.  A syntactically valid sentence doesn't always mean something, but we recognize it as valid.

Lanauges are made up of symbols, like letters, numbers, and punctuation.  We know that words can start with letters, but not numbers or punctuation.  That's a rule.  Grammer defines the rules of what syntax is allowed.  We also know that words are separated by punctuation like spaces, commas, periods, etc.  In a general sense, we can identify words (recognized by applying the rules in the Grammer), by looking at how the punctuation separates symbols from our alphabet.  These rules about how to separate valid words is also in the Grammmer.  While we may not be familiar with the definition of all the words, we can still identify distinct words in a sentnece by applying these rules.  Let's call the act of separating valid words and symbols "tokenization", and valid words are "tokens".  Another term for this is "parsing".  

If we have rigid rules about the allowed symbols and rigid rules about how symbols near each other, we can parse a list of symbols into tokens without knowing what those tokens actually mean.

"The horse drove to the hospital to put it's wheels in the ocean" is valid english syntax.  The verbs, articles, nouns and prepositions are all in the right places, as are the spaces, commas, apostrophes, etc.  The meaning, or semantics, is nonsense, but we can diagram the sentence based on the rules of English Grammar.  Compilation requires syntactical validity because part of the process is exactly like diagramming a sentence.  Diagramming a sentence doesn't requie knowledge about the sentence before it or after it, only the symbols inbetween certain punctuation.  Likewise, it doesnt' need to know the meaning of the sentnce, and it doesn't even need to know the definition of the words, just that certain words are expected in certain places (like a, the, an, of, etc), and that these placesments indicate that other words are nouns (the name of a variable for instance), and others are verbs (a function or operator).

"Drove man hospital I" isn't syntactically valid english, even though we can infer some meaning.  Computers can't infer.  They require rigid rules to operate.   Compilation is asking a computer to read text and perform some complex operations, and so rigid rules are required.  The details of these rules is a large topic, but is related to learning what is and isn't valid C++, and so it's related to the task of learning to program.

Syntax, with repsect to programming, is the set of valid symbols we can use, and how we can order those symbols in a file.  Think spelling and punctuation.  Literally, think that.   The totality of valid combinations of syntax is called the Grammar of a language.  Remember that determining validity is applying a rule to a specific combination of symbols and determining if that is valid.  Different languages use different Grammars, but they all have some concept of valid syntax.  Learning a new language is a mix of learning the grammar while also learning the semantics ie the concepts which are invoked by valid grammar.

For the purposes of these guides, we'll define semantics at the machine code, or architecture level.  To get a grasp on what that means exactly, take a look at :ref:`architecture_introduction`.  Compilation will produce machine code based on a bunch of rules.  The machine we execute the code on will determine what that actually means (i.e. blink an LED.)

So, source files have to be written with a specific syntax which is allowed by a version of a programming language.  Let's assume we've done that from here on out.

Now we compile the program.  We tell the computer which syntax we are using (C++, c++11 version) and it then looks at the file and parses it.

PreProcessing
=============

The first step to compiling is "preprocess" the files.  There's a bunch of things in the files which are not valid C++ instructions for the target processor, but are things like comments in the code or preprossor directives.  These usually begin with #, or are related to things which begin with #.

Comments
--------
The first one we usually encounter are comments.  These are things preceeded by `//` or surrounded by `/* */`.  The preprocessor removes these.  They are for humans and not computers.  Might seem obviously, but we are trying to explain exactly how the compiler goes from a source file to architecture dependent binary, so let's note the obvious as well.

Includes
--------
The first preprocessor directive we usually encounter is  an `#include` statement.  These tell the preprocessor that there are files it needs to fully compile the program.  These files can include function definitions, variables, or anything else that the code in *this* file might need and aren't defined in *this* file.  So the preprocessor is building a list of all the files it expects to have in order to compile.  

But how does it know where these files are located?  The `#include` directives don't give the full path to every file.  When compiling a program, you have to tell the compiler which directories to look for files.  This is done using the `-I` options.  It's important to understand it's not sufficient to just `#include` other files, you also have to tell the compiler which directories your includes are in.

Remember that the compiler is effective constructing a single file which represents all the code you need to run.  While not strictly true, let's assume that is for now.   It needs to acquire all the information in one place as a single monolithic program.  Each specific source file only has partial information.

Maybe you haven't used includes, or maybe think there is some magic between the correspondnace between ".h" files and ".cpp".  Not so.  You can consider a the preprocessor is just blindly copying and pasting the contents of a file mentioned as in an `#include` statement right where the `#include` statement is.  It can be slightly smarter than that, but not by much.  Read on.

Defines
-------
A third preprocessing directive is a `#define` statement.  While these can look like variables, but are not.  The preprocessor just does a a simple find/replace.  This is why when you do something like `#define NUM 30` you don't need to set the type of NUM to an int.  However, wherever you use NUM, it had better accept an int.  It's similar to doing a find/replace all insteances of NUM with 30 before compiling.

Conditionals
------------
With `#define` comes conditional statements like `#ifdef`, `#ifndef` and `#endif`.  These mean "if defined", "if not defined", and "end the if statement".  And these are all directives to the preprocessor, not statements in the compiled code.  They tell the preprocessor to include or exclude the lines in the file from the compilation.  If a condition is not met, then the code is not used.

.. code:: C++

    #define NUM 30
    #ifdef NUM
        int x = NUM;
    #endif
    #ifndef NUM
        int x = 100;
    #endif

The above is a bit silly, but the idea here is the preprocessor will only include `int x = NUM;` when compiling code, because NUM is defined to the preprocessor.  `int x = 100;` will not be included in the compiled code.  

You'll see these directives protecting header files.  The reason here is that headers should ony be included once, even if they are required by many files.  protecting them with

.. code:: C++

    #ifndef SOMEUNIQUE_VAR_NAME
    #define SOMEUNIQUE_VAR_NAME
    <rest of file>
    #endif

defines the SOMEUNIQUE_VAR_NAME only if it's not defined already.  If it is defined, it skips including the contents again.  

The more directives you come across, you should learn about them.  `There's good resources for this online <http://www.cplusplus.com/doc/tutorial/preprocessor/>`_.

The preprocessor collects information about all the source code needed to compile the program, including those from other libraries.  It can find problems like missing deifntions, which seems like a pain but it's really just finding mistakes well before they end up as bugs in the code.  That's why it's there.  There's a lot more than can be done in a preprocessor, but this is enough for the sake of this guide.

Final note on preprocessor directives
-------------------------------------
While they seem complicated, they trigger some very straight forward behavior.  Copy/paste the contents of a file, or find and replace a value in a file.  Nothing fancy at all yet.  No magic.  You could do this by hand in a text editor.

Compiling
=========
Once finished preprocessing the actual compilation step occurs.

Compilation transforms the human-readable, but syntactically valid  source code into something called assembly, or assembly language.  It's important to note that, while assembly is sort of a language, it's specific to the target architecture.  It's really only a lanauge in that it's meant to be human readable.  But that's relatively to binary.  It's basically 1 step above the machine code, and it is literally translated directly to 1s and 0s.

This step requires information about the target architecture, as the assembly is related to what instructions can be understood by the processor that will run the code.

Architecture
------------
Architecture in this sense refers to what instructions a processor can run, and the format of the instructions.  It's another language in the sense that there is a correct syntaxt to use.  It's architecture specific in that processors only understand certain instructions.  You'll see the abbreviation ISA which stands for Instruction Set Architecture.

For most microcontrollers, the companies that create them release instruction set documentation.  For example, here's the `AVR ATMega328P ISA <http://ww1.microchip.com/downloads/en/devicedoc/atmel-0856-avr-instruction-set-manual.pdf>`_.

It's a great read, but check out :ref:`architecture_introduction` for a basic example of what processor architecture is.

Binary
------
Binary has a number of meanings.  In the context of compilation, it's the final result.  This result is a list of instructions which are formatted for the target architecture.  To get an idea of what that means, head over to :ref:`architecture_introduction`.  The final result of compilation gets stored in a file, and uploaded to the target microprocessor in some fasion.  Binary also means this file (as in "the binary").  Binary can also refer to the format of the instructions in the ISA.  Again, head over to :ref:`architecture_introduction` to get a better idea.

Source
------
Well, we have to start somewhere, so we start at the source.  The beginning of compilation, after preprocessing, is all the valid syntax in the files you wrote, plus whatever libraries you included.  It's really a big mess.  How does the compilier know where to start?   Well, that's part of the specification of C++.  The general rule is that the compiler starts at the top and reads to the bottom, and therefore must have a definition of everything it uses before it sees something used.  This is why `#includes` are at the top of the file, and you see function prototypes declares when using a function before defining it. 

Note this is not where the program starts executing.  That is determiend by the special function, `main()`.

.. _maincpp:

Main
----
Every C++ program must have a main function, and only one main function.  You must have noted that arduino doesn't use main, they use loop and setup.  That's because arduino's main program is located somewhere else entirely.  What's that look like you say?  Funny you should ask.  For arduino 1.8.10, it's located in `/home/marcidy/arduino-1.8.10/hardware/arduino/avr/cores/arduino/main.cpp`

.. literalinclude:: main.cpp
    :language: C++

Well, well, well.  Look at what we have here.  Ignoring everything else, notice the "setup()" function and the "loop()" function?  Do you see why "loop" isn't a loop by itself, but gets executed inside a loop?  

And look at that, it does some serial stuff, USB stuff.  Stuff that "just worked" when we were just using sketches.

The point to make here is that there is one, and only one, main function.  The C++ Grammar defines where function names can appear, and how they can be named.  For execution purposes, the compilier looks for a special one called "main" and knows that the start of this function is the start of the program execution.

It also displays some more features of writing C++, namely the `__attribute__` syntax.  This kind of statement is used to manipulate properties of the objects the code creates.  For example, you can manipulate the properites of variables that are not related to their types, values or names.  Why you'd do that is a more complicated subject, but it's useful to get some understanding of what the bits of syntax mean. 


Object Code
-----------
The process to compile source code is well described `here <http://www.cplusplus.com/articles/yAqpX9L8/>`_, which covers why header files, and specifically declarations, prototypes, and definitions are important.  

The gist is that the compiler actually knows very little relative to the entire program at any given point during compilation.  It knows what is has already seen, and nothing more, meaning it does not know what will come below the line it is currently processing.

If you try to use a function before it's defined, the compilier will stop.

Each .cpp file is visited by the compiler which tranlates it to object code.  The output of this step are files of the same name, but with a `".o"` extension, called an object file.  Object code is ready to be linked together in the next step of the compilation process.

Object code is part of the completed compilation processs, but may contain placeholders for information not yet fully known.  For example, calling a function or retrieving the value of a variable requires knowing where in memory they are located.  Those memory locations can not be known until all the object files are compiled and linked together.

Object code is partially completed assembly code.  It requires linking to fill in any missing references.

Assembly
--------
Assembly here is more like a verb, where assembly code is converted to machine reable binary for the target architecture.