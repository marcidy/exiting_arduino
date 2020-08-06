.. _build_automation:

================
Build Automation
================
Build automation is about getting to compiled code with as little manual steps as possible.  "Building" refers to building the binary, so the idea is to invoke the compilation process with one command and a binary is produced.

Arduino comes with a build system, invoked with the :ref:`arduino_builder`.  The build system is configurable for a large number of different hardware, so it knows how to interpret options you tell it to invoke the different tool-chains it has access to with the right options required to compile the code you've written to a binary for the target architecture.

Let's take an approach to build one up from scratch to understand what decisions it's making, how it's making them, and how we can have a build system make decision based on information.

Build Scripts
=============
At your computer, you can invoke commands a number of ways.  These days most people poke at the screen with a mouse or a finger, interacting with the GUI (graphical user interface).  Programmers should be using the CLI (command line interface).  This is done via `terminals` (programs which are launched which represent the "termination" of information), `consoles` (Where a computer can give you grief), and `shells` (programs which provide a "shell" around the guts of the commputer).

These are different things, but generally when people want to access the command line, they will use one of these terms.

I'm going to use the term `shell` since it's ubiquitous.  You can read about the differences `Here <https://superuser.com/questions/144666/what-is-the-difference-between-shell-console-and-terminal>`_.

Scripting is writing a bunch of commands into a file that you would otherwise type at the command line and executing them in the order they are written.  Shells provide the commands that you can use with scripting to manage control flow and provide access to the running system.   Shell programming, just like other programming, has certain syntax rules and allowable commands.


