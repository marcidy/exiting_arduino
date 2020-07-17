.. _uploading_firmware:

==================
Uploading Firmware
==================
Once we've generated the `.eep` and `.hex` files from :ref:`command_line_compilation`, we need to upload them to the chip.

To do this, we need a programmer.   In the case of Arduino, the USB port is used.

The topic is more complex than this section let's on.  In reality, it's another electronic circuit on the board which allows us to program over the serial port.

First, let's look at how to send the data from our computer user `avr-dude`.  Then we will briefly discuss what's happening when the chip receives the string of 1s and 0s over the USB cable.

avr-dude
========

`avr-dude` is the tool used to upload and download data from an AVR microprocessor.  Note that it's bi-directinoal.  It's possible to extract firmware from a chip as well, which we'll briefly look at towards the end.

This is how the IDE invokes the command:

::

    /home/marcidy/.arduino15/packages/arduino/tools/avrdude/6.3.0-arduino17/bin/avrdude 
        -C/home/marcidy/.arduino15/packages/arduino/tools/avrdude/6.3.0-arduino17/etc/avrdude.conf 
        -v 
        -patmega328p 
        -carduino 
        -P/dev/ttyACM0 
        -b115200 
        -D 
        -Uflash:w:/tmp/arduino_build_608217/Fade.ino.hex:i 

Just one command this time, let's take a look at the arguements.

::

    avrdude
    Usage: avrdude [options]
    Options:
      -p <partno>                Required. Specify AVR device.
      -b <baudrate>              Override RS-232 baud rate.
      -B <bitclock>              Specify JTAG/STK500v2 bit clock period (us).
      -C <config-file>           Specify location of configuration file.
      -c <programmer>            Specify programmer type.
      -D                         Disable auto erase for flash memory
      -i <delay>                 ISP Clock Delay [in microseconds]
      -P <port>                  Specify connection port.
      -F                         Override invalid signature check.
      -e                         Perform a chip erase.
      -O                         Perform RC oscillator calibration (see AVR053). 
      -U <memtype>:r|w|v:<filename>[:format]
                                 Memory operation specification.
                                 Multiple -U options are allowed, each request
                                 is performed in the order specified.
      -n                         Do not write anything to the device.
      -V                         Do not verify.
      -u                         Disable safemode, default when running from a script.
      -s                         Silent safemode operation, will not ask you if
                                 fuses should be changed back.
      -t                         Enter terminal mode.
      -E <exitspec>[,<exitspec>] List programmer exit specifications.
      -x <extended_param>        Pass <extended_param> to programmer.
      -y                         Count # erase cycles in EEPROM.
      -Y <number>                Initialize erase cycle # in EEPROM.
      -v                         Verbose output. -v -v for more.
      -q                         Quell progress output. -q -q for less.
      -l logfile                 Use logfile rather than stderr for diagnostics.
      -?                         Display this usage.
    
    avrdude version 6.3-20171130, URL: <http://savannah.nongnu.org/projects/avrdude/>

`-C`
    The configuration file for avrdude.  Take a look at this file, it explains details about how different programmers work, as well as the expectations of various `parts`.  The top of this file is commented and explains how to interpret the rest of the file.
`-v`
    Verbose output.  This flag is set when selecting `verbose` in the IDE preferences, and useful to inspect what's going on.
`-p`
    Part, or what part is being programmed.  If you look in the configuration file, you'll see information related to various parts, how to read and write data from the parts, how to set fuses, etc. 
`-c`
    Which programmer is being used.  Boards can be programmed by different types of devices.  The `arduino` programmer is really a USB connection to a USB-TTL converter, which uses the STK500v2 protocol, as noted on the config file.
`-P`
    The port for programming, which for me on linux is /dev/ttyUSB0.  Will be different based on your operating system and available ports.
`-b`
    programming baud rate.  How fast data is sent across the connection, and must be matched between the sender and receiver.
`-D` 
    Don't auto-erase flash memory.
`-U`
    This tells the programmer to take data from one place and put it in another.  For example, `flash:w:/tmp/arduino_build_608217/Fade.ino.hex:i` means write the contents of Fade.ino.hex into flash memory.  The hex file is in 'i' format.  'i' here means 'Intel hex' which is a specific format, and matches the 'ihex' option used in :ref:`avr-objcopy`.


`Intel Hex <https://en.wikipedia.org/wiki/Intel_HEX`>_ is a file format as well.  The 1s and 0s of the binary is stored on your computer's harddisk in a specific format, just like all other data on your computer.

This format contains extra information, like checksums and byte counts, which are used to veryify the data, and provide extra information to programs which read it, and so provides ways to determine if the file has been corrupted on your harddrive.


Check out `AVRDUDE's documentation <https://www.nongnu.org/avrdude/user-manual/avrdude.html>`_  to get a full explanation of all the options.


Firmware Extraction
===================

We can use AVRDUDE to extract firmware from a chip the same way we load firmware, with slightly different options.

The `-U` parameter can be used to read instead of write.

::

    -Uflash:r:<file_name>:i

This will read the flask memory and write it out to <file_name> in Intel Hex format.

Once it's in that format, you can disassemble it, and try to reverse engineer it if you are so inclined.


Programmer
==========

A programmer translates data from your computer to a set of electrical signals which are send over a USB cable to the USB port on the Arduino.  Note this isn't the only method.  There are usually pins exposed on the board for the ISP, In-circuit Serial Programmer.  We won't cover this explicitly, but the concept is the same:  You have a file somewhere with the firmware, and you need to convert it to electrical signals.

This topic requires knowledge of the specific architecture of the hardware you are programming, as well as some basic electrical knowledge.  Due to that, it's covered in :ref:`electrically_programming`.


