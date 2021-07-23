---
layout: post
title:  "Linux Kernel Summit - Rust for Linux"
date:   2021-07-22 08:10:00 -0300
categories: linux kernel rust
---

The proposal for the talk "Rust for Linux" for the Linux Kernel Summit led to many interesting discussions.

In this post I collected some of them, but I recommend reading everything from the beginning:
[[TECH TOPIC] Rust for Linux](https://lore.kernel.org/ksummit/CANiq72kF7AbiJCTHca4A0CxDDJU90j89uh80S3pDqDt7-jthOg@mail.gmail.com/)

[PL061 driver in C and Rust - side by side](/assets/html/pl061-c-vs-rust.html)

- [Using RAII for error paths](#using-raii-for-error-paths)
- [devm_k*alloc() are a nightmare](#devm_kalloc-are-a-nightmare)
- [Multiple References challenges in C](#multiple-references-challenges-in-c)
- [Rust `Rc<T>` prevents UB and data races](#rust-rct-prevents-ub-and-data-races)
- [Rust `Rc` vs `Arc`](#rust-rc-vs-arc)
- [Safety of Rust `Ref<T>` drop](#safety-of-rust-reft-drop)
- [Rust aliasing and lifetime rules](#rust-aliasing-and-lifetime-rules)
- [Rust Weak References](#rust-weak-references)
- [Rust lifetimes, aliasing rules, sync & send traits](#rust-lifetimes-aliasing-rules-sync--send-traits)
- [virtio device](#virtio-device)
- [PL061 driver in Rust](#pl061-driver-in-rust)
- [Rust for loop](#rust-for-loop)
- [Standard Rust Style](#standard-rust-style)
- [Rust Conditional Compilation](#rust-conditional-compilation)
- [How to model APIs between drivers and subsystems](#how-to-model-apis-between-drivers-and-subsystems)
- [Rust Compilation Model](#rust-compilation-model)
- [Rust Endianess](#rust-endianess)
- [virtio backend for I2C in Rust](#virtio-backend-for-i2c-in-rust)
- [Pattern Matching](#pattern-matching)
- [Modules vs Devices Lifespans](#modules-vs-devices-lifespans)
- [Improving the Driver Writing Experience](#improving-the-driver-writing-experience)
- [Why not C++ in the Linux Kernel](#why-not-c-in-the-linux-kernel)


### Using RAII for error paths

[From: Roland Dreier](https://lore.kernel.org/ksummit/CAG4TOxMzf1Wn6PcWk=XfB+SV+MHwbxUq8t1RNswie5e3=Y+OXQ@mail.gmail.com/)


On Mon, Jul 5, 2021 at 4:51 PM Linus Walleij <linus.walleij@linaro.org> wrote:

> I also repeat my challenge from the mailing list, which is that
> while I understand that leaf drivers is a convenient place to start
> doing Rust code, the question is if that is where Rust as a language
> will pay off. My suggestion is to investigate git logs for the kind of
> problems that Rust solves best and then introduce Rust where
> those problems are. I believe that this would make be the
> Perfect(TM) selling point for the language in the strict
> hypothetico-deductive sense:

One area I see where Rust could make a big improvement for drivers is
in using RAII for error paths.  Drivers often have a lot of code like

```c
    if (something())
        return err;

    if (something_else())
        goto undo_something;

    if (a_third_thing())
        goto undo_two_things;
```

and so on, which is difficult to get right in the first place and even
harder to keep correct in the face of changes.

That leads to a lot of bugs in error paths, which maybe affect no one
until they become exploitable security bugs, and also means that
maintainers get a lot of patches like
https://lore.kernel.org/linux-rdma/20200523030457.16160-1-wu000273@umn.edu/
that need careful review.

"devres" / devm_xxx was an attempt to deal with this in C, but it only
solves some cases of this and has not received a lot of adoption (we
can argue about the reasons).  Better language-level support for
ownership / lifetime tied to scope would probably lead to driver code
that is better in several dimensions.  Even in C++, the RAII idiom is
quite nice and I often miss it in kernel code.


### Example using "devres" / devm_xxx

[From: Linus Walleij](https://lore.kernel.org/ksummit/CACRpkdZyJd0TW5aVRfxSSWknzCyVhjMwQuAj9i9iuQ6pW9vftQ@mail.gmail.com/)


Dmitry in the input subsystem even insist to use it for e.g. powering
down and disabling regulators on remove(), like recently in
drivers/input/touchscreen/cy8ctma140.c

```c
/* Called from the registered devm action */
static void cy8ctma140_power_off_action(void *d)
{
        struct cy8ctma140 *ts = d;

        cy8ctma140_power_down(ts);
}
(...)
error = devm_add_action_or_reset(dev, cy8ctma140_power_off_action, ts);
if (error) {
        dev_err(dev, "failed to install power off handler\n");
        return error;
}
```

I think it's a formidable success, people just need to learn to do it more.

### devm_k*alloc() are a nightmare

[From: Laurent Pinchart](https://lore.kernel.org/ksummit/YOTSYy2J2COzOY9l@pendragon.ideasonboard.com/)


devres is interesting and has good use cases, but the devm_k*alloc() are
a nightmare. They're used through drivers without any consideration of
life time management of the allocated memory, to allocate data
structures that can be accessed in userspace-controlled code paths.
Open a device node, keep it open, unplug the device, close the device
node, and in many cases you'll find that the kernel can crash :-( When
the first line of the probe function of a driver that exposes a chardev
if a devm_kzalloc(), it's usually a sign that something is wrong.

I wonder if we could also benefit from the gcc cleanup attribute
([https://gcc.gnu.org/onlinedocs/gcc/Common-Variable-Attributes.html#index-cleanup-variable-attribute](https://gcc.gnu.org/onlinedocs/gcc/Common-Variable-Attributes.html#index-cleanup-variable-attribute)).
I don't know if it's supported by the other compilers we care about
though.

### Multiple References challenges in C

[From: Greg KH](https://lore.kernel.org/ksummit/YOVbsS9evoCx0isz@kroah.com/)


This is going to be the "interesting" part of the rust work, where it
has to figure out how to map the reference counted objects that we
currently have in the driver model across to rust-controlled objects and
keep everything in sync properly.

For the normal code, the fact that the memory was assigned to one
specific object (the struct device) but yet referenced from another
object (the cdev).  devm_* users like this do not seem to realize there
are two separate object lifecycles happening here as the interactions
are subtle at times.

I am looking forward to how the rust implementation is going to handle
all of this as I have no idea.


### Rust `Rc<T>` prevents UB and data races

[From: Miguel Ojeda](https://lore.kernel.org/ksummit/CANiq72n=8Do_9suDeJdwoF_8ZR-uLEj2r9cRSB_k=yTk_q0FHw@mail.gmail.com/)


Even if you are using `Rc<T>`, Rust buys you a *lot*. For starters, it
still prevents UB and data races -- e.g. it prevents mistakenly
sharing the `Rc` with other threads.


### Rust `Rc` vs `Arc`

[From: Miguel Ojeda](https://lore.kernel.org/ksummit/CANiq72=qa2jMUKyfPmH1q1jr8n_Tm7FMy0QyWRhhpinUrvMiNA@mail.gmail.com/)


Yes, `Rc` is a type meant for single-threaded reference-counting
scenarios. For multi-threaded scenarios, Rust provides `Arc` instead.
It still guarantees no UB and no data races.

This is actually a good example of another benefit. `Rc` is there
because there is a performance advantage when you do not need to share
it among threads (you do not need to use atomics) -- and Rust checks
you do not do so. In C, you can still have an `Rc`, but you would not
have a compile-time guarantee that it is not shared. So in C you may
decide to avoid your single-threaded `Rc` and use `Arc` "just in case"
-- and, in fact, this is the only one C++ has (`std::shared_ptr`), and
even then, it does not statically prevent UB.

--

[From: Wedson Almeida Filho](https://lore.kernel.org/ksummit/YOXd9WoafgBr1Nkv@google.com/)

For cases when you want to hold on to something without incrementing its
refcount, the common pattern in Rust is to use "weak" pointers. `Ref<T>` doesn't
support them because we haven't needed them yet and they have extra cost.
(`Arc<T>` supports them though.)


### Safety of Rust `Ref<T>` drop

[From: Wedson Almeida Filho](https://lore.kernel.org/ksummit/YOXd9WoafgBr1Nkv@google.com/)


>On Wed, Jul 07, 2021 at 05:44:41PM +0200, Greg KH wrote:<br>
> >On Wed, Jul 07, 2021 at 05:15:01PM +0200, Miguel Ojeda wrote:<br>
> > For instance, we have a `Ref` type that is similar to `Arc` but reuses
> > the `refcount_t` from C and introduces saturation instead of aborting
> > [3]
> > 
> > [3] https://github.com/Rust-for-Linux/linux/blob/rust/rust/kernel/sync/arc.rs
> 
> This is interesting code in that I think you are missing the part where
> you still need a lock on the object to prevent one thread from grabbing
> a reference while another one is dropping the last reference.  Or am I
> missing something?

You are missing something :)

> The code here:

```rust 
    fn drop(&mut self) {
         // SAFETY: By the type invariant, there is necessarily a reference to the object. We cannot
         // touch `refcount` after it's decremented to a non-zero value because another thread/CPU
         // may concurrently decrement it to zero and free it. It is ok to have a raw pointer to
         // freed/invalid memory as long as it is never dereferenced.
         let refcount = unsafe { self.ptr.as_ref() }.refcount.get();
 
         // INVARIANT: If the refcount reaches zero, there are no other instances of `Ref`, and
         // this instance is being dropped, so the broken invariant is not observable.
         // SAFETY: Also by the type invariant, we are allowed to decrement the refcount.
         let is_zero = unsafe { rust_helper_refcount_dec_and_test(refcount) };
         if is_zero {
             // The count reached zero, we must free the memory.
             //
             // SAFETY: The pointer was initialised from the result of `Box::leak`.
             unsafe { Box::from_raw(self.ptr.as_ptr()) };
         }
    }
```

> 
> Has a lot of interesting comments, and maybe just because I know nothing
> about Rust, but why on the first line of the comment is there always
> guaranteed a reference to the object at this point in time?

It's an invariant of the `Ref<T>` type: if a `Ref<T>` exists, there necessarily
is a non-zero reference count. You cannot have a `Ref<T>` with a zero refcount.

`drop` is called when a `Ref<T>` is about to be destroyed. Since it is about to
be destroyed, it still exists, therefore the ref-count is necessarily non-zero.

> And yes
> it's ok to have a pointer to memory that is not dereferenced, but is
> that what is happening here?

`refcount` is a raw pointer. When it is declared and initialised, it points to
valid memory. The comment is saying that we should be careful with the case
where another thread ends up freeing the object (after this thread has
decremented its share of the count); and that we're not violating any Rust
aliasing rules by having this raw pointer (as long as we don't dereference it).
 
> I feel you are trying to duplicate the logic of a "struct kref" here,

`struct kref` would give us the ability to specify a `release` function when
calling `refcount_dec_and_test`, but we don't need this round-trip to C code
because we know at compile-time what the `release` function is: it's the `drop`
implementation for the wrapped type (`T` in `Ref<T>`).

> and that requires a lock to work properly.  Where is the lock here?

We don't need a lock. Once the refcount reaches zero, we know that nothing else
has pointers to the memory block; the lifetime rules guarantee that if there is
a reference to a `Ref<T>`, then it cannot outlive the `Ref<T>` itself. To
produce a new `Ref<T>` from an existing one, `clone` is called, which increments
the refcount.


### Rust aliasing and lifetime rules


[From: Wedson Almeida Filho](https://lore.kernel.org/ksummit/YOX+N1D7AqmrY+Oa@google.com/)


> What enforces that?  Where is the lock on the "back end" for `Ref<T>`
> that one CPU from grabbing a reference at the same time the "last"
> reference is dropped on a different CPU?

There is no lock. I think you might be conflating kobject concepts with general
reference counting.

The enforcement is done at compile time by the Rust aliasing and lifetime rules:
the owner of a piece of memory has exclusive access to it, except when it is
borrowed. When it is borrowed, the borrow *must not* outlive the memory being
borrowed.

Unsafe code may break these rules, but if it exposes a safe interface (which
`Ref` does), then this interface must obey these rules.

Here's a simple example. Suppose we have a struct like:

```rust
struct X {
    a: u64,
    b: u64,
}
```

And we want to create a reference-counted instance of it, we would write:

```rust
  let ptr = Ref::new(X { a: 10, b: 20 });
```

(Note that we don't need to embed anything in the struct, we just wrap it with
the `Ref` type. In this case, the type of `ptr` is `Ref<X>` which is a
ref-counted instance of `X`.)

At this point we can agree that there are no other pointers to this. So if we
`ptr` went out of scope, the refcount would drop to zero and the memory would be
freed.

Now suppose I want to call some function that takes a reference to `X` (a
const pointer to `X` in C parlance), say:

```rust
fn testfunc(ptr_ref: &X) {
}
```

This reference has a lifetime associated with it. The compiler won't allow
implementations where using `ptr_ref` would outlive the original `ptr`, for
example if it escapes `testfunc` (for example, to another thread) but doesn't
"come back" before the end of the function (for example, if `testfunc` "joined"
the thread). Here's a trivial example with scopes to demonstrate the sort of
compiler error we'd get:

```rust
fn main() {
    let ptr_ref;
    {
        let ptr = Ref::new(X { a: 10, b: 20 });
        ptr_ref = &ptr;
    }
    println!("{}", ptr_ref.a);
}
```

Compiling this results in the following error:

```
error[E0597]: `ptr` does not live long enough
  --> src/main.rs:12:19
   |
12 |         ptr_ref = &ptr;
   |                   ^^^^ borrowed value does not live long enough
13 |     }
   |     - `ptr` dropped here while still borrowed
14 |     println!("{}", ptr_ref.a);
   |                    ------- borrow later used here
```

Following these rules, the compiler *guarantees* that if a thread or CPU somehow
has access to a reference to a `Ref<T>`, then it *must* be backed by a real
`Ref<T>` somewhere: a borrowed value must never outlive what it's borrowing. So
incrementing the refcount is necessarily from n to n+1 where n > 0 because the
existing reference guarantees that n > 0.

There are real cases when you can't guarantee that lifetimes line up as required
by the compiler to guarantee safety. In such cases, you can "clone" ptr (which
increments the refcount, again from n to n+1, where n > 0), then you end up with
your own reference to the underlying `X`, for example:

```rust
fn main() {
    let ptr_clone;
    {
        let ptr = Ref::new(X { a: 10, b: 20 });
        ptr_clone = ptr.clone();
    }
    println!("{}", ptr_clone.a);
}
```

(Note that the reference owned by `ptr` has been destroyed by the time
`ptr_clone.a` is used in `println`, but `ptr_clone` has its own reference due to
the clone call.)

The ideas above apply equally well if instead of thinking in terms of scope, you
think in terms of threads/CPUs. If a thread needs a refcounted object to
potentially outlive the borrow keeping it alive, then it needs to increment
the refcount: if you can't prove the lifetime rules, then you must clone the
reference.

Given that by construction the refcount starts at 1, there is no path to go from
0 to 1. Ever.

> Does Rust provide "architecture-specific" locks like this somehow that
> are "built in"?  If so, what happens when we need to fix those locks?
> Does that get fixed in the compiler, not the kernel code?

There are no magic locks implemented by the compiler.

> Even with multiple CPUs?  What enforces these lifetime rules?

The compiler does, at compile-time. Each lifetime usage is a constraint that
must be satisfied by the compiler; once all constraints are gathered for a given
function, the compiler tries to solve them, if it can find a solution, then the
code is accepted; if it can't find a solution, the code is rejected. Note that
this means that some correct code may be rejected the compiler by design: it is
conservative in that it only accepts what it can prove is correct.


### Rust Weak References

[From: Wedson Almeida Filho](https://lore.kernel.org/ksummit/YOY0HLj5ld6zHJHU@google.com/)

> So I think Greg speaks about a situation where you have multiple threads
> and the refcounted object can be looked up through some structure all the
> threads see. And the problem is that the shared data structure cannot hold
> ref to the object it points to because you want to detect the situation
> where the data structure is the only place pointing to the object and
> reclaim the object in that case. Currently I don't see how to model this
> idiom with Rust refs.

The normal idiom in Rust for this is "weak" pointers. With it, each
reference-counted object has two counts: strong and weak refs. Objects are
"destroyed" when the strong count goes to zero and "freed" when the weak count
goes to zero.

Weak references need to upgraded to strong references before the underlying
objects can be accessed; upgrading may fail if the strong count has gone to
zero. It is, naturally, implemented as an increment that avoids going from 0 to 1.
It is safe to try to do it because the memory is kept alive while there are
weak references.

For the case you mention, the list would be based on weak references. If the
object's destructor also removes the object from the list, both counts will go
to zero and the object will be freed as well. (If it fails to do so, the
*memory* will linger allocated until someone removes the object from the list,
but all attempts to upgrade the weak reference to a strong one will fail.)

The obvious cost is that we need an extra 32-bit number per reference-counted
allocation. But if we have specialized cases, like the underlying object always
being in some data structure until the ref count goes to zero, then we can build
a zero-cost abstraction for such a scenario.

We can also build specialised zero-cost abstractions for the case when we want
to avoid the 1 -> 0 transition unless we're holding some lock to prevent others
observing the object-with-zero-ref. For this I'd have to spend more time to see
if we can do safely (i.e., with compile-time guarantees that the object was
actually removed from the data structure).


### Rust lifetimes, aliasing rules, sync & send traits

[From: Wedson Almeida Filho](https://lore.kernel.org/ksummit/YOb%2FaJC2VuOcz3YY@google.com/)

> Thanks for the detailed explainations, it seems rust can "get away" with
> some things with regards to reference counts that the kernel can not.
> Userspace has it easy, but note that now that rust is not in userspace,
> dealing with multiple cpus/threads is going to be interesting for the
> language.
>
> So, along those lines, how are you going to tie rust's reference count
> logic in with the kernel's reference count logic?  How are you going to
> handle dentries, inodes, kobjects, devices and the like?  That's the
> real question that I don't seem to see anyone even starting to answer
> just yet.

None of what I described before is specific to userspace, but I understand your
need to see something more concrete.

I'll describe what we've done for `task_struct` and how lifetimes, aliasing
rules, sync & send traits help us achieve zero cost (when compared to C) but
also gives us safety. The intention here is to show that similar things can (and
will) be done to other kernel reference-counted objects when we get to them.

So we begin with `current`. It gives us access to the current task without
incurring any increments or decrements of the refcount; in Rust, we'd do the
following:

```rust
  let current = Task::current();
```

We can use it for however long we'd like as long as it's in the same task. But
how do we restrict that? Rust has this `Send` trait that tells it that a type
can be used by another thread/CPU; `current`'s type doesn't isn't `Send`, so
attempting to send it another thread fails. For example, if we tried this:

```rust
  send_to_thread(current);
```

We'd get the following compiler error:

```
    |
195 | fn send_to_thread<T: Send>(t: T) {
    |                      ---- required by this bound in `send_to_thread`
...
201 |     send_to_thread(current);
    |     ^^^^^^^^^^^^^^ `*mut ()` cannot be sent between threads safely
    |
    = help: within `TaskRef<'_>`, the trait `Send` is not implemented for `*mut ()`
    = note: required because it appears within the type `(&(), *mut ())`
    = note: required because it appears within the type `PhantomData<(&(), *mut ())>`
note: required because it appears within the type `TaskRef<'_>`
```

One option we do have is to send a "reference" to `current` to another thread.
This works and is zero-cost. It is safe because the lifetime of the reference is
tied to that of `current`. (And `current`'s type is `Sync`, which means that a
reference to it is safely shareable with another thread/CPU.)

So calling:

```rust
  send_to_thread(&current);
```

Works fine. But its implementation must convince the compiler that by the time
it returns the sharing with another thread is over. Otherwise it would be a
violation of lifetime requirements (a borrow cannot outlive the borrowed value)
and compilation would fail.

Now, similarly to my example in another email, if you really want the task to
outlive `current`, then you can call `clone`. This results in `get_task_struct`
being called, so now we can send it another thread that can hold on to it and
this is all safe. For example:

```rust
    let current = Task::current();
    let task = current.clone();
    send_to_thread(task);
```

Now, the other task *owns* the reference, so we're not supposed to use `task` at
all (suppose for a moment that it's some arbitrary task, not current). In C,
given that there is no ownership discipline enforced by the compiler, one could
easily make the mistake of using `task` (which is unsafe because the other
thread/CPU may have decremented its refcount and freed it by now). In Rust, an
attempt to use `task` would fail; for example:

```rust
    let current = Task::current();
    let task = Task::current().clone();
    send_to_thread(task);
    pr_info!("Pid is {}", task.pid());
```

Would result in the following compilation error:

```
error[E0382]: borrow of moved value: `task`
   --> rust/kernel/task.rs:203:34
    |
201 |     let task = Task::current().clone();
    |         ---- move occurs because `task` has type `Task`, which does not implement the `Copy` trait
202 |     send_to_thread(task);
    |                    ---- value moved here
203 |     pr_info!("Pid is {}", task.pid());
    |                           ^^^^ value borrowed here after move
```

(Note that `task` is inaccessible despite still being in scope.)

Another common mistake is to leak references by forgetting to call
`put_task_struct`. Rust helps prevent this by automatically calling it when
needed, including error paths. Suppose we have a function that allocates some
struct that includes a task field, for example:

```rust
struct X {
    task: Task,
    a: u64,
    b: u64,
}

fn alloc_X(task: Task) -> Result<Box<X>> {
    Box::try_new(X {
        task: task,
        a: 10,
        b: 20
    })
}
```

Here, if the function fails (e.g., the allocation in `try_new` fails), the task
refcount is automatically decremented when the function returns. If it succeeds,
ownership is transferred to the new instance of X, and the refcount will be
decremented automatically when this instance of X is eventually freed.

Another example that shows lifetimes clearly is that of `group_leader`. Given a
task, its group leader can be accessed with zero-cost but this access is
subjected to lifetime requirements. For example:

```rust
    let task = get_some_task();
    let leader = task.group_leader();
    pr_info!("Pid is {}", leader.pid());
```

Works with zero cost (i.e., no extra inc/ref of the group leader). But the
following:

```rust
    let leader;
    {
        let task = get_some_task();
        leader = task.group_leader();
    }
    pr_info!("Pid is {}", leader.pid());
```

Fails with the following error:

```
error[E0597]: `task` does not live long enough
   --> rust/kernel/task.rs:209:18
    |
209 |         leader = task.group_leader();
    |                  ^^^^ borrowed value does not live long enough
210 |     }
    |     - `task` dropped here while still borrowed
211 |     pr_info!("Pid is {}", leader.pid());
    |                           ------ borrow later used here
```

Because task's refcount is decremented at the end of the scope.


### virtio device

[From: Andy Lutomirski](https://lore.kernel.org/ksummit/3D69F05C-A039-4730-957B-02BE4E06C547@amacapital.net/)


I would suggest a virtio device.  The core virtio guest code is horribly unsafe, and various stakeholders keep trying to make it slightly less unsafe. A Rust implementation of the entire virtio ring (or at least the modern bits) and of some concrete device (virtio-blk?) would be quite nice.

I think that, at least for an initial implementation, erroring out in the !use_dma case would be fine. The !use_dma virtio variant is, in my opinion, a historical error and does not really deserve to survive going forward. The powerpc people may beg to differ.  Any port of the !use_dma variant to Rust (or anything that safely manages device memory) will quickly notice that the resulting use of device memory is nonsense and ought not to compile without unsafe annotations in various places. And those unsafe annotations may well deserve comments like /* yes, this is indeed wrong.*/.

Fortunately, at least on non-powerpc, you can easily boot a system in the sane use_dma mode. Virtme will do this for you with minimal effort.

--

[From: Andy Lutomirski](https://lore.kernel.org/ksummit/CALCETrWKbNuEjdFA3FEuD6WJH7mcCAr8HReFjaTe9TQ9=XNztQ@mail.gmail.com/)


> Another deviation from real hardware drivers? ;-)
> How many kernel maintainers have experience with writing a virtio driver?
>

Me?  Definitely more than zero :)

> What is wrong with something driving real hardware?

Nothing! But with virt hardware, anyone can test it.

That being said, the virtio case has real use cases from the secure
virt folks.  A Linux guest in a VM host that has untrusted virtio host
devices (TDX, SEV, various Xen or userspace host device driver
implementations, etc) wants to avoid being compromised by a
misbehaving virtio device.


### PL061 driver in Rust

[From: Wedson Almeida Filho](https://lore.kernel.org/ksummit/YPVvEZgcP1LMGjcy@google.com/)


[Code in C and Rust - side by side](/assets/html/pl061-c-vs-rust.html)


Here is a working PL061 driver in Rust (converted form the C one):
[https://github.com/wedsonaf/linux/blob/pl061/drivers/gpio/gpio_pl061_rust.rs](https://github.com/wedsonaf/linux/blob/pl061/drivers/gpio/gpio_pl061_rust.rs)

(I tested it on QEMU through the sysfs interface and also gpio-keys as QEMU
uses one of the PL061 pins as the power button.)

I have a long list of ways in which Rust affords us extra guarantees but in the
interest of brevity I will try to describe how Rust helps us address the two (or
more) lifetime issues Greg mentioned the other day.

Rust allows us to build abstractions that guarantee safety. Here are the ones I
used/built for this:

1. State created on `probe` is ref-counted.
2. Hardware resources (device mem and irq in this case) are "revocable".
3. On `remove`, we automatically revoke access to hardware resources, then free
them.

What this gives us:
1. With ref-counted objects Rust allows us to avoid dangling pointers. No more
UAF because memory was freed when the device was removed. (C can also do this,
of course, but the compiler doesn't help us if/when we forget to
increment/decrement the ref count.)
2. Given that references to device state may outlive the device, revocable hw
resources allows us to prevent the use of these resources after the device is
gone. Rust ensures that such access is only allowed before resources are
revoked. (In C we can also do something similar, but the compiler won't enforce
this invariant for us, i.e., we can make mistakes where we forget to check if
something was revoked, or forget to hold locks keeping resources alive, etc.)
3. After revoking access, we need to ensure that existing concurrent users
finish before we can free resources. In this implementation, we use RCU so that
resource users need to hold an RCU read lock and we ensure that they've also
completed their use before freeing the resources (synchronize_rcu between
revoking & freeing). Locking/unlocking happens automatically.

This, naturally, doesn't solve any problems with the existing C code. However, I
think it addresses things on the Rust side. For example, suppose that in
addition to registering with gpio, we also wanted to expose the device as a
miscdev (I use this as an example because we have miscdevs in Rust). The
refcounted device state can be stored in the miscdev registration, and each
opened file can also have a reference to it (device state). We don't control
when the latter gets released, but it's ok for them to hold on to state because
they won't be able to use hw resources after the device is removed; once all
file descriptors are closed, the refcount goes to zero and the memory is freed.


### Rust for loop

[From: Miguel Ojeda](https://lore.kernel.org/ksummit/CANiq72n706_u-q3=rTL9UCNhJrji3VvOpGg+6uoEttBWWpVfrw@mail.gmail.com/)


>On Mon, Jul 19, 2021 at 4:43 PM Geert Uytterhoeven <> wrote:
>
> Turns out "a..b" in Rust does mean "range from a to b-1".
> That's gonna be hard to (un)learn...

It may help to think about it as the usual `for` loop in C, typically
written with `<` and the "size" (instead of `<=` and the "size - 1"),
i.e.:

```rust
    for i in 0..N
    for (i = 0; i < N; ++i)
```

--

[From: Steven Rostedt](https://lore.kernel.org/ksummit/20210719144756.307e5d4d@oasis.local.home/)


Or think of it as the python range() function, but not as a git log, where

	git log sha1..sha2

is a list of commits from sha1 + 1 through to sha2 inclusive :-p


### Standard Rust Style

[From: Miguel Ojeda](https://lore.kernel.org/ksummit/CANiq72knmcA_9ZTk13ECSC1jUeVQx9di9hqHjFF6M+-HRRDWoA@mail.gmail.com/)


As Linus said, it is a way to
easily differentiate what is what, instead of using a prefix or
suffix:
  - Types (and similar) are in `CamelCase`.
  - Functions (and similar) are in `snake_case`.
  - Statics/consts are in `SCREAMING_CASE`.

See [https://rust-lang.github.io/api-guidelines/naming.html](https://rust-lang.github.io/api-guidelines/naming.html)

### Rust Conditional Compilation

[From: Miguel Ojeda](https://lore.kernel.org/ksummit/CANiq72=tm_ERbkG1jyWwuyJqzgA9M8Hd7rCQ-tJ7Tf_E-v2wgA@mail.gmail.com/)



We support conditional compilation with the kernel configuration, e.g.:

```rust
    #[cfg(CONFIG_X)]      // `CONFIG_X` is enabled (`y` or `m`)
    #[cfg(CONFIG_X="y")]  // `CONFIG_X` is enabled as a built-in (`y`)
    #[cfg(CONFIG_X="m")]  // `CONFIG_X` is enabled as a module   (`m`)
    #[cfg(not(CONFIG_X))] // `CONFIG_X` is disabled
```

One can use it, like in C, to fully remove any item as a user, to make
an implementation a noop (so that callers do not need to care), etc.


### How to model APIs between drivers and subsystems

[From: Laurent Pinchart](https://lore.kernel.org/ksummit/YPdiMHr%2Ft5K6RJck@pendragon.ideasonboard.com/)




> - Attempts to access hardware resources freed during `remove` *must* be
>   prevented, that's where the calls to `resources()` are relevant -- if a
>   subsystem calls into the driver with one of the references it held on to, they
>   won't be able to access the (already released) hw resources.

That's where our opinions differ. Yes, those accesses must be prevented,
but I don't think the right way to do so is to check if the I/O memory
resource is still valid. We should instead prevent reaching driver code
paths that make those I/O accesses, by waiting for all calls in progress
to return, and preventing new calls from being made. This is a more
generic solution in the sense that it doesn't prevent accessing I/O
memory only, but avoids any operation that is not supposed to take
place. My reasoning is that drivers will be written with the assumption
that, for instance, nobody will try to set the GPIO direction once
.remove() returns. Even if the direction_output() function correctly
checks if the I/O memory is available and returns an error if it isn't,
it may also contain other logic that will not work correctly after
.remove() as the developer will not have considered that case. This
uncorrect logic may or may not lead to bugs, and some categories of bugs
may be prevented by rust (such as accessing I/O memory after .remove()),
but I don't think that's relevant. The subsystem, with minimal help from
the driver's implementation of the .remove() function if necessary,
should prevent operations from being called when they shouldn't, and
especially when the driver's author will not expect them to be called.
That way we'll address whole classes of issues in one go. And if we do
so, checking if I/O memory access has been revoked isn't required
anymore, as we guarantee if isn't.

True, this won't prevent I/O memory from being accessed after .remove()
in other contexts, for instance in a timer handler that the driver would
have registered and forgotten to cancel in .remove(). And maybe the I/O
memory revoking mechanism runtime overhead may be a reasonable price to
pay for avoiding this, I don't know. I however believe that regardless
of whether I/O memory is revoked or not, implementing a mechanism in the
subsytem to avoid erroneous conditions from happening in the first place
is where we'll get the largest benefit with a (hopefully) reasonable
effort.

> We have this problem specifically in gpio: as Linus explained, they created an
> indirection via a pointer which is checked in most entry points, but there is no
> synchronisation that guarantees that the pointer will remain valid during a
> call, and nothing forces uses of the pointer to be checked (so as Linus points
> out, they may need more checks).
> 
> For Rust drivers, if the registration with other subsystems were done by doing
> references to driver data in Rust, this extra "protection" (that has race
> conditions that, timed correctly, lead to use-after-free vulnerabilities) would
> be obviated; all would be handled safely on the Rust side (e.g., all accesses
> must be checked, there is no way to get to resources without a check, and use of
> the resources is guarded by a guard that uses RCU read-side lock).
> 
> Do you still think we don't have a problem?

We do have a problem, we just try to address it in different ways. And
of course mine is better, and I don't expect you to agree with this
statement right away ;-) Jokes aside, this has little to do with C vs.
rust in this case though, it's about how to model APIs between drivers
and subsystems.

--

[From: Andy Lutomirski](https://lore.kernel.org/ksummit/CALCETrWH4N17C+uHaDbzGkgS005feaOVQ25yGo9Zy0cb3+eeGA@mail.gmail.com/)


Preventing functions from being called, when those functions are
scattered all over the place (sysfs, etc) may be hard.  Preventing
access to a resource seems much more tractable.  That being said,
preventing access to the I/O resource in particular seems a bit odd to
me.  It's really the whole device that's gone after it has been
removed.  So maybe the whole device (for some reasonable definition of
"whole device") could get revoked?

--

[From: Laurent Pinchart](https://lore.kernel.org/ksummit/YPd7byfwcfbOvPyn@pendragon.ideasonboard.com/)



I agree that we should view this from a resource point of view, or
perhaps an object point of view, instead of looking at individual
functions. A GPIO driver creates GPIO controllers, which are objects
that expose an API to consumers. In most cases, it's the whole API that
shouldn't be called after .remove(), not individual functions (there are
a few exceptions, for instead the .release() file operation for objects
exposed through userspace as a cdev may need to reach the object). What
I'd like to see is the subsystem blocking this, instead of having
individual drivers tasked with correctly handling API calls from
consumers after .remove(). The correctness of the latter may be less
difficult to achieve and even guarantee with rust, but that's solving a
problem that shouldn't exist in the first place.


### Rust Compilation Model

[From: Miguel Ojeda](https://lore.kernel.org/ksummit/CANiq72kHY=w8VVHUH8EyLcfRXQzq+OXOBCrrW7dHk9kkzJ_BHQ@mail.gmail.com/)



In any case, please note that the compilation model is different in
Rust than in C. In Rust, the translation unit is not each `.rs` file,
but the "crate" (i.e. a set of `.rs` files that are found by the
compiler given a `.rs` entry point file), there are no header files,
etc. Thus some differences apply. For instance, currently having
everything in the same "crate" means the compiler can see everything
even without LTO.


### Rust Endianess

[From: Miguel Ojeda](https://lore.kernel.org/ksummit/CANiq72nzWrs2RoLGYVy3s=bXCxywtOvQLMV7w6mnBgeMK8uwNQ@mail.gmail.com/)


> Can you express endianness here as well? There are often cases where
> the hardware is always big endian independent of the CPU
> endianness. Some of the readl/writel macros handle this, adding a swap
> when the two don't match.

Yes, of course!

Querying the endianness can be done as usual, similar to the C preprocessor:

```rust
    pub fn f(n: i32) -> i32 {
        #[cfg(target_endian = "little")]
        return n + 42;

        #[cfg(target_endian = "big")]
        return n;
    }
```

As well as with the `cfg!` macro, more ergonomic for small things like this:

```rust
    pub fn f(n: i32) -> i32 {
        if cfg!(target_endian = "little") {
            n + 42
        } else {
            n
        }
    }
```

As well in macros (like the `memory_map!` shown above).

If you just need the most common case, i.e. converting from/to a given
endianness, the primitive types come with functions for that, e.g. the
following would include a `bswap` in x86:

```rust
    pub fn f(n: i32) -> i32 {
        i32::from_be(n)
    }
```

See `{from,to}_{be,le}` in [https://doc.rust-lang.org/std/primitive.i32.html](https://doc.rust-lang.org/std/primitive.i32.html)


### virtio backend for I2C in Rust

[From: Viresh Kumar](https://lore.kernel.org/ksummit/20210709070329.uhpoqnttqu3uw2am@vireshk-i7/)



Just so everyone is aware of our work with Rust, we (at Linaro's
Project Stratos [1] initiative) are looking to write Hypervisor
agnostic low-level virtio based vhost-user backends, which can be used
to process low-level (like I2C, GPIO, RPMB, etc) requests from guests
running on VMs to the host having access to the real hardware.

We are still working on getting the virtio GPIO spec merged, but the
I2C stuff has progressed well. Here is the virtio backend I have
written for I2C in Rust:

[https://github.com/vireshk/vhost-device](https://github.com/vireshk/vhost-device)

It is under review [2] currently to get merged in rust-vmm project.

[1] [https://linaro.atlassian.net/wiki/spaces/STR/overview](https://linaro.atlassian.net/wiki/spaces/STR/overview)<br>
[2] [https://github.com/rust-vmm/vhost-device/pull/1](https://github.com/rust-vmm/vhost-device/pull/1)


### Pattern Matching

[From: Andy Lutomirski](https://lore.kernel.org/ksummit/CALCETrVGL6sPPoxPZmGy0xcfakL3XAF+KX27Ofid76__SxS4XA@mail.gmail.com/)



> What if I would ignore that? I.e.
>
>    let Some(content) = strong;
>    println!("{}", content);
>
> ?

That won’t compile. Pattern matching like this is fundamentally a
conditional operation. In Rust, or in pretty much any language with
proper “sum types” if you try to “destructure” something (the value
‘strong’ here) and there’s a possibility that the pattern doesn’t
match, then the compiler will require that you have some sort of
appropriate condition.

Personally, for some types of programming, I find sum types by
themselves to be an even bigger win than Rust-style safety.  C has
unions, which are painful to use. Python has various hacks that are
unpleasant. C++ likes to pretend that std::variant is decent, but it’s
really quite moderate.  Rust, Haskell, etc all have very nice sum
types.

As the most basic example, C often uses a null pointer to mean
“nothing here”, but you’re on your own to remember not to reference a
null pointer. In Rust a reference can’t be null, and Option can be
used to mean “maybe something here, maybe not”.  You can’t even
attempt to dereference the pointer without checking if it’s present.

For a more interesting example, a pointer to a page table entry could
be arranged to be a union of:

Empty: nothing there, but still has several bits of usable data
NextTable: a pointer to the next level of page tables
Page: a pointer to the physical address of the data along with access
rights and such

And one could manipulate page tables with no fear of forgetting about
the existence of huge pages — if you try to write code that
misinterprets a huge page (Page) as a pointer to the next table down
(NextLevel), the language won’t let you.



### Modules vs Devices Lifespans

[From: Greg KH](https://lore.kernel.org/ksummit/YOWh0Dq+2v+wH3B4@kroah.com/)



> > module code lifespans are different than device structure lifespans,
> > it's when people get them confused, as here, that we have problems.
> 
> I'm not claiming they are.  What I am claiming is that module lifetime
> must always encompass the device lifetimes.  Therefore, you can never
> be incorrect by using a module lifetime for anything attached to a
> device, just inefficient for using memory longer than potentially
> needed.  However, in a lot of use cases, the device is created on
> module init and destroyed on module exit, so the inefficiency is barely
> noticeable.

In almost no use case is the device created on module init of a driver.
devices are created by busses, look at the USB code as an example.  The
usb bus creates the devices and then individual modules bind to that
device as needed.  If the device is removed from the system, wonderful,
the device is unbound, but the module is still loaded.  So if you really
wanted to, with your change, just do a insert/remove of a USB device a
few zillion times and then memory is all gone :(

> The question I'm asking is shouldn't we optimize for this?

No.

> so let people allocate devm memory safe in the knowledge it will be
> freed on module release?

No, see above.  Modules are never removed in a normal system.  devices
are.

And the drm developers have done great work in unwinding some of these
types of mistakes in their drivers, let's not go backwards please.


### Improving the Driver Writing Experience

[From: Laurent Pinchart](https://lore.kernel.org/ksummit/YOXhlDsMAZUn1EBg@pendragon.ideasonboard.com/)


I've given this lots of thoughts lately, in the context of V4L2, but it
should be roughly the same for character devices in general. Here's what
I think should be done for drivers that expose character devices.

- Drivers must stop allocating their internal data structure (the one
  they set as device data with dev_set_drvdata()) with devm_kzalloc().
  The data structure must instead be allocated with a plain kzalloc()
  and reference-counted.

  Most drivers will register a single character device using a
  subsystem-specific API (e.g. video_register_device() in V4L2). The
  subsystem needs to provide a .release() callback, called when the
  last reference to the character device is released. Drivers must
  implement this, and can simply free their internal data structure at
  this point.

  For drivers that register multiple character devices, or in general
  expose multiple interfaces to userspace or other parts of the kernel,
  the internal data structure must be properly reference-counted, with a
  reference released in each .release() callback. There may be ways to
  simplify this.

  This can be seen as going back to the pre-devm_kzalloc() era, but it's
  only about undoing a mistake that was done way too often (to be fair,
  many drivers used to just kfree() the data in the driver's .remove()
  operation, so it wasn't always a regression, only enshrining a
  preexisting bad practice).

  This is only part of the puzzle though. There's a remove/use race that
  still needs to be solved.

- In .remove(), drivers must inform the character device that new access
  from userspace are not allowed anymore. This would set a flag in
  struct cdev that would result in all new calls from userspace through
  file operations to be rejected (with -ENXIO for instance). This should
  be wrapped in subsystem-specific functions (e.g.
  video_device_disconnect() wrapping cdev_disconnect()). From now on, no
  new calls are possible, but existing calls may be in progress.

- Drivers then need to cancel all pending I/O and wait for completion.
  I/O (either direct memory-mapped I/O or through bus APIs, such as I2C
  or USB transactions) are not allowed anymore after .remove() returns.
  This will have the side effect of waking up userspace contexts that
  are waiting for I/O completion (through a blocking file I/O operation
  for instance). Existing calls from userspace will start completing.

  This is also a good place to start shutting down the device, for
  instance disabling interrupts.

- The next step is for drivers to wait until all calls from userspace
  complete. This should be done with a call count in struct cdev that is
  updated upon entry and exit of calls from userspace, and a wait queue
  to wait for that count to go to 0. This should be wrapped in
  subsystem-specific APIs. As the flag that indicates device removal is
  set, no new calls are allowed so the counter can only decrease, and as
  all pending I/O have terminated or have been cancelled, no pending
  calls should be blocked.

- Finally, drivers should unregister the character device, through the
  appropriate subsystem API.

  At this point, memory mappings and file handles referencing the device
  may still exist, but no file operation is in progress. The device is
  quiescent.

  Care needs to be taken in drivers and subsystems to not start any I/O
  operation when handling the file .release() operation or the
  destruction of memory mappings. Overall I don't expect much issues,
  but I'm sure some drivers do strange things in those code paths.

- When the least memory mapping is gone and the last file handle is
  closed, the subsystem will call the driver's .release() callback. At
  this point, the driver will perform the operations listed in the first
  item of this list.


The challenge here will be to make this as easy as possible for drivers,
to avoid risk of drivers getting it wrong. The DRM/KMS subsystem has
created a devres-based system to handle this, with the devres associated
with the drm_device (the abstraction of the cdevs exposed by DRM to
userspace), *not* the physical device. It has a drawback though, it
assumes that a DRM driver will only ever want to register a drm_device
and nothing else, and hardcodes that assumption in the way it releases
resources. That's fine for most DRM drivers, but if a driver was to
register a drm_device and something else (such as a V4L2 video_device,
an input device, ...), the DRM subsystem will get in the way.

I have two questions:

- Does the above make sense ?
- Assuming it does, how do we get from the current mess to a situation
  where writing a driver will be a pleasure, not a punishment ? :-)

--

[From: Greg KH
](https://lore.kernel.org/ksummit/YOagA4bgdGYos5aa@kroah.com/)


> - Assuming it does, how do we get from the current mess to a situation
>   where writing a driver will be a pleasure, not a punishment ? :-)

That's the real question.  Thanks to a lot of cleanups that Christoph
has recently done to the lower-level cdev code, the lower levels are now
in a shape where we can work on them better.  I'm going to try to carve
out some time in the next few months to start to work on these things.
I think that once we get the ideas of what needs to be done, and a
working core change, I can unleash some interns on doing tree-wide
cleanups/changes to help bring everything into alignment.

--

[From: Mauro Carvalho](https://lore.kernel.org/ksummit/20210708110852.1c4f8148@coco.lan/)

Good point. Yeah, indeed some work seems to be required on that area.

Yet, V4L2 is somewhat different here, as most (if not all) devices 
expose multiple cdevs. 

Also, in the case of V4L2 USB and PCI devices, their "dev->parent" is 
usually a PCI bridge, or an USB device with multiple functions on it,
as those hardware contain both audio and video and sometimes input
(either buttons or remote controllers), typically using different 
drivers. So, when the hardware is hot-unplugged or unbind, several 
drivers and multiple cdevs will be released. Ensuring that those will
happen at the right time can be a challenge, specially if there are
pending syscalls and/or threads by the time the device is unbound.

The DRM subsystem likely fits on the same case, as drivers also 
usually create multiple cdevs, and there are DMABUF objects shared
between different struct devices.

So, yeah, using devm_* for V4L2 and DRM can indeed bring troubles.
I can't see a solution there currently but to avoid using devm*,
handling it using an approach similar to the one you described.

-

I'm not so sure if using devm_* is a problem on several cases, though.
I mean, when the hardware is not hot-pluggable, the data lifetime
is a lot simpler.

So, for instance, a regulator driver probably can use devm_* without
any issues, as it doesn't seem to make much sense to unbind a regulator
once the device was probed. On drivers like that, not using devm_*
would just make the probing part of the driver more complex, without
bringing any real benefit.

--

[From: Andy Lutomirski](https://lore.kernel.org/ksummit/CALCETrXbStLhO80n=HehDAXfysvKmJ=5PDD3WzdK3rdzXGAvdw@mail.gmail.com/)

> Don't forget that regulators, GPIO controllers and other such core
> resources can be part of unpluggable devices.

It sounds like this type of use might be a good fit for a simple
precise garbage collector.  This could be arranged in C in a
reasonably safe way (with the assistance of sparse, perhaps).  Every
struct that could contain a GC pointer to a device would have a
special attribute, and the whole graph of devices could be walked as
needed to collect garbage.

Alternatively, weak references might be a good fit for this,
especially if hotplug is involved.  If a device gets hot-unplugged
without warning, every subsequent attempt to follow a weak reference
to that device could be made to fail.

--

[From: Linus Walleij](https://lore.kernel.org/ksummit/CACRpkdasOaNgBAZVx5qpKJdU7h41jHDG2jWi2+pi9a1JBh7RTQ@mail.gmail.com/)


What we did in GPIO to get around the whole issue is to split
our device in two abstractions. The device that spawns a GPIO
driver can be a GPIO on any bus (platform, PCI, SPI, I2C, ...)
then that populates and registers one (or more) gpio_chips.
That is a struct that deal with handling the hardware but does
not contain any struct device.

When registering gpio_chip we create a struct gpio_device which
contains a struct device and a struct cdev (exposed to userspace)
and all in-kernel consumer handles (called struct gpio_desc). This
is reference counted on the struct device and uses
->release() to eventually clean itself up.

The crucial part is what happens when a device with GPIOs
disappears, if e.g. the USB device is unplugged or the driver
is rmmod:ed by force from the command line. We then unregister
the struct gpio_chip and drop all devm_*  resources taken by the
driver (referencing the struct dev in the USB device or so) so these
go away, but the struct gpio_device stays around
until the last reference from userspace is dropped.

In order to not crash calls from the character device the device is
"numbed", so any calls will just return "OK" but nothing happens.
We then hope userspace will be so nice to terminate once it realizes
that it is no longer needed, closing the chardev and releasing the
resources held.

This works for us but I admit it is a bit kludgy. I guess it is not a
generally useful practice, we just had to come up with something.

--

[From: Dan Williams](https://lore.kernel.org/ksummit/CAPcyv4gy5+9Xcd-G4oAVAs6T5UcE8Z75vZv5Sd+CNBGsbiBJ+w@mail.gmail.com/)


We do something similar in the CXL driver. The driver data is
allocated with devm. When ->remove() is triggered the driver data is
disconnected from the cdev (because it's about to be freed by devres)
and all future ioctls attempts fail until userspace finally gives up
and closes the device-file to release the cdev.

I clumsily attempted to solve this problem in a generic way here:

[https://lore.kernel.org/r/CAPcyv4hEpdh_aGcs_73w5KmYWdvR29KB2M2-NNXsaXwxf35Hwg@mail.gmail.com](https://lore.kernel.org/r/CAPcyv4hEpdh_aGcs_73w5KmYWdvR29KB2M2-NNXsaXwxf35Hwg@mail.gmail.com)

...Christoph rightly pointed out that this is something debugfs
already handles with its file_operations proxy implementation. I'm
thinking that concept can be extended for drivers to deploy their
ioctl implementation behind a proxy that handles the ioctl shutdown
synchronization in the core and not have every driver open code it.
See fs/debugfs/file.c::FULL_PROXY_FUNC().

--


[From: Daniel Vetter](https://lore.kernel.org/ksummit/CAKMK7uHgtGc9ncD3LjHzWxF1eOJ5-M+u=45ZG8-vDtgEAHVJ4Q@mail.gmail.com/)

Since we're dropping notes, a few of my thoughts:

- Personally I think an uapi resource cleanup system needs a different
namespace, so that you don't mix these up. Hence drmm_ vs devres_ for
drm, and maybe the generic version could be called uapires or ures or
whatever. It needs to stick out.

- I'm wondering whether we could enlist checkers (maybe a runtime one)
to scream anytime we do an unprotected dereference from ures memory to
devres memory. This would help in subsystem where this problem is
solved by trying to decouple the uapi side from the device side (like
gpio and other subsystem with simpler interfaces). We have undefined
behaviour and data race checkers already, this should be doable. But I
have no idea how :-)

- Core infrastructure like cdev_disconnect sounds really good.
Especially if someone figures out how to punch out mmap without races
in a generic way.

- Many drivers are perfectly ok with their ures on the single cdev
they expose, but like Laurent points out, there's plenty where you
need to group them. We need some way to merge ures groups together so
a driver can say "only release ures for _any_ of these cdev if _all_
of them are released". I'm not sure how to do that, but can also be
done as an additional later on. Maybe an explicit struct ures_group
that complex drivers (or complex subsystems like drm/v4l) can
optionally pass to subordinate uapi interfaces like cdev could make
this work.

- Another good reason for a commont struct to manage uapi resources is
to that we don't need a per-subsystem dupe for everything. Once your
driver is more than a few lines you need more than just drmm_kzalloc,
you might have you own slab, other allocater, a few workqueues,
whatever else there is ....

- Automagic cleanup is great for small drivers, but is hitting a bit a
scaling wall for big drivers. The trouble there is the unwind code to
quiescent the driver with all it's kthread, work queues, workers and
everything else. That needs to happen in a very careful order (if you
flush the work before you disable the interrupt that schedules it,
it's no good) and is an absolute pain to validate. There's a few
reasons why we have barriers here:
* often the same or similar quiescent code is needed for
suspend/resume, so you're not gaining anything if the module unload is
automatic
* unwind as you create/init it not always the right thing to do, or at
least not obviously so, e.g. in i915 we have on interrupt handler, but
it's hiearchical internally, and we arm new subsystems irq sources as
we initialize these parts of the driver
* as soon as you hit something where there's not yet a devres/uapi
wrapper available it gets annoying and giving up is easier :-)

- the real cool stuff starts to happen if you combine devres with
ures, e.g. see devm_drm_dev_alloc(). With that you can achieve drivers
with no ->remove callback that actually get all the lifetime right,
unfortunately we're not there yet, we're missing a
devm_drm_dev_register (to call drm_dev_unplug() outmatically on
remove) and a devm_drm_atomic_reset (which calls drm_atomic_shutdown
on remove) so that for simplest drivers the ->remove hook becomes
empty (e.g. hx8357d_remove()). But you really need the ures side fully
rolled out first or things just go very wrong on hotunplug :-)

Anyway I'm very much interested in this topic and seeing some kind of
solution lifted to core code.

Cheers, Daniel


### Why not C++ in the Linux Kernel

[From: Bart Van Assche](https://lore.kernel.org/ksummit/f391c00d-7f4f-a60c-0230-4aca5ea2d4ed@acm.org/)

As a sidenote, I'm surprised that C++ is not supported for Linux kernel
code since C++ supports multiple mechanisms that are useful in kernel
code, e.g. RAII, lambda functions, range-based for loops, std::span<>
and std::string_view<>. Lambda functions combined with std::function<>
allow to implement type-safe callbacks. Implementing type-safe callbacks
in C without macro trickery is not possible.


--

[From: Linus Torvalds](https://lore.kernel.org/ksummit/CAHk-=wiwZWAo_Ki587FD2BrAQVK71TBN=uKimdBf1Pxg3=+nTw@mail.gmail.com/)

> As a sidenote, I'm surprised that C++ is not supported for Linux kernel
> code since C++ supports multiple mechanisms [..]

You'd have to get rid of some of the complete garbage from C++ for it
to be usable.

One of the trivial ones is "new" - not only is it a horribly stupid
namespace violation, but it depends on exception handling that isn't
viable for the kernel, so it's a namespace violation that has no
upsides, only downsides.

Could we fix it with some kind of "-Dnew=New" trickery? Yes, but
considering all the other issues, it's just not worth the pain. C++ is
simply not a good language. It doesn't fix any of the fundamental
issues in C (ie no actual safety), and instead it introduces a lot of
new problems due to bad designs.

--

[From: Bart Van Assche](https://lore.kernel.org/ksummit/22460501-fe09-f8e7-1051-b6b692500859@acm.org/)


Hi Linus,

Thank you for having shared your opinion. You may want to know that
every C++ project I have worked on so far enabled at least the following
compiler flags: -fno-exceptions and -fno-rtti.

What the C++ operator new does if not enough memory is available depends
on the implementation of that operator. We could e.g. modify the
behavior of operator new as follows:
- Add -fno-builtin-new to the compiler flags.
- Define a custom version of operator new.

An example (user space code):

```c++
#include <stdlib.h>
#include <stdio.h>

void *operator new(size_t size)
{
  printf("%s\n", __func__);
  return malloc(size);
}

void operator delete(void *p)
{
  printf("%s\n", __func__);
  free(p);
}

void operator delete(void *p, size_t size)
{
  printf("%s\n", __func__);
  free(p);
}

void *operator new[](size_t size)
{
  printf("%s\n", __func__);
  return malloc(size);
}

void operator delete[](void *p)
{
  printf("%s\n", __func__);
  free(p);
}

void operator delete[](void *p, size_t size)
{
  printf("%s\n", __func__);
  free(p);
}

int main(int, char **)
{
  int *intp = new int;
  long *arrayp = new long[37];
  delete[] arrayp;
  delete intp;
  return 0;
}

The output of the above code:

operator new
operator new []
operator delete []
operator delete
```

--

[From: Linus Torvalds](https://lore.kernel.org/ksummit/CAHk-=wjYGDtLafGB6wabjZCyPUiTJSda0c8h5+_8BeFNdCdrNg@mail.gmail.com/)

The point is, C++ really has some fundamental problems. Yes, you can
work around them, but it doesn't change the fact that it doesn't
actually *fix* any of the issues that make C problematic.

For example, do you go as far as to disallow classes because member
functions are horrible garbage? Maybe newer versions of C++ fixed it,
but it used to be the case that you couldn't sanely even split a
member function into multiple functions to make it easier to read,
because every single helper function that worked on that class then
had to be declared in the class definition.

Which makes simple things like just re-organizing code to be legible a
huge pain.

At the same time, C++ offer no real new type or runtime safety, and
makes the problem space just bigger. It forces you to use _more_
casts, which then just make for more problems when it turns out the
casts were incorrect and hid the real problem.

So no. We're not switching to a new language that causes pain and
offers no actual upsides.

At least the argument is that Rust _fixes_ some of the C safety
issues. C++ would not.

--

[From: Laurent Pinchart](https://lore.kernel.org/ksummit/YOYfPWW1zgrL4DiQ@pendragon.ideasonboard.com/)

> Which makes simple things like just re-organizing code to be legible a
> huge pain.
> 
> At the same time, C++ offer no real new type or runtime safety, and
> makes the problem space just bigger. It forces you to use _more_
> casts, which then just make for more problems when it turns out the
> casts were incorrect and hid the real problem.

I beg to differ on that one. There are features of C++ that would be
very helpful for kernel development. Among them is native support for
destructors, which allow implementing RAII idioms. Unique pointers would
also be very helpful to explicitly expose object ownership rules (shared
pointers are a different story though, it's very easy to use them
incorrectly and infect the whole code base as a result). Templates are
another feature that is widely criticized (and often for good reasons),
but when seen as a type-safe implementation of macros, they can bring
increased safety to the code base.

C++ has upsides and fixes real issues. It also causes pain, and it's not
the only language to provide the above features, so I wouldn't call for
its usage in the kernel. I just wish we had objects with destructors in
plain C.

--

[Miguel Ojeda](https://lore.kernel.org/ksummit/CANiq72nKao=rz89yajChtsM8Nvv2LM-xZfX+iwk686SDMhv5iw@mail.gmail.com/)

The issue is that, even if people liked C++ a lot, there is little
point in using C++ once Rust is an option.

Even if you discuss "modern C++" (i.e. post-C++11, and even
post-C++17), there is really no comparison.

For instance, you mentioned `std::span` from the very latest C++20
standard; let's build one:

```c++
    std::span<int> f() {
        int a[] = { foo() };
        std::span<int> s(a);
        return s;
    }
```

Now anybody that accesses the returned `std::span` has just introduced
UB. From a quick test, neither Clang nor GCC warn about it. Even if
they end up detecting such a simple case, it is impossible to do so in
the general case.

Yes, it is a stupid mistake, we should not do that, and the usual
arguments. But the point is UB is still as easy as has always been to
introduce in both C and C++. In Rust, that mistake does not happen.

--

[From: Miguel Ojeda](https://lore.kernel.org/ksummit/CANiq72=o5hKZyFqnGvd-3LeqjbR+JDsWhf=rJkimTKQSqf45pg@mail.gmail.com/)


> You're comparing apples and pears though. A C++ function that is meant
> to transfer unique ownership of an object to the caller should return a
> std::unique_ptr<> on a container that stores the data. We're getting

Nope, I am not comparing apples and pears. I just showed you a trivial
way to make UB in C++20 with one of the types someone else mentioned.

You mention `std::unique_ptr` now. Same deal:

```c++
    std::unique_ptr<int> f() {
        return {}; // returns a `nullptr`
    }
```

You will now reply: "oh, but you are *not* supposed to use it like
that!". But the point is: it is not about the simple examples, it is
about the complex cases where objects are alive for a long time,
passed across chains of calls, manipulated in several places, across
different threads, etc., etc. so that reasoning is non-local.

Don't get me wrong, `std::unique_ptr` is nice, and I have used it many
times in my career with good results. Still, it is far from what Rust
offers.

Another extremely typical example:

```c++
    std::vector<int> v;
    ...
    int * p = v.data(); // looks OK
    ...
    v.push_back(42); // looks OK
    ...
    f(p); // oh, wait...
```

> off-topic though, this mail thread isn't about comparing C++ and rust
> :-)

Well, if people bring up C++ as an alternative to Rust, they are
implying Rust does not offer much more than C++. Which is false,
misleading, and directly counters the Rust support proposal, thus I
feel the need to answer.

Again, don't get me wrong: while one can definitely see Rust as a
"cleaned up" C/C++ (it is, in a way); the key is that it *also* offers
major advantages using a few new research ideas that no other system
language had (even SPARK had to catch up [1]). It is not just a
sweeter syntax or a few fancy features here and there as many seem to
imply in many fora.

[1] [https://blog.adacore.com/using-pointers-in-spark](https://blog.adacore.com/using-pointers-in-spark)
