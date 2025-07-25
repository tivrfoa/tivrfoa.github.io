---
layout: post
title:  "Intrusive Data Structures"
date:   2025-07-27 08:10:00 -0300
categories: rust data-structure
---

- [What is a Intrusive Data Structures?](#why-use-intrusive-data-structures)
- [Why use them?](#why-use-intrusive-data-structures)
- [How they look like?](#how-they-look-like)
  - [C example](#c-example)
  - [C++ example](#c++-example)
  - [Rust example](#rust-example)
  - [Zig example](#zig-example)
- [Quotes](#quotes)
- [References](#references)


# What is a Intrusive Data Structures?

# Why use them?

## From: Boost - Intrusive and non-intrusive containers

> Intrusive containers have some important advantages:
>
>    - Operating with intrusive containers doesn't invoke any memory management at all. The time and size overhead associated with dynamic memory can be minimized.
>    - Iterating an Intrusive container needs less memory accesses than the semantically equivalent container of pointers: iteration is faster.
>    - Intrusive containers offer better exception guarantees than non-intrusive containers. In some situations intrusive containers offer a no-throw guarantee that can't be achieved with non-intrusive containers.
>    - The computation of an iterator to an element from a pointer or reference to that element is a constant time operation (computing the position of T* in a std::list<T*> has linear complexity).
>    - Intrusive containers offer predictability when inserting and erasing objects since no memory management is done with intrusive containers. Memory management usually is not a predictable operation so complexity guarantees from non-intrusive containers are looser than the guarantees offered by intrusive containers.
>
>[https://www.boost.org/doc/libs/1_60_0/doc/html/intrusive/intrusive_vs_nontrusive.html](https://www.boost.org/doc/libs/1_60_0/doc/html/intrusive/intrusive_vs_nontrusive.html)
>
>Performance comparison between Intrusive and Non-intrusive containers:
>[https://www.boost.org/doc/libs/1_60_0/doc/html/intrusive/performance.html](https://www.boost.org/doc/libs/1_60_0/doc/html/intrusive/performance.html)

## From: Crate intrusive_collections

>The main difference between an intrusive collection and a normal one is that while normal collections allocate memory behind your back to keep track of a set of values, intrusive collections never allocate memory themselves and instead keep track of a set of objects. Such collections are called intrusive because they requires explicit support in objects to allow them to be inserted into a collection.
>
>[https://docs.rs/intrusive-collections/latest/intrusive_collections/](https://docs.rs/intrusive-collections/latest/intrusive_collections/)

>Amanieu:
>
>The main use case for intrusive collections is to allow an object to be a member of multiple collections simultaneously. Say you are writing a game and you have a set of objects. You will have one main list of all objects, and a separate list of objects that are "owned" by a player. If a player dies, all objects owned by him need to be removed. This can be implemented by making objects belong to two lists at once.
>
>[https://www.reddit.com/r/rust/comments/46yubp/comment/d090zxg/](https://www.reddit.com/r/rust/comments/46yubp/comment/d090zxg/)

## From: Data Structures in the Linux Kernel - Doubly linked list

>Usually a linked list structure contains a pointer to the item. The implementation of linked list in Linux kernel does not. So the main question is - where does the list store the data? The actual implementation of linked list in the kernel is - Intrusive list. An intrusive linked list does not contain data in its nodes - A node just contains pointers to the next and previous node and list nodes part of the data that are added to the list. This makes the data structure generic, so it does not care about entry data type anymore.
>
>[https://github.com/0xAX/linux-insides/blob/master/DataStructures/linux-datastructures-1.md](https://github.com/0xAX/linux-insides/blob/master/DataStructures/linux-datastructures-1.md)

This link is great and it has a clear explanation and examples. Take a look! =)

## Gemini Definition

A. Definition and Core Characteristics

Intrusive data structures represent a specialized class of linked data structures where the organizational metadata, such as next and prev pointers for a list or child/parent pointers for a tree, is embedded directly within the user's data object. This contrasts sharply with traditional data structures, which typically reside in a separate, externally managed node object. The design paradigm of intrusive structures necessitates that the data object itself "cooperates" with the data structure by containing the necessary "hooks" or linking fields.  

A fundamental characteristic of this approach is that intrusive containers do not store copies of user-provided values; instead, they directly manage and link the original objects, with the linking information intrinsically part of those objects. This means that the nodes of the data structure contain only the metadata (i.e., pointers to other nodes) but not the data itself. The programmer embeds these nodes into their data, thereby avoiding additional pointer indirections. This paradigm suggests a profound shift in design philosophy: **instead of a container owning and managing external nodes that point to data, the data itself becomes "aware" of its participation in a data structure. This "self-awareness" (or "intrusion")** is the root cause of both the primary benefits, such as memory efficiency and cache locality, and the primary drawbacks, including the requirement for object modification and more complex lifetime management. This approach points to a tighter coupling between the data and its organizational structure, which is a hallmark of low-level systems programming and high-performance computing.

More here:
[https://g.co/gemini/share/2ed8ecea357e](https://g.co/gemini/share/2ed8ecea357e)

## Nice comment by tornewuff



# Why use them?

Gemini:

>The core motivations for their use are multifaceted and directly tied to performance optimization. They aim to avoid costly pointer indirections, which can incur run-time costs on every data read. Furthermore, they minimize dynamic memory allocations, which can be computationally expensive and unpredictable. A significant advantage is their ability to facilitate the inclusion of a single data object in multiple distinct data structures simultaneously. For example, an element might be part of several search trees and a priority queue, allowing efficient retrieval in different orders.


# Quotes

## [https://news.ycombinator.com/item?id=43682475](https://news.ycombinator.com/item?id=43682475)

Diggsey:

If you just mean using array indexes instead of pointers, that can work in some cases, but can run into concurrency issues when the container needs to be resized. With pointers to nodes, new nodes can be allocated and old nodes freed on a separate thread without interfering with the performance sensitive thread, whereas resizing an array requires all access to stop while the backing memory is re-allocated.

I don't think you mean this, but elaborating on why a non-intrusive linked list is not very useful, that's because it takes linear time to find an object in a non-intrusive linked list, but constant time in an intrusive linked list.

For example:

```c
    struct Node {
        Node* prev_by_last_update;
        Node* next_by_last_update;
        Node* prev_with_owner;
        Node* next_with_owner;
        Payload data;
    }
```

This intrusive structure allows reassigning and updating nodes, whilst being able to efficiently find eg. the next most recently updated node, or to iterate over nodes with the same owner.

## [https://users.rust-lang.org/t/what-are-intrusive-linked-lists-and-does-rust-really-struggle-with-them/117943/24](https://users.rust-lang.org/t/what-are-intrusive-linked-lists-and-does-rust-really-struggle-with-them/117943/24)

Reply from tornewuff:

>>akrauze:
>>
>>Still, how does this differ from something like this?
>>
>>```rs
>>    struct DoubleNode<T> {
>>        first_next: *mut DoubleNode,
>>        first_priv: *mut DoubleNode,
>>        second_next: *mut DoubleNode,
>>        second_priv: *mut DoubleNode,
>>        data: T,
>>    }
>>```

Yes, your structure definition here (which includes the data by value, not a pointer) gives you the ability to put an object on more than one linked list at once without needing to be intrusive. That's not the only benefit of intrusive linked lists, though.

Intrusive linked lists are typically used in cases where the number and kinds of lists that a particular data type is going to be part of is tightly tied to the definition of the data type. When that's the case, there are some advantages to having the pointers be part of the data type itself, instead of a wrapper:

- One of the benefits of doubly-linked lists is that there are a number of ways to manipulate the list using just a reference to a particular member of the list, without needing a reference to the head of the list (e.g. removing the object, or inserting another object immediately before/after the current one). With a non-intrusive list, only functions that are given a reference to the actual list node can do these things, but with intrusive pointers any methods you define on the data type itself can do these kinds of list manipulations since the list pointers are just like any other member.
  - This is probably less likely to come up in most Rust code, though, because this is effectively a really tricky kind of shared mutability that isn't going to be easy to provide a safe API for, even for the case of things only being on a single list.
- With a wrapper you would need to copy/move the T into the wrapper (or construct it in-place) before you can actually make it part of a list. With intrusive pointers, any instance of the type can always be added to a list without paying any copy/move costs and without its address changing. This is particularly important if there are also other pointers to the object that aren't related to the lists: for example, a process object might be on one or more doubly-linked lists for various reasons, but each thread might also have a regular pointer to the process that owns it.
  - This is also probably less likely to come up in most Rust code, though, because Rust requires everything to be movable under most circumstances and also guarantees that moves are just a memcpy at worst.
- You can name the pointers something use-case-specific, instead of "first" and "second". Having the definition of T contain foo: DoubleLinkPtrs and bar: DoubleLinkPtrs members makes it clear that it's supposed to be on one "foo" list and one "bar" list.

Another difference that doesn't apply to Rust is that in a language without generics, you can generally only store the data by-value by writing one wrapper for each type of data to be stored - at which point there isn't much benefit to having it be a separate wrapper instead of just part of the data type.

So, yeah - typically you're going to see intrusive lists in cases where there are specific benefits to having the data type and the list(s) it is a part of be tied tightly together. Most of the time, these don't really apply, and it's going to be simpler and cleaner for data to not be aware of its container.

# Rust for Linux

## Add container_of and offset_of macros

[https://github.com/Rust-for-Linux/linux/pull/158](https://github.com/Rust-for-Linux/linux/pull/158)


```
impl_list_item!
    container_of!
```


## `macro_rules! container_of`

[https://github.com/Rust-for-Linux/linux/blob/28753212e0f9c61afd859acf1d678f5de7faa4b8/rust/kernel/lib.rs#L238C1-L238C26](https://github.com/Rust-for-Linux/linux/blob/28753212e0f9c61afd859acf1d678f5de7faa4b8/rust/kernel/lib.rs#L238C1-L238C26)

## `macro_rules! impl_has_list_links`

### offset_of

[https://github.com/Rust-for-Linux/linux/blob/28753212e0f9c61afd859acf1d678f5de7faa4b8/rust/kernel/list/impl_list_item_mod.rs#L47](https://github.com/Rust-for-Linux/linux/blob/28753212e0f9c61afd859acf1d678f5de7faa4b8/rust/kernel/list/impl_list_item_mod.rs#L47)


# References

[https://github.com/0xAX/linux-insides/blob/master/DataStructures/linux-datastructures-1.md](https://github.com/0xAX/linux-insides/blob/master/DataStructures/linux-datastructures-1.md)

[Pacific++ 2017: Matt Bentley "Can we make a faster linked list?"](https://www.youtube.com/watch?v=SPXNy0-dUbw)

[Gemini: Intrusive Data Structures Analysis](https://g.co/gemini/share/2ed8ecea357e)

[Zig's @fieldParentPtr for dumbos like me](https://www.ryanliptak.com/blog/zig-fieldparentptr-for-dumbos/)

[Adopt intrusive data structures for better performance](https://github.com/doitsujin/dxvk/issues/3796)

## Rust

[https://docs.rs/intrusive-collections/latest/intrusive_collections/](https://docs.rs/intrusive-collections/latest/intrusive_collections/)

[Safe Intrusive Collections with Pinning](https://internals.rust-lang.org/t/safe-intrusive-collections-with-pinning/7281)

[What are intrusive linked lists and does Rust really struggle with them?](https://users.rust-lang.org/t/what-are-intrusive-linked-lists-and-does-rust-really-struggle-with-them/117943)

[https://www.kernel.org/doc/rustdoc/next/kernel/list/struct.List.html](https://www.kernel.org/doc/rustdoc/next/kernel/list/struct.List.html)

[https://rust.docs.kernel.org/src/kernel/list.rs.html#255](https://rust.docs.kernel.org/src/kernel/list.rs.html#255)

[https://github.com/bbatha/movecell_graph/blob/master/src/lib.rs](https://github.com/bbatha/movecell_graph/blob/master/src/lib.rs)
