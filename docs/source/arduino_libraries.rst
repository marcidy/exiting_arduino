.. _arduino_libraries:

=================
Arduino Libraries
=================
We'll go through some of the libraries included with Arduino here, starting with the famous `Arduino.h`, which is included in most sketches.


Arduino.h
---------

Where is it?  Well, you can check the output on-screen during compilation to get a clue from the '-I' options passed to avr-g++, covered more in :ref:`command_line_compilation`

Mine's here:  /home/marcidy/.arduino15/packages/arduino/hardware/avr/1.8.2/cores/arduino

::

    abi.cpp              HardwareSerial3.cpp       new.cpp           Stream.cpp   WCharacter.h      wiring_shift.c
    Arduino.h            HardwareSerial.cpp        new.h             Stream.h     WInterrupts.c     WMath.cpp
    binary.h             HardwareSerial.h          PluggableUSB.cpp  Tone.cpp     wiring_analog.c   WString.cpp
    CDC.cpp              HardwareSerial_private.h  PluggableUSB.h    Udp.h        wiring.c          WString.h
    Client.h             hooks.c                   Printable.h       USBAPI.h     wiring_digital.c
    HardwareSerial0.cpp  IPAddress.cpp             Print.cpp         USBCore.cpp  wiring_private.h
    HardwareSerial1.cpp  IPAddress.h               Print.h           USBCore.h    wiring_pulse.c
    HardwareSerial2.cpp  main.cpp                  Server.h          USBDesc.h    wiring_pulse.S


Look at that, there it is, the famous `Arduino.h`.  You should read that file [#]_.  We'll use part of this file to take a deep dive through Arduino libs.   And all those files, really.  INTERESTING.  Hey, wtf is a ".S" file?  Did you read :ref:`architecture_introduction`?  It's assembly code.  ALSO INTERESTING.  YOU WRITE ASSEMBLY DIRECTLY...*ahem* sorry.  You can write assembly and "compile" it with avr-g++, and use it with Arduino.  Neat!

Ok, take a deep breath.  We're going deep.  Architecture Deep.  If you don't know what I mean by Architecture, take a look at :ref:`architecture_introduction`.

We're going to head through a bunch of includes to where the specifics of architecture actually matter.  For example, differnt chips have different pins which can do different things.  They are variables when you type them in the IDE, but where are they defined?

Nothing up to this stage reflects architecture differences.  Which is great, we don't have to care, but we're curious, so let's do it.

Let's look at some details of Arduino.h

::

    #define HIGH 0x1
    #define LOW  0x0

    #define INPUT 0x0
    #define OUTPUT 0x1
    #define INPUT_PULLUP 0x2


Ah, some preprocesor directives.  So that's how Arduino defines those.  For those unaware the `0x` prefix is hexadecimal, which is another way to display numeric data.  It also is nice for expressing binary data.  You should read up on that for sure, it's critical for programming hardware to know hex.

So these includes are clearly mentioning avr:

::

    #include <avr/pgmspace.h>
    #include <avr/io.h>
    #include <avr/interrupt.h>

We'll tackle those later, let's keep digging through Arduino.h

::

    #if defined(__AVR_ATtiny24__) || defined(__AVR_ATtiny44__) || defined(__AVR_ATtiny84__)
      #define DEFAULT 0
      #define EXTERNAL 1
      #define INTERNAL1V1 2
      #define INTERNAL INTERNAL1V1
    #elif defined(__AVR_ATtiny25__) || defined(__AVR_ATtiny45__) || defined(__AVR_ATtiny85__)
      #define DEFAULT 0
      #define EXTERNAL 4
      #define INTERNAL1V1 8
      #define INTERNAL INTERNAL1V1
      #define INTERNAL2V56 9
      #define INTERNAL2V56_EXTCAP 13
    #else
    #if defined(__AVR_ATmega1280__) || defined(__AVR_ATmega2560__) || defined(__AVR_ATmega1284__) || defined(__AVR_ATmega1284P__) || defined(__AVR_ATmega644__) || defined(__AVR_ATmega644A__) || defined(__AVR_ATmega644P__) || defined(__AVR_ATmega644PA__)
    #define INTERNAL1V1 2
    #define INTERNAL2V56 3
    #else
    #define INTERNAL 3
    #endif
    #define DEFAULT 1
    #define EXTERNAL 0
    #endif

Ok, this what mine looks like.  Yours actually might be different.  Why?  Becuase I have the board files for ATTiny installed.  What's interesting here, is we actually haven't defined those anywhere yet. But we also don't see the architecture we're using, AVR ATmega328p.  IT was indicated in the "mmcu" flag, but it's not here.

::

    void pinMode(uint8_t pin, uint8_t mode);
    void digitalWrite(uint8_t pin, uint8_t val);
    int digitalRead(uint8_t pin);
    int analogRead(uint8_t pin);
    void analogReference(uint8_t mode);
    void analogWrite(uint8_t pin, int val);

Well, these look familiar.

Ok, ok, go to the end of the file.  What I want to show is here:

::

    #include "pins_arduino.h"

Ok, let's look at THAT file.  Why's it at the end?  Well, if you read through :ref:`compilation_primer` you know that the `#include` just causes the preprocessor to copy/past the contents of the file in place of the include.  so no biggie about where they actually are.  The requirement is that definitions occur before usages. not that `#include` is at the top of the file.

pins_arduino.h
--------------
Ok, well, i couldn't find pins_arduino.h in the same directory as Arduino.h.  We'll have to check the other directory from the '-I' options passed to `avr-g++`.

::

    -I/home/marcidy/.arduino15/packages/arduino/hardware/avr/1.8.2/variants/standard 

Well, there it is.  The system works!  Becuase it has to!  No magic here, `avr-g++` was told *explicitly* where to look for files.

::

    #define LED_BUILTIN 13

    #define PIN_A0   (14)
    #define PIN_A1   (15)
    #define PIN_A2   (16)
    
    static const uint8_t A0 = PIN_A0;
    static const uint8_t A1 = PIN_A1;
    static const uint8_t A2 = PIN_A2;



Just picking a couple here.  This is how you can write things like 'A0' as a pin name in sketch and Arduino will interpret it as a pin number.  It's just a variable defined here.  Also the LED_BUILDIN is defined.

Now where in a case where these pins are actual numbers.  We're getting closer to hardware.  Time to checkout those files in the `avr/` directory.

.. [#]  You should read ALL code you can find.  It makes you a better programmer just by exposure, you'll pick up subtle ways that other people structure their code.  It will also make you google things and learn how to learn programming.  It's the best thing you can do.  Are you using a library?  Read it's code.  You already know why it's doing something, reading the code will show you how it's done.  Ideally, you read it becuase you are curious.
