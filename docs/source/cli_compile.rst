.. _command_line_compilation:

========================
Command Line Compilation
========================
At this stage we have a valid C++ file, maybe output from :ref:`arduino_builder` preprocessing the sketch.  We saw in that section that the IDE actually executes a number of commands not echoed to the screen:

.. literalinclude:: _static/arduino_builder_compile_clean.txt

It's a lot of repitition, so we'll look at the different types of invokations.  

I've expanded the first to split out all the options passed to avr-g++.

Pre-processing
==============

The first few commands pre-process input files (the sketch Fade.ino in this case) and generate files in temporary directories for use in later steps.

Note that these two commands are identical except for the `-o` flag, which determines where the output will be placed.   You could run it without the `-o` flag to see the output on screen, which is then placed in the file `/tmp/arduino_build_709419/preproc/ctags_target_for_gcc_minus_e.cpp`.

.. code:: bash
    
  Detecting libraries used...
  /home/marcidy/.arduino15/packages/arduino/tools/avr-gcc/7.3.0-atmel3.6.1-arduino5/bin/avr-g++
    -c
    -g
    -Os
    -w
    -std=gnu++11
    -fpermissive
    -fno-exceptions
    -ffunction-sections
    -fdata-sections
    -fno-threadsafe-statics
    -Wno-error=narrowing
    -flto
    -w
    -x C++
    -E
    -CC
    -mmcu=atmega328p
    -DF_CPU=16000000L
    -DARDUINO=10810
    -DARDUINO_AVR_UNO
    -DARDUINO_ARCH_AVR
    -I/home/marcidy/.arduino15/packages/arduino/hardware/avr/1.8.2/cores/arduino
    -I/home/marcidy/.arduino15/packages/arduino/hardware/avr/1.8.2/variants/standard 
    /tmp/arduino_build_709419/sketch/Fade.ino.cpp
    -o /tmp/arduino_build_709419/preproc/ctags_target_for_gcc_minus_e.cpp

Here we see the beginning of the compilation tool-chain for avr. 

Quick pause to explain what's happening.  Your computer where you write arduino code is probably Windows, Mac, or Linux, and most likely runs an Intel or AMD processor. There's nothing fundamentally different from those processors and an avr.  You can compile code for them, and you end up with a binary file which they can execture.  This is all explained in detail in :ref:`compilation_primer`.  

However, key to what we are doing, is that we are not compiling for your computer.  We are compiling for a completely difference processor.  This is called `cross compiling`.  Our computer is the host where we are executing commands to compile for the target machine, in this case an Arduino Uno which has an Atmel AVR ATMega328p microcontroller.

Let's look at `avr-g++` command and the options it takes.

avr-g++
-------

This is where you need to really start paying close attention.  

You need to understand which options passed to this command will change based on the chip you are programming.  Once you use this command, you are outside of the Arduino build environment and need valid C++ (ie not a .ino file) and where all the libraries you use are located.  Especially the libraries you didn't know you used, like some libraries which ship with the Ardiuno IDE.  

Not that we need to stick to the Arduino libraries, but we will for now.  Just understand they are libraries going forward, and we compile against them like any other library.  In a different section look at how to replace these.  

See :ref:`arduino_builder_preproc` to see how the sketch gets converted too valid C++ in the build directory.

`avr-g++` is part of `gcc` which is the Gnu C Compilier.  It gets a bit compilicated talking about what exactly is `avr-g++` vs `gcc`, so don't worry too much about it.  It suffices to say that `avr-g++` is a free, open source compilier for code which needs to compile to a binary that an avr microcontroller can run.  It's not the only compilier, but it being free and open source means it can be included with the Arduino IDE, and likewise you can use it, without paying.  

If you want a better idea of what it means for a binary to be run by a microcontroller, check out :ref:`architecture_introduction`.

The compilier will read source files, and convert what is written in those source files into a binary executable. A natural question is "how?", which of course has many layers and is complicated, and discussed in :ref:`compilation_primer`.

However, it is critical to know (not understand just yet) that "compilation" is a multi-step process:

    1. Pre-processing
    2. Compilation
    3. Assembly
    4. Linking

Note that the first step "Pre-processing" mentioned here is not the same pre-processing that `arduino_builder` does.  `arduino_builder` did it's own pre-processing, collecting data about the project stored by the IDE, and converted that into data used by these commands.  This step is preprocessing related only to C++ source files.

Let's look at the options passed to `avr-g++` and why, and note these steps in order.  Oh look, even though we are compiling, one of the steps to compiling is compiling.  It's more appropriate to think of "compiling a program" as a 4 step process, where one of those steps is "compile the code to assembly".

First, let's quickly look at `avr-g++ --help`:

:: 

    $ avr-g++ --help
      --version                Display compiler version information
      -dumpspecs               Display all of the built in spec strings
      -dumpversion             Display the version of the compiler
      -dumpmachine             Display the compiler's target processor
      -print-search-dirs       Display the directories in the compiler's search path
      -print-libgcc-file-name  Display the name of the compiler's companion library
      -print-file-name=<lib>   Display the full path to library <lib>
      -print-prog-name=<prog>  Display the full path to compiler component <prog>
      -print-multiarch         Display the target's normalized GNU triplet, used as
                               a component in the library path
      -print-multi-directory   Display the root directory for versions of libgcc
      -print-multi-lib         Display the mapping between command line options and
                               multiple library search directories
      -print-multi-os-directory Display the relative path to OS libraries
      -print-sysroot           Display the target libraries directory
      -print-sysroot-headers-suffix Display the sysroot suffix used to find headers
      -Wa,<options>            Pass comma-separated <options> on to the assembler
      -Wp,<options>            Pass comma-separated <options> on to the preprocessor
      -Wl,<options>            Pass comma-separated <options> on to the linker
      -Xassembler <arg>        Pass <arg> on to the assembler
      -Xpreprocessor <arg>     Pass <arg> on to the preprocessor
      -Xlinker <arg>           Pass <arg> on to the linker
      -save-temps              Do not delete intermediate files
      -save-temps=<arg>        Do not delete intermediate files
      -no-canonical-prefixes   Do not canonicalize paths when building relative
                               prefixes to other gcc components
      -pipe                    Use pipes rather than intermediate files
      -time                    Time the execution of each subprocess
      -specs=<file>            Override built-in specs with the contents of <file>
      -std=<standard>          Assume that the input sources are for <standard>
      --sysroot=<directory>    Use <directory> as the root directory for headers
                               and libraries
      -B <directory>           Add <directory> to the compiler's search paths
      -v                       Display the programs invoked by the compiler
      -###                     Like -v but options quoted and commands not executed
      -E                       Preprocess only; do not compile, assemble or link
      -S                       Compile only; do not assemble or link
      -c                       Compile and assemble, but do not link
      -o <file>                Place the output into <file>
      -pie                     Create a position independent executable
      -shared                  Create a shared library
      -x <language>            Specify the language of the following input files
                               Permissible languages include: c C++ assembler none
                               'none' means revert to the default behavior of
                               guessing the language based on the file's extension
    
    Options starting with -g, -f, -m, -O, -W, or --param are automatically
     passed on to the various sub-processes invoked by avr-g++.  In order to pass
     other options on to these processes the -W<letter> options must be used.
    
So we see that some options (paramters) are consumed by `avr-g++` directly while others are passed along to 'various sub-processes invoked by avr-g++'.  This is related to the multi-step process called `compilation`.

The relevent `avr-g++` options invoked by Arduino:

    `-c`
        Compile and assemble, but do not link
    `-std=gnu++11`
        Assume that the input sources are for gnu++11.  This is the standard to which the code is written, and is like a flavor of C++.  As with all things, programming languages evolve over time, and compilers and code must understand the same version or flavor of the language.  
    `-x C++`
        We're using C++, not C or some other language.
    `-E`
        Preprocess only; do not compile, assemble or link.  This seems at odds with `-c`, however when used together it means "First preprocess only, then compile and and assemble only", which is steps 1 through 3 of compilation.
    `-o <filename>`
        Output the results into <filename>.  Here, the /dev/null is a special file on Linux which looks like a file, but really just discards the output.  Seems weird, but what's actually happening is that the useful output is produced by sub-processes.

The other options aren't consumed directly by avr-g++ and are passed to the sub-processes that avr-g++ uses.  Let's take a look at some of them.

    `-D`
        These set variables which are used during the execution of the rest of the sub-processes.  Anytime you see a `-D`X=Y`, the X becomes a variable and Y is the value of that variable for the sub-processes.

    `-DF_CPU=16000000L`
        This sets a variable to the frequency of the microcontroller clock on the target board.  The `L` means "long" as in "long integer".
    `-DARDUINO=10810`
        My version of othe arduino IDE again.
    `-DARDUINO_AVR_UNO`
        This is setting the existance of a variable.  You can think of it like a boolean.  If this variable exists, then the sub-processes can do things which are required by the UNO target.
    `-DARDUINO_ARCH_AVR`
        Another boolean-like variable.  Since the AVR architecture is used by more than just the UNO, this varialbe lets sub-processes do things that are AVR specific rather than just UNO specific.
    `-I`
        These options tell the sub-processes where to look for important files, like libraries
    `-I/home/marcidy/.arduino15/packages/arduino/hardware/avr/1.8.2/cores/arduino`
        You should look in this directory on your machine.  You'll note some very interesting files here.  We'll dig into these in other sections on this site.
    `-I/home/marcidy/.arduino15/packages/arduino/hardware/avr/1.8.2/variants/standard`
        Really you should read all files in all directories indicated by these steps are they are clearly important to compiling your code.  These are low-level definitions of variables, macros, and structures used by Arduino libraries.
    `/tmp/arduino_build_709419/sketch/Fade.ino.cpp`
       This file is in the build directory for your project, created by `arduino-builder`, and a ".cpp" extension has been added to the ".ino".  Part of what `arduino-builder` does is rewrite your file so it's actually valid C++.  Sketches themselves are not a complete program, nor strictly valid C++, but make up part of a larger program which gets compiled.  We'll see that as we move along.  When you write code outside the ecosystem, it will have to be valid C++, so it's important to see where and how this step occurs between "sketch" and code.  See :ref:`arduino_builder_preproc` for some on this.


Ctags
-----

Ctags is outside the scope of this guide, but you can read about it on line.  Basically it's a tool which creates an index of all the C++ "stuff" you are using, so it can be looked up and found by other tools.  The following step uses this file.  

The first invocation of avr-g++ preprocess the cpp file for use with ctags.  

The next line invokes the ctags tool.  Again, we're going to ignore it.
::

    /home/marcidy/arduino-1.8.10/tools-builder/ctags/5.8-arduino11/ctags
        -u
        --language-force=c++
        -f
        -
        --c++-kinds=svpf
        --fields=KSTtzns
        --line-directives 
        /tmp/arduino_build_709419/preproc/ctags_target_for_gcc_minus_e.cpp

Compilation
===========

Compilation requires knowing things like the types of all variables, the types of data returned by functions, the types of all arguments to all functions (they are just variables, after all), and the order of the arguments in each function.

The compilier doesn't need ot know the definition of each function and variable at this stage, but does need to know the types associated with function returns, function arguements, and variable types.  These are typically defined in header files.


Libraries and #includes
-----------------------

Note that this isn't the same thing as knowing the definition of each function or the value of each variable.  Type information tells the compilier how to interpret some binary strings like `11011001` as an integer or a memory address.  

With this information, the compilier can do two things:
    1. Check that we don't accidentally try to use functions or variables inconsistently, ie use an address where a value is required
    2. Create the right instruction, as some times we want to do the same operation, but on different types.

To get a better idea of this, read through :ref:`architecture_introduction` and why sometimes we want to do something like `AND 1 0` which is with values and other times we want to do `AND A B` which is with values stored in memory.

For any specific file being compiled, we need to have those definitions either in the file, or in a file referenced by a `#include` directive.  Then we need to tell the compiler where it should look for the files.

Let's see how that's handled.

::

    /home/marcidy/.arduino15/packages/arduino/tools/avr-gcc/7.3.0-atmel3.6.1-arduino5/bin/avr-g++
        -c
        -g
        -Os
        -w
        -std=gnu++11
        -fpermissive
        -fno-exceptions
        -ffunction-sections
        -fdata-sections
        -fno-threadsafe-statics
        -Wno-error=narrowing
        -MMD
        -flto
        -mmcu=atmega328p
        -DF_CPU=16000000L
        -DARDUINO=10810
        -DARDUINO_AVR_UNO
        -DARDUINO_ARCH_AVR
        -I/home/marcidy/.arduino15/packages/arduino/hardware/avr/1.8.2/cores/arduino
        -I/home/marcidy/.arduino15/packages/arduino/hardware/avr/1.8.2/variants/standard 
        /tmp/arduino_build_709419/sketch/Fade.ino.cpp
        -o /tmp/arduino_build_709419/sketch/Fade.ino.cpp.o
       
Ok, here we see an actual compilation step.  The input is the `.cpp` file and the output is a `.o` object file.  Had we more files, we'd see more lines indicating compilation of the individual files to object code.  

Looking in the destination directory `/tmp/arduino_build_709419/sketch/` we actually see two files created: `Fade.ino.cpp.o` and `Fade.ino.cpp.d`. 

Before looking at the files, let's look at the options passed to avr-g++ actually.  Let's start from the bottom.

::

    -o /tmp/arduino_build_709419/sketch/Fade.ino.cpp.o

The "-o" flag tells the program to produce a file with the name provided.

::

    /tmp/arduino_build_709419/sketch/Fade.ino.cpp

This is the file being compiled. It's the input to the compiler.

::

    -I/home/marcidy/.arduino15/packages/arduino/hardware/avr/1.8.2/cores/arduino
    -I/home/marcidy/.arduino15/packages/arduino/hardware/avr/1.8.2/variants/standard 

Remember the "-I" is for libraries.  Specifically "other files to include when compiling".  We need files in these directories becuase they are either explicitly in `#include` statements in our file, or are included by other files which have been included.  Let's look at what's included in Fade.ino.

::

    #include <Arduino.h>

Only `Arduino.h`.  Hmm.  If you look in the directories mentioned above in the `-I` options, you'll find this file.

Well, if you are curious, checkout :ref:`arduino_libraries` to dig deeper through it and it's includes.  This part of the guide is just going to focus on command line compilation.  If you are curious where all the definitions are for the things which are otherwise magically working, that's a good place to look.

The take away from here is that you need to tell the compiler where to look for files that you include in the compilation.  If it can't find a file mentioned in an `#include`, it will fail.  There's no magic about where the compilier is going to look for files.

Build Variables
---------------

If you went through :ref:`arduino_libraries`, you learned where the libraries will make compiliation decisions based on the target hardware.  We can pass variables into the pre-processing step of compilation by using the `-D` option on the command line

::

    -DF_CPU=16000000L
    -DARDUINO=10810
    -DARDUINO_AVR_UNO
    -DARDUINO_ARCH_AVR

By setting build variables on the command line, we can make decisions about what code to include and how to implement code for different architectures, and the compilier will only include the code relevent for our architecture.

For example, `-DF_CPU` sends the default clock frequency of the target.  Some boards at 8MHz, some are 16MHz, etc.  This extremely important for things like serial ports and other communication.  Now any code which requires specific timing, like a 9600 baud rate (which means 9600 bits/s), we can write code which can know exactly how many instruction cycles it needs to maintain that baud rate.  CPUs don't have any knowledge of time, and difference processors can do more or less amount of work in a given amount of time.  They work in a cycle, and the CPU clock frequency tells the processor how long a cycle is.  Once it has this information, it can convert back and forth between cycles and time.

There's nothing particularly special about build variabled, they provide a very useful way to pass information into the compiliation process via setting values that will be used in preprocessing directives.  

So we need to do some work to tell the compilier exactly whwat code will work on the target architecture, both when specifying on the command line, and in the code itself.  A major example for that is pin assignments, since all MCUs don't have the same pins.  Using switches on the command line we can tell the compilier which board we're using, and define the pins list in a file that is specific to the board for which we are compiling.

Compilation Output
------------------

The output of compilation in this case are two files, `Fade.ino.cpp.o` and `Fade.ino.cpp.d`.  

Dependencies
____________

`Fade.ino.cpp.d` is a helper file which tells the compilier what files are required to compile `Fade.ino.cpp`.  It's a list of dependencies.  It's a simple file, let's take a look:

::

    /tmp/arduino_build_709419/sketch/Fade.ino.cpp.o: \
        /tmp/arduino_build_709419/sketch/Fade.ino.cpp \
        /home/marcidy/.arduino15/packages/arduino/hardware/avr/1.8.2/cores/arduino/Arduino.h \
        /home/marcidy/.arduino15/packages/arduino/hardware/avr/1.8.2/cores/arduino/binary.h \
        /home/marcidy/.arduino15/packages/arduino/hardware/avr/1.8.2/cores/arduino/WCharacter.h \
        /home/marcidy/.arduino15/packages/arduino/hardware/avr/1.8.2/cores/arduino/WString.h \
        /home/marcidy/.arduino15/packages/arduino/hardware/avr/1.8.2/cores/arduino/HardwareSerial.h \
        /home/marcidy/.arduino15/packages/arduino/hardware/avr/1.8.2/cores/arduino/Stream.h \
        /home/marcidy/.arduino15/packages/arduino/hardware/avr/1.8.2/cores/arduino/Print.h \
        /home/marcidy/.arduino15/packages/arduino/hardware/avr/1.8.2/cores/arduino/Printable.h \
        /home/marcidy/.arduino15/packages/arduino/hardware/avr/1.8.2/cores/arduino/USBAPI.h \
        /home/marcidy/.arduino15/packages/arduino/hardware/avr/1.8.2/cores/arduino/Arduino.h \
        /home/marcidy/.arduino15/packages/arduino/hardware/avr/1.8.2/variants/standard/pins_arduino.h

This file has searched through all the includes of all the files included in `Fade.ino.cpp`.  We know that `Arduino.h` is the only file included directly, however, `Arduino.h` also has a list of includes, and those includes have includes, etc.  Each define types required to finally compile `Fade.ino.cpp`.


Object Code
___________

`Fade.ino.cpp.o` is compiled object code.  This code reflect the source code in `Fade.ino.cpp` compiled to architecture specific assembly, but without filling in definitions which are not yet known.  So it is a combination of assembly code and references.

It's possile to inspect the file using a disassembler, conveniently included with the avr tool chain.  It's called "dumping" and invoked as:

::

    avr-objdump -D Fade.ino.cpp.o

If you do this, you'll see the assembly code represented by the object:

::

    Fade.ino.cpp.o:     file format elf32-avr


    Disassembly of section .gnu.lto_.profile.c49527905a8810bc:
    
    00000000 <.gnu.lto_.profile.c49527905a8810bc>:
       0:   78 9c           mul     r7, r8
       2:   63 63           ori     r22, 0x33       ; 51
       4:   60 60           ori     r22, 0x00       ; 0
       6:   60 01           movw    r12, r0
       8:   62 46           sbci    r22, 0x62       ; 98
       a:   06 13           cpse    r16, r22
       c:   71 00           .word   0x0071  ; ????
       e:   00 f7           brcc    .-64            ; 0xffffffd0 <__SREG__+0xffffff91>
      10:   00 57           subi    r16, 0x70       ; 112
    
    Disassembly of section .gnu.lto_.icf.c49527905a8810bc:
    
    00000000 <.gnu.lto_.icf.c49527905a8810bc>:
       0:   78 9c           mul     r7, r8
       2:   63 63           ori     r22, 0x33       ; 51
       4:   60 60           ori     r22, 0x00       ; 0
       6:   50 60           ori     r21, 0x00       ; 0
       8:   40 00           .word   0x0040  ; ????
       a:   56 86           std     Z+14, r5        ; 0x0e
       c:   bf eb           ldi     r27, 0xBF       ; 191
       e:   9e 1d           adc     r25, r14
      10:   e2 67           ori     r30, 0x72       ; 114
      12:   3c 72           andi    r19, 0x2C       ; 44
      14:   6b e5           ldi     r22, 0x5B       ; 91
      16:   14 2e           mov     r1, r20
      18:   a6 7d           andi    r26, 0xD6       ; 214
      1a:   9b ae           std     Y+59, r9        ; 0x3b
      1c:   cc 65           ori     r28, 0x5C       ; 92
      1e:   63 de           rcall   .-826           ; 0xfffffce6 <__SREG__+0xfffffca7>
      20:   d4 b0           in      r13, 0x04       ; 4
      22:   f4 2e           mov     r15, r20
      24:   37 cb           rjmp    .-2450          ; 0xfffff694 <__SREG__+0xfffff655>
      26:   e5 8f           std     Z+29, r30       ; 0x1d
      28:   2d 8f           std     Y+29, r18       ; 0x1d
      2a:   78 19           sub     r23, r8
      2c:   00 04           cpc     r0, r0
      2e:   bc 0f           add     r27, r28
      30:   5a 73           Address 0x0000000000000030 is out of bounds.
    .word   0xffff  ; ????
    
    Disassembly of section .gnu.lto_.jmpfuncs.c49527905a8810bc:
    
    00000000 <.gnu.lto_.jmpfuncs.c49527905a8810bc>:
       0:   78 9c           mul     r7, r8
       2:   63 63           ori     r22, 0x33       ; 51
       4:   60 60           ori     r22, 0x00       ; 0


Here we see the instructions in binary and in assembly, along with "sections".  This information is used to link all the difference objects together or order them in the final binary.

Looking at specific instructions, you can reference back to the avr instruction set to understand each specific line.  In many for things like `rjmp` and `rcall` and other instructions which represent moving around the program, placeholders are used until the final binary is linked together.


There's now a lot of repition of compiliation, once for each file which we need.  There's a few notable exceptions.

Compiling with Assembly
_______________________

The next line deals with including assembly directly in a `.S` file.

::
    
    /home/marcidy/.arduino15/packages/arduino/tools/avr-gcc/7.3.0-atmel3.6.1-arduino5/bin/avr-gcc 
       
        -c
        -g
        -x assembler-with-cpp
        -flto
        -MMD
        -mmcu=atmega328p 
        -DF_CPU=16000000L
        -DARDUINO=10810
        -DARDUINO_AVR_UNO
        -DARDUINO_ARCH_AVR 
        -I/home/marcidy/.arduino15/packages/arduino/hardware/avr/1.8.2/cores/arduino 
        -I/home/marcidy/.arduino15/packages/arduino/hardware/avr/1.8.2/variants/standard 
        /home/marcidy/.arduino15/packages/arduino/hardware/avr/1.8.2/cores/arduino/wiring_pulse.S 
        -o /tmp/arduino_build_709419/core/wiring_pulse.S.o

Recalling that `-x` means the language we'll compile for, we see it's `assembler-with-cpp`.  If you look at that file, you'll see exactly that, some assembly.   Note that the extension is `.S`.  It's also possible to include assembly in `.s` files, but these mean different things, and require different steps.  For now, beyond the scope of this guide.

Note also that the command is `avr-gcc` rather than `avr-g++`.  This is the `Gnu C Compilier`.  The next line also uses `avr-gcc` but to compile `C` code.  The differences between these languages is in what the typed symbols mean, and is reflected in the different Grammar for each lanauge. :ref:`compilation_primer` to understand what that means.

Compiling with C code
_____________________

::

    /home/marcidy/.arduino15/packages/arduino/tools/avr-gcc/7.3.0-atmel3.6.1-arduino5/bin/avr-gcc 
        -c
        -g
        -Os
        -w
        -std=gnu11
        -ffunction-sections
        -fdata-sections
        -MMD
        -flto
        -fno-fat-lto-objects 
        -mmcu=atmega328p
        -DF_CPU=16000000L
        -DARDUINO=10810
        -DARDUINO_AVR_UNO
        -DARDUINO_ARCH_AVR 
        -I/home/marcidy/.arduino15/packages/arduino/hardware/avr/1.8.2/cores/arduino 
        -I/home/marcidy/.arduino15/packages/arduino/hardware/avr/1.8.2/variants/standard 
        /home/marcidy/.arduino15/packages/arduino/hardware/avr/1.8.2/cores/arduino/WInterrupts.c 
        -o /tmp/arduino_build_709419/core/WInterrupts.c.o

Here's an example of compiling `C` code as part of the project.  All these files that aren't Fade.ino are included by Arduino and aren't part of the avr-gcc libraries, which is why they are compiled as part of the project.

Maybe you're wondering why since Fade.ino doens't use wiring or interrupts.  Actually the Serial port does, and if you inspected `main.cpp`, you'd see that we do need them to use the serial port.

main.cpp
________

As noted in :ref:`compilation_primer`, code must include a main function.  By default, arduino uses the main function in `main.cpp`

::

    /home/marcidy/.arduino15/packages/arduino/tools/avr-gcc/7.3.0-atmel3.6.1-arduino5/bin/avr-g++
    -c
    -g
    -Os
    -w
    -std=gnu++11
    -fpermissive
    -fno-exceptions
    -ffunction-sections
    -fdata-sections
    -fno-threadsafe-statics
    -Wno-error=narrowing
    -MMD
    -flto
    -mmcu=atmega328p
    -DF_CPU=16000000L
    -DARDUINO=10810
    -DARDUINO_AVR_UNO
    -DARDUINO_ARCH_AVR
    -I/home/marcidy/.arduino15/packages/arduino/hardware/avr/1.8.2/cores/arduino
    -I/home/marcidy/.arduino15/packages/arduino/hardware/avr/1.8.2/variants/standard /home/marcidy/.arduino15/packages/arduino/hardware/avr/1.8.2/cores/arduino/main.cpp
    -o /tmp/arduino_build_709419/core/main.cpp.o


If you are writing your own code, you would write the main function yourself.  This file just sets up the serial port, runs the `setup()` function once, and runs `loop()` forever, tucked inside a `for` loop with no stop condition.  If you write your own `main`, arduino will skip this inclusion during pre-processing.


The `main` function tells the compiler where to place the first instruction to be executed.  More specifically, every microcontroller has some special address to load the first instruction to start the program.  The first instruction generated by compiling `main` will go into that address.  It means "start here", and whatever that translates to in terms of loading the program into memory will be managed based on the target architecture.


Archive files
_____________

The remainder of the comamnds look like this:

::

    /home/marcidy/.arduino15/packages/arduino/tools/avr-gcc/7.3.0-atmel3.6.1-arduino5/bin/avr-gcc-ar 
    rcs 
    /tmp/arduino_build_709419/core/core.a 
    /tmp/arduino_build_709419/core/wiring_pulse.S.o

The `avr-gcc-ar` command creates an archive file (`core.a` in this case) and packs the object files into it.  Here, the compilier is taking all the intermediary object files (`.o`) and sticking them in the build directory in the `core.a` archive.

These files won't need to be compiled again, as they are part of Arduino and you shouldn't edit them.  Note that `Fade.ino.cpp.o` is not packed into that archive.

This step is useful for any project to cache compiled objects which won't change.  It's not strictly necessary, but speeds up build times for subsequent compiles.  

In the linking step, the linker will make use of this archive to find all the compiled objects in one place.


After running all these commands, the directory `/tmp/arduino_build_709419/core` looks like

::

    abi.cpp.d              HardwareSerial3.cpp.d  new.cpp.o           WInterrupts.c.d     wiring_pulse.S.o
    abi.cpp.o              HardwareSerial3.cpp.o  PluggableUSB.cpp.d  WInterrupts.c.o     wiring_shift.c.d
    CDC.cpp.d              HardwareSerial.cpp.d   PluggableUSB.cpp.o  wiring_analog.c.d   wiring_shift.c.o
    CDC.cpp.o              HardwareSerial.cpp.o   Print.cpp.d         wiring_analog.c.o   WMath.cpp.d
    core.a                 hooks.c.d              Print.cpp.o         wiring.c.d          WMath.cpp.o
    HardwareSerial0.cpp.d  hooks.c.o              Stream.cpp.d        wiring.c.o          WString.cpp.d
    HardwareSerial0.cpp.o  IPAddress.cpp.d        Stream.cpp.o        wiring_digital.c.d  WString.cpp.o
    HardwareSerial1.cpp.d  IPAddress.cpp.o        Tone.cpp.d          wiring_digital.c.o
    HardwareSerial1.cpp.o  main.cpp.d             Tone.cpp.o          wiring_pulse.c.d
    HardwareSerial2.cpp.d  main.cpp.o             USBCore.cpp.d       wiring_pulse.c.o
    HardwareSerial2.cpp.o  new.cpp.d              USBCore.cpp.o       wiring_pulse.S.d


We have object files and dependency files for all the source files needed to compile `Fade.ino.cpp`.


Linking
=======
Once the compilier has found all the necessary inputs and compiled them, it assembles a single file.  Without going into the details, this means it has taken all the different functions which have been compiled to target specific assembly code and aranged them into a single file on the host operating system.  It has made a lot of decisions behind the scenes, like where to place each function, which is now just a list of machine instructions.

Looking at the command which does the linking, we can see the inputs and outputs fromm that command.

::

    /home/marcidy/.arduino15/packages/arduino/tools/avr-gcc/7.3.0-atmel3.6.1-arduino5/bin/avr-gcc
        -w
        -Os
        -g
        -flto
        -fuse-linker-plugin
        -Wl,--gc-sections
        -mmcu=atmega328p
        -o /tmp/arduino_build_709419/Fade.ino.elf 
        /tmp/arduino_build_709419/sketch/Fade.ino.cpp.o 
        /tmp/arduino_build_709419/../arduino_cache_430568/core/core_arduino_avr_uno_2bd201547ef1722ab59b0c23270fe17e.a
        -L/tmp/arduino_build_709419
        -lm

This command tells `avr-gcc` to link the files `Fade.ino.cpp.o` and `core.a` 

The output of linking is an `.elf` file, which stands for 'executable and linkable format.  You can read more about it at `Wikipedia <https://en.wikipedia.org/wiki/Executable_and_Linkable_Format>`_.  This file contains more information that what is required to run the program.  It also has metadata which explains how to interpret different sections of the file.  This file cannot be directly interpretted by the target architecture as a binary, but cains all the information needed to do so.  It's a representation of the firmware, but not yet the firmware we are going to load onto the processor.

The `-L` option tells the linker where to find object code that has been compiled elsewhere. While that sounds similar to the `-I` directive above, the files passed to `-L` must have been already compiled for the target architecture.  These are also libraries, but can be referred to as _shared_ libraries, which isn't too relevent for this guide.  They are shared in the sense that all programs on a machine can share them without including them specifically, so they can be compiled once and saved, and then any program can link to them.  It makes far more sense when programming on a machine capable of running multiple programs than for an Arduino target.  


`'-l` is another way to specify code to include in the program.  It takes abbreviated names for system libraries.  System libraries are part of the language which are included with the lanauge, but not necessarily included in compilation without explicit request.  IF you've heard of the standard library, which is always included, that's called `libc`.  The math library is called `libm`'.  With the `-l` option, we can drop the 'lib' prefix.  So here we are telling the linker to include the math library.  The standard library is always included.

So these options tell the linker what to link together:

::

        /tmp/arduino_build_709419/sketch/Fade.ino.cpp.o
        /tmp/arduino_build_709419/../arduino_cache_430568/core/core_arduino_avr_uno_2bd201547ef1722ab59b0c23270fe17e.a
        -L/tmp/arduino_build_709419
        -lm


The `arduino_cache_430568` directory caches, or saves, the output from previous steps, and we can re-use those results without haveing to recompile files which haven't changed.  Arduino builder created the file and placed some required information in it.  


We now have `Fade.ino.elf`.  Let's take a look at the last few commands run during the compilation.

.. _avr-objcopy:

avr-objcopy
===========
We need to translate the ELF file into something that a programmer can understand, since our goal is to use a programmer to load the file onto the chip.

The programmer needs to know where various parts of the code go.  Some of the code is executable instructions, some is data.  Different programmers use different file formats to create the stream of data send over the serial cable to program the chip, to the same net effect.

`eep` files are for the EEPROM (Electrically Erasable Programmable Read Only Memory).  These contain data.
`hex` files are the instructions, or program.

`avr-objcopy` is used for a number of tasks, but here it's to create the `hex` and `eep` files.

::

    /home/marcidy/.arduino15/packages/arduino/tools/avr-gcc/7.3.0-atmel3.6.1-arduino5/bin/avr-objcopy 
        -O ihex 
        -j .eeprom 
        --set-section-flags=.eeprom=alloc,load 
        --no-change-warnings 
        --change-section-lma .eeprom=0 
        /tmp/arduino_build_709419/Fade.ino.elf 
        /tmp/arduino_build_709419/Fade.ino.eep
    /home/marcidy/.arduino15/packages/arduino/tools/avr-gcc/7.3.0-atmel3.6.1-arduino5/bin/avr-objcopy 
        -O ihex 
        -R .eeprom 
        /tmp/arduino_build_709419/Fade.ino.elf 
        /tmp/arduino_build_709419/Fade.ino.hex
        
These files will be used by the programmer to load the firmware onto the chip.

And finally, to get the final size of the firmware:

::

    /home/marcidy/.arduino15/packages/arduino/tools/avr-gcc/7.3.0-atmel3.6.1-arduino5/bin/avr-size -A /tmp/arduino_build_709419/Fade.ino.elf

    /tmp/arduino_build_709419/Fade.ino.elf  :
    section                     size      addr
    .data                          2   8388864
    .text                       1142         0
    .bss                          11   8388866
    .comment                      17         0
    .note.gnu.avr.deviceinfo      64         0
    .debug_aranges               136         0
    .debug_info                 3850         0
    .debug_abbrev               2148         0
    .debug_line                 1573         0
    .debug_frame                 140         0
    .debug_str                  1209         0
    .debug_loc                  1803         0
    .debug_ranges                160         0
    Total                      12255

To understand what these sections are, search ELF file format and meaning for each section.

At this stage, have all the data we need to upload the firmware.  :ref:`uploading_firmware` handles that.
