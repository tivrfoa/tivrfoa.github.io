---
layout: post
title:  "Go vs Rust Concurrency"
date:   2025-07-06 08:10:00 -0300
categories: go rust concurrency
---

**Prompt**

>"Go's memory model, for example, has no concept of isolating memory between co-routines or threads, nor of controlling mutability.
Everything can always race on the same global mutable heap."
>
>  Graydon Hoare https://www.infoq.com/news/2012/08/Interview-Rust/


Create a Go program where this behavior can lead to bugs, then show a Rust equivalent where it is safe, because Rust prevents that kind of bug.

- [Gemini](#gemini)
- [Grok](#grok)
- [ChatGPT](#chatgpt)

# Gemini

Of course. This is a fantastic question that gets to the very heart of the philosophical differences between Go and Rust regarding concurrency and memory safety.

I will create two programs to illustrate this. The first is a Go program that demonstrates a classic race condition, a bug that can be subtle and hard to track down. The second is the equivalent program in Rust, which will not even compile without the proper safety mechanisms, effectively preventing the bug from ever existing.

**The Go Example: Unchecked Shared Memory**

Here is a Go program that starts several concurrent "goroutines." Each goroutine attempts to increment a shared counter 1,000 times. Because Go's memory model allows any goroutine to access the shared counter variable without restriction, they will interfere with each other, leading to an incorrect final count.

[Go Playground](https://go.dev/play/p/uoJtnduVo1V)

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	// We expect the final count to be 10 * 1000 = 10,000.
	const numGoroutines = 10
	const incrementsPerGoroutine = 1000

	// A WaitGroup is used to wait for all the goroutines to finish.
	// This is for control flow, not for memory safety.
	var wg sync.WaitGroup
	wg.Add(numGoroutines)

	// This is our shared counter. It lives on the heap and is accessible
	// by all goroutines launched from main.
	var counter int // Defaults to 0

	// Launch 10 concurrent goroutines.
	for i := 0; i < numGoroutines; i++ {
		go func() {
			// Decrement the WaitGroup counter when the goroutine finishes.
			defer wg.Done()

			// Each goroutine will try to increment the counter 1000 times.
			for j := 0; j < incrementsPerGoroutine; j++ {
				// --- THIS IS THE DANGER ZONE ---
				// The `counter++` operation is not atomic. It involves three steps:
				// 1. Read the current value of `counter`.
				// 2. Add 1 to that value.
				// 3. Write the new value back to `counter`.
				// A different goroutine can read the value *after* this one has
				// read it but *before* it has written the new value back.
				counter++
			}
		}()
	}

	// Wait for all goroutines to complete their work.
	wg.Wait()

	// The final result will almost certainly NOT be 10,000.
	// Run this program multiple times, and you'll likely get a different result each time.
	fmt.Printf("Expected final counter: %d\n", numGoroutines*incrementsPerGoroutine)
	fmt.Printf("Actual final counter:   %d\n", counter)
}
```

When you run the Go program, the output will show an "Actual final counter" that is less than the expected 10,000. This is because multiple goroutines read the same value of counter before any of them can write their incremented value back, causing some increments to be lost. This is a classic race condition.

The Rust Equivalent: Compile-Time Guarantees

Now, let's try to write the same logic in Rust. A naive, direct translation of the Go code would be blocked by the compiler. Rust's ownership and borrowing rules prevent you from sharing mutable data between threads without a synchronization primitive.

The compiler forces you to be explicit about shared ownership (Arc) and mutual exclusion (Mutex). This makes the code slightly more verbose but guarantees at compile time that a race condition of this nature is impossible.

[Rust playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2024&gist=ff466fac48c7303121d9db387398fc8b)

```rs
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    // We expect the final count to be 10 * 1000 = 10,000.
    const NUM_THREADS: usize = 10;
    const INCREMENTS_PER_THREAD: usize = 1000;

    // To share data between threads, Rust requires special "wrapper" types.
    // `Arc` stands for "Atomically Referenced Counter". It allows multiple
    // owners of the same data by keeping track of how many references exist.
    // `Mutex` stands for "Mutual Exclusion". It ensures that only one thread
    // can access the data inside it at any given time.
    //
    // By combining them, we create a thread-safe, shared, mutable integer.
    let counter = Arc::new(Mutex::new(0));

    // We'll store the handles to our spawned threads in this vector.
    let mut handles = vec![];

    for _ in 0..NUM_THREADS {
        // We clone the `Arc` to give ownership of a reference to the new thread.
        // This just increments the reference count; it doesn't copy the data.
        let counter_clone = Arc::clone(&counter);

        let handle = thread::spawn(move || {
            for _ in 0..INCREMENTS_PER_THREAD {
                // --- THIS IS THE SAFE ZONE ---
                // To access the data, we must first `lock()` the mutex.
                // This call will block until the mutex is available.
                // The `.unwrap()` handles a potential error if a thread panics
                // while holding the lock.
                //
                // `num` is a "MutexGuard", a smart pointer to the data (our `i32`).
                let mut num = counter_clone.lock().unwrap();

                // Now that we have the lock, we can safely modify the data.
                // No other thread can acquire the lock until `num` goes out of scope.
                *num += 1;

                // The lock is automatically released when `num` goes out of scope
                // at the end of this statement. This is a key feature of Rust's
                // RAII (Resource Acquisition Is Initialization) pattern.
            }
        });
        handles.push(handle);
    }

    // Wait for all threads to complete their work.
    for handle in handles {
        handle.join().unwrap();
    }

    // The result will ALWAYS be 10,000 because the Mutex ensured that
    // the increments were orderly and atomic from the perspective of the counter.
    // We lock it one last time to read the final value.
    println!("Expected final counter: {}", NUM_THREADS * INCREMENTS_PER_THREAD);
    println!("Actual final counter:   {}", *counter.lock().unwrap());
}
```

I've just provided the two code examples.

The Go program is simple and concise, but it contains a hidden bug that might only appear under heavy load. The burden is on the developer to know when to use synchronization primitives like sync.Mutex.

The Rust program requires you to think about memory sharing and locking upfront. The compiler acts as a strict partner, refusing to compile code that could lead to data races. This prevents the bug entirely, shifting the safety check from runtime (or a developer's code review) to compile-time.

# Grok

To address the query, we need to demonstrate how Go's memory model, which lacks memory isolation between goroutines and control over mutability, can lead to concurrency bugs like data races. Then, we’ll provide a Rust equivalent where its safety features prevent such issues. Let’s dive into the examples.

---

### Go Program: Demonstrating a Data Race Bug

In Go, goroutines share the same memory space, and variables are mutable by default. Without explicit synchronization, multiple goroutines accessing and modifying shared data can result in race conditions—unpredictable behavior due to unsynchronized memory access. Here’s a Go program that illustrates this problem:

```go
package main

import (
    "sync"
)

var counter int

func increment(wg *sync.WaitGroup) {
    for i := 0; i < 1000; i++ {
        counter++ // Unsynchronized access to shared variable
    }
    wg.Done()
}

func main() {
    var wg sync.WaitGroup
    for i := 0; i < 10; i++ {
        wg.Add(1)
        go increment(&wg) // Launch 10 goroutines
    }
    wg.Wait()        // Wait for all goroutines to complete
    println(counter) // Expected: 10000, Actual: Varies
}
```

#### Explanation
- **Shared Mutable State**: The global variable `counter` is an integer accessible by all goroutines.
- **Goroutines**: We launch 10 goroutines, each incrementing `counter` 1000 times. Ideally, `counter` should reach 10 × 1000 = 10,000.
- **Race Condition**: Since there’s no synchronization (e.g., mutex), goroutines may read and write `counter` simultaneously. For example, two goroutines might read `counter` as 42, increment it to 43 locally, and write back 43, losing an update. This leads to a final value less than 10,000, varying per run.
- **Detection**: Running this with `go run -race main.go` will flag a data race, confirming the bug, but Go compiles and executes it regardless, leaving the error to manifest at runtime.

This showcases Go’s memory model: goroutines operate on a shared, mutable heap without inherent protection, making such bugs possible if synchronization is neglected.

---

### Rust Equivalent: Safety Through Compiler Enforcement

Rust’s ownership model and borrow checker prevent data races at compile time by enforcing strict rules: only one mutable reference or multiple immutable references to data can exist at once, and shared data across threads requires synchronization. Let’s first attempt a direct equivalent to the Go program and see why it fails, then provide a safe version.

#### Attempted Unsafe Rust Program (Won’t Compile)
```rust
use std::thread;

fn main() {
    let mut counter = 0;
    let handles: Vec<_> = (0..10).map(|_| {
        thread::spawn(|| {
            for _ in 0..1000 {
                counter += 1; // Attempt to mutate shared variable
            }
        })
    }).collect();

    for handle in handles {
        handle.join().unwrap();
    }
    println!("{}", counter);
}
```

#### Compilation Error
This code doesn’t compile. Rust’s compiler produces an error like:

```
error[E0373]: closure may outlive the current function, but it borrows `counter`, which is owned by the current function
  --> src/main.rs:6:23
   |
6  |         thread::spawn(|| {
   |                       ^^ may outlive borrowed value `counter`
7  |             for _ in 0..1000 {
8  |                 counter += 1;
   |                 ------- `counter` is borrowed here
   |
note: function requires argument type to outlive `'static`
```

- **Why It Fails**: The closure captures `counter` by reference and is moved to a new thread, which may outlive `main`. Rust requires thread-spawned closures to have a `'static` lifetime, but `counter` is a local variable. More critically, even if we adjusted lifetimes, Rust prohibits multiple threads from mutating `counter` without synchronization, preventing the race condition outright.

#### Safe Rust Program with Synchronization
To achieve the same functionality safely, we use Rust’s synchronization primitives, such as `Mutex` with `Arc` (Atomic Reference Counting) for thread-safe sharing:

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    // Wrap counter in Arc<Mutex<i32>> for safe sharing and mutation
    let counter = Arc::new(Mutex::new(0));
    
    // Spawn 10 threads
    let handles: Vec<_> = (0..10).map(|_| {
        let counter_clone = Arc::clone(&counter); // Clone Arc for each thread
        thread::spawn(move || {
            for _ in 0..1000 {
                let mut num = counter_clone.lock().unwrap(); // Lock for exclusive access
                *num += 1; // Safely increment
            }
        })
    }).collect();

    // Wait for all threads to finish
    for handle in handles {
        handle.join().unwrap();
    }

    // Print the final value
    println!("{}", *counter.lock().unwrap()); // Always prints 10000
}
```

#### Alternative with Atomic Operations
For efficiency, we can use `AtomicUsize` instead of `Mutex`, which provides lock-free atomic operations:

```rust
use std::sync::atomic::{AtomicUsize, Ordering};
use std::sync::Arc;
use std::thread;

fn main() {
    let counter = Arc::new(AtomicUsize::new(0));
    let handles: Vec<_> = (0..10).map(|_| {
        let counter_clone = Arc::clone(&counter);
        thread::spawn(move || {
            for _ in 0..1000 {
                counter_clone.fetch_add(1, Ordering::Relaxed); // Atomic increment
            }
        })
    }).collect();

    for handle in handles {
        handle.join().unwrap();
    }

    println!("{}", counter.load(Ordering::Relaxed)); // Always prints 10000
}
```

#### Explanation
- **Arc**: Enables multiple threads to share ownership of the counter.
- **Mutex**: Ensures only one thread modifies the counter at a time, preventing races. `lock()` provides exclusive access.
- **AtomicUsize**: Offers lock-free increments via `fetch_add`, suitable for simple operations like this, with memory ordering (e.g., `Relaxed`) specifying consistency rules.
- **Safety Guarantee**: Rust forces us to use these constructs. The unsafe version doesn’t compile, so we must write safe code, ensuring `counter` reaches exactly 10,000 every time.

---

### Comparison and Conclusion

- **Go’s Vulnerability**: The Go program compiles despite the race condition, relying on the programmer to add synchronization (e.g., `sync.Mutex`) manually. Forgetting this leads to subtle, runtime bugs detectable only with tools like the race detector.
- **Rust’s Prevention**: Rust prevents the bug at compile time. The naive attempt fails due to ownership and borrowing rules, forcing the use of `Mutex` or `AtomicUsize`. This eliminates data races by design, aligning with the query’s requirement to show a safe Rust equivalent.

Thus, Go’s shared, mutable memory model permits concurrency bugs, while Rust’s type system and ownership enforce safety, making the latter immune to this class of errors when using its safe abstractions.

# ChatGPT

Great question. You're right: Go's memory model offers concurrency through goroutines and channels, but it **does not enforce isolation or immutability** at the type level. This makes it easy to accidentally write code with **data races** if shared memory is accessed without synchronization.

Rust, on the other hand, uses **ownership, borrowing, and type system checks at compile time** to **prevent data races by design**.

---

### ✅ Goal

We will:

1. Write a **Go program with a race condition**.
2. Rewrite it in **Rust where the same code won’t even compile** until it’s made race-free.

---

## ⚠️ Go Version (With Race Condition)

```go
package main

import (
    "fmt"
    "time"
)

var shared = 0

func write() {
    for i := 0; i < 1000; i++ {
        shared++
    }
}

func main() {
    go write()
    go write()
    time.Sleep(100 * time.Millisecond)
    fmt.Println("Final value:", shared)
}
```

### 🔍 What's Wrong?

This code starts **two goroutines** that both increment a global variable `shared` **without synchronization**. This will almost certainly lead to a **data race**.

You can verify this by running:

```bash
go run -race main.go
```

You'll see something like:

```
WARNING: DATA RACE
```

---

## ✅ Rust Version (Compile-time Safe)

### ✅ 1. **Rust will prevent the race** at compile time:

```rust
use std::thread;

fn main() {
    let mut shared = 0;

    let handle1 = thread::spawn(|| {
        for _ in 0..1000 {
            // Cannot access shared here
        }
    });

    let handle2 = thread::spawn(|| {
        for _ in 0..1000 {
            // Cannot access shared here
        }
    });

    // This won’t compile: `shared` cannot be shared across threads without synchronization.
    // handle1.join().unwrap();
    // handle2.join().unwrap();

    println!("Final value: {}", shared);
}
```

### 🛑 Error Message

Rust will say something like:

```
error[E0373]: closure may outlive the current function, but it borrows `shared`, which is owned by the current function
```

---

## ✅ Rust Version (Correct and Safe with Synchronization)

You **must** use thread-safe types, like `Mutex` or `Atomic*`.

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let shared = Arc::new(Mutex::new(0));

    let mut handles = vec![];

    for _ in 0..2 {
        let shared_clone = Arc::clone(&shared);
        let handle = thread::spawn(move || {
            for _ in 0..1000 {
                let mut num = shared_clone.lock().unwrap();
                *num += 1;
            }
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Final value: {}", *shared.lock().unwrap());
}
```

### ✅ What’s Different

* `Arc` is an atomic reference counter for shared ownership across threads.
* `Mutex` ensures **only one thread can mutate at a time**.
* Rust forces you to **wrap shared state in safe abstractions**.

---

## ✅ Summary

| Language | Code Compiles?                        | Data Race Possible?      | Safety Mechanism      |
| -------- | ------------------------------------- | ------------------------ | --------------------- |
| Go       | ✅                                     | ✅ Yes                    | Manual sync           |
| Rust     | ❌ (unsafe version) / ✅ (safe version) | ❌ Never (if it compiles) | Enforced via compiler |

Would you like to see the same kind of example using channels instead of shared memory?

**prompt: what does "-race" do?**

The `-race` flag in Go enables the **race detector**, a powerful tool that **detects data races** in your program **at runtime**.

---

### ✅ What it does

When you compile or run a Go program with `-race`, the Go toolchain:

1. **Instruments your code** to track memory accesses.
2. **Monitors all reads and writes** to shared memory.
3. **Detects concurrent accesses** where:

   * At least one access is a write,
   * And they happen in different goroutines,
   * And there’s no synchronization between them.

---

### ✅ How to use it

* Run your Go program with race detection:

```bash
go run -race main.go
```

* Or build it with race detection:

```bash
go build -race -o myapp main.go
./myapp
```

---

### ✅ Example output

For a program with a data race, you'll see output like:

```
==================
WARNING: DATA RACE
Read at 0x00c0000b6008 by goroutine 6:
  main.write()
      /path/to/main.go:10 +0x3c

Previous write at 0x00c0000b6008 by goroutine 5:
  main.write()
      /path/to/main.go:10 +0x3c
==================
```

It tells you:

* The memory address involved.
* The line of code that caused the access.
* Which goroutines are racing.

---

### ✅ When to use it

* During development or testing.
* Before releasing production binaries.
* In CI pipelines to catch unsafe concurrency early.

---

### ⚠️ Limitations

* It **slows down execution** (2–20× slower).
* Uses **more memory**.
* Doesn’t catch **every** race — just ones that actually happen during that run.

---

### ✅ Summary

| Flag    | Purpose                                                |
| ------- | ------------------------------------------------------ |
| `-race` | Enables runtime detection of data races in Go programs |

Would you like help detecting or fixing a specific race in your code?
The `-race` flag in Go enables the **race detector**, a powerful tool that **detects data races** in your program **at runtime**.

---

### ✅ What it does

When you compile or run a Go program with `-race`, the Go toolchain:

1. **Instruments your code** to track memory accesses.
2. **Monitors all reads and writes** to shared memory.
3. **Detects concurrent accesses** where:

   * At least one access is a write,
   * And they happen in different goroutines,
   * And there’s no synchronization between them.

---

### ✅ How to use it

* Run your Go program with race detection:

```bash
go run -race main.go
```

* Or build it with race detection:

```bash
go build -race -o myapp main.go
./myapp
```

---

### ✅ Example output

For a program with a data race, you'll see output like:

```
==================
WARNING: DATA RACE
Read at 0x00c0000b6008 by goroutine 6:
  main.write()
      /path/to/main.go:10 +0x3c

Previous write at 0x00c0000b6008 by goroutine 5:
  main.write()
      /path/to/main.go:10 +0x3c
==================
```

It tells you:

* The memory address involved.
* The line of code that caused the access.
* Which goroutines are racing.

---

### ✅ When to use it

* During development or testing.
* Before releasing production binaries.
* In CI pipelines to catch unsafe concurrency early.

---

### ⚠️ Limitations

* It **slows down execution** (2–20× slower).
* Uses **more memory**.
* Doesn’t catch **every** race — just ones that actually happen during that run.

---

### ✅ Summary

| Flag    | Purpose                                                |
| ------- | ------------------------------------------------------ |
| `-race` | Enables runtime detection of data races in Go programs |

Would you like help detecting or fixing a specific race in your code?
