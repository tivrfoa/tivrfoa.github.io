---
layout: post
title:  "Rust in-kernel TLS handshake"
date:   2023-01-21 11:30:00 -0300
categories: Rust Linux kernel TLS
---

<!--
<div style="background-color: #a2b9bc; padding: 15px; border-radius: 10px; margin-bottom: 10px">
Me: Doesn't the second version creates new String objects on every call, hence causing an impact in GC?
</div>
-->

[https://github.com/fujita/rust-tls](https://github.com/fujita/rust-tls)

>This experiment is for figuring out how well Rust could work for in-kernel TLS 1.3 handshake.
>
>There is some debate over in-kernel TLS handshake mainly because of the complexity. Rust could help >auditing the complicated security-relevant code.
>
>This can establish a QUIC connection with Quinn's example client, Rust QUIC implementation. Only minimum >server side functionality and connection establishment are supported for now.
>
>I'll work on Rust crypto support for mainline. Meanwhile you can compile this kernel module with my fork of Linux kernel.
>
>$ make KDIR=~/git/linux LLVM=1


Quick look at the code ...

## Use lots of *mut

eg:

```rust
struct QuicWork {
    sock: *mut bindings::sock,
    work: workqueue::Work,
}
```

Why do you need `*mut` in Rust?

[https://doc.rust-lang.org/std/keyword.mut.html](https://doc.rust-lang.org/std/keyword.mut.html)

>Mutable raw pointers work much like mutable references, with the added possibility of not pointing to a valid object. The syntax is *mut Type.

[https://doc.rust-lang.org/reference/types/pointer.html#mutable-references-mut](https://doc.rust-lang.org/reference/types/pointer.html#mutable-references-mut)

>Raw pointers (*const and *mut)
>
>Raw pointers are pointers without safety or liveness guarantees. Raw pointers are written as *const T or *mut T. For example *const i32 means a raw pointer to a 32-bit integer. Copying or dropping a raw pointer has no effect on the lifecycle of any other value. Dereferencing a raw pointer is an unsafe operation. This can also be used to convert a raw pointer to a reference by reborrowing it (&* or &mut *). Raw pointers are generally discouraged; **they exist to support interoperability with foreign code, and writing performance-critical or low-level functions.**


## Return `Result` in many places

This makes the logic more clean, with the use of `?`

