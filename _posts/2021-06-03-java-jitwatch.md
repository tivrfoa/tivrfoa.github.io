---
layout: post
title:  "Java JITWatch"
date:   2021-06-03 13:40:00 -0300
categories: java jit jitwatch
---

From the project [wiki page](https://github.com/AdoptOpenJDK/jitwatch/wiki):

>Description
>
>A tool for understanding the behaviour of the Java HotSpot Just-In-Time (JIT) compiler during the execution of your program.
>
>Works by processing the hotspot.log file output by the JVM.

I found this tool because I was trying to understand the just in time (JIT) compilation performed by Java.

The tool shows some interesting things, like:
  - inlined methods
  - compiled methods
  - compilation order


### Test class

```java
package jitwatch.tests;

import java.util.concurrent.ThreadLocalRandom;

public class loop1 {
	public static void main(String[] args) {
		long t = 0;
		for (int i = 0; i < 10_000_000; i++) {
			int randomNum = ThreadLocalRandom.current().nextInt(0, 10 + 1);
			int r = new loop1().go(randomNum);
			if (r > 300)
				t += r;
		}
		System.out.println(t);
	}

	int go(int s) {
		int[] a = new int[19];
		for (int i = 0; i < 10; i++)
			for (int j = 0; j < 10; j++)
				a[i + j] = f1(i, j) + s;
		return sum(a);
	}

	int f1(int a, int b) { return a + b; }

	int sum(int[] a) {
		int s = 0;
		for (int i : a) s += i;
		return s;
	}
}
```

**Compiling**

```sh
javac -g -d classes/ src/jitwatch/tests/loop1.java
```

**Running the class and creating the log file**

```sh
java -cp classes/ -XX:+UnlockDiagnosticVMOptions -XX:+TraceClassLoading -XX:+LogCompilation -XX:+PrintAssembly -XX:+DebugNonSafepoints -XX:-UseCompressedOops jitwatch.tests.loop1
```

You might get the error:<br>
`Could not load hsdis-amd64.so; library not loadable; PrintAssembly is disabled`

My workaround was to get `hsdis-amd64.so` file from [https://github.com/liuzhengyang/hsdis](https://github.com/liuzhengyang/hsdis)
and put it in `$JAVA_HOME/lib/server`.

### Running JITWatch

```sh
# Clone the repo
git clone https://github.com/AdoptOpenJDK/jitwatch

cd jitwatch

mvn compile exec:java
```

**Configuring sources and classes**

Before loading the log file, it's important to configure the source and classes, eg:

![JITWatch Configuration](/assets/images/jitwatch-configuration.png)

Then `Open Log` and finally `Start`.

### Screenshots

- JIT compiled class members
[![JIT compiled class members](/assets/images/JIT-compiled-class-members.png)](/assets/images/JIT-compiled-class-members.png)

- TriView: Source code - Bytecode - Assembly
[![TriView: Source code - Bytecode - Assembly](/assets/images/TriView-source-code-bytecode-assembly.png)](/assets/images/TriView-source-code-bytecode-assembly.png)

- Compile Chain - Inlined and Compiled methods
[![Compile Chain - Inlined and Compiled methods](/assets/images/compile-chain-inlined-and-compiled-methods.png)](/assets/images/compile-chain-inlined-and-compiled-methods.png)

- JIT compilation order
[![JIT compilation order](/assets/images/jit-compilation-order.png)](/assets/images/jit-compilation-order.png)

- Method in Assembly - C2 Level 4
[![Method in Assembly - C2 Level 4](/assets/images/method-in-assembly-c2-level4.png)](/assets/images/method-in-assembly-c2-level4.png)
