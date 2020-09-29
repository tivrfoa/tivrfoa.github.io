---
layout: post
title:  "My notes and transcriptions from: The Java Language Environment - A White Paper"
date:   2020-09-27 20:26:44 -0300
categories: java software jvm
---

[https://www.stroustrup.com/1995_Java_whitepaper.pdf](https://www.stroustrup.com/1995_Java_whitepaper.pdf)

The paper is from October, 1995


### The Better Way is Here Now

>Imagine, if you will, this development world…
>
>  - Your programming language is object oriented, yet it’s still dead simple.
>  - Your development cycle is much faster because Java is interpreted. The
compile-link-load-test-crash-debug cycle is obsolete—now you just compile
and run.
>  - Your applications are portable across multiple platforms. Write your
applications once, and you never need to port them—they will run without
modification on multiple operating systems and hardware architectures.
>  - Your applications are robust because the Java run-time system manages
memory for you.
>  - Your interactive graphical applications have high performance because
multiple concurrent threads of activity in your application are supported by
the multithreading built into Java environment.
>  - Your applications are adaptable to changing environments because you can
dynamically download code modules from anywhere on the network.
>  - Your end users can trust that your applications are secure, even though
they’re downloading code from all over the Internet; the Java run-time
system has built-in protection against viruses and tampering.

### Beginnings of the Java Language Project

> Design and architecture decisions drew from a variety of
languages such as Eiffel, SmallTalk, Objective C, and Cedar/Mesa. The result is
a language environment that has proven ideal for developing secure,
distributed, network-based end-user applications in environments ranging
from networked-embedded devices to the World-Wide Web and the desktop.


### Architecture Neutral and Portable

>The architecture-neutral and portable language environment of Java is known
as the Java Virtual Machine.


### Design Goals

>Simplicity

>Programmers familiar with C, Objective C, C++, Eiffel, Ada, and related languages should find their Java language learning curve quite short—on the order of a couple of weeks.

### Character Data Types

>Java language character data is a departure from traditional C. Java’s char data
type defines a sixteen-bit Unicode character. Unicode characters are unsigned
16-bit values that define character codes in the range 0 through 65,535.

### Unsigned (logical) right shift

>All the familiar C and C++ operators apply. Because Java lacks unsigned data
types, the >>> operator has been added to the language to indicate an
unsigned (logical) right shift.

**The sign bit becomes 0, so the result is always non-negative.**

```shell
jshell> -1 >>> 1
$1 ==> 2147483647

jshell> Integer.MAX_VALUE
$2 ==> 2147483647

jshell> 1 >>> 1
$3 ==> 0

jshell> Integer.toBinaryString(-1 >>> 1)
$4 ==> "1111111111111111111111111111111" // 31 bits

jshell> Integer.toBinaryString(-1)
$5 ==> "11111111111111111111111111111111" // 32 bits

jshell> Integer.toBinaryString(-2)
$9 ==> "11111111111111111111111111111110"

jshell> Integer.toBinaryString(Integer.MIN_VALUE)
$6 ==> "10000000000000000000000000000000" // 32 bits

jshell> Integer.MIN_VALUE
$10 ==> -2147483648

jshell> Integer.MAX_VALUE + 1
$11 ==> -2147483648

jshell> Integer.MAX_VALUE + 1 == Integer.MIN_VALUE
$12 ==> true

jshell> -5 >>> 2
$13 ==> 1073741822

jshell> Integer.toBinaryString(-5)
$14 ==> "11111111111111111111111111111011" // 32 bits

jshell> Integer.toBinaryString(-5 >>> 2)
$15 ==> "111111111111111111111111111110" // 30 bits

jshell> Integer.toBinaryString(-5 >>> 3)
$20 ==> "11111111111111111111111111111" // 29 bits
```

### Arrays

>In contrast to C and C++, Java language arrays are first-class language objects.
An array in Java is a real object with a run-time representation. You can declare
and allocate arrays of any type, and you can allocate arrays of arrays to obtain
multi-dimensional arrays.
>
>You declare an array of, say, Points (a class you’ve declared elsewhere) with a
declaration like this:
>```	Point myPoints[];```
>
>This code states that `myPoints` is an uninitialized array of `Points`. At this
time, the only storage allocated for myPoints is a reference handle. At some
future time you must allocate the amount of storage you need, as in:
>
>```	myPoints = new Point[10];```
>
>**The C notion of a pointer to an array of memory elements is gone, and with it,
the arbitrary pointer arithmetic that leads to unreliable code in C**. No longer
can you walk off the end of an array, possibly trashing memory and leading to
the famous “delayed-crash” syndrome, where a memory-access violation today
manifests itself hours or days later. Programmers can be confident that array
checking in Java will lead to more robust and reliable code.

### Multi-Level Break

> Use of labelled blocks in Java leads to considerable
simplification in programming effort and a major reduction in maintenance.

### Memory Management and Garbage Collection

> Explicit memory management has proved to
be a fruitful source of bugs, crashes, memory leaks, and poor performance.
Java completely removes the memory management load from the programmer.
C-style pointers, pointer arithmetic, malloc, and free do not exist. Automatic
garbage collection is an integral part of Java and its run-time system. **While Java
has a `new` operator to allocate memory for objects, there is no explicit `free`
function.** Once you have allocated an object, the run-time system keeps track of
the object’s status and automatically reclaims memory when objects are no
longer in use, freeing memory for future use.

> The Java memory manager keeps track of references to objects. When an object has
no more references, the object is a candidate for garbage collection.

### The Background Garbage Collector

>The Java garbage collector achieves high performance by taking advantage of
the nature of a user’s behavior when **interacting** with software applications
such as the HotJava browser. The typical user of the typical interactive
application has many natural pauses where they’re contemplating the scene in
front of them or thinking of what to do next. The Java run-time system takes
advantage of these idle periods and runs the garbage collector in a low priority
thread when no other threads are competing for CPU cycles.

This is an interesting part, because here he's targeting interactive applications.<br>
Ok, but this idle period is sometimes non-existent or very rare for many **non-interactive** applications, then I language with a garbage collector would not be the best approach.

### No More Typedefs, Defines, or Preprocessor

>Source code written in Java is *simple*. There is no preprocessor, no #define and
related capabilities, no typedef, and absent those features, no longer any need
for header files. Instead of header files, Java language source files provide the
definitions of other classes and their methods.
>
>A major problem with C and C++ is the amount of context you need to
understand another programmer’s code: you have to read all related header
files, all related #defines, and all related typedefs before you can even begin
to analyze a program. In essence, programming with #defines and typedefs
results in every programmer inventing a new programming language that’s
incomprehensible to anybody other than its creator, thus defeating the goals of
good programming practices.

<blockquote style="margin-left: 60px; border-left: none; font-size: x-large;">
	<p style="padding: 15px; border-radius: 5px; margin-left: 20px; background: #eee;">
	In essence, programming with #defines and typedefs results in every programmer <span style="font-weight: bold;">inventing a new programming language</span> that’s incomprehensible to anybody other than its creator, thus defeating the goals of good programming practices.
	</p>
</blockquote>

I really like James Gosling approach to simplicity, and Java being simple for sure is one of the reasons of its success.


### TODO CONTINUE ...