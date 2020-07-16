.. _arduino_builder:

===============
Arduino Builder
===============

'arduino-builder' is a tool provided along with the Arduino IDE.  Let's take a look at how the command is invoked when the commpile button is pressed:
    
::
    
        /home/marcidy/arduino-1.8.10/arduino-builder -dump-prefs -logger=machine -hardware /home/marcidy/arduino-1.8.10/hardware -hardware /home/marcidy/.arduino15/packages -hardware /home/marcidy/Arduino/hardware -tools /home/marcidy/arduino-1.8.10/tools-builder -tools /home/marcidy/arduino-1.8.10/hardware/tools/avr -tools /home/marcidy/.arduino15/packages -built-in-libraries /home/marcidy/arduino-1.8.10/libraries -libraries /home/marcidy/Arduino/libraries -fqbn=arduino:avr:uno -ide-version=10810 -build-path /tmp/arduino_build_709419 -warnings=none -build-cache /tmp/arduino_cache_430568 -prefs=build.warn_data_percentage=75 -prefs=runtime.tools.arduinoOTA.path=/home/marcidy/.arduino15/packages/arduino/tools/arduinoOTA/1.3.0 -prefs=runtime.tools.arduinoOTA-1.3.0.path=/home/marcidy/.arduino15/packages/arduino/tools/arduinoOTA/1.3.0 -prefs=runtime.tools.avr-gcc.path=/home/marcidy/.arduino15/packages/arduino/tools/avr-gcc/7.3.0-atmel3.6.1-arduino5 -prefs=runtime.tools.avr-gcc-7.3.0-atmel3.6.1-arduino5.path=/home/marcidy/.arduino15/packages/arduino/tools/avr-gcc/7.3.0-atmel3.6.1-arduino5 -prefs=runtime.tools.avrdude.path=/home/marcidy/.arduino15/packages/arduino/tools/avrdude/6.3.0-arduino17 -prefs=runtime.tools.avrdude-6.3.0-arduino17.path=/home/marcidy/.arduino15/packages/arduino/tools/avrdude/6.3.0-arduino17 -verbose /home/marcidy/arduino-1.8.10/examples/01.Basics/Fade/Fade.ino
        /home/marcidy/arduino-1.8.10/arduino-builder -compile -logger=machine -hardware /home/marcidy/arduino-1.8.10/hardware -hardware /home/marcidy/.arduino15/packages -hardware /home/marcidy/Arduino/hardware -tools /home/marcidy/arduino-1.8.10/tools-builder -tools /home/marcidy/arduino-1.8.10/hardware/tools/avr -tools /home/marcidy/.arduino15/packages -built-in-libraries /home/marcidy/arduino-1.8.10/libraries -libraries /home/marcidy/Arduino/libraries -fqbn=arduino:avr:uno -ide-version=10810 -build-path /tmp/arduino_build_709419 -warnings=none -build-cache /tmp/arduino_cache_430568 -prefs=build.warn_data_percentage=75 -prefs=runtime.tools.arduinoOTA.path=/home/marcidy/.arduino15/packages/arduino/tools/arduinoOTA/1.3.0 -prefs=runtime.tools.arduinoOTA-1.3.0.path=/home/marcidy/.arduino15/packages/arduino/tools/arduinoOTA/1.3.0 -prefs=runtime.tools.avr-gcc.path=/home/marcidy/.arduino15/packages/arduino/tools/avr-gcc/7.3.0-atmel3.6.1-arduino5 -prefs=runtime.tools.avr-gcc-7.3.0-atmel3.6.1-arduino5.path=/home/marcidy/.arduino15/packages/arduino/tools/avr-gcc/7.3.0-atmel3.6.1-arduino5 -prefs=runtime.tools.avrdude.path=/home/marcidy/.arduino15/packages/arduino/tools/avrdude/6.3.0-arduino17 -prefs=runtime.tools.avrdude-6.3.0-arduino17.path=/home/marcidy/.arduino15/packages/arduino/tools/avrdude/6.3.0-arduino17 -verbose /home/marcidy/arduino-1.8.10/examples/01.Basics/Fade/Fade.ino
      
        Using board 'uno' from platform in folder: /home/marcidy/.arduino15/packages/arduino/hardware/avr/1.8.2
        Using core 'arduino' from platform in folder: /home/marcidy/.arduino15/packages/arduino/hardware/avr/1.8.2
    

These are two distinct steps goverend by the `arduino-build` command: first gather information about the project using `-dump-perfs` and then use that information to compile the code using `-compile`.  Let's explore both in detail.

Dump Perfs
==========
This step gathers information stored in settings from the Arduino IDE and from the code being compiled.

Expanding the first line so it's more readable:
    
::

      /home/marcidy/arduino-1.8.10/arduino-builder
        -dump-prefs
        -logger=machine
        -hardware /home/marcidy/arduino-1.8.10/hardware
        -hardware /home/marcidy/.arduino15/packages
        -hardware /home/marcidy/Arduino/hardware
        -tools /home/marcidy/arduino-1.8.10/tools-builder
        -tools /home/marcidy/arduino-1.8.10/hardware/tools/avr
        -tools /home/marcidy/.arduino15/packages
        -built-in-libraries /home/marcidy/arduino-1.8.10/libraries
        -libraries /home/marcidy/Arduino/libraries
        -fqbn=arduino:avr:uno
        -ide-version=10810
        -build-path /tmp/arduino_build_709419
        -warnings=none
        -build-cache /tmp/arduino_cache_430568
        -prefs=build.warn_data_percentage=75
        -prefs=runtime.tools.arduinoOTA.path=/home/marcidy/.arduino15/packages/arduino/tools/arduinoOTA/1.3.0
        -prefs=runtime.tools.arduinoOTA-1.3.0.path=/home/marcidy/.arduino15/packages/arduino/tools/arduinoOTA/1.3.0
        -prefs=runtime.tools.avr-gcc.path=/home/marcidy/.arduino15/packages/arduino/tools/avr-gcc/7.3.0-atmel3.6.1-arduino5
        -prefs=runtime.tools.avr-gcc-7.3.0-atmel3.6.1-arduino5.path=/home/marcidy/.arduino15/packages/arduino/tools/avr-gcc/7.3.0-atmel3.6.1-arduino5
        -prefs=runtime.tools.avrdude.path=/home/marcidy/.arduino15/packages/arduino/tools/avrdude/6.3.0-arduino17
        -prefs=runtime.tools.avrdude-6.3.0-arduino17.path=/home/marcidy/.arduino15/packages/arduino/tools/avrdude/6.3.0-arduino17
        -verbose /home/marcidy/arduino-1.8.10/examples/01.Basics/Fade/Fade.ino
    
Again since I'm using linux, the `/home/marcidy/arduino-1.8.10/` is the path to the programs that come with the Arduino IDE.  They'll be different for everyone but the idea here is to learn to read this output and deconstruct it into its parts, and discover from those parts what is happening. 
    
The command has a number of repeated options like `-hardware` and `-tools` which are telling the program where to look for files important to compiliation.  Let's go over the options.


Generic options
---------------
    `-dump-perfs`
        The command, what is to be done, i.e. gather information about the project and preferences from the Arduino IDE.
    `-logger`
        Probably configures logging for executing on the target hardware, but not sure
    `-ide-version`
        The version of the Arduino IDE used to commpile the program.  Mine is 1.8.10 which translates to 10810.
    `-build-path`
        directory on the machine to look for / save files related to compilation
    `-build-cache`
        When compiling, look here to see if we can reuse any compiliation results.  Save compilation results here when required.
    `-warnings`
        Compiliers emit warnings that aren't always necessary for successful compilation.  This probably translates down to a flag during the actual compilation step to turn off warnings.
    `-verbose`
        Print information to the screen relevent to this step

    Note that the file I'm compiling is `/home/marcidy/arduino-1.8.10/examples/01.Basics/Fade/Fade.ino`, which is the final input into the command.

Library options
---------------
    Libraries are code required by the project that arent' written directly as part of the project.  For example, a servo library, or the libraries that make up the Arduino ecosystem.

    `-built-in-libraries`
        This is the path to the libraries which come with the Arduino IDE.  Only the libraries required for the project will be compiled in, but this path tells the compiler where to find the libraries.

    `-libraries`
        Again tells the compiler where to find libraries required for compilation.  This would be the libraries downloaded from the Arduino IDE via the library manager or possibly placed manually.  

    `-tools`
        paths to programs which help in pre-compiliation steps, also knows as 'preprocessing' or 'precompiling'.  

Hardware options
----------------
    `-hardware`
        tell the program where to find files with hardware.  It's repeated a number of times to tell the
        program to look in various locations, meaning the hardware definitions can be in any of the listed directories.
            
    `-fqbn`
        "fully qualified board name", which is the board for which we are compiling.  

Toolchain options
-----------------
    `-prefs=runtime.tools.arduinoOTA.path`
        Path to Over The Air updater
    `-prefs=runtime.tools.arduinoOTA-1.3.0.path`
        Path to version specific OTA updater
    `-prefs=runtime.tools.avr-gcc.path`
        avr-gcc is the AVR Gnu C Compiler.  AVR is the type of processor used in arduino UNOs (and others, but not all arduino compatible processors are AVRs).  This option tells the program where to find it.
    `-prefs=runtime.tools.avr-gcc-7.3.0-atmel3.6.1-arduino5.path`
        path to a specific avr-gcc version
    `-prefs=runtime.tools.avrdude.path`
        Path to the AVRDownloadersUploader (avr-dude).  We'll go into this more in depth in other sections.  It's the tool used to upload a compiled program to an avr processor.
    `-prefs=runtime.tools.avrdude-6.3.0-arduino17.path`
        Version specific path to avr-dude.

Dump perfs summary
------------------
From this command, we see that the tool has determined what hardware we are using and what libraries and tools are required for the project.  We also see where intermediate steps are saved on the machine, specifically in the `-build-path` and `-build-cache` locations.  We'll be investigating those in a separate section.  Pay close attention to the values used when using the IDE, they change, even for the same project, and they get deleted with the program exits.

If you run this command on it's own, you'll see a wealth of information.

.. literalinclude:: _static/arduino_build_dump_perfs.txt
    :language: none

Yikes, that's a lot.  I think my bank code is in there.  Well, you can peruse that if you want.  It's listing some programs it will use, options for those programs, and locations in the filesystem.

This command, on it's own, doesn't modify anything.

Compile
=======
As you can guess from the name, compiling means 'putting it together'.  Specifically gathering all the necessary information and coverting the human readable source code into a machine readable binary.

:: 

    /home/marcidy/arduino-1.8.10/arduino-builder
        -compile
        -logger=machine
        -hardware /home/marcidy/arduino-1.8.10/hardware
        -hardware /home/marcidy/.arduino15/packages
        -hardware /home/marcidy/Arduino/hardware
        -tools /home/marcidy/arduino-1.8.10/tools-builder
        -tools /home/marcidy/arduino-1.8.10/hardware/tools/avr
        -tools /home/marcidy/.arduino15/packages
        -built-in-libraries /home/marcidy/arduino-1.8.10/libraries
        -libraries /home/marcidy/Arduino/libraries
        -fqbn=arduino:avr:uno
        -ide-version=10810
        -build-path /tmp/arduino_build_709419
        -warnings=none
        -build-cache /tmp/arduino_cache_430568
        -prefs=build.warn_data_percentage=75
        -prefs=runtime.tools.arduinoOTA.path=/home/marcidy/.arduino15/packages/arduino/tools/arduinoOTA/1.3.0
        -prefs=runtime.tools.arduinoOTA-1.3.0.path=/home/marcidy/.arduino15/packages/arduino/tools/arduinoOTA/1.3.0
        -prefs=runtime.tools.avr-gcc.path=/home/marcidy/.arduino15/packages/arduino/tools/avr-gcc/7.3.0-atmel3.6.1-arduino5
        -prefs=runtime.tools.avr-gcc-7.3.0-atmel3.6.1-arduino5.path=/home/marcidy/.arduino15/packages/arduino/tools/avr-gcc/7.3.0-atmel3.6.1-arduino5
        -prefs=runtime.tools.avrdude.path=/home/marcidy/.arduino15/packages/arduino/tools/avrdude/6.3.0-arduino17
        -prefs=runtime.tools.avrdude-6.3.0-arduino17.path=/home/marcidy/.arduino15/packages/arduino/tools/avrdude/6.3.0-arduino17
        -verbose /home/marcidy/arduino-1.8.10/examples/01.Basics/Fade/Fade.ino
    
Well, these are identical to the `-dump-perfs` invokation, so no need to reiterate their defintions.  The only difference here is that the program is actually going to compile the program using this information.  Compiliation is covered in the Quickstart.

Running the command on it's own displays output related to compilation.

The lines like `===info |||` looks like it's information for the IDE itself.  Removing those, we see these commands run:

.. literalinclude:: _static/arduino_builder_compile_clean.txt
    :language: none


Lot's of output, but it falls into specific classes of commands which are further covered in :ref:`command_line_compilation`.

Let's look at the build directory `/tmp/arduino_build_709419`, and specifically order the files by the time they were creates.

::

    total 108
    -rw-r--r--  1 marcidy marcidy  1295 Jun 20 20:24 build.options.json
    -rw-r--r--  1 marcidy marcidy   421 Jun 20 20:24 includes.cache
    drwxr-xr-x  2 marcidy marcidy  4096 Jun 20 20:24 preproc
    drwxr-xr-x  2 marcidy marcidy  4096 Jun 20 20:24 sketch
    drwxr-xr-x  2 marcidy marcidy  4096 Jun 20 20:24 libraries
    drwxr-xr-x  2 marcidy marcidy  4096 Jun 20 20:24 core
    -rwxr-xr-x  1 marcidy marcidy 15952 Jun 20 20:24 Fade.ino.elf
    drwxrwxrwt 22 root    root    45056 Jun 20 20:24 ..
    -rw-r--r--  1 marcidy marcidy  4596 Jun 20 20:24 Fade.ino.with_bootloader.hex
    -rw-r--r--  1 marcidy marcidy  3250 Jun 20 20:24 Fade.ino.hex
    -rw-r--r--  1 marcidy marcidy    13 Jun 20 20:24 Fade.ino.eep
    drwxr-xr-x  6 marcidy marcidy  4096 Jun 20 20:24 .


Look at build.options.json and include.cache, but we're going to look in the "preproc" and "sketch" directories.

In the 'preproc' directory, there's just one file.

.. literalinclude:: _static/ctags_target_for_gcc_minus_e.cpp
    :language: none

This file looks a lot like the sketch, but with a change on the first line.  We'll, at least they give us a hint about what the file is for in the filename:  `ctags target for gcc -E`.

You can google ctags to see what it's all about.  I'm not going to cover it here.

In the 'sketch' directory, there are 3 files:
    
::

    Fade.ino.cpp
    Fade.ino.cpp.d
    Fade.ino.cpp.o

.. _arduino_builder_preproc:

Sketch Preprocessing
====================

`arduino-builder` converted the sketch into valid C++ by automatically making changes to the .ino file.  Take a look at Fade.ino.cpp.

.. literalinclude:: _static/Fade.ino.cpp
    :language: none

We see the additions of `#line` directives, along with *function prototypes* for `setup()` and `loop()`.  

`C++ directives <http://www.cplusplus.com/doc/tutorial/preprocessor/>`_ covers what the `#line` does as well as cover everything you see that starts with a `#`.

The added function function prototypes are required as a rule of writing C++:

::

    void setup();
    void loop();

This is covered in gerenal by learning C++.  I'll add here that C++ required you to declare the type of all variables *before* using the variable.  Here we are telling the compilier that "setup" and "loop" are functions which doesn't return anything (void).  This guide, or at least this part of it, isn't concerned with learning C/C++, but there's plenty of resources for that.  `arduino-builder` is doing a bit of extra work for us at this stage so that writing *sketches* is a bit easier than writing valid C++.  Not much easier, but slightly easier.


Now that we have valid C++ thanks to the arduino-builder pre-processing the .ino files, we can move to the topic of compilation.  This will cover the rest of the files created during this process.

Head over to :ref:`command_line_compilation` to keep moving forward.
