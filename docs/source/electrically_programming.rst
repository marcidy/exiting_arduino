.. _electrically_programming:

=======================================
Electrically Programming an Arduino Uno
=======================================

Once we have data exiting the USB port of your computer heading to the Arduino Uno, it's a stream of electrical HIGH and LOW signals on the wire.  AVRDUDE along with your computer's hardware has done the work to create the data, sending HIGHs and LOWs based on a serial communication protocol.  While this translation is complicated due to your computer being a very complicated processor with an operating system, chipsets, drivers, etc, we can focus on the Ardiuno Uno side and understand the same idea.  After all, if I send you a message over a wire, I have to send it in a way you can understand it.  We have to agree ahead of time what a HIGH and LOW means, and how often I will be sending HIGHs and LOWs.


