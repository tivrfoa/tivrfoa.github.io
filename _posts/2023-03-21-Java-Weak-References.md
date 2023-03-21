---
layout: post
title:  "Java Object Reachability - SoftReference, WeakReference, PhantomReference"
date:   2023-03-21 08:10:00 -0300
categories: Java Object Reachability
---

Nice summary from: [Caches, Canonicalizing Mappings, and Pre-Mortem Cleanup](https://www.artima.com/insidejvm/ed2/gc17.html), by Bill Venners

<div class="quote1">
The garbage collector treats soft, weak, and phantom objects differently because each is intended to provide a different kind of service to the program. Soft references enable you to create in-memory caches that are sensitive to the overall memory needs of the program. Weak references enable you to create canonicalizing mappings, such as a hash table whose keys and values will be removed from the map if they become otherwise unreferenced by the program. Phantom references enable you to establish more flexible pre-mortem cleanup policies than are possible with finalizers. 
</div>

# Reachability

From: [https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/ref/package-summary.html#reachability](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/ref/package-summary.html#reachability)

*start-ref*

Going from strongest to weakest, the different levels of reachability reflect the life cycle of an object. They are operationally defined as follows:

- An object is strongly reachable if it can be reached by some thread without traversing any reference objects. A newly-created object is strongly reachable by the thread that created it.
- An object is softly reachable if it is not strongly reachable but can be reached by traversing a soft reference.
- An object is weakly reachable if it is neither strongly nor softly reachable but can be reached by traversing a weak reference. When the weak references to a weakly-reachable object are cleared, the object becomes eligible for finalization.
- An object is phantom reachable if it is neither strongly, softly, nor weakly reachable, it has been finalized, and some phantom reference refers to it.
- Finally, an object is unreachable, and therefore eligible for reclamation, when it is not reachable in any of the above ways.

`PhantomReference<T>` Phantom reference objects, which are enqueued after the collector determines that their referents may otherwise be reclaimed.

`SoftReference<T>` Soft reference objects, which are cleared at the discretion of the garbage collector in response to memory demand.

`WeakReference<T>` Weak reference objects, which do not prevent their referents from being made finalizable, finalized, and then reclaimed.


*end-ref*

From: [What's the difference between SoftReference and WeakReference in Java?](https://stackoverflow.com/a/299702/339561)

<div class="quote1">
From Understanding Weak References, by Ethan Nicholas:

<div class="quote2">
<h3>Weak references</h3>

<p>A weak reference, simply put, is a reference that isn't strong enough to force an object to remain in memory. Weak references allow you to leverage the garbage collector's ability to determine reachability for you, so you don't have to do it yourself. You create a weak reference like this:</p>

<p>
WeakReference weakWidget = new WeakReference(widget);
</p>


<p>and then elsewhere in the code you can use weakWidget.get() to get the actual Widget object. Of course the weak reference isn't strong enough to prevent garbage collection, so you may find (if there are no strong references to the widget) that weakWidget.get() suddenly starts returning null.</p>

...

<h3>Soft references</h3>

<p>A soft reference is exactly like a weak reference, except that it is less eager to throw away the object to which it refers. An object which is only weakly reachable (the strongest references to it are WeakReferences) will be discarded at the next garbage collection cycle, but an object which is softly reachable will generally stick around for a while.</p>

<p>SoftReferences aren't required to behave any differently than WeakReferences, but in practice softly reachable objects are generally retained as long as memory is in plentiful supply. This makes them an excellent foundation for a cache, such as the image cache described above, since you can let the garbage collector worry about both how reachable the objects are (a strongly reachable object will never be removed from the cache) and how badly it needs the memory they are consuming.</p>
</div>

<p>And Peter Kessler added in a comment:</p>

<div class="quote2">
The Sun JRE does treat SoftReferences differently from WeakReferences. We attempt to hold on to object referenced by a SoftReference if there isn't pressure on the available memory. One detail: the policy for the "-client" and "-server" JRE's are different: the -client JRE tries to keep your footprint small by preferring to clear SoftReferences rather than expand the heap, whereas the -server JRE tries to keep your performance high by preferring to expand the heap (if possible) rather than clear SoftReferences. One size does not fit all.
</div>

</div>


# SoftReference

>Soft reference objects, which are cleared at the discretion of the garbage collector in response to memory demand. Soft references are most often used to implement memory-sensitive caches.
>
>[https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/ref/SoftReference.html](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/ref/SoftReference.html)

Interesting example of a use case:
[https://stackoverflow.com/a/299663/339561](https://stackoverflow.com/a/299663/339561)

<div class="quote1">
Typical use case example is keeping a parsed form of a contents from a file. You'd implement a system where you'd load a file, parse it, and keep a SoftReference to the root object of the parsed representation. Next time you need the file, you'll try to retrieve it through the SoftReference. If you can retrieve it, you spared yourself another load/parse, and if the GC cleared it in the meantime, you reload it. That way, you utilize free memory for performance optimization, but don't risk an OOME.
</div>

# WeakReference

>Weak reference objects, which do not prevent their referents from being made finalizable, finalized, and then reclaimed. Weak references are most often used to implement canonicalizing mappings.
>
>[https://docs.oracle.com/en/java/javase/17/docs/api//java.base/java/lang/ref/WeakReference.html](https://docs.oracle.com/en/java/javase/17/docs/api//java.base/java/lang/ref/WeakReference.html)

### Canonicalizing Mappings

Canonical:

>Mathematics. (of an equation, coordinate, etc.) in simplest or standard form.
>
>[https://www.dictionary.com/browse/canonical](https://www.dictionary.com/browse/canonical)

>In computer science, canonicalization (sometimes standardization or normalization) is a process for converting data that has more than one possible representation into a "standard", "normal", or canonical form.
>
>[https://en.wikipedia.org/wiki/Canonicalization](https://en.wikipedia.org/wiki/Canonicalization)

>Canonicalization is the process of converting data that involves more than one representation into a standard approved format.
>
>[https://www.techopedia.com/definition/15646/canonicalization](https://www.techopedia.com/definition/15646/canonicalization)


Example:

```java
import java.lang.ref.WeakReference;
import java.util.*;

public class WeakTest1 {

    private static final int LEN = 10;

    private static int countNotNull(Set<WeakReference<String>> set) {
        int notNullCount = 0;
        for (var wr : set) {
            notNullCount += wr.get() == null ? 0 : 1;
        }
        return notNullCount;
    }

    public static void main(String[] args) throws Exception {
        Set<WeakReference<String>> set = new HashSet<>();

        String IamTwoStrong = "2";
        for (int i = 0; i < LEN; i++) {
            if (i == 2)
                set.add(new WeakReference(IamTwoStrong));
            else
                set.add(new WeakReference("" + i));
        }

        assert set.size() == LEN;
        assert countNotNull(set) == LEN;
        System.gc();
        // Size does not change, but weak references are now null
        assert set.size() == LEN;

        // should be 1, because of "IamTwoStrong" strong reference
        assert countNotNull(set) == 1;
    }
}
```

# PhantomReferences


Use cases, from: [Understanding Weak References](http://euler.mat.uson.mx/~havillam/pa/Common/Understanding-Weak-References.pdf) 

<div class="quote1">
<p>What good are PhantomReferences? I'm only aware of two serious cases for them: first,
they allow you to determine exactly when an object was removed from memory. They
are in fact the only way to determine that. This isn't generally that useful, but might
come in handy in certain very specific circumstances like manipulating large images: if
you know for sure that an image should be garbage collected, you can wait until it
actually is before attempting to load the next image, and therefore make the
dreaded OutOfMemoryError less likely.</p>

<p>Second, PhantomReferences avoid a fundamental problem with finalization: finalize()
methods can "resurrect" objects by creating new strong references to them. So what,
you say? Well, the problem is that an object which overrides finalize() must now be
determined to be garbage in at least two separate garbage collection cycles in order to
be collected. When the first cycle determines that it is garbage, it becomes eligible for
finalization. Because of the (slim, but unfortunately real) possibility that the object was
"resurrected" during finalization, the garbage collector has to run again before the object
can actually be removed. And because finalization might not have happened in a timely
fashion, an arbitrary number of garbage collection cycles might have happened while
the object was waiting for finalization. This can mean serious delays in actually cleaning
up garbage objects, and is why you can get OutOfMemoryErrors even when most of the
heap is garbage.</p>
</div>

# References

[Understanding Weak References](http://euler.mat.uson.mx/~havillam/pa/Common/Understanding-Weak-References.pdf)

[Caches, Canonicalizing Mappings, and Pre-Mortem Cleanup](https://www.artima.com/insidejvm/ed2/gc17.html)

[https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/ref/SoftReference.html](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/ref/SoftReference.html)

[https://docs.oracle.com/en/java/javase/17/docs/api//java.base/java/lang/ref/WeakReference.html](https://docs.oracle.com/en/java/javase/17/docs/api//java.base/java/lang/ref/WeakReference.html)

[How to Use Java SoftReferences to Build an Efficient Cache](https://blog.shiftleft.io/understanding-jvm-soft-references-for-great-good-and-building-a-cache-244a4f7bb85d)

[HotSpot Virtual Machine Garbage Collection Tuning Guide - Other Considerations](https://docs.oracle.com/en/java/javase/17/gctuning/other-considerations.html)
