---
published: true
title: Linux Commandline Tips
tags: Tips git
---

When using the Linux command line and related tools, there are many useful tricks and common practices worth noting. In this post, I’m keeping a running list of tips related to Linux command-line tools for future reference.

***Table of Contents***
* TOC
{:toc}

# Cd
* ```cd -``` moves back to the previous directory.
* ```pushd <path>``` moves to the path and save the current directory.
    * ```popd``` to return to the saved directory.

# Command Line
* ```ctrl-A``` moves cursor to the beginning of line.
* ```ctrl-E``` moves cursor to the end of the line.
* ```ctrl-U``` cuts everything from cursor to beginning of the line.
* ```ctrl-K``` cuts everything from cursor to end of the line.
* ```ctrl-Y``` yanks the last cut text.
* ```!!``` runs the previous command.

# Echo
* ```echo $?``` displays the return value of the most recently run program.

# Hexdump
* ```hexdump -C <filename>``` dumps a binary file in canonical hex+ASCII format.

# Git
## Branch
* In case of ambiguity between file names and branch names (e.g. when both a file and a branch are named as ```object```).
    * ```git checkout object --``` to checkout branch.
    * ```git checkout -- object``` to checkout file.

# Gdb
* See https://sourceware.org/gdb/current/onlinedocs/gdb.html/ for a super detailed doc.
* ```-x <filename>``` takes the gdb script and runs it in gdb.
* ```l``` lists 10 lines of source code around the current program counter.
    * ```l <n>``` lists 10 lines of source code around line n in the current source file.
    * ```l main``` lists 10 lines of source code around the start of function ```main```.
    * ```l <filename>:<n>```: lists 10 lines of source code around line n in the specified source file.
* ```tui``` can be triggered by ```layout <format>```.
    * ```layout src``` opens a tui window with source code.
    * ```ctrl-X A``` toggles tui on/off.
    * ```ctrl-X O``` switches between tui windows.
    * ```layout asm``` opens a tui window with assembly.
    * ```layout regs``` opens a tui window with CPU registers.


# Grep
* ```-n``` displays line numbers.
* ```-c``` displays **just** the total number of matching lines.
* ```-E``` for extended-regex pattern. For example, ```grep -E 'dsb|dmb|isb'```. 
* ```-v``` excludes matched lines.
* ```--exclude=".."``` excludes certain filenames.   

# Objdump
* ```-z``` shows all symbols (to include those with zero values).
* To perform objdump on arm binary, 
    * For bare-metal, install ```binutils-arm-none-eabi```, and use ```arm-none-eabi-objdump```.
    * For arm64, install ```binutils-aarch64-linux-gnu```, and use ```aarch64-linux-gnu-objdump```.

# Ps 
* ```ps auxwf``` 
    * ```a``` shows processes for all users.
    * ```u``` shows the user/owner of each process.
    * ```x``` shows processes not attached to a terminal.
    * ```w``` shows wide output (to include the entire command line).
    * ```f``` shows processes in a tree format.

# Set
* When writing a shell script, use ```set -e``` at the beginning to specify that the script should exit immediately if any command exits with a non-zero status.

# Tee
* ```<command> | tee <filename>``` displays the output while saving it into the file. 
* ```<command> | tee -a <filename>``` displays the output while appending it at the end of the file. 
* ```<command> 2>&1 | tee <filename>``` displays the stderr and stdout while saving them in the file.
    * Shorthand: ```<command> |& tee <filename>```