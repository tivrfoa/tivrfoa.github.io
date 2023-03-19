---
layout: post
title:  "Java CompletableFuture Part 1"
date:   2023-03-19 07:30:00 -0300
categories: Java CompletableFuture Asynchronous Programming
---

# Java CompletableFuture Part 1

A look at some of the classes and methods used by CompletableFuture.

I'm just trying to learn how it works under the hood.

I thought it would use an event loop like Node.js, but it seems
it uses a thread pool.

```java
    // VarHandle mechanics
    private static final VarHandle RESULT;
    private static final VarHandle STACK;
    private static final VarHandle NEXT;
    static {
        try {
            MethodHandles.Lookup l = MethodHandles.lookup();
            RESULT = l.findVarHandle(CompletableFuture.class, "result", Object.class);
            STACK = l.findVarHandle(CompletableFuture.class, "stack", Completion.class);
            NEXT = l.findVarHandle(Completion.class, "next", Completion.class);
        } catch (ReflectiveOperationException e) {
            throw new ExceptionInInitializerError(e);
        }

        // Reduce the risk of rare disastrous classloading in first call to
        // LockSupport.park: https://bugs.openjdk.org/browse/JDK-8074773
        Class<?> ensureLoaded = LockSupport.class;
    }
```

[https://docs.oracle.com/javase/9/docs/api/java/lang/invoke/VarHandle.html](https://docs.oracle.com/javase/9/docs/api/java/lang/invoke/VarHandle.html)

[https://docs.oracle.com/javase/9/docs/api/java/lang/invoke/MethodHandles.Lookup.html#findVarHandle-java.lang.Class-java.lang.String-java.lang.Class-](https://docs.oracle.com/javase/9/docs/api/java/lang/invoke/MethodHandles.Lookup.html#findVarHandle-java.lang.Class-java.lang.String-java.lang.Class-)

```java
public VarHandle findVarHandle​(Class<?> recv,
                               String name,
                               Class<?> type)
                        throws NoSuchFieldException,
                               IllegalAccessException
```

>Produces a VarHandle giving access to a non-static field name of type type declared in a class of type recv. The VarHandle's variable type is type and it has one coordinate type, recv.

[https://docs.oracle.com/javase/9/docs/api/java/lang/invoke/VarHandle.html#compareAndSet-java.lang.Object...-](https://docs.oracle.com/javase/9/docs/api/java/lang/invoke/VarHandle.html#compareAndSet-java.lang.Object...-)

```java
public final boolean compareAndSet​(Object... args)
```

>Atomically sets the value of a variable to the newValue with the memory semantics of setVolatile(java.lang.Object...) if the variable's current value, referred to as the witness value, == the expectedValue, as accessed with the memory semantics of getVolatile(java.lang.Object...).



```java
    public T get() throws InterruptedException, ExecutionException {
        Object r;
        if ((r = result) == null)
            r = waitingGet(true);
        return (T) reportGet(r);
    }
    
    /**
     * Returns raw result after waiting, or null if interruptible and
     * interrupted.
     */
    private Object waitingGet(boolean interruptible) {
        if (interruptible && Thread.interrupted())
            return null;
        Signaller q = null;
        boolean queued = false;
        Object r;
        while ((r = result) == null) { // ? where is result set ?
            if (q == null) {
                q = new Signaller(interruptible, 0L, 0L);
                if (Thread.currentThread() instanceof ForkJoinWorkerThread)
                    ForkJoinPool.helpAsyncBlocker(defaultExecutor(), q);
            }
            else if (!queued)
                queued = tryPushStack(q);
            else if (interruptible && q.interrupted) {
                q.thread = null;
                cleanStack();
                return null;
            }
            else {
                try {
                    ForkJoinPool.managedBlock(q);
                } catch (InterruptedException ie) { // currently cannot happen
                    q.interrupted = true;
                }
            }
        }
        if (q != null) {
            q.thread = null;
            if (q.interrupted)
                Thread.currentThread().interrupt();
        }
        postComplete();
        return r;
    }
    
    /**
     * Pops and tries to trigger all reachable dependents.  Call only
     * when known to be done.
     */
    final void postComplete() {
        /*
         * On each step, variable f holds current dependents to pop
         * and run.  It is extended along only one path at a time,
         * pushing others to avoid unbounded recursion.
         */
        CompletableFuture<?> f = this; Completion h;
        while ((h = f.stack) != null ||
               (f != this && (h = (f = this).stack) != null)) {
            CompletableFuture<?> d; Completion t;
            if (STACK.compareAndSet(f, h, t = h.next)) {
                if (t != null) {
                    if (f != this) {
                        pushStack(h);
                        continue;
                    }
                    NEXT.compareAndSet(h, t, null); // try to detach
                }
                f = (d = h.tryFire(NESTED)) == null ? this : d;
            }
        }
    }
```

`result` is being changed in one of these:

- internalComplete
- completeValue



```Java
```

[https://github.com/openjdk/jdk/blob/master/src/java.base/share/classes/java/util/concurrent/ForkJoinPool.java](https://github.com/openjdk/jdk/blob/master/src/java.base/share/classes/java/util/concurrent/ForkJoinPool.java)

```java

    /**
     * If the given executor is a ForkJoinPool, poll and execute
     * AsynchronousCompletionTasks from worker's queue until none are
     * available or blocker is released.
     */
    static void helpAsyncBlocker(Executor e, ManagedBlocker blocker) {
        WorkQueue w = null; Thread t; ForkJoinWorkerThread wt;
        if ((t = Thread.currentThread()) instanceof ForkJoinWorkerThread) {
            if ((wt = (ForkJoinWorkerThread)t).pool == e)
                w = wt.workQueue;
        }
        else if (e instanceof ForkJoinPool)
            w = ((ForkJoinPool)e).externalQueue();
        if (w != null)
            w.helpAsyncBlocker(blocker);
    }
    
        final void helpAsyncBlocker(ManagedBlocker blocker) {
            if (blocker != null) {
                for (;;) {
                    int b = base, cap; ForkJoinTask<?>[] a;
                    if ((a = array) == null || (cap = a.length) <= 0 || b == top)
                        break;
                    int k = (cap - 1) & b, nb = b + 1, nk = (cap - 1) & nb;
                    ForkJoinTask<?> t = a[k];
                    U.loadFence();                     // for re-reads
                    if (base != b)
                        ;
                    else if (blocker.isReleasable())
                        break;
                    else if (a[k] != t)
                        ;
                    else if (t != null) {
                        if (!(t instanceof CompletableFuture
                              .AsynchronousCompletionTask))
                            break;
                        else if (casSlotToNull(a, k, t)) {
                            base = nb;
                            U.storeStoreFence();
                            t.doExec();
                        }
                    }
                    else if (a[nk] == null)
                        break;
                }
            }
        }

```


# Debugging

ForkJoinPool has an array of WorkQueue;
WorkQueue has an array of ForkJoinTask;
CompletableFuture$AsyncSupply is a subtype of ForkJoinTask;
CompletableFuture$AsyncSupply has:
  - dep: CompletableFuture
  - fn: Supplier

CompletableFuture.get
	CompletableFuture.reportGet

```java
    static <U> CompletableFuture<U> asyncSupplyStage(Executor e,
                                                     Supplier<U> f) {
        if (f == null) throw new NullPointerException();
        CompletableFuture<U> d = new CompletableFuture<U>();
        e.execute(new AsyncSupply<U>(d, f));
        return d;
    }
    
    /** Returns true if successfully pushed c onto stack. */
    final boolean tryPushStack(Completion c) {
        Completion h = stack;
        NEXT.set(c, h);         // CAS piggyback
        return STACK.compareAndSet(this, h, c);
    }

        public boolean isReleasable() {
            if (Thread.interrupted())
                interrupted = true;
            return ((interrupted && interruptible) ||
                    (deadline != 0L &&
                     (nanos <= 0L ||
                      (nanos = deadline - System.nanoTime()) <= 0L)) ||
                    thread == null);
        }
        public boolean block() {
            while (!isReleasable()) {
                if (deadline == 0L)
                    LockSupport.park(this);
                else
                    LockSupport.parkNanos(this, nanos);
            }
            return true;
        }
```

LockSupport

```java
    /**
     * Disables the current thread for thread scheduling purposes unless the
     * permit is available.
     *
     * <p>If the permit is available then it is consumed and the call returns
     * immediately; otherwise
     * the current thread becomes disabled for thread scheduling
     * purposes and lies dormant until one of three things happens:
     *
     * <ul>
     * <li>Some other thread invokes {@link #unpark unpark} with the
     * current thread as the target; or
     *
     * <li>Some other thread {@linkplain Thread#interrupt interrupts}
     * the current thread; or
     *
     * <li>The call spuriously (that is, for no reason) returns.
     * </ul>
     *
     * <p>This method does <em>not</em> report which of these caused the
     * method to return. Callers should re-check the conditions which caused
     * the thread to park in the first place. Callers may also determine,
     * for example, the interrupt status of the thread upon return.
     *
     * @param blocker the synchronization object responsible for this
     *        thread parking
     * @since 1.6
     */
    public static void park(Object blocker) {
        Thread t = Thread.currentThread();
        setBlocker(t, blocker);
        U.park(false, 0L);
        setBlocker(t, null);
    }
    
    private static void setBlocker(Thread t, Object arg) {
        U.putReferenceOpaque(t, PARKBLOCKER, arg);
    }
```

Unsafe

```java
    /** Opaque version of {@link #putReferenceVolatile(Object, long, Object)} */
    @IntrinsicCandidate
    public final void putReferenceOpaque(Object o, long offset, Object x) {
        putReferenceVolatile(o, offset, x);
    }
    
    /**
     * Stores a reference value into a given Java variable, with
     * volatile store semantics. Otherwise identical to {@link #putReference(Object, long, Object)}
     */
    @IntrinsicCandidate
    public native void putReferenceVolatile(Object o, long offset, Object x);
```

ForkJoinPool

```java
    public void execute(Runnable task) {
        externalSubmit((task instanceof ForkJoinTask<?>)
                       ? (ForkJoinTask<Void>) task // avoid re-wrap
                       : new ForkJoinTask.RunnableExecuteAction(task));
    }
    
    /**
     * Pushes a possibly-external submission.
     */
    private <T> ForkJoinTask<T> externalSubmit(ForkJoinTask<T> task) {
        Thread t; ForkJoinWorkerThread wt; WorkQueue q;
        if (task == null)
            throw new NullPointerException();
        if (((t = Thread.currentThread()) instanceof ForkJoinWorkerThread) &&
            (q = (wt = (ForkJoinWorkerThread)t).workQueue) != null &&
            wt.pool == this)
            q.push(task, this);
        else
            externalPush(task);
        return task;
    }
    
    /**
     * Adds the given task to an external submission queue, or throws
     * exception if shutdown or terminating.
     *
     * @param task the task. Caller must ensure non-null.
     */
    final void externalPush(ForkJoinTask<?> task) {
        WorkQueue q;
        if ((q = submissionQueue()) == null)
            throw new RejectedExecutionException(); // shutdown or disabled
        else if (q.lockedPush(task))
            signalWork();
    }

    /**
     * Finds and locks a WorkQueue for an external submitter, or
     * returns null if shutdown or terminating.
     */
    final WorkQueue submissionQueue() {
        int r;
        if ((r = ThreadLocalRandom.getProbe()) == 0) {
            ThreadLocalRandom.localInit();           // initialize caller's probe
            r = ThreadLocalRandom.getProbe();
        }
        for (int id = r << 1;;) {                    // even indices only
            int md = mode, n, i; WorkQueue q; ReentrantLock lock;
            WorkQueue[] qs = queues; // length = 16
            if ((md & SHUTDOWN) != 0 || qs == null || (n = qs.length) <= 0)
                return null;
            else if ((q = qs[i = (n - 1) & id]) == null) {
                if ((lock = registrationLock) != null) {
                    WorkQueue w = new WorkQueue(id | SRC);
                    lock.lock();                    // install under lock
                    if (qs[i] == null)
                        qs[i] = w;                  // else lost race; discard
                    lock.unlock();
                }
            }
            else if (!q.tryLock())                  // move and restart
                id = (r = ThreadLocalRandom.advanceProbe(r)) << 1;
            else
                return q;
        }
    }
    
        /**
         * Constructor used for external queues.
         */
        WorkQueue(int config) {
            array = new ForkJoinTask<?>[INITIAL_QUEUE_CAPACITY];
            this.config = config;
            owner = null;
            phase = -1;
        }
        
    /**
     * Tries to construct and start one worker. Assumes that total
     * count has already been incremented as a reservation.  Invokes
     * deregisterWorker on any failure.
     *
     * @return true if successful
     */
    private boolean createWorker() {
        ForkJoinWorkerThreadFactory fac = factory;
        Throwable ex = null;
        ForkJoinWorkerThread wt = null;
        try {
            if (fac != null && (wt = fac.newThread(this)) != null) {
                wt.start();
                return true;
            }
        } catch (Throwable rex) {
            ex = rex;
        }
        deregisterWorker(wt, ex);
        return false;
    }
```

It's not only Rust that has `unsafe` ... =)

ThreadLocalRandom

```java
    // Unsafe mechanics
    private static final Unsafe U = Unsafe.getUnsafe();
    private static final long PROBE
        = U.objectFieldOffset(Thread.class, "threadLocalRandomProbe");
    
    /**
     * Returns the probe value for the current thread without forcing
     * initialization. Note that invoking ThreadLocalRandom.current()
     * can be used to force initialization on zero return.
     */
    static final int getProbe() {
        return U.getInt(Thread.currentThread(), PROBE);
    }
    
    /**
     * Initialize Thread fields for the current thread.  Called only
     * when Thread.threadLocalRandomProbe is zero, indicating that a
     * thread local seed value needs to be generated. Note that even
     * though the initialization is purely thread-local, we need to
     * rely on (static) atomic generators to initialize the values.
     */
    static final void localInit() {
        int p = probeGenerator.addAndGet(PROBE_INCREMENT);
        int probe = (p == 0) ? 1 : p; // skip 0
        long seed = RandomSupport.mixMurmur64(seeder.getAndAdd(SEEDER_INCREMENT));
        Thread t = Thread.currentThread();
        U.putLong(t, SEED, seed);
        U.putInt(t, PROBE, probe);
    }
```

ForkJoinWorkerThread

```java
    /**
     * Full nonpublic constructor.
     */
    ForkJoinWorkerThread(ThreadGroup group, ForkJoinPool pool,
                         boolean useSystemClassLoader, boolean isInnocuous) {
        super(group, null, pool.nextWorkerThreadName(), 0L);
        UncaughtExceptionHandler handler = (this.pool = pool).ueh;
        this.workQueue = new ForkJoinPool.WorkQueue(this, isInnocuous);
        super.setDaemon(true);
        if (handler != null)
            super.setUncaughtExceptionHandler(handler);
        if (useSystemClassLoader)
            super.setContextClassLoader(ClassLoader.getSystemClassLoader());
    }
```

ForkJoinTask

```java

    /**
     * Unless done, calls exec and records status if completed, but
     * doesn't wait for completion otherwise.
     *
     * @return status on exit from this method
     */
    final int doExec() {
        int s; boolean completed;
        if ((s = status) >= 0) {
            try {
                completed = exec();
            } catch (Throwable rex) {
                s = trySetException(rex);
                completed = false;
            }
            if (completed)
                s = setDone();
        }
        return s;
    }
    
    /**
     * Immediately performs the base action of this task and returns
     * true if, upon return from this method, this task is guaranteed
     * to have completed. This method may return false otherwise, to
     * indicate that this task is not necessarily complete (or is not
     * known to be complete), for example in asynchronous actions that
     * require explicit invocations of completion methods. This method
     * may also throw an (unchecked) exception to indicate abnormal
     * exit. This method is designed to support extensions, and should
     * not in general be called otherwise.
     *
     * @return {@code true} if this task is known to have completed normally
     */
    protected abstract boolean exec();
```

Thread

```java
    @IntrinsicCandidate
    public static native Thread currentThread();
    
    /**
     * Tests if this thread is alive. A thread is alive if it has
     * been started and has not yet died.
     *
     * @return  {@code true} if this thread is alive;
     *          {@code false} otherwise.
     */
    public final native boolean isAlive();
    
    private native void start0();
```

ReentrantLock

```java
    public void lock() {
        sync.lock();
    }
    
        @ReservedStackAccess
        final void lock() {
            if (!initialTryLock())
                acquire(1);
        }
```

VarHandleGuards

```java
    @ForceInline
    @LambdaForm.Compiled
    @Hidden
    static final boolean guard_LII_Z(VarHandle handle, Object arg0, int arg1, int arg2, VarHandle.AccessDescriptor ad) throws Throwable {
        handle.checkExactAccessMode(ad);
        if (handle.isDirect() && handle.vform.methodType_table[ad.type] == ad.symbolicMethodTypeErased) {
            return (boolean) MethodHandle.linkToStatic(handle, arg0, arg1, arg2, handle.vform.getMemberName(ad.mode));
        } else {
            MethodHandle mh = handle.getMethodHandle(ad.mode);
            return (boolean) mh.asType(ad.symbolicMethodTypeInvoker).invokeBasic(handle.asDirect(), arg0, arg1, arg2);
        }
    }
```

# Cancelling a task:

[https://www.reddit.com/r/java/comments/rsa40l/comment/hqm46n6/](https://www.reddit.com/r/java/comments/rsa40l/comment/hqm46n6/?)

```txt
Glass__Editor
·

Yes, I mostly like it.

There are a lot of methods to read through, but the only problem I've really had using it is interrupting the thread completing a task. The cancel method wont interrupt the thread executing the task (even if you pass true for mayInterruptIfRunning), so when I wanted to do that I instead submitted the task to an ExecutorService and stored the Future it returned in a CompletableFuture subclass with an overridden cancel method (and completed it at the end of the task).

Other than that it's pretty useful for coordinating threads and I use it quite a bit.
```

# References

[Java Completabe Future](https://github.com/openjdk/jdk/blob/master/src/java.base/share/classes/java/util/concurrent/CompletableFuture.java)

[The Design and Engineering of
Concurrency Libraries - Doug Lea](http://www.sti.uniurb.it/events/sfm15mp/slides/lea.2.pdf)

[http://concurrencyfreaks.blogspot.com/2014/11/doug-lea-at-splash-on-parallel.html](http://concurrencyfreaks.blogspot.com/2014/11/doug-lea-at-splash-on-parallel.html)

[JEP 266: More Concurrency Updates](https://openjdk.org/jeps/266)

[CompletableFuture updates and CompletionStage](https://mail.openjdk.org/pipermail/core-libs-dev/2013-July/018859.html)

[State of Loom: Part 2](https://cr.openjdk.org/~rpressler/loom/loom/sol1_part2.html)

[https://www.reddit.com/r/java/comments/rsa40l/does_anybody_enjoy_completablefuture/](https://www.reddit.com/r/java/comments/rsa40l/does_anybody_enjoy_completablefuture/)

[https://stackoverflow.com/questions/66281348/why-does-cs-async-await-not-need-an-event-loop](https://stackoverflow.com/questions/66281348/why-does-cs-async-await-not-need-an-event-loop)

[https://nodejs.org/en/about](https://nodejs.org/en/about)

[https://nodejs.org/en/docs/guides/blocking-vs-non-blocking](https://nodejs.org/en/docs/guides/blocking-vs-non-blocking)

