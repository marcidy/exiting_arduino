.. _start_from_ide:

=============================
Starting from the Arduino IDE
=============================
This is an overview with minimal explanation which provides the key details needed to develop, compile, and load firmware without using the Arduino IDE.  The goal is to point out what the IDE is actually *doing*, why each step is necessary, and how you can do it manaully or with scripts.  It will move in steps from using the IDE as is, to executing the same commands on the command line and explaining what the commands are doing and what needs to be changed.

For someone who has never compiled their own program, the more detailed explanation about compilation specifically may also be interesting.

I'll try to cross link the sections so a more detailed explanaiton is accessible.

IDE Compilation
===============
Load up the Arduino IDE, and load the "fade.ino" sketch.  Note that sketches (.ino files) are not exactly valid C or C++, but this will be explained later.

    .. image:: img/compile-1.png
        :height: 480px
        :width: 640px
        :alt: Arduino IDE Preferences

Not much info given at all!  We will change that now to see what commands the IDE is executing to compile the program.

IDE Configuration
-----------------
The IDE provides all the information we need, but we need to tick a couple boxes so it will output the relevant information.

    .. image:: img/arduino-preferences-2.png
        :width: 640px
        :alt: Open Arduino Preferences

Open Arduino preferences.

    .. image:: img/ide-preferences.png
        :width: 640px
        :alt: Arduino IDE Preferences

Tick these boxes to enable more output.

Detailed IDE Compilation
------------------------
Hit compile again.

.. image:: img/compile-2.png
    :width: 640px
    :alt: Arduino IDE Preferences

Lot's more output!  This is a combination of commands tht were run and output from those commands.  Let's look at my output and take a look.

.. code:: bash

  /home/marcidy/arduino-1.8.10/arduino-builder -dump-prefs -logger=machine -hardware /home/marcidy/arduino-1.8.10/hardware -hardware /home/marcidy/.arduino15/packages -hardware /home/marcidy/Arduino/hardware -tools /home/marcidy/arduino-1.8.10/tools-builder -tools /home/marcidy/arduino-1.8.10/hardware/tools/avr -tools /home/marcidy/.arduino15/packages -built-in-libraries /home/marcidy/arduino-1.8.10/libraries -libraries /home/marcidy/Arduino/libraries -fqbn=arduino:avr:uno -ide-version=10810 -build-path /tmp/arduino_build_709419 -warnings=none -build-cache /tmp/arduino_cache_430568 -prefs=build.warn_data_percentage=75 -prefs=runtime.tools.arduinoOTA.path=/home/marcidy/.arduino15/packages/arduino/tools/arduinoOTA/1.3.0 -prefs=runtime.tools.arduinoOTA-1.3.0.path=/home/marcidy/.arduino15/packages/arduino/tools/arduinoOTA/1.3.0 -prefs=runtime.tools.avr-gcc.path=/home/marcidy/.arduino15/packages/arduino/tools/avr-gcc/7.3.0-atmel3.6.1-arduino5 -prefs=runtime.tools.avr-gcc-7.3.0-atmel3.6.1-arduino5.path=/home/marcidy/.arduino15/packages/arduino/tools/avr-gcc/7.3.0-atmel3.6.1-arduino5 -prefs=runtime.tools.avrdude.path=/home/marcidy/.arduino15/packages/arduino/tools/avrdude/6.3.0-arduino17 -prefs=runtime.tools.avrdude-6.3.0-arduino17.path=/home/marcidy/.arduino15/packages/arduino/tools/avrdude/6.3.0-arduino17 -verbose /home/marcidy/arduino-1.8.10/examples/01.Basics/Fade/Fade.ino
  /home/marcidy/arduino-1.8.10/arduino-builder -compile -logger=machine -hardware /home/marcidy/arduino-1.8.10/hardware -hardware /home/marcidy/.arduino15/packages -hardware /home/marcidy/Arduino/hardware -tools /home/marcidy/arduino-1.8.10/tools-builder -tools /home/marcidy/arduino-1.8.10/hardware/tools/avr -tools /home/marcidy/.arduino15/packages -built-in-libraries /home/marcidy/arduino-1.8.10/libraries -libraries /home/marcidy/Arduino/libraries -fqbn=arduino:avr:uno -ide-version=10810 -build-path /tmp/arduino_build_709419 -warnings=none -build-cache /tmp/arduino_cache_430568 -prefs=build.warn_data_percentage=75 -prefs=runtime.tools.arduinoOTA.path=/home/marcidy/.arduino15/packages/arduino/tools/arduinoOTA/1.3.0 -prefs=runtime.tools.arduinoOTA-1.3.0.path=/home/marcidy/.arduino15/packages/arduino/tools/arduinoOTA/1.3.0 -prefs=runtime.tools.avr-gcc.path=/home/marcidy/.arduino15/packages/arduino/tools/avr-gcc/7.3.0-atmel3.6.1-arduino5 -prefs=runtime.tools.avr-gcc-7.3.0-atmel3.6.1-arduino5.path=/home/marcidy/.arduino15/packages/arduino/tools/avr-gcc/7.3.0-atmel3.6.1-arduino5 -prefs=runtime.tools.avrdude.path=/home/marcidy/.arduino15/packages/arduino/tools/avrdude/6.3.0-arduino17 -prefs=runtime.tools.avrdude-6.3.0-arduino17.path=/home/marcidy/.arduino15/packages/arduino/tools/avrdude/6.3.0-arduino17 -verbose /home/marcidy/arduino-1.8.10/examples/01.Basics/Fade/Fade.ino
  Using board 'uno' from platform in folder: /home/marcidy/.arduino15/packages/arduino/hardware/avr/1.8.2
  Using core 'arduino' from platform in folder: /home/marcidy/.arduino15/packages/arduino/hardware/avr/1.8.2
  Detecting libraries used...
  /home/marcidy/.arduino15/packages/arduino/tools/avr-gcc/7.3.0-atmel3.6.1-arduino5/bin/avr-g++ -c -g -Os -w -std=gnu++11 -fpermissive -fno-exceptions -ffunction-sections -fdata-sections -fno-threadsafe-statics -Wno-error=narrowing -flto -w -x C++ -E -CC -mmcu=atmega328p -DF_CPU=16000000L -DARDUINO=10810 -DARDUINO_AVR_UNO -DARDUINO_ARCH_AVR -I/home/marcidy/.arduino15/packages/arduino/hardware/avr/1.8.2/cores/arduino -I/home/marcidy/.arduino15/packages/arduino/hardware/avr/1.8.2/variants/standard /tmp/arduino_build_709419/sketch/Fade.ino.cpp -o /dev/null
  Generating function prototypes...
  /home/marcidy/.arduino15/packages/arduino/tools/avr-gcc/7.3.0-atmel3.6.1-arduino5/bin/avr-g++ -c -g -Os -w -std=gnu++11 -fpermissive -fno-exceptions -ffunction-sections -fdata-sections -fno-threadsafe-statics -Wno-error=narrowing -flto -w -x C++ -E -CC -mmcu=atmega328p -DF_CPU=16000000L -DARDUINO=10810 -DARDUINO_AVR_UNO -DARDUINO_ARCH_AVR -I/home/marcidy/.arduino15/packages/arduino/hardware/avr/1.8.2/cores/arduino -I/home/marcidy/.arduino15/packages/arduino/hardware/avr/1.8.2/variants/standard /tmp/arduino_build_709419/sketch/Fade.ino.cpp -o /tmp/arduino_build_709419/preproc/ctags_target_for_gcc_minus_e.cpp
  /home/marcidy/arduino-1.8.10/tools-builder/ctags/5.8-arduino11/ctags -u --language-force=C++ -f - --C++-kinds=svpf --fields=KSTtzns --line-directives /tmp/arduino_build_709419/preproc/ctags_target_for_gcc_minus_e.cpp
  Compiling sketch...
  /home/marcidy/.arduino15/packages/arduino/tools/avr-gcc/7.3.0-atmel3.6.1-arduino5/bin/avr-g++ -c -g -Os -w -std=gnu++11 -fpermissive -fno-exceptions -ffunction-sections -fdata-sections -fno-threadsafe-statics -Wno-error=narrowing -MMD -flto -mmcu=atmega328p -DF_CPU=16000000L -DARDUINO=10810 -DARDUINO_AVR_UNO -DARDUINO_ARCH_AVR -I/home/marcidy/.arduino15/packages/arduino/hardware/avr/1.8.2/cores/arduino -I/home/marcidy/.arduino15/packages/arduino/hardware/avr/1.8.2/variants/standard /tmp/arduino_build_709419/sketch/Fade.ino.cpp -o /tmp/arduino_build_709419/sketch/Fade.ino.cpp.o
  Compiling libraries...
  Compiling core...
  Using precompiled core: /tmp/arduino_cache_430568/core/core_arduino_avr_uno_2bd201547ef1722ab59b0c23270fe17e.a
  Linking everything together...
  /home/marcidy/.arduino15/packages/arduino/tools/avr-gcc/7.3.0-atmel3.6.1-arduino5/bin/avr-gcc -w -Os -g -flto -fuse-linker-plugin -Wl,--gc-sections -mmcu=atmega328p -o /tmp/arduino_build_709419/Fade.ino.elf /tmp/arduino_build_709419/sketch/Fade.ino.cpp.o /tmp/arduino_build_709419/../arduino_cache_430568/core/core_arduino_avr_uno_2bd201547ef1722ab59b0c23270fe17e.a -L/tmp/arduino_build_709419 -lm
  /home/marcidy/.arduino15/packages/arduino/tools/avr-gcc/7.3.0-atmel3.6.1-arduino5/bin/avr-objcopy -O ihex -j .eeprom --set-section-flags=.eeprom=alloc,load --no-change-warnings --change-section-lma .eeprom=0 /tmp/arduino_build_709419/Fade.ino.elf /tmp/arduino_build_709419/Fade.ino.eep
  /home/marcidy/.arduino15/packages/arduino/tools/avr-gcc/7.3.0-atmel3.6.1-arduino5/bin/avr-objcopy -O ihex -R .eeprom /tmp/arduino_build_709419/Fade.ino.elf /tmp/arduino_build_709419/Fade.ino.hex
  /home/marcidy/.arduino15/packages/arduino/tools/avr-gcc/7.3.0-atmel3.6.1-arduino5/bin/avr-size -A /tmp/arduino_build_709419/Fade.ino.elf
  Sketch uses 1144 bytes (3%) of program storage space. Maximum is 32256 bytes.
  Global variables use 13 bytes (0%) of dynamic memory, leaving 2035 bytes for local variables. Maximum is 2048 bytes.


.. code:: bash

  /home/marcidy/arduino-1.8.10/arduino-builder -dump-prefs -logger=machine -hardware /home/marcidy/arduino-1.8.10/hardware -hardware /home/marcidy/.arduino15/packages -hardware /home/marcidy/Arduino/hardware -tools /home/marcidy/arduino-1.8.10/tools-builder -tools /home/marcidy/arduino-1.8.10/hardware/tools/avr -tools /home/marcidy/.arduino15/packages -built-in-libraries /home/marcidy/arduino-1.8.10/libraries -libraries /home/marcidy/Arduino/libraries -fqbn=arduino:avr:uno -ide-version=10810 -build-path /tmp/arduino_build_709419 -warnings=none -build-cache /tmp/arduino_cache_430568 -prefs=build.warn_data_percentage=75 -prefs=runtime.tools.arduinoOTA.path=/home/marcidy/.arduino15/packages/arduino/tools/arduinoOTA/1.3.0 -prefs=runtime.tools.arduinoOTA-1.3.0.path=/home/marcidy/.arduino15/packages/arduino/tools/arduinoOTA/1.3.0 -prefs=runtime.tools.avr-gcc.path=/home/marcidy/.arduino15/packages/arduino/tools/avr-gcc/7.3.0-atmel3.6.1-arduino5 -prefs=runtime.tools.avr-gcc-7.3.0-atmel3.6.1-arduino5.path=/home/marcidy/.arduino15/packages/arduino/tools/avr-gcc/7.3.0-atmel3.6.1-arduino5 -prefs=runtime.tools.avrdude.path=/home/marcidy/.arduino15/packages/arduino/tools/avrdude/6.3.0-arduino17 -prefs=runtime.tools.avrdude-6.3.0-arduino17.path=/home/marcidy/.arduino15/packages/arduino/tools/avrdude/6.3.0-arduino17 -verbose /home/marcidy/arduino-1.8.10/examples/01.Basics/Fade/Fade.ino
  /home/marcidy/arduino-1.8.10/arduino-builder -compile -logger=machine -hardware /home/marcidy/arduino-1.8.10/hardware -hardware /home/marcidy/.arduino15/packages -hardware /home/marcidy/Arduino/hardware -tools /home/marcidy/arduino-1.8.10/tools-builder -tools /home/marcidy/arduino-1.8.10/hardware/tools/avr -tools /home/marcidy/.arduino15/packages -built-in-libraries /home/marcidy/arduino-1.8.10/libraries -libraries /home/marcidy/Arduino/libraries -fqbn=arduino:avr:uno -ide-version=10810 -build-path /tmp/arduino_build_709419 -warnings=none -build-cache /tmp/arduino_cache_430568 -prefs=build.warn_data_percentage=75 -prefs=runtime.tools.arduinoOTA.path=/home/marcidy/.arduino15/packages/arduino/tools/arduinoOTA/1.3.0 -prefs=runtime.tools.arduinoOTA-1.3.0.path=/home/marcidy/.arduino15/packages/arduino/tools/arduinoOTA/1.3.0 -prefs=runtime.tools.avr-gcc.path=/home/marcidy/.arduino15/packages/arduino/tools/avr-gcc/7.3.0-atmel3.6.1-arduino5 -prefs=runtime.tools.avr-gcc-7.3.0-atmel3.6.1-arduino5.path=/home/marcidy/.arduino15/packages/arduino/tools/avr-gcc/7.3.0-atmel3.6.1-arduino5 -prefs=runtime.tools.avrdude.path=/home/marcidy/.arduino15/packages/arduino/tools/avrdude/6.3.0-arduino17 -prefs=runtime.tools.avrdude-6.3.0-arduino17.path=/home/marcidy/.arduino15/packages/arduino/tools/avrdude/6.3.0-arduino17 -verbose /home/marcidy/arduino-1.8.10/examples/01.Basics/Fade/Fade.ino
  Using board 'uno' from platform in folder: /home/marcidy/.arduino15/packages/arduino/hardware/avr/1.8.2
  Using core 'arduino' from platform in folder: /home/marcidy/.arduino15/packages/arduino/hardware/avr/1.8.2

I'm on a Linux machine so `/home/marcidy/arduino-1.8.10` are the path to the arduino builder tool.  This tool is a program that comes with the arduino IDE to collect information about the code you want to compile.  It uses information from things you've set in the IDE (like the board) and libraries installed through the IDE.  

Check out :ref:`arduino_builder` to learn more about that part of the too.

Now, we're on the command line.  Great job.  Head over to :ref:`command_line_compilation`.

