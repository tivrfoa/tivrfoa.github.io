---
layout: post
title:  "#1BRC Timeline"
date:   2024-02-05 11:30:00 -0300
categories: Java benchmark performance
---

# 1BRC Timeline

*Important moments, key ideas and most performance improvements in [The One Billion Row Challenge](https://github.com/gunnarmorling/1brc)*

The goal is to learn which changes most improved performance,
and also to give credit to the original authors.

*ps: it doesn't necessarily mean that other people copied the code.
Others might have arrived at the same/similar solution on their own.*

# Leaderboard Changes

`git log -p --reverse  README.md | rg "\+\|\s1"`

| # | Result (m:s.ms) | Implementation     | JDK | Submitter     | Notes     |
|---|-----------------|--------------------|-----|---------------|-----------|
| 1.|        04:13.449| [CalculateAverage.java](https://github.com/gunnarmorling/onebrc/blob/main/src/main/java/dev/morling/onebrc/CalculateAverage.java) (baseline)| 21.0.1-open| Gunnar Morling|
| 1.|        02:08.845| [CalculateAverage_royvanrijn.java](https://github.com/gunnarmorling/1brc/blob/main/src/main/java/dev/morling/onebrc/CalculateAverage_royvanrijn.java) (baseline)| 21.0.1-open| Roy van Rijn| Process lines in parallel
| 1.|        02:08.845| [CalculateAverage_royvanrijn.java](https://github.com/gunnarmorling/1brc/blob/main/src/main/java/dev/morling/onebrc/CalculateAverage_royvanrijn.java)| 21.0.1-open| Roy van Rijn|
| 1.|        02:08.845| [CalculateAverage_itaske.java](https://github.com/gunnarmorling/1brc/blob/main/src/main/java/dev/morling/onebrc/CalculateAverage_itaske.java)| 21.0.1-open| [itaske](https://github.com/itaske)|
| 1.|        02:08.315| [CalculateAverage_itaske.java](https://github.com/gunnarmorling/1brc/blob/main/src/main/java/dev/morling/onebrc/CalculateAverage_itaske.java)| 21.0.1-open| [itaske](https://github.com/itaske)|
| 1.|        00:38.510| [CalculateAverage_bjhara.java](https://github.com/gunnarmorling/1brc/blob/main/src/main/java/dev/morling/onebrc/CalculateAverage_bjhara.java)| 21.0.1-open| [Hampus Ram](https://github.com/bjhara)|Memory mapped file using `FileChannel` and `MappedByteBuffer`
| 1.|        00:23.366| [link](https://github.com/gunnarmorling/1brc/pull/5/)| 21.0.1-open| [Roy van Rijn](https://github.com/royvanrijn)|SWAR
| 1.|        00:14.848| [link](https://github.com/gunnarmorling/1brc/blob/main/src/main/java/dev/morling/onebrc/CalculateAverage_spullara.java)| 21.0.1-graalce| [Sam Pullara](https://github.com/spullara)|Lazy String creation and better hash table?
| 1.|        00:12.063| [link](https://github.com/gunnarmorling/1brc/blob/main/src/main/java/dev/morling/onebrc/CalculateAverage_spullara.java)| 21.0.1-graalce| [Sam Pullara](https://github.com/spullara)|
| 1.|        00:10.870| [link](https://github.com/gunnarmorling/1brc/blob/main/src/main/java/dev/morling/onebrc/CalculateAverage_ebarlas.java)| 21.0.1-graalce| [Elliot Barlas](https://github.com/ebarlas)|
| 1.|        00:07.999| [link](https://github.com/gunnarmorling/1brc/blob/main/src/main/java/dev/morling/onebrc/CalculateAverage_royvanrijn.java)| 21.0.1-GraalVM Native - Unsafe   | [Roy van Rijn](https://github.com/royvanrijn)|
| 1.|        00:07.620| [link](https://github.com/gunnarmorling/1brc/blob/main/src/main/java/dev/morling/onebrc/CalculateAverage_merykitty.java)| 21.0.1-open   | [Quan Anh Mai](https://github.com/merykitty)|Vector API, Magic Branchless Number Parsing
| 1.|        00:06.159| [link](https://github.com/gunnarmorling/1brc/blob/main/src/main/java/dev/morling/onebrc/CalculateAverage_royvanrijn.java)| 21.0.1-GraalVM Native - Unsafe | [royvanrijn](https://github.com/royvanrijn)|
| 1 | 00:03.604 | [link](https://github.com/gunnarmorling/1brc/blob/main/src/main/java/dev/morling/onebrc/CalculateAverage_royvanrijn.java)| 21.0.1-GraalVM Native - Unsafe | [Roy van Rijn](https://github.com/royvanrijn) |New server
| 1 | 00:03.044 | [link](https://github.com/gunnarmorling/1brc/blob/main/src/main/java/dev/morling/onebrc/CalculateAverage_thomaswue.java)| 21.0.1-GraalVM Native - Unsafe | [Thomas Wuerthinger](https://github.com/thomaswue), [Quan Anh Mai](https://github.com/merykitty), [Alfonso² Peterssen](https://github.com/mukel) | 
| 1 | 00:02.941 | [link](https://github.com/gunnarmorling/1brc/blob/main/src/main/java/dev/morling/onebrc/CalculateAverage_royvanrijn.java)| 21.0.1-GraalVM Native - Unsafe | [Roy van Rijn](https://github.com/royvanrijn) |  |
| 1 | 00:02.904 | [link](https://github.com/gunnarmorling/1brc/blob/main/src/main/java/dev/morling/onebrc/CalculateAverage_royvanrijn.java)| 21.0.1-GraalVM Native - Unsafe | [Roy van Rijn](https://github.com/royvanrijn) |  |
| 1 | 00:02.575 | [link](https://github.com/gunnarmorling/1brc/blob/main/src/main/java/dev/morling/onebrc/CalculateAverage_merykittyunsafe.java)| 21.0.1-open | [Quan Anh Mai](https://github.com/merykitty) | Use `Unsafe` *His previous time after server changed was 00:03.258, so 683ms faster.* |
| 1 | 00:02.552 | [link](https://github.com/gunnarmorling/1brc/blob/main/src/main/java/dev/morling/onebrc/CalculateAverage_thomaswue.java)| 21.0.1-GraalVM Native - Unsafe | [Thomas Wuerthinger](https://github.com/thomaswue), [Quan Anh Mai](https://github.com/merykitty), [Alfonso² Peterssen](https://github.com/mukel) |  |
| 1* | 00:02.461 | [link](https://github.com/gunnarmorling/1brc/blob/main/src/main/java/dev/morling/onebrc/CalculateAverage_artsiomkorzun.java)| 21.0.1-GraalVM Native - Unsafe | [Artsiom Korzun](https://github.com/artsiomkorzun) |  |
| 1* | 00:02.477 | [link](https://github.com/gunnarmorling/1brc/blob/main/src/main/java/dev/morling/onebrc/CalculateAverage_abeobk.java)| 21.0.1-GraalVM Native - Unsafe | [Van Phu DO](https://github.com/abeobk) |  |
| 1 | 00:02.336 | [link](https://github.com/gunnarmorling/1brc/blob/main/src/main/java/dev/morling/onebrc/CalculateAverage_abeobk.java)| 21.0.1-GraalVM Native - Unsafe | [Van Phu DO](https://github.com/abeobk) |  |
| 1 | 00:02.195 | [link](https://github.com/gunnarmorling/1brc/blob/main/src/main/java/dev/morling/onebrc/CalculateAverage_thomaswue.java)| 21.0.1-GraalVM Native - Unsafe | [Thomas Wuerthinger](https://github.com/thomaswue), [Quan Anh Mai](https://github.com/merykitty), [Alfonso² Peterssen](https://github.com/mukel) | Subprocess spawn trick |
| 1 | 00:02.019 | [link](https://github.com/gunnarmorling/1brc/blob/main/src/main/java/dev/morling/onebrc/CalculateAverage_artsiomkorzun.java)| 21.0.2-GraalVM Native - Unsafe | [Artsiom Korzun](https://github.com/artsiomkorzun) |  |
| 1 | 00:01.893 | [link](https://github.com/gunnarmorling/1brc/blob/main/src/main/java/dev/morling/onebrc/CalculateAverage_thomaswue.java)| 21.0.2-GraalVM Native - Unsafe | [Thomas Wuerthinger](https://github.com/thomaswue), [Quan Anh Mai](https://github.com/merykitty), [Alfonso² Peterssen](https://github.com/mukel) | Process 2MB segments to make all threads finish at the same time. Process with 3 scanners in parallel in the same thread. |
| 1 | 00:01.878 | [link](https://github.com/gunnarmorling/1brc/blob/main/src/main/java/dev/morling/onebrc/CalculateAverage_thomaswue.java)| 21.0.2-GraalVM Native - Unsafe | [Thomas Wuerthinger](https://github.com/thomaswue), [Quan Anh Mai](https://github.com/merykitty), [Alfonso² Peterssen](https://github.com/mukel) |  |
| 1 | 00:01.832 | [link](https://github.com/gunnarmorling/1brc/blob/main/src/main/java/dev/morling/onebrc/CalculateAverage_thomaswue.java)| 21.0.2-GraalVM Native - Unsafe | [Thomas Wuerthinger](https://github.com/thomaswue), [Quan Anh Mai](https://github.com/merykitty), [Alfonso² Peterssen](https://github.com/mukel) |  |
| 1 | 00:01.645 | [link](https://github.com/gunnarmorling/1brc/blob/main/src/main/java/dev/morling/onebrc/CalculateAverage_jerrinot.java)| 21.0.2-GraalVM Native - Unsafe | [Jaromir Hamala](https://github.com/jerrinot) |  |
| 1 | 00:01.535 | [link](https://github.com/gunnarmorling/1brc/blob/main/src/main/java/dev/morling/onebrc/CalculateAverage_thomaswue.java)| 21.0.2-GraalVM Native - Unsafe | [Thomas Wuerthinger](https://github.com/thomaswue), [Quan Anh Mai](https://github.com/merykitty), [Alfonso² Peterssen](https://github.com/mukel) |  |


# Timeline

### Dec 28, 2023 07:44:58 - Project is created

### Jan 1, 2024 11:38:16 - Baseline time: 04:13.449

### Jan 1, 2024 14:33:40 - royvanrijn - Processing lines in parallel - 02:08.845

```java
Map<String, Measurement> resultMap = Files.lines(Path.of(FILE)).parallel()
```

### Jan 2, 2024 10:04:29 - bjhara - Memory Mapped File - 00:38.510

[https://github.com/gunnarmorling/1brc/pull/10](https://github.com/gunnarmorling/1brc/pull/10)

[https://github.com/bjhara/1brc/blob/a7edadedaef09702982bbd3618fcdcaf9bf66e12/src/main/java/dev/morling/onebrc/CalculateAverage_bjhara.java](https://github.com/bjhara/1brc/blob/a7edadedaef09702982bbd3618fcdcaf9bf66e12/src/main/java/dev/morling/onebrc/CalculateAverage_bjhara.java)

Key idea:
  - Memory mapped file using `FileChannel` and `MappedByteBuffer`

```java
    private static Stream<ByteBuffer> splitFileChannel(final FileChannel fileChannel) throws IOException {
        return StreamSupport.stream(Spliterators.spliteratorUnknownSize(new Iterator<ByteBuffer>() {
            private static final long CHUNK_SIZE = 1024 * 1024 * 10L;

            private final long size = fileChannel.size();
            private long start = 0;

            @Override
            public boolean hasNext() {
                return start < size;
            }

            @Override
            public ByteBuffer next() {
                try {
                    MappedByteBuffer mappedByteBuffer = fileChannel.map(FileChannel.MapMode.READ_ONLY, start,
                            Math.min(CHUNK_SIZE, size - start));

                    // don't split the data in the middle of lines
                    // find the closest previous newline
                    int realEnd = mappedByteBuffer.limit() - 1;
                    while (mappedByteBuffer.get(realEnd) != '\n')
                        realEnd--;

                    realEnd++;

                    mappedByteBuffer.limit(realEnd);
                    start += realEnd;

                    return mappedByteBuffer;
                }
                catch (IOException ex) {
                    throw new UncheckedIOException(ex);
                }
            }
        }, Spliterator.IMMUTABLE), false);
    }
```

### Jan 2, 2024 16:14:32 - padreati - Vector API - 00:50.547 

Key ideas:
  - First use of the Vector API

```sh
JAVA_OPTS="--enable-preview --add-modules jdk.incubator.vector"
time java $JAVA_OPTS --class-path target/average-1.0.0-SNAPSHOT.jar dev.morling.onebrc.CalculateAverage_padreati
```

```xml
+            <compilerArgs>
+              <compilerArg>--enable-preview</compilerArg>
+              <compilerArg>--add-modules</compilerArg>
+              <compilerArg>java.base,jdk.incubator.vector</compilerArg>
+            </compilerArgs>
```

```java
import jdk.incubator.vector.ByteVector;
import jdk.incubator.vector.VectorOperators;
import jdk.incubator.vector.VectorSpecies;
```

### Jan 3, 2024 - royvanrijn - SWAR - 00:23.366

Key ideas:
  - SIMD Within A Register (SWAR) for finding ';'.
    - Iterates over a long, instead of a byte.
  - Use `int` instead of `double`
  - Branchless parse `int`

https://github.com/gunnarmorling/1brc/pull/5

https://github.com/gunnarmorling/1brc/blob/5570f1b60a557baf9ec6af412f8d5bd75fc44891/src/main/java/dev/morling/onebrc/CalculateAverage_royvanrijn.java

```java
/**
 * Changelog:
 *
 * Initial submission:          62000 ms
 * Chunked reader:              16000 ms
 * Optimized parser:            13000 ms
 * Branchless methods:          11000 ms
 * Adding memory mapped files:  6500 ms (based on bjhara's submission)
 * Skipping string creation:    4700 ms
 * Custom hashmap...            4200 ms
 * Added SWAR token checks:     3900 ms
 * Skipped String creation:     3500 ms (idea from kgonia)
 * Improved String skip:        3250 ms
 * Segmenting files:            3150 ms (based on spullara's code)
 * Not using SWAR for EOL:      2850 ms
 *
 * Best performing JVM on MacBook M2 Pro: 21.0.1-graal
 * `sdk use java 21.0.1-graal`
 *
 */
```

#### findNextSWAR

```java
    /**
     * -------- This section contains SWAR code (SIMD Within A Register) which processes a bytebuffer as longs to find values:
     */
    private static final long SEPARATOR_PATTERN = compilePattern((byte) ';');

    private int findNextSWAR(ByteBuffer bb, long pattern, int start, int limit) {
        int i;
        for (i = start; i <= limit - 8; i += 8) {
            long word = bb.getLong(i);
            int index = firstAnyPattern(word, pattern);
            if (index < Long.BYTES) {
                return i + index;
            }
        }
        // Handle remaining bytes
        for (; i < limit; i++) {
            if (bb.get(i) == (byte) pattern) {
                return i;
            }
        }
        return limit; // delimiter not found
    }

    private static int firstAnyPattern(long word, long pattern) {
        final long match = word ^ pattern;
        long mask = match - 0x0101010101010101L;
        mask &= ~match;
        mask &= 0x8080808080808080L;
        return Long.numberOfTrailingZeros(mask) >>> 3;
    }
```

Read more about this here: https://richardstartin.github.io/posts/finding-bytes

#### compilePattern

It replicates the byte value in a long.

```java
    private static long compilePattern(byte value) {
        return ((long) value << 56) | ((long) value << 48) | ((long) value << 40) | ((long) value << 32) |
                ((long) value << 24) | ((long) value << 16) | ((long) value << 8) | (long) value;
    }
```

```java
jshell> var p = compilePattern((byte) ';')
p ==> 4268070197446523707

jshell> Long.toHexString(p)
$4 ==> "3b3b3b3b3b3b3b3b"

jshell> Integer.toHexString((byte) ';')
$5 ==> "3b"
```

#### branchlessParseInt

```java
    /**
     * Branchless parser, goes from String to int (10x):
     * "-1.2" to -12
     * "40.1" to 401
     * etc.
     *
     * @param input
     * @return int value x10
     */
    private static int branchlessParseInt(final byte[] input, int start, int length) {
        // 0 if positive, 1 if negative
        final int negative = ~(input[start] >> 4) & 1;
        // 0 if nr length is 3, 1 if length is 4
        final int has4 = ((length - negative) >> 2) & 1;

        final int digit1 = input[start + negative] - '0';
        final int digit2 = input[start + negative + has4];
        final int digit3 = input[start + negative + has4 + 2];

        return (-negative ^ (has4 * (digit1 * 100) + digit2 * 10 + digit3 - 528) - negative); // 528 == ('0' * 10 + '0')
    }
```


franz1981 comment on ByteBuffer and big-endian

>By default ByteBuffers uses Big Indian, which is native for M1/M2...but not on x86, which is the one used for the benchmark.
>It translates in:
>
>    wrong position, while found
>    a useless reverse bytes
>
>It would be better to store in a static final, the native byte order and based on it, correctly reverse the bytes yourself (look at what I have done in the Netty code).
>Conversely you can always assume little endian, setting the mapped byte buffer order, and leave M1 to have the performance hit

https://github.com/gunnarmorling/1brc/pull/5#discussion_r1440157267


franz1981 - get long idea

>Hash code here has a data dependency: you either manually unroll this or just relax the hash code by using a var handle and use getLong amortizing the data dependency in batches, handing only the last 7 (or less) bytes separately, using the array.
>In this way the most of computation would like resolve in much less loop iterations too, similar to https://github.com/apache/activemq-artemis/blob/25fc0342275b29cd73123523a46e6e94582597cd/artemis-commons/src/main/java/org/apache/activemq/artemis/utils/ByteUtil.java#L299


### Jan 3, 2024 - Interesting comment about ZGC by fisk:

https://github.com/gunnarmorling/1brc/pull/15#issuecomment-1875495420

https://github.com/fisk/jdk/commit/8ce820de84b7031ced52fb63d190d9c8546f6730

```txt
@lobaorn do you really think you can nerd snipe me like that with benchmarking games?

Having said that... I implemented some experimental leyden support for generational ZGC, with some object streaming shenanigans allowing loading of archived objects, and support for archived code from the JIT cache so we can run with compiled code immediately and dodge most of the warmup costs.

On my machine with 256 cores...

real 0m2.318s
user 1m24.229s
sys 0m2.009s

...beat that!

Oh and here are the JVM flags: -XX:+UseLargePages -XX:+UseZGC -XX:+ZGenerational -Xms8G -Xmx8G -XX:+AlwaysPreTouch -XX:ConcGCThreads=4 -XX:+UnlockDiagnosticVMOptions -XX:-ZProactive -XX:ZTenuringThreshold=1 -Xlog:gc*:file=fatroom_gc.log -XX:SharedArchiveFile=Fatroom-dynamic.jsa -XX:+ReplayTraining -XX:+ArchiveInvokeDynamic -XX:+LoadCachedCode -XX:CachedCodeFile=Fatroom-dynamic.jsa-sc

Here is my experimental Generational ZGC leyden branch: https://github.com/fisk/jdk/tree/1brc_genz_leyden

Takes about 1/3 of the time compared to normal ZGC on my machine.
```

### Jan 3, 2024 11:35:51 - Nice 35 lines solution by Sam Pullara

https://github.com/spullara/1brc/blob/dd10a02e075fcdc11eb7ca9dbcb245ba9db739d2/src/main/java/dev/morling/onebrc/CalculateAverage_naive.java

```java
package dev.morling.onebrc;

import java.io.BufferedReader;
import java.io.FileNotFoundException;
import java.io.FileReader;
import java.util.concurrent.ConcurrentSkipListMap;
import java.util.stream.Collectors;

public class CalculateAverage_naive {

    record Result(double min, double max, double sum, long count) {
    }

    public static void main(String[] args) throws FileNotFoundException {
        long start = System.currentTimeMillis();
        var results = new BufferedReader(new FileReader("./measurements.txt"))
                .lines()
                //.parallel() // I don't know why Sam removed this. But it makes it much faster.
                .map(l -> l.split(";"))
                .collect(Collectors.toMap(
                        parts -> parts[0],
                        parts -> {
                            double temperature = Double.parseDouble(parts[1]);
                            return new Result(temperature, temperature, temperature, 1);
                        },
                        (oldResult, newResult) -> {
                            double min = Math.min(oldResult.min, newResult.min);
                            double max = Math.max(oldResult.max, newResult.max);
                            double sum = oldResult.sum + newResult.sum;
                            long count = oldResult.count + newResult.count;
                            return new Result(min, max, sum, count);
                        }, ConcurrentSkipListMap::new));
        System.out.println(System.currentTimeMillis() - start);
        System.out.println(results);
    }
}
```

### Jan 3, 2024 11:35:51 - spullara - lazy String creation and better hash table (?) - 00:14.848

[https://github.com/spullara/1brc/blob/dd10a02e075fcdc11eb7ca9dbcb245ba9db739d2/src/main/java/dev/morling/onebrc/CalculateAverage_spullara.java](https://github.com/spullara/1brc/blob/dd10a02e075fcdc11eb7ca9dbcb245ba9db739d2/src/main/java/dev/morling/onebrc/CalculateAverage_spullara.java)

[https://github.com/gunnarmorling/1brc/pull/21](https://github.com/gunnarmorling/1brc/pull/21)

It was not clear at first what made such big difference,
because his solution:
  - does **not** use SWAR
  - work with doubles

..., but creating String name only when aggregating certainly speeds things up.

```java
class ByteArrayToResultMap {
  public static final int MAPSIZE = 1024*128;
  Result[] slots = new Result[MAPSIZE];
  byte[][] keys = new byte[MAPSIZE][];

  private int hashCode(byte[] a, int fromIndex, int length) {
    int result = 0;
    int end = fromIndex + length;
    for (int i = fromIndex; i < end; i++) {
      result = 31 * result + a[i];
    }
    return result;
  }
  // ...
}
```

### Jan 3, 2024 - ebarlas - Changed JVM to GraalVM CE 21.0.1

[https://github.com/gunnarmorling/1brc/pull/45](https://github.com/gunnarmorling/1brc/pull/45)


# Jan 3, 2024 - ddimtirov - use Epsilon GC, MemorySegment and Arena

Nice example of how to replace `MappedByteBuffer` with `MemorySegment`:

[https://github.com/gunnarmorling/1brc/pull/32](https://github.com/gunnarmorling/1brc/pull/32)

>switched to the foreign memory access preview API for another 10% speedup

```diff
-JAVA_OPTS="-XX:+UseParallelGC"
+# --enable-preview to use the new memory mapped segments
+# We don't allocate much, so just give it 1G heap and turn off GC; the AlwaysPreTouch was suggested by the ergonomics
+JAVA_OPTS="--enable-preview -Xms1g -Xmx1g -XX:+UnlockExperimentalVMOptions -XX:+UseEpsilonGC  -XX:+AlwaysPreTouch"
 time java $JAVA_OPTS --class-path target/average-1.0.0-SNAPSHOT.jar dev.morling.onebrc.CalculateAverage_ddimtirov

```

### Jan 4, 2024 - royvanrijn - Trying to fix the endian issue #68

https://github.com/gunnarmorling/1brc/pull/68/files

### Jan 4, 2024 - artsiomkorzun first submission

https://github.com/gunnarmorling/1brc/pull/83


### Jan 5, 2024 - yemreinci - Calculate the hashcode while reading the data

https://github.com/gunnarmorling/1brc/pull/86

>Improvements
>
>- Calculate the hashcode while reading the data, instead of later in the hashmap implementation. This is expected to increase instruction level parallelism as CPU can work on the math while waiting for data from memory/cache.
>- Convert the number parsing while loop into a few if statements. While loop with a switch-case inside is likely not so great for the branch predictor.


```java
int hash = 0;
while ((b = bb.get(currentPosition++)) != ';') {
    buffer[offset++] = b;
    hash = (hash << 5) - hash + b;
}
```

### Jan 5, 2024 3:02 GMT-3 - artsiomkorzun - AtomicInteger, AtomicReference

```java
        AtomicInteger counter = new AtomicInteger();
        AtomicReference<Aggregates> result = new AtomicReference<>();
        Aggregator[] aggregators = new Aggregator[PARALLELISM];
```


### Jan 6, 2024 6:55 AM GMT-3 - thomaswue - Unsafe, GraalVM Native Image - 9.625

[https://github.com/gunnarmorling/1brc/pull/70](https://github.com/gunnarmorling/1brc/pull/70)

[https://github.com/thomaswue/1brc/blob/c67abfcd469cbc00f89f5850cbf64a5407e21549/src/main/java/dev/morling/onebrc/CalculateAverage_thomaswue.java](https://github.com/thomaswue/1brc/blob/c67abfcd469cbc00f89f5850cbf64a5407e21549/src/main/java/dev/morling/onebrc/CalculateAverage_thomaswue.java)

>Update: Thanks to tuning from @mukel using sun.misc.Unsafe to directly access the mapped memory, it is now down to 1.28s (total CPU user time 32.2s) on my machine. Also, instead of PGO, this is now just using a single native image build run with tuning flags "-O3" and "-march=native".

>mmap the entire file, use Unsafe directly instead of ByteBuffer, avoid byte[] copies.
>These tricks give a ~30% speedup, over an already fast implementation.


```sh
source "$HOME/.sdkman/bin/sdkman-init.sh"
sdk use java 21.0.1-graal 1>&2
NATIVE_IMAGE_OPTS="--gc=epsilon -O3 -march=native --enable-preview"
native-image $NATIVE_IMAGE_OPTS -cp target/average-1.0.0-SNAPSHOT.jar -o image_calculateaverage_thomaswue dev.morling.onebrc.CalculateAverage_thomaswue
```

```java
    private static final Unsafe UNSAFE = initUnsafe();

    private static Unsafe initUnsafe() {
        try {
            Field theUnsafe = Unsafe.class.getDeclaredField("theUnsafe");
            theUnsafe.setAccessible(true);
            return (Unsafe) theUnsafe.get(Unsafe.class);
        }
        catch (NoSuchFieldException | IllegalAccessException e) {
            throw new RuntimeException(e);
        }
    }
```

### Jan 6, 2024 1:01 PM GMT-3 - merykitty - Vector API, magic branchless number parsing - 00:07.620

[https://github.com/gunnarmorling/1brc/pull/114](https://github.com/gunnarmorling/1brc/pull/114)

[https://github.com/merykitty/1brc/blob/aa5f311ae156a04369dc854b4cc091a63370cee3/src/main/java/dev/morling/onebrc/CalculateAverage_merykitty.java](https://github.com/merykitty/1brc/blob/aa5f311ae156a04369dc854b4cc091a63370cee3/src/main/java/dev/morling/onebrc/CalculateAverage_merykitty.java)


```java
    // The technique here is to align the key in both vectors so that we can do an
    // element-wise comparison and check if all characters match
    var nodeKey = ByteVector.fromArray(BYTE_SPECIES, node.data, BYTE_SPECIES.length() - localOffset);
    var eqMask = line.compare(VectorOperators.EQ, nodeKey).toLong();
    long validMask = (-1L >>> -semicolonPos) << localOffset;
    if ((eqMask & validMask) == validMask) {
        aggr = node.aggr;
        break;
    }
```

```java
    // Parse a number that may/may not contain a minus sign followed by a decimal with
    // 1 - 2 digits to the left and 1 digits to the right of the separator to a
    // fix-precision format. It returns the offset of the next line (presumably followed
    // the final digit and a '\n')
    private static long parseDataPoint(Aggregator aggr, MemorySegment data, long offset) {
        long word = data.get(JAVA_LONG_LT, offset);
        // The 4th binary digit of the ascii of a digit is 1 while
        // that of the '.' is 0. This finds the decimal separator
        // The value can be 12, 20, 28
        int decimalSepPos = Long.numberOfTrailingZeros(~word & 0x10101000);
        int shift = 28 - decimalSepPos;
        // signed is -1 if negative, 0 otherwise
        long signed = (~word << 59) >> 63;
        long designMask = ~(signed & 0xFF);
        // Align the number to a specific position and transform the ascii code
        // to actual digit value in each byte
        long digits = ((word & designMask) << shift) & 0x0F000F0F00L;

        // Now digits is in the form 0xUU00TTHH00 (UU: units digit, TT: tens digit, HH: hundreds digit)
        // 0xUU00TTHH00 * (100 * 0x1000000 + 10 * 0x10000 + 1) =
        // 0x000000UU00TTHH00 +
        // 0x00UU00TTHH000000 * 10 +
        // 0xUU00TTHH00000000 * 100
        // Now TT * 100 has 2 trailing zeroes and HH * 100 + TT * 10 + UU < 0x400
        // This results in our value lies in the bit 32 to 41 of this product
        // That was close :)
        long absValue = ((digits * 0x640a0001) >>> 32) & 0x3FF;
        long value = (absValue ^ signed) - signed;
        aggr.min = Math.min(value, aggr.min);
        aggr.max = Math.max(value, aggr.max);
        aggr.sum += value;
        aggr.count++;
        return offset + (decimalSepPos >>> 3) + 3;
    }
```

**TODO** Understand how this code works.

Interesting way to create a `HashMap`, using a static function
instead of the constructor `HashMap(int initialCapacity)`

[https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/HashMap.html#newHashMap(int)](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/HashMap.html#newHashMap(int))

```java
var res = HashMap.<String, Aggregator> newHashMap(processorCnt);
```

API note in the JDK:

[https://github.com/openjdk/jdk/blob/master/src/java.base/share/classes/java/util/HashMap.java#L462](https://github.com/openjdk/jdk/blob/master/src/java.base/share/classes/java/util/HashMap.java#L462)

```java
    /**
     * Constructs an empty {@code HashMap} with the specified initial
     * capacity and the default load factor (0.75).
     *
     * @apiNote
     * To create a {@code HashMap} with an initial capacity that accommodates
     * an expected number of mappings, use {@link #newHashMap(int) newHashMap}.
     *
     * @param  initialCapacity the initial capacity.
     * @throws IllegalArgumentException if the initial capacity is negative.
     */
    public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }
```

### Jan 7, 2024 - royvanrijn - Adding Unsafe and merykitty's branchless parser - 00:06.159

[https://github.com/gunnarmorling/1brc/pull/194](https://github.com/gunnarmorling/1brc/pull/194)


### Jan 7, 2024 - thomaswue - search and compare > 1 byte - 00:06.532

[https://github.com/gunnarmorling/1brc/pull/213](https://github.com/gunnarmorling/1brc/pull/213)

>This is some tuning over the initial submission now searching for the delimiter and comparing names more than 1 byte at a time. Should be ~30% faster than initial version.


### Jan 9, 2024 - mukel - Optimizing `findDelimiter`

[https://github.com/gunnarmorling/1brc/pull/263#discussion_r1446699670](https://github.com/gunnarmorling/1brc/pull/263#discussion_r1446699670)

```java
private static int findDelimiter(long word) {
    long input = word ^ 0x3B3B3B3B3B3B3B3BL;
    long tmp = (input - 0x0101010101010101L) & ~input & 0x8080808080808080L;
    return Long.numberOfTrailingZeros(tmp) >>> 3;
}
```

**TODO** Understand how this code works.

### Jan 10, 2024 - Server change - Solutions got faster

Old server: dedicated virtual cloud environment (Hetzner CCX33)

New server: [Hetzner AX161](https://www.hetzner.com/dedicated-rootserver/ax161) dedicated server (32 core AMD EPYC™ 7502P (Zen2), 128 GB RAM)

```txt
royvanrijn: 6.159 -> 3.604
thomaswue.: 6.532 -> 3.911
merykitty.: 7.620 -> 4.496
```

### Jan 11, 2024 - mtopolnik - focus on 10k (what was really asked for!)

[https://github.com/gunnarmorling/1brc/pull/246](https://github.com/gunnarmorling/1brc/pull/246)

>My primary focus was performance on the "new" dataset with 10,000 keys and names in the full range of 1-100 bytes. I find that the timing on that dataset degrades to 10.1 seconds.
>
>You can generate the dataset with create_measurements3.sh, and I encourage contestants to try it out and use as the optimization target. It's unquestionably a kick to see oneself at the top of the leaderboard, but it's more fun (and useful!) to design a solution that works well beyond the 416 station names of max length 26 chars.

### Jan 14, 2024 - Cliff Click submission =D - 4.741

[https://github.com/cliffclick/1brc/blob/262270c12b904e110bc468e132e578a2e1842ac3/src/main/java/dev/morling/onebrc/CalculateAverage_cliffclick.java](https://github.com/cliffclick/1brc/blob/262270c12b904e110bc468e132e578a2e1842ac3/src/main/java/dev/morling/onebrc/CalculateAverage_cliffclick.java)

Adding Unsafe:

[https://github.com/gunnarmorling/1brc/pull/185/commits/5f8fb7ce09d3f74ab5beb68522008ac2687e8c32](https://github.com/gunnarmorling/1brc/pull/185/commits/5f8fb7ce09d3f74ab5beb68522008ac2687e8c32)

### Jan 15, 2024 - jerrinot - Instruction-Level Parallelism (ILP) - 3.409

[https://github.com/gunnarmorling/1brc/pull/424](https://github.com/gunnarmorling/1brc/pull/424)

```txt
This initial submission is aiming to exploit instruction-level parallelism: Each thread pulls from multiple chunks in each iteration. This breaks nice sequential access, but a CPU gets independent streams so it can go bananas with out-of-order processing.

There are optimization opportunities left on the table, this is to give me some feeling of how it may perform on the testing box.

Credits:

    @mtopolnik - for many conversations we have had in the last 2 weeks!
    @merykitty - for the amazing branch-less parser
    @royvanrijn - for the hashing-as-you-go idea
```

```java
            chunkStartOffsets[0] = inputBase;
            chunkStartOffsets[chunkCount] = inputBase + length;

            Processor[] processors = new Processor[THREAD_COUNT];
            Thread[] threads = new Thread[THREAD_COUNT];

            for (int i = 0; i < THREAD_COUNT; i++) {
                long startA = chunkStartOffsets[i * chunkPerThread];
                long endA = chunkStartOffsets[i * chunkPerThread + 1];
                long startB = chunkStartOffsets[i * chunkPerThread + 1];
                long endB = chunkStartOffsets[i * chunkPerThread + 2];
                long startC = chunkStartOffsets[i * chunkPerThread + 2];
                long endC = chunkStartOffsets[i * chunkPerThread + 3];
                long startD = chunkStartOffsets[i * chunkPerThread + 3];
                long endD = chunkStartOffsets[i * chunkPerThread + 4];

                Processor processor = new Processor(startA, endA, startB, endB, startC, endC, startD, endD);
                processors[i] = processor;
                Thread thread = new Thread(processor);
                threads[i] = thread;
                thread.start();
            }
```

### Jan 16, 2024 - plevart: Look Mom No Unsafe! - 5.336

ffb09bf4bf0b41835b3340415be4f3c34565c126

Final version - 4.676

[https://github.com/plevart/1brc/blob/7777e9ee4e2302fdab3671d2aae401f05e0394d5/src/main/java/dev/morling/onebrc/CalculateAverage_plevart.java](https://github.com/plevart/1brc/blob/7777e9ee4e2302fdab3671d2aae401f05e0394d5/src/main/java/dev/morling/onebrc/CalculateAverage_plevart.java)

### Jan 31, 2024 - Shiplëv submission =D - No Unsafe! - 4.884

Beautiful solution with great comments!!!

[https://github.com/shipilev/1brc/blob/c2bb7d2621a070c5e4f0c592aece0f55fa50bee7/src/main/java/dev/morling/onebrc/CalculateAverage_shipilev.java](https://github.com/shipilev/1brc/blob/c2bb7d2621a070c5e4f0c592aece0f55fa50bee7/src/main/java/dev/morling/onebrc/CalculateAverage_shipilev.java)


### Jan 31, 2024 - Serkan ÖZAL - Fastest Solution running on the JVM!!! - 21.0.1-open - 1.880

Amazing code using the Vector API.

[https://github.com/gunnarmorling/1brc/pull/679](https://github.com/gunnarmorling/1brc/pull/679)

[https://github.com/serkan-ozal/1brc/blob/c74cddbe8d6139af9833bb9c2546da44b8f7a065/src/main/java/dev/morling/onebrc/CalculateAverage_serkan_ozal.java](https://github.com/serkan-ozal/1brc/blob/c74cddbe8d6139af9833bb9c2546da44b8f7a065/src/main/java/dev/morling/onebrc/CalculateAverage_serkan_ozal.java)

# First Use of Important Concepts


### MappedByteBuffer

```
$ git log -S"MappedBy" --reverse
commit 6b13d52b676d4ccfe7d766ac46f81abee8225816
Author: Hampus Ram
Date:   Tue Jan 2 14:04:29 2024 +0100

    Implementation using memory mapped file
```

### Vector API

```
$ git log -S"jdk.incubator.vector" --reverse
commit 17218485709af71ebd7cb7c2c47f3abc262f5a2d
Author: Aurelian Tutuianu
Date:   Tue Jan 2 21:14:32 2024 +0200

     - implementation by padreati
```

### AtomicReference

```
git log -S"AtomicReference" --reverse
commit 0ba5cf33d4a4ce290dbe4fa5e4e4c37b424c8675
Author: Richard Startin <richardstartin@apache.org>
Date:   Wed Jan 3 18:15:06 2024 +0000

    richardstartin submission

+import java.util.concurrent.atomic.AtomicIntegerFieldUpdater;
+import java.util.concurrent.atomic.AtomicReferenceArray;
+import java.util.concurrent.atomic.AtomicReferenceFieldUpdater;
```

```
$ git log -S"AtomicReference;" --reverse
commit cec579b506d448afd2365e0197ed5b98bd0d6a5e
Author: Artsiom Korzun <akorzun@deltixlab.com>
Date:   Fri Jan 5 12:15:54 2024 +0100

    improved artsiomkorzun solution
```

He uses it to merge the results!

```java
        AtomicInteger counter = new AtomicInteger();
        AtomicReference<Aggregates> result = new AtomicReference<>();
        int parallelism = Runtime.getRuntime().availableProcessors();
        Aggregator[] aggregators = new Aggregator[parallelism];

        for (int i = 0; i < aggregators.length; i++) {
            aggregators[i] = new Aggregator(counter, result, fileAddress, fileSize, segmentCount);
            aggregators[i].start();
        }

        // ...

            while (!result.compareAndSet(null, aggregates)) {
                Aggregates rights = result.getAndSet(null);

                if (rights != null) {
                    aggregates.merge(rights);
                }
            }
```

That's a great idea, because when joining on threads sequentially,
the first thread you're waiting might be the last one to finish ...

**TODO**: can't that code get in an infinite loop?

>You can use AtomicReference when applying optimistic locks. You have a shared object and you want to change it from more than 1 thread.
>
>    You can create a copy of the shared object
>    Modify the shared object
>    You need to check that the shared object is still the same as before - if yes, then update with the reference of the modified copy.
>
>As other thread might have modified it and/can modify between these 2 steps. You need to do it in an atomic operation. this is where AtomicReference can help

HamoriZ answered Jun 12, 2015 at 12:30
https://stackoverflow.com/a/30803137/339561


### MemorySegment

```
$ git log -S"MemorySegm" --reverse
commit d73457872f0b9990ab6258d0a96d3edad3b24b1a
Author: Dimitar Dimitrov
Date:   Thu Jan 4 04:22:39 2024 +0900

    ddimtirov - switched to the foreign memory access preview API for another 10% speedup
```

### Epsilon GC

```
$ git log -S"Epsilon" --reverse
commit d73457872f0b9990ab6258d0a96d3edad3b24b1a
Author: Dimitar Dimitrov
Date:   Thu Jan 4 04:22:39 2024 +0900
```

### Unsafe

```
$ git log -S"Unsafe" --reverse
commit a53aa2e6fdbcc74e0c6a9a308f0a8c3507b0e06a
Author: Thomas Wuerthinger
Date:   Sat Jan 6 10:55:07 2024 +0100
```

### Instruction-Level Parallelism (ILP)

*I didn't look at all other solutions, so if there was a previous one, let me know =)*

```
commit dbdd89a84779761ca092e5aaeb6f6e92394a422d
Author: Jaromir Hamala
Date:   Mon Jan 15 18:55:22 2024 +0100

    jerrinot's initial submission (#424)
    
    * initial version
    
    let's exploit that superscalar beauty!
```

# Conclusion

There were a few changes that really changed the game.<br>
The others were minor improvements and mostly people combining code
from other solutions.

Here are the important changes:
  - Process lines in parallel (one line change =));
  - Memory map the file, split in chunks and process them in parallel;
  - Great hash table;
  - Create UTF-8 String for each city after the main loop;
  - SWAR;
  - Vector API;
  - Unsafe;
  - ILP.

That's it for the timeline. If I missed something, let me know here: [https://twitter.com/tivrfoa/status/1754607677362106552](https://twitter.com/tivrfoa/status/1754607677362106552)

In the next post I'll explore the performance impact of using `Unsafe`.

# References

[https://github.com/gunnarmorling/1brc](https://github.com/gunnarmorling/1brc)

[https://en.wikipedia.org/wiki/SWAR](https://en.wikipedia.org/wiki/SWAR)

[Multiple byte processing with full-word instructions - 1975](https://dl.acm.org/doi/pdf/10.1145/360933.360994)

[GENERAL-PURPOSE SIMD WITHIN A REGISTER:PARALLEL PROCESSING ON CONSUMER MICROPROCESSORS - 2003](http://aggregate.org/SWAR/Dis/dissertation.pdf)

[Finding Bytes in Arrays - 2019](https://richardstartin.github.io/posts/finding-bytes)
