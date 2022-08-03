+++
title = "Recommended reading"
description = "Some books and resources that I like."
date = 2022-08-03T02:13:50Z
author = "Torbjörn Molin"
+++

## About the list

This is a compilation of some of the things I've read and some videos that I like and also some things that are on my list that I haven't gotten to yet. I think that some of it may be useful for people like me who are interested in and perhaps work with computers, programming and computer science but are self taught.

### Programming from the ground up by Jonathan Bartlett

This free (available [here](https://download-mirror.savannah.gnu.org/releases/pgubook/ProgrammingGroundUp-1-0-booksize.pdf) under the GNU Free documentation license) really is what the title says. It introduces the reader to programming using 32 bit x86 assembly language on Linux. Being 32 bit, it is a bit out of date but I remember seeing the code ported to 64 bit ASM somewhere on the internet.

I remember getting completely hooked and feeling that so many things fell in place. I had a "mind blown" kind of moment when reading the last bullet item under "going further" in the chapter that takes the reader through writing a primitive memory allocator.

> Change the name of the functions to malloc and free, and build them into a shared library. Use LD_PRELOAD to force them to be used as your memory manager instead of the default one. Add some write system calls to STDOUT to verify that your memory manager is being used instead of the default one.

Being shown that memory allocation is not magic and then the suggestion to test using the primitive allocator as the system default (malloc and free are the C standard library functions for allocating and freeing memory) was eye opening to me.

I did not, however, read it as an introduction to programming and I hade written a decent amount of code in a few different languages before reading it. I think that for most people the approach of getting to something useful fast and not worrying about too much detail in the beginning is a better approach to learning to program.

### Programming: Principles and Practice Using C++ by Bjarne Stroustrup

I have written some, but not all that much C++ and the part I remember from this book is a chapter that is about writing a simple parser for mathematical expressions. I remember it being well written and showing a good iterative approach to the problem. Even though it was a very simple implementation it did give me some insight into an area that previously seemed like magic (writing a simple interpreter).

### Philipp Oppermann's blog - Writing an OS in Rust.

A series of [blog posts](https://os.phil-opp.com) that are a good mix of operating systems for beginners, x86 architecture and certain aspects of the Rust language. Another one of those resources that have given me a better understanding of how things work on a lower abstraction level.

### Debugging under fire

This is a [video recording](https://youtu.be/30jNsCVLpAE) of a talk given by Bryan Cantrill at the GOTO conference.
Cantrill is often entertaining and here he goes through some thoughts on catastrophic failures and debugging production systems.
Other good stuff also featuring Cantrill are [Oral Tradition in Software Engineering](https://youtu.be/4PaWFYm0kEw) (the Hemingway parody competition entry at 21:39 in particular) and [On the metal podcast](https://oxide.computer/podcasts).

### Crafting interpreters by Robert Nystrom

This is one of the books on my list to read, with the added benefit of being a possible implementation project to aid in learning a new programming language. It's available [here](https://craftinginterpreters.com).

### Raytracer in one weekend by Peter Shirley

Another one on the to-read-list. Also a nice implementation project for a new language. Available [here](https://raytracing.github.io)
