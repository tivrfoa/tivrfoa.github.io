---
layout: post
title:  "Why do programmers use hexadecimal numbers?"
date:   2020-09-30 10:05:44 -0300
categories: software hexadecimal binary
---

I watched a great video by Jacob Sorber with a very clear explanation about the use of hexadecimal numbers:

[Why do programmers use hexadecimal numbers?](https://www.youtube.com/watch?v=dPxCGlW9lfM)

>Why do we bother with yet another way to represent numbers? Why don't we just stick with binary?
>
>And the answer is is that binary is very long.
>
>And that is where hex comes in, because hex is basically condesend binary.
>
>I say that because it's really easy to convert from hexadecimal to binary, because 16 is a power of 2.
>
>Each digit in an hex number maps to exactly 4 bits.
>
>So for example I can take the number `0x16D` and I can decompose it like this: 0b`0001 0110 1101`

```
      1|6|D
	  /   \ 
 0001|0110|1101
```

>And doing that with decimal numbers isn't that easy.
>
>Another advantage is it's really easy to look at a hexadecimal number and tell how many bytes are in it. And also it's really reasy to tell where one byte ends and the other byte begins.

>When you should actually use hexadecimal numbers?
>
>You should use hex whenever the binary representation is important, for example RGB colors. It matters because each of those bytes represent a different component. **Anytime they're representing numbers where it's important to see where one byte ends and the other byte begins, use hex.**

>Learning hex well is going to save you a ton of time!

```java
jshell> for (int i = 0; i <= 0xf; ++i) {
   ...> System.out.printf("%2d\t%4s\n", i, Integer.toBinaryString(i));
   ...> }
 0         0
 1         1
 2        10
 3        11
 4       100
 5       101
 6       110
 7       111
 8      1000
 9      1001
10      1010
11      1011
12      1100
13      1101
14      1110
15      1111
```
