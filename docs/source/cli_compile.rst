.. _command_line_compilation:

========================
Command Line Compilation
========================
At this stage we have a valid C++ file, maybe output from :ref:`arduino_builder` preprocessing the sketch.  We saw in that section that the IDE actually executes a number of commands not echoed to the screen:

.. literalinclude:: arduino_build_compile_compressed.txt

We'll go line by line.  I've expanded the first to split out all the options passed to avr-g++.

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
    -o /dev/null

Here we see the beginning of the compilation tool-chain for avr.

Quick pause to explain what's happening.  Your computer where you write arduino code is probably Windows, Mac, or Linux, and most likely runs an Intel or AMD processor. There's nothing fundamentally different from those processors and an avr.  You can compile code for them, and you end up with a binary file which they can execture.  This is all explained in detail in :ref:`compilation_primer`.  

However, key to what we are doing, is that we are not compiling for your computer.  We are compiling for a completely difference processor.  This is called `cross compiling`.  Our computer is the host where we are executing commands to compile for the target machine, in this case an Arduino Uno which has an Atmel AVR ATMega328p microcontroller.

Let's look at this command in more depth.

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
        -flto
        -w
        -x c++
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

The first invocation of avr-g++ preprocess the cpp file for use with ctags.  

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

This is the actual ctags tool running.  Again, we're going to ignore it.

Libraries and #includes
-----------------------

Compilation requires knowing things like the types of all variables, the types of data returned by functions, the types of all arguments to all functions (they are just variables, after all), and the order of the arguments in each function.

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

It's time to start looking at what those options to avr-g++ actually mean.  Let's start from the bottom.

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

Only `Arduino.h`.  Hmm.  If you look in the directories mentioned above, you'll find this file.

Well, if your more curious, checkout :ref:`arduino_libraries` to dig deeper through it and it's includes.  This part of the guide is just going to focus on command line compilation.  If you are curious where all the definitions are for the things which are otherwise magically working, that's a good place to look.

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

We see that a different command as been invoked, `avr-gcc`. 

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

If we compile without it, we get an error:

::

    /home/marcidy/.arduino15/packages/arduino/tools/avr-gcc/7.3.0-atmel3.6.1-arduino5/bin/../lib/gcc/avr/7.3.0/../../../../avr/lib/avr5/crtatmega328p.o:(.init9+0x0): undefined reference to `main'
    collect2: error: ld returned 1 exit status


This is a bit cryptic, but C/C++ requires that there be one, and only one, function named "main".  The location of this function is mentioned in :ref:`arduino_libraries`, and detailed further in :ref:`maincpp`.

However, the linker is complaining about crtatemega328p.o

If we look in that location of that file, we see a large number of files related to different chips, as well as `libc` and `libm`.

This directory is storing the architecture specific object code required for compiliation to our target architecture.  

The issue here is a step performed by `arduino_builder` has not been echoed to the screen, and we've missed the step which picks up all the architecture specific object code and puts it in the build cache.

The major inputs to the linker are compiled files.  These can be code compiled specifically as part of your project, or other files found elsewhere that have been compiled for the target architecture.  
