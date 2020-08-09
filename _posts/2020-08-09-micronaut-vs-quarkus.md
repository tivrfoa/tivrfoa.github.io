---
layout: post
title:  "Micronaut vs Quarkus Simple Benchmark"
date:   2020-08-09 12:30:44 -0300
categories: quarkus micronaut graalvm java microservice native
---

### Goal of this simple benchmark

I watched a nice comparison made by Micronaut's creator  Graeme Rocher:<br>
[Micronaut 2.0 vs Quarkus 1.3.1 vs Spring Boot 2.3 Performance on JDK 14](https://www.youtube.com/watch?v=rJFgdFIs_k8)

And I read the nice post ["A IO thread and a worker thread walk into a bar: a microbenchmark story"](https://quarkus.io/blog/io-thread-benchmark/)
by Emmanuel Bernard explaining the benchmark made above.

**So the goal of this benchmark is:**
  - to include the `reactive` approach presented in the Quarkus post;
  - also compare the native images produced by by GraalVM for both frameworks;
  - introduce two new tests that have block operations.

### Spoiler regarding native image

As the JVM warms up, it has much higher throughput than native images.<br>
But for short lived functions the native image is better, because of
the startup time.

  - without -Xmx128m
	- Quarkus JVM - 244,9 MiB (RSS) - Requests per second: 2399.87 -> 28531.32
    - Quarkus Native - 300 MiB (RSS) - Requests per second: 18108.92
  - with -Xmx128m
    - Quarkus JVM - 263,9 MiB (RSS) - Requests per second: 2539.22 -> 28841.80
    - Quarkus Native - 54 MiB (RSS) - Requests per second: 1951.44

### JDK and Java versions

```shell
$ java --version
openjdk 11.0.8 2020-07-14
OpenJDK Runtime Environment (build 11.0.8+10-post-Ubuntu-0ubuntu118.04.1)
OpenJDK 64-Bit Server VM (build 11.0.8+10-post-Ubuntu-0ubuntu118.04.1, mixed mode, sharing)
```

```shell
$ javac --version
javac 11.0.8
```

ps: I was first running with the version below, but then I found that it has
some memory issues :/

```shell
$ java --version
openjdk 11.0.7 2020-04-14
OpenJDK Runtime Environment GraalVM CE 20.1.0 (build 11.0.7+10-jvmci-20.1-b02)
OpenJDK 64-Bit Server VM GraalVM CE 20.1.0 (build 11.0.7+10-jvmci-20.1-b02, mixed mode, sharing)
```

### Hardware/System Config

  - Ubuntu 18.04.4 LTS
  - 64-bit
  - RAM: 7,7 GB
  - Intel® Core™ i5-7200U CPU @ 2.50GHz × 4
  
```shell
$ uname -a
Linux 4.15.0-112-generic #113-Ubuntu SMP Thu Jul 9 23:41:39 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
```

### Quarkus IO Thread vs Worker Pool

To compare the 'reactive' style I copied the code from the `quarkus-io-thread` folder:<br>
[https://github.com/johnaohara/quarkus-iothread-workerpool/tree/1.3.1.Final](https://github.com/johnaohara/quarkus-iothread-workerpool/tree/1.3.1.Final)

For the tests I called:
  - 'Worker Pool' -> imperative
  - 'IO Thread'   -> reactive

### Let's start!

The instructions to create the native images
[are at the end](#creating-native-images---graalvm)
of this post.

### ./mvnw clean compile
   
Micronaut 2.0.0.M2 | Quarkus 1.3.1.Final (imperative) | Quarkus 1.3.1.Final (reactive) 
------------------ | -------------------------------- | ------------------------------
2.030 s | 2.543 s | 2.692 s

### ./mvnw test
    
Micronaut 2.0.0.M2 | Quarkus 1.3.1.Final (imperative) | Quarkus 1.3.1.Final (reactive) 
------------------ | -------------------------------- | ------------------------------
4.093 s | 7.410 s | 7.326 s

### ./mvnw clean package

Micronaut 2.0.0.M2 | Quarkus 1.3.1.Final (imperative) | Quarkus 1.3.1.Final (reactive) 
------------------ | -------------------------------- | ------------------------------
6.748 s | 12.910 s | 13.131 s

### Time it takes to start in development mode

  - Micronaut: ./mvnw mn:run
  - Quarkus: ./mvnw quarkus:dev

Micronaut 2.0.0.M2 | Quarkus 1.3.1.Final (imperative) | Quarkus 1.3.1.Final (reactive) 
------------------ | -------------------------------- | ------------------------------
718 ms | 1.420 s | 1.252 s

ps: if you disable debug for Quarkus ( -Ddebug ), it will save you around 60 ms ...
not worth the effort.

It's nice that both frameworks detect changes:

  - Micronaut
```log
[INFO] Detected change in .../framework-comparison-2020/micronaut-example-2.0.0.M2/src/main/java/micronaut/example/MessageService.java
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] Copying 2 resources
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 4 source files to .../framework-comparison-2020/micronaut-example-2.0.0.M2/target/classes
```

  - Quarkus
```log
2020-08-07 12:15:47,111 INFO  [io.qua.dev] (vert.x-worker-thread-0) Changed source files detected, recompiling [.../framework-comparison-2020/quarkus-example-reactive-1.3.1.Final/src/main/java/org/acme/MessageService.java]
2020-08-07 12:15:47,487 INFO  [io.quarkus] (vert.x-worker-thread-0) Quarkus stopped in 0.008s
__  ____  __  _____   ___  __ ____  ______ 
 --/ __ \/ / / / _ | / _ \/ //_/ / / / __/ 
 -/ /_/ / /_/ / __ |/ , _/ ,< / /_/ /\ \   
--\___\_\____/_/ |_/_/|_/_/|_|\____/___/   
2020-08-07 12:15:47,762 INFO  [io.quarkus] (vert.x-worker-thread-0) code-with-quarkus 1.0.0-SNAPSHOT (powered by Quarkus 1.3.1.Final) started in 0.043s. Listening on: http://0.0.0.0:8080
2020-08-07 12:15:47,762 INFO  [io.quarkus] (vert.x-worker-thread-0) Profile dev activated. Live Coding activated.
2020-08-07 12:15:47,762 INFO  [io.quarkus] (vert.x-worker-thread-0) Installed features: [cdi, hibernate-validator, mutiny, vertx, vertx-web]
2020-08-07 12:15:47,762 INFO  [io.qua.dev] (vert.x-worker-thread-0) Hot replace total time: 0.653s
```

### Time it takes to start the jars

java -jar target/micronaut-example-0.1.jar<br>
java -jar target/code-with-quarkus-1.0.0-SNAPSHOT-runner.jar<br>
java -jar target/quarkus-io-thread-runner.jar

Micronaut 2.0.0.M2 | Quarkus 1.3.1.Final (imperative) | Quarkus 1.3.1.Final (reactive) 
------------------ | -------------------------------- | ------------------------------
827 ms | 1.182 s | 1.088 s

### Time to First Response

It was measured using the time.js script, eg:
`$ node time.js micronaut-example-0.1.jar`

And for the native images:
`$ node time-native.js micronaut-example`

Micronaut 2.0.0.M2 | Quarkus 1.3.1.Final (imperative) | Quarkus 1.3.1.Final (reactive) | Native Quarkus 1.3.1.Final (imperative) | Native Quarkus 1.3.1.Final (reactive) | Native Micronaut 2.0.0.M2
------------------ | -------------------------------- | ------------------------------ | --------------------------------------- | ------------------------------------- | -------------------------
1428 ms | 1499 ms | 1301 ms | 26 ms | 25 ms | 36 ms

### ab - Requests per Second: /hello/John

java -jar path/to/jar.jar<br>
ab -k -c 20 -n 10000 http://localhost:8080/hello/John
  - `-k`: keep-alive
  - `-c 20`: 20 concurrency requests
  - `-n 10000`: 10.000 requests

Micronaut 2.0.0.M2 | Quarkus 1.3.1.Final (imperative) | Quarkus 1.3.1.Final (reactive) | Native Quarkus 1.3.1.Final (imperative) | Native Quarkus 1.3.1.Final (reactive) | Native Micronaut 2.0.0.M2
------------------ | -------------------------------- | ------------------------------ | --------------------------------------- | ------------------------------------- | -------------------------
38551.84 | 27956.78 | **78252.77** | 17315.99 | 39053.04 | 19053.28

You need to run multiple times to warmp up the JVM!<br>
And it's incredible how the number grows. For example, for Quarkus (reactive), it
starts at 5563.98 and goes to 78252.77

* ps: I'm not sure this *"reactive"* style is how most people would
code in Quarkus. It's also a bit lower abstraction.
But it's good to know that we have it at our disposal.

Also one important point about the reactive approach, from
[Vert.x docs:](https://vertx.io/docs/vertx-core/java/)

> **The Golden Rule - Don’t Block the Event Loop**
>
>We already know that the Vert.x APIs are non blocking and won’t block the event loop, but that’s not much help if you block the event loop yourself in a handler.
>
>If you do that, then that event loop will not be able to do anything else while it’s blocked. If you block all of the event loops in Vertx instance then your application will grind to a complete halt!
>
>So don’t do it! You have been warned.

So let's do some testings introducing just a little blocking ...

### ab Test Sleeping 2 milliseconds each 5 requests: /split/John


Micronaut 2.0.0.M2 | Quarkus 1.3.1.Final (imperative) | Quarkus 1.3.1.Final (reactive) | Native Quarkus 1.3.1.Final (imperative) | Native Quarkus 1.3.1.Final (reactive) | Native Micronaut 2.0.0.M2
------------------ | -------------------------------- | ------------------------------ | --------------------------------------- | ------------------------------------- | -------------------------
14816.86 | **24395** | 17365.75 | 15355.25 | 14060.33 | 13223.58

That's an interesting result! Because this is probably the scenario that
you will find in most applications.

It's nice explained here
[Quarkus IO Thread Microbencmark:](https://quarkus.io/blog/io-thread-benchmark/)

*"When your workload needs to interact with a database or another remote
service, it relies on blocking IO. The thread is blocked waiting for the
answer. Other requests running on different threads are not slowed down
significantly. But this means one thread for every concurrent request,
which limits the overall concurrency."*

### ab Test Sleeping 2 milliseconds for **each** request: /sleep

Micronaut 2.0.0.M2 | Quarkus 1.3.1.Final (imperative) | Quarkus 1.3.1.Final (reactive) | Native Quarkus 1.3.1.Final (imperative) | Native Quarkus 1.3.1.Final (reactive) | Native Micronaut 2.0.0.M2
------------------ | -------------------------------- | ------------------------------ | --------------------------------------- | ------------------------------------- | -------------------------
3137.98 | **8692.10** | 3419.26 | 8457.52 | 2970.71 | 2844.78

### Memory Comsumption - RSS

**!!! Huge difference between JVMs below !!!**
  - openjdk 11.0.8 2020-07-14 (1)
OpenJDK Runtime Environment (build 11.0.8+10-post-Ubuntu-0ubuntu118.04.1)
  - openjdk 11.0.7 2020-04-14 (2)
OpenJDK Runtime Environment GraalVM CE 20.1.0 (build 11.0.7+10-jvmci-20.1-b02)

So I ran with both versions ...

Mesuared after running 15x the command below:<br>
```shell
ab -k -c 20 -n 10000 http://localhost:8080/hello/John
```

#### No -Xmx - openjdk 11.0.8 2020-07-14 (1)

Micronaut 2.0.0.M2 | Quarkus 1.3.1.Final (imperative) | Quarkus 1.3.1.Final (reactive) | Native Quarkus 1.3.1.Final (imperative) | Native Quarkus 1.3.1.Final (reactive) | Native Micronaut 2.0.0.M2
------------------ | -------------------------------- | ------------------------------ | --------------------------------------- | ------------------------------------- | -------------------------
345,8 MiB | 257,5 MiB | 200,9 MiB | 293,1 MiB | 282,6 MiB | 427,8 MiB

#### With -Xmx128m - openjdk 11.0.8 2020-07-14 (1)

Micronaut 2.0.0.M2 | Quarkus 1.3.1.Final (imperative) | Quarkus 1.3.1.Final (reactive) | Native Quarkus 1.3.1.Final (imperative) | Native Quarkus 1.3.1.Final (reactive) | Native Micronaut 2.0.0.M2
------------------ | -------------------------------- | ------------------------------ | --------------------------------------- | ------------------------------------- | -------------------------
233,6 MiB | 270,6 MiB | 200,1 MiB | 52 MiB | 42 MiB | ?¹

¹ Maybe I did something wrong when I created Micronaut native image, because
running with -Xmx128m `./micronaut-example -Xmx128m` I got the error:
```log
11:47:11.021 [nioEventLoopGroup-1-9] ERROR i.m.h.s.netty.RoutingInBoundHandler - Micronaut Server Error - No request state present. Cause: Direct buffer memory
java.lang.OutOfMemoryError: Direct buffer memory
	at java.nio.Bits.reserveMemory(Bits.java:175)
	at java.nio.DirectByteBuffer.<init>(DirectByteBuffer.java:118)
	at java.nio.ByteBuffer.allocateDirect(ByteBuffer.java:317)
	at io.netty.buffer.PoolArena$DirectArena.allocateDirect(PoolArena.java:758)
	at io.netty.buffer.PoolArena$DirectArena.newChunk(PoolArena.java:734)
	at io.netty.buffer.PoolArena.allocateNormal(PoolArena.java:245)
	at io.netty.buffer.PoolArena.allocate(PoolArena.java:215)
	at io.netty.buffer.PoolArena.allocate(PoolArena.java:147)
	at io.netty.buffer.PooledByteBufAllocator.newDirectBuffer(PooledByteBufAllocator.java:356)
	at io.netty.buffer.AbstractByteBufAllocator.directBuffer(AbstractByteBufAllocator.java:187)
	at io.netty.buffer.AbstractByteBufAllocator.directBuffer(AbstractByteBufAllocator.java:178)
	at io.netty.buffer.AbstractByteBufAllocator.ioBuffer(AbstractByteBufAllocator.java:139)
	at io.netty.channel.DefaultMaxMessagesRecvByteBufAllocator$MaxMessageHandle.allocate(DefaultMaxMessagesRecvByteBufAllocator.java:114)
	at io.netty.channel.nio.AbstractNioByteChannel$NioByteUnsafe.read(AbstractNioByteChannel.java:147)
	at io.netty.channel.nio.NioEventLoop.processSelectedKey(NioEventLoop.java:714)
	at io.netty.channel.nio.NioEventLoop.processSelectedKeysOptimized(NioEventLoop.java:650)
	at io.netty.channel.nio.NioEventLoop.processSelectedKeys(NioEventLoop.java:576)
	at io.netty.channel.nio.NioEventLoop.run(NioEventLoop.java:493)
	at io.netty.util.concurrent.SingleThreadEventExecutor$4.run(SingleThreadEventExecutor.java:989)
	at io.netty.util.internal.ThreadExecutorMap$2.run(ThreadExecutorMap.java:74)
	at io.netty.util.concurrent.FastThreadLocalRunnable.run(FastThreadLocalRunnable.java:30)
	at java.lang.Thread.run(Thread.java:834)
	at com.oracle.svm.core.thread.JavaThreads.threadStartRoutine(JavaThreads.java:517)
	at com.oracle.svm.core.posix.thread.PosixJavaThreads.pthreadStartRoutine(PosixJavaThreads.java:193)
```

#### No -Xmx - GraalVM runtime (2)

Micronaut 2.0.0.M2 | Quarkus 1.3.1.Final (imperative) | Quarkus 1.3.1.Final (reactive)
------------------ | -------------------------------- | ------------------------------
608,4 MiB | 486,1 MiB | 467,6 MiB

#### With -Xmx128m - GraalVM runtime (2)

Micronaut 2.0.0.M2 | Quarkus 1.3.1.Final (imperative) | Quarkus 1.3.1.Final (reactive)
------------------ | -------------------------------- | ------------------------------
498,8 MiB | 482,2 MiB | 456 MiB


### Conclusion

Even what seems a simple benchmark can be hard to do :( ...<br>
JDK and JVM versions and their configurations can have great impact on the
benchmark.

Both frameworks are fast, use low memory, offers great productivity
and have support for native images using GraalVM.

People should also consider other factors when making their choice:
  - Size of community
  - How active is the project
  - How often regressions happen
  - How fast bugs are fixed, and if the fixes are backported¹
  - Quality of documentation
  - Company support

¹ Or should we always migrate to new releases if we want the fixes?
I understand it must be a real trouble to have a different release *only*
with the bug fixes. But otherwise users can be stuck in an endless loop:
bugs -> upgrade -> bugs


### Creating Native Images - GraalVM

I used this version of GraalVM to compile both frameworks:

[graalvm-ce-java11-linux-amd64-20.1.0.tar.gz](https://github.com/graalvm/graalvm-ce-builds/releases/download/vm-20.1.0/graalvm-ce-java11-linux-amd64-20.1.0.tar.gz)


#### Instructions for Quarkus

[Instructions for Quarkus and GraalVM](https://quarkus.io/guides/building-native-image#configuring-graalvm)

$ ./mvnw package -Pnative

The binary was built faster than I thought! 2:17min

```log
[code-with-quarkus-1.0.0-SNAPSHOT-runner:30080]    classlist:   5,491.56 ms,  0.96 GB
[code-with-quarkus-1.0.0-SNAPSHOT-runner:30080]        (cap):   2,335.22 ms,  0.94 GB
[code-with-quarkus-1.0.0-SNAPSHOT-runner:30080]        setup:   4,851.33 ms,  0.94 GB
20:14:27,377 INFO  [org.hib.val.int.uti.Version] HV000001: Hibernate Validator 6.1.2.Final
20:14:29,779 INFO  [org.jbo.threads] JBoss Threads version 3.0.1.Final
[code-with-quarkus-1.0.0-SNAPSHOT-runner:30080]     (clinit):     549.80 ms,  2.62 GB
[code-with-quarkus-1.0.0-SNAPSHOT-runner:30080]   (typeflow):  25,044.51 ms,  2.62 GB
[code-with-quarkus-1.0.0-SNAPSHOT-runner:30080]    (objects):  20,975.97 ms,  2.62 GB
[code-with-quarkus-1.0.0-SNAPSHOT-runner:30080]   (features):     470.03 ms,  2.62 GB
[code-with-quarkus-1.0.0-SNAPSHOT-runner:30080]     analysis:  48,785.14 ms,  2.62 GB
[code-with-quarkus-1.0.0-SNAPSHOT-runner:30080]     universe:   1,254.35 ms,  2.62 GB
[code-with-quarkus-1.0.0-SNAPSHOT-runner:30080]      (parse):   5,190.79 ms,  2.62 GB
[code-with-quarkus-1.0.0-SNAPSHOT-runner:30080]     (inline):   7,324.82 ms,  2.94 GB
[code-with-quarkus-1.0.0-SNAPSHOT-runner:30080]    (compile):  37,087.20 ms,  2.92 GB
[code-with-quarkus-1.0.0-SNAPSHOT-runner:30080]      compile:  51,278.90 ms,  2.92 GB
[code-with-quarkus-1.0.0-SNAPSHOT-runner:30080]        image:   3,023.71 ms,  2.88 GB
[code-with-quarkus-1.0.0-SNAPSHOT-runner:30080]        write:   3,567.54 ms,  2.88 GB
[code-with-quarkus-1.0.0-SNAPSHOT-runner:30080]      [total]: 119,568.72 ms,  2.88 GB
[INFO] [io.quarkus.deployment.QuarkusAugmentor] Quarkus augmentation completed in 125054ms
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  02:17 min
[INFO] Finished at: 2020-08-08T20:16:17-03:00
[INFO] ------------------------------------------------------------------------
```

$ ./target/code-with-quarkus-1.0.0-SNAPSHOT-runner 
```java
__  ____  __  _____   ___  __ ____  ______ 
 --/ __ \/ / / / _ | / _ \/ //_/ / / / __/ 
 -/ /_/ / /_/ / __ |/ , _/ ,< / /_/ /\ \   
--\___\_\____/_/ |_/_/|_/_/|_|\____/___/   
2020-08-08 12:08:15,077 INFO  [io.quarkus] (main) code-with-quarkus 1.0.0-SNAPSHOT (powered by Quarkus 1.3.1.Final) started in 0.011s. Listening on: http://0.0.0.0:8080
2020-08-08 12:08:15,077 INFO  [io.quarkus] (main) Profile prod activated. 
2020-08-08 12:08:15,077 INFO  [io.quarkus] (main) Installed features: [cdi, hibernate-validator, resteasy]
```

#### Instructions for Micronaut

  - [Micronaut first Graal app](https://guides.micronaut.io/micronaut-creating-first-graal-app/guide/index.html#graal)
  - [Micronaut guide: GraalVM](https://docs.micronaut.io/latest/guide/index.html#graal)

$ native-image --no-server -cp target/micronaut-example-0.1.jar 
```log
[micronaut-example:12298]    classlist:   4,894.61 ms,  0.94 GB
[micronaut-example:12298]        (cap):   2,885.53 ms,  0.94 GB
[micronaut-example:12298]        setup:   6,435.89 ms,  0.94 GB
WARNING GR-10238: VarHandle for static field is currently not fully supported. Static field private static volatile java.lang.System$Logger jdk.internal.event.EventHelper.securityLogger is not properly marked for Unsafe access!
Warning: class initialization of class io.micronaut.http.bind.binders.ContinuationArgumentBinder failed with exception java.lang.NoClassDefFoundError: kotlin/TypeCastException. This class will be initialized at run time because option --allow-incomplete-classpath is used for image building. Use the option --initialize-at-run-time=io.micronaut.http.bind.binders.ContinuationArgumentBinder to explicitly request delayed initialization of this class.
Warning: class initialization of class io.micronaut.http.bind.binders.ContinuationArgumentBinder$Companion failed with exception java.lang.NoClassDefFoundError: kotlin/TypeCastException. This class will be initialized at run time because option --allow-incomplete-classpath is used for image building. Use the option --initialize-at-run-time=io.micronaut.http.bind.binders.ContinuationArgumentBinder$Companion to explicitly request delayed initialization of this class.
[micronaut-example:12298]     (clinit):   1,031.40 ms,  2.88 GB
[micronaut-example:12298]   (typeflow):  33,285.50 ms,  2.88 GB
[micronaut-example:12298]    (objects):  31,182.64 ms,  2.88 GB
[micronaut-example:12298]   (features):   3,973.39 ms,  2.88 GB
[micronaut-example:12298]     analysis:  72,308.19 ms,  2.88 GB
[micronaut-example:12298]     universe:   2,084.96 ms,  2.91 GB
[micronaut-example:12298]      (parse):   8,151.94 ms,  2.93 GB
[micronaut-example:12298]     (inline):  11,121.85 ms,  3.27 GB
[micronaut-example:12298]    (compile):  57,107.71 ms,  3.27 GB
[micronaut-example:12298]      compile:  78,775.50 ms,  3.27 GB
[micronaut-example:12298]        image:   4,824.30 ms,  3.26 GB
[micronaut-example:12298]        write:   9,926.53 ms,  3.26 GB
[micronaut-example:12298]      [total]: 180,872.39 ms,  3.26 GB
```

$ ./micronaut-example 
```java
13:57:25.508 [main] INFO  io.micronaut.runtime.Micronaut - Startup completed in 11ms. Server Running: http://localhost:8080
```
