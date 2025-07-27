---
layout: post
title:  "Intrusive Data Structures"
date:   2025-07-27 08:10:00 -0300
categories: rust data-structure
---
<style>
h1 {
  font-size: 2.5em;
}
h2 {
  font-size: 2em;
}
</style>

- [What is an Intrusive Data Structure?](#why-use-intrusive-data-structure)
- [Why use them?](#why-use-them)
- [Examples](#examples)
  - [C](#c)
  - [C Generic List](#c-generic-list)
  - [Rust](#rust)
  - [Zig](#zig)
- [Comparing `container_of` implementations](#comparing-container_of-implementations)
- [References](#references)

# What is an Intrusive Data Structure?

A simple linked list in C:

```c
typedef struct Node {
    int data;
    struct Node *next;
} Node;
```

An **intrusive** linked list:

```c
typedef struct IntrusiveLink {
    struct IntrusiveLink *next;
} IntrusiveLink;
```

Where is the **data**?! =)

Here it is:

```c
// Our data structure that we want to put into the list
// It *must* contain an IntrusiveLink member.
typedef struct MyIntData {
    int value;           // The actual data we care about
    IntrusiveLink node;  // The embedded list hook
} MyIntData;
```

How it works?

The real *magic* is in this macro:

```c
// ptr: pointer to the member (e.g., &myIntDataInstance->node)
// type: the type of the containing struct (e.g., MyIntData)
// member: the name of the member (e.g., node)
#define container_of(ptr, type, member) ({ \
    const typeof( ((type *)0)->member ) *__mptr = (ptr); \
    (type *)( (char *)__mptr - offsetof(type,member) );})
```

With this macro, you can go to the struct containing the intrusive node (*container_of*) by passing the node pointer and the name of the field, eg:

```c
MyIntData* data_item = container_of(current_link, MyIntData, node);
printf("%d -> ", data_item->value);
```

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

Intrusive data structures represent a specialized class of linked data structures where the organizational metadata, such as next and prev pointers for a list or child/parent pointers for a tree, is embedded directly within the user's data object. This contrasts sharply with traditional data structures, which typically reside in a separate, externally managed node object. The design paradigm of intrusive structures necessitates that the data object itself "cooperates" with the data structure by containing the necessary "hooks" or linking fields.  

A fundamental characteristic of this approach is that intrusive containers do not store copies of user-provided values; instead, they directly manage and link the original objects, with the linking information intrinsically part of those objects. This means that the nodes of the data structure contain only the metadata (i.e., pointers to other nodes) but not the data itself. The programmer embeds these nodes into their data, thereby avoiding additional pointer indirections. This paradigm suggests a profound shift in design philosophy: **instead of a container owning and managing external nodes that point to data, the data itself becomes "aware" of its participation in a data structure. This "self-awareness" (or "intrusion")** is the root cause of both the primary benefits, such as memory efficiency and cache locality, and the primary drawbacks, including the requirement for object modification and more complex lifetime management. This approach points to a tighter coupling between the data and its organizational structure, which is a hallmark of low-level systems programming and high-performance computing.

More here:
[https://g.co/gemini/share/2ed8ecea357e](https://g.co/gemini/share/2ed8ecea357e)

# Why use them?

## From: Boost - Intrusive and non-intrusive containers

Intrusive containers have some important advantages:

- Operating with intrusive containers doesn't invoke any memory management at all. The time and size overhead associated with dynamic memory can be minimized.
- Iterating an Intrusive container needs less memory accesses than the semantically equivalent container of pointers: iteration is faster.
- Intrusive containers offer better exception guarantees than non-intrusive containers. In some situations intrusive containers offer a no-throw guarantee that can't be achieved with non-intrusive containers.
- The computation of an iterator to an element from a pointer or reference to that element is a constant time operation (computing the position of T* in a std::list<T*> has linear complexity).
- Intrusive containers offer predictability when inserting and erasing objects since no memory management is done with intrusive containers. Memory management usually is not a predictable operation so complexity guarantees from non-intrusive containers are looser than the guarantees offered by intrusive containers.

[https://www.boost.org/doc/libs/1_60_0/doc/html/intrusive/intrusive_vs_nontrusive.html](https://www.boost.org/doc/libs/1_60_0/doc/html/intrusive/intrusive_vs_nontrusive.html)

Performance comparison between Intrusive and Non-intrusive containers:
[https://www.boost.org/doc/libs/1_60_0/doc/html/intrusive/performance.html](https://www.boost.org/doc/libs/1_60_0/doc/html/intrusive/performance.html)

## Gemini:

The core motivations for their use are multifaceted and directly tied to performance optimization. They aim to avoid costly pointer indirections, which can incur run-time costs on every data read. Furthermore, they minimize dynamic memory allocations, which can be computationally expensive and unpredictable. A significant advantage is their ability to facilitate the inclusion of a single data object in multiple distinct data structures simultaneously. For example, an element might be part of several search trees and a priority queue, allowing efficient retrieval in different orders.

# Examples

## C

By Gemini:

```c
#include <stdio.h>
#include <stdlib.h> // For malloc, free
#include <stddef.h> // For offsetof

// Define the "list hook" that will be embedded in our data structures
// For a singly linked list, we only need a 'next' pointer.
typedef struct IntrusiveLink {
    struct IntrusiveLink *next;
} IntrusiveLink;

// Our data structure that we want to put into the list
// It *must* contain an IntrusiveLink member.
typedef struct MyIntData {
    int value;           // The actual data we care about
    IntrusiveLink node;  // The embedded list hook
} MyIntData;

MyIntData* newIntData(int value) {
    MyIntData* data = (MyIntData*)malloc(sizeof(MyIntData));
    data->value = value;
    data->node.next = NULL; // Initialize its embedded link
    return data;
}

// A macro to get the containing structure from a pointer to its member
// This is crucial for intrusive lists.
// ptr: pointer to the member (e.g., &myIntDataInstance->node)
// type: the type of the containing struct (e.g., MyIntData)
// member: the name of the member (e.g., node)
#define container_of(ptr, type, member) ({ \
    const typeof( ((type *)0)->member ) *__mptr = (ptr); \
    (type *)( (char *)__mptr - offsetof(type,member) );})

// Function to insert a new data structure at the beginning of the intrusive list
// headRef: A pointer to the head of the list (which is an IntrusiveLink*)
// new_data_ptr: A pointer to the MyIntData instance to add
void intrusive_insert_at_beginning(IntrusiveLink** headRef, MyIntData* new_data_ptr) {
    // The new node's 'next' points to the current head
    new_data_ptr->node.next = *headRef;
    // The head is updated to point to the new node's embedded link
    *headRef = &(new_data_ptr->node);
}

// Function to print the intrusive linked list
void intrusive_print_list(IntrusiveLink* head) {
    IntrusiveLink* current_link = head;
    printf("Intrusive Linked List: ");
    while (current_link != NULL) {
        // Use container_of to get the MyIntData structure from its embedded link
        MyIntData* data_item = container_of(current_link, MyIntData, node);
        printf("%d -> ", data_item->value);
        current_link = current_link->next;
    }
    printf("NULL\n");
}

// Function to free the memory allocated for the intrusive linked list
// Note: This frees the MyIntData objects themselves, not just "nodes".
void intrusive_free_list(IntrusiveLink** headRef) {
    IntrusiveLink* current_link = *headRef;
    IntrusiveLink* next_link;
    while (current_link != NULL) {
        next_link = current_link->next;
        // Get the containing data structure
        MyIntData* data_item = container_of(current_link, MyIntData, node);
        free(data_item); // Free the entire MyIntData object
        current_link = next_link;
    }
    *headRef = NULL;
    printf("Intrusive List freed.\n");
}

int main() {
    // The head of our intrusive list. It points to an IntrusiveLink
    // embedded within a MyIntData structure.
    IntrusiveLink* head = NULL;

    MyIntData* data1 = newIntData(10);
    MyIntData* data2 = newIntData(20);
    MyIntData* data3 = newIntData(30);
    MyIntData* data4 = newIntData(40);

    // Insert elements into the intrusive list
    // We pass the address of the embedded 'node' member
    intrusive_insert_at_beginning(&head, data1); // List: 10 -> NULL
    intrusive_insert_at_beginning(&head, data2); // List: 20 -> 10 -> NULL
    intrusive_insert_at_beginning(&head, data3); // List: 30 -> 20 -> 10 -> NULL
    intrusive_insert_at_beginning(&head, data4); // List: 40 -> 30 -> 20 -> 10 -> NULL

    // Print the list
    intrusive_print_list(head);

    // Add another element
    MyIntData* data5 = (MyIntData*)malloc(sizeof(MyIntData));
    data5->value = 5;
    data5->node.next = NULL;
    intrusive_insert_at_beginning(&head, data5);
    intrusive_print_list(head);

    // Free the allocated memory
    intrusive_free_list(&head);

    // Try to print after freeing (should show NULL or be empty)
    intrusive_print_list(head);

    return 0;
}
```

## C Generic List

By Gemini:

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <stddef.h> // <-- Add this header for offsetof


// Define a generic intrusive node structure.
// Any structure that wants to be part of an intrusive list will embed this.
typedef struct IntrusiveNode {
    struct IntrusiveNode *next;
} IntrusiveNode;

// The LinkedList now just holds a pointer to the head of the intrusive nodes.
typedef struct IntrusiveLinkedList {
    IntrusiveNode *head;
} IntrusiveLinkedList;

void initIntrusiveList(IntrusiveLinkedList *list) {
    list->head = NULL;
}

// Function to insert a node at the end of the intrusive list.
// The 'node_to_insert' is already part of the data structure.
void intrusiveInsertEnd(IntrusiveLinkedList *list, IntrusiveNode *node_to_insert) {
    node_to_insert->next = NULL; // Ensure the new node points to NULL initially

    if (list->head == NULL) {
        list->head = node_to_insert;
        return;
    }

    IntrusiveNode *temp = list->head;
    while (temp->next != NULL) {
        temp = temp->next;
    }
    temp->next = node_to_insert;
}

// Function to print the intrusive list.
// Requires a print function that can cast the IntrusiveNode* back to the original struct.
void printIntrusiveList(IntrusiveLinkedList *list, void (*printFunc)(IntrusiveNode *)) {
    IntrusiveNode *temp = list->head;
    while (temp != NULL) {
        printFunc(temp);
        printf(", ");
        temp = temp->next;
    }
    printf("\n");
}

// The free function for an intrusive list typically does nothing, as the memory
// for the nodes is managed by the data structures themselves.
// If the data structures themselves allocate internal memory (like char* for name),
// that should be freed by the caller or a specific free function for the data type.
void freeIntrusiveList(IntrusiveLinkedList *list) {
    // In an intrusive list, we don't free the nodes here,
    // as they are part of larger allocated structures.
    // The responsibility of freeing the memory lies with the owner of the data structures.
    list->head = NULL; // Just clear the head of the list
}

// Example data structure: Person, now containing the intrusive node.
typedef struct Person {
    IntrusiveNode node; // Embedded intrusive node
    char *name;
    int age;
} Person;

// Helper to get the containing Person struct from the IntrusiveNode
// This is a common pattern for intrusive lists, often implemented with offsetof
// For simplicity, here we'll assume a direct cast is safe for this example,
// but for more complex scenarios, offsetof and container_of macro are preferred.
#define container_of(ptr, type, member) ({ \
    const typeof( ((type *)0)->member ) *__mptr = (ptr); \
    (type *)( (char *)__mptr - offsetof(type,member) );})


void printPerson(IntrusiveNode *node) {
    // Cast the IntrusiveNode back to the containing Person struct
    Person *p = container_of(node, Person, node);
    printf("%s is %d years old", p->name, p->age);
}

// Example data structure: IntegerWrapper, for demonstrating integers
typedef struct IntegerWrapper {
    IntrusiveNode node;
    int value;
} IntegerWrapper;

void printInt(IntrusiveNode *node) {
    IntegerWrapper *iw = container_of(node, IntegerWrapper, node);
    printf("%d ", iw->value);
}


void testIntList() {
    IntrusiveLinkedList list;
    initIntrusiveList(&list);

    // Allocate IntegerWrapper structs on the heap or stack
    IntegerWrapper val1 = {.value = 10};
    IntegerWrapper val2 = {.value = 20};
    IntegerWrapper val3 = {.value = 30};
    IntegerWrapper val4 = {.value = 40};

    intrusiveInsertEnd(&list, &val1.node);
    intrusiveInsertEnd(&list, &val2.node);
    intrusiveInsertEnd(&list, &val3.node);
    intrusiveInsertEnd(&list, &val4.node);

    printIntrusiveList(&list, printInt);

    // For stack-allocated data, no explicit free is needed here.
    // If allocated on heap, they would need to be freed by the caller.
    freeIntrusiveList(&list); // This just clears the list head, doesn't free nodes
}

void testPersonList() {
    IntrusiveLinkedList list;
    initIntrusiveList(&list);

    // Allocate Person structs on the heap to demonstrate memory management
    Person *p1 = (Person *)malloc(sizeof(Person));
    p1->name = strdup("Marco"); // strdup allocates memory for the string
    p1->age = 22;

    Person *p2 = (Person *)malloc(sizeof(Person));
    p2->name = strdup("Mary");
    p2->age = 20;

    intrusiveInsertEnd(&list, &p1->node);
    intrusiveInsertEnd(&list, &p2->node);

    printIntrusiveList(&list, printPerson);

    // When dealing with heap-allocated intrusive nodes, you must manually free them
    // along with any dynamically allocated members (like 'name' here).
    // This often involves iterating the list and freeing each container.
    IntrusiveNode *current = list.head;
    IntrusiveNode *next;
    while (current != NULL) {
        next = current->next;
        Person *p = container_of(current, Person, node);
        free(p->name); // Free the dynamically allocated name
        free(p);        // Free the Person struct itself
        current = next;
    }

    freeIntrusiveList(&list); // Clears the list head
}

int main() {
    testIntList();
    printf("\n");
    testPersonList();

    return 0;
}
```

## Rust

```rs
#![allow(unused_variables)] // Allow unused variables for demonstration purposes
#![allow(dead_code)] // Allow dead code for demonstration purposes

use std::{mem, ptr}; // Import mem for offset_of, ptr for raw pointer operations

// Helper to get the containing struct from the IntrusiveNode.
// This is the Rust equivalent of the C `container_of` macro.
// It relies on `offset_of!` from `core::mem`.
// IMPORTANT: This macro MUST be defined BEFORE any code that uses it.
macro_rules! container_of {
    ($ptr:expr, $Type:ty, $member:ident) => {{
        // Ensure the pointer is mutable. `*mut u8` is the generic byte pointer.
        let member_ptr = $ptr as *mut u8;
        // Calculate the offset of the member within the struct.
        // This requires the actual type name ($Type) and member name ($member).
        let offset = mem::offset_of!($Type, $member);
        // Subtract the offset from the member's address to get the struct's base address.
        (member_ptr.sub(offset)) as *mut $Type
    }};
}

// Define a generic intrusive node structure.
// Any structure that wants to be part of an intrusive list will embed this.
// #[repr(C)] ensures C-compatible memory layout.
#[repr(C)]
pub struct IntrusiveNode {
    pub next: *mut IntrusiveNode, // Raw pointer to the next node
}

impl IntrusiveNode {
    // A constructor for a new (unlinked) node.
    pub const fn new() -> Self {
        IntrusiveNode {
            next: ptr::null_mut(), // Initialize next as a null pointer
        }
    }
}

// The IntrusiveLinkedList now just holds a raw pointer to the head.
pub struct IntrusiveLinkedList {
    head: *mut IntrusiveNode, // Raw pointer to the head of the intrusive nodes
}

impl IntrusiveLinkedList {
    pub fn new() -> Self {
        IntrusiveLinkedList {
            head: ptr::null_mut(),
        }
    }

    // Function to insert a node at the end of the intrusive list.
    // 'node_to_insert' is a raw pointer to the embedded IntrusiveNode.
    // This function is `unsafe` because it deals with raw pointers and
    // doesn't guarantee memory safety on its own.
    pub unsafe fn intrusive_insert_end(&mut self, node_to_insert: *mut IntrusiveNode) {
        // Ensure the new node points to NULL initially
        unsafe {
            (*node_to_insert).next = ptr::null_mut();
        }

        if self.head.is_null() {
            self.head = node_to_insert;
            return;
        }

        let mut temp = self.head;
        unsafe {
            while !(*temp).next.is_null() {
                temp = (*temp).next;
            }
            (*temp).next = node_to_insert;
        }
    }

    // Function to print the intrusive list.
    // Requires a print function that can cast the IntrusiveNode* back to the original struct.
    pub unsafe fn print_intrusive_list(&self, print_func: unsafe fn(*mut IntrusiveNode)) {
        let mut temp = self.head;
        while !temp.is_null() {
            unsafe {
                print_func(temp);
            }
            print!(", ");
            unsafe {
                temp = (*temp).next;
            }
        }
        println!();
    }

    // This function will iterate through the list and free the containing structures.
    // This is the user's responsibility in an intrusive list.
    // This is highly unsafe and requires knowledge of the actual type.
    pub unsafe fn free_list_person(&mut self) {
        println!("free_list_person");
        // let mut current = self.head;
        // while !current.is_null() {
        //     let next_node = (*current).next; // Save the next pointer before freeing current
        //
        //     // Cast the IntrusiveNode back to the containing Person struct using the macro
        //     println!("casting");
        //     let p_ptr = container_of!(current, Person, node);
        //
        //     // Reconstruct Box from raw pointer and let it drop, which frees 'name' and 'Person'
        //     println!("reconstructing box");
        //     let _ = Box::from_raw(p_ptr);

        //     current = next_node;
        // }
        println!("clearing list");
        self.head = ptr::null_mut(); // Clear the list head after freeing all elements
    }
}

// Example data structure: Person, now containing the intrusive node.
// #[repr(C)] is essential for predictable memory layout.
#[repr(C)]
pub struct Person {
    pub node: IntrusiveNode, // Embedded intrusive node
    pub name: String,
    pub age: i32,
}

impl Drop for Person {
    fn drop(&mut self) {
        // `String` handles its own memory, so no explicit free for `name` here.
        // This `Drop` impl will be called when a `Person` value goes out of scope
        // or is explicitly dropped (e.g., via `Box::from_raw`).
        println!("Dropping Person: {}", self.name); // For debugging drops
    }
}

// Print function for Person. Takes a raw pointer to IntrusiveNode.
unsafe fn print_person(node_ptr: *mut IntrusiveNode) {
    unsafe {
        let person_ptr = container_of!(node_ptr, Person, node);
        let person_ref = &*person_ptr; // Dereference the raw pointer to get a reference
        print!("{} is {} years old", person_ref.name, person_ref.age);
    }
}

// Example data structure: IntegerWrapper, for demonstrating integers
#[repr(C)]
pub struct IntegerWrapper {
    pub node: IntrusiveNode,
    pub value: i32,
}

// Print function for IntegerWrapper. Takes a raw pointer to IntrusiveNode.
unsafe fn print_int(node_ptr: *mut IntrusiveNode) {
    unsafe {
        let int_wrapper_ptr = container_of!(node_ptr, IntegerWrapper, node);
        let int_wrapper_ref = &*int_wrapper_ptr;
        print!("{}", int_wrapper_ref.value);
    }
}

fn test_int_list() {
    let mut list = IntrusiveLinkedList::new();

    let mut val1 = IntegerWrapper {
        node: IntrusiveNode::new(),
        value: 10,
    };
    let mut val2 = IntegerWrapper {
        node: IntrusiveNode::new(),
        value: 20,
    };
    let mut val3 = IntegerWrapper {
        node: IntrusiveNode::new(),
        value: 30,
    };
    let mut val4 = IntegerWrapper {
        node: IntrusiveNode::new(),
        value: 40,
    };

    unsafe {
        list.intrusive_insert_end(&mut val1.node);
        list.intrusive_insert_end(&mut val2.node);
        list.intrusive_insert_end(&mut val3.node);
        list.intrusive_insert_end(&mut val4.node);

        list.print_intrusive_list(print_int);
    }

    list.head = ptr::null_mut();
}

fn test_person_list() {
    let mut list = IntrusiveLinkedList::new();

    let mut p1 = Box::new(Person {
        node: IntrusiveNode::new(),
        name: String::from("Marco"),
        age: 22,
    });
    let mut p2 = Box::new(Person {
        node: IntrusiveNode::new(),
        name: String::from("Mary"),
        age: 20,
    });

    unsafe {
        let p1_node_ptr = &mut p1.node as *mut IntrusiveNode;
        let p2_node_ptr = &mut p2.node as *mut IntrusiveNode;

        list.intrusive_insert_end(p1_node_ptr);
        list.intrusive_insert_end(p2_node_ptr);

        list.print_intrusive_list(print_person);

        list.free_list_person();
        println!("bye usnafe");
    }
    println!("bye test_person_list");
}

fn main() {
    test_int_list();
    println!();
    test_person_list();
}
```

## Zig

[https://www.openmymind.net/Zigs-New-LinkedList-API/](https://www.openmymind.net/Zigs-New-LinkedList-API/)

```zig
pub const SinglyLinkedList = struct {
  first: ?*Node = null,

  pub const Node = struct {
    next: ?*Node = null,
  };
};
```

# Comparing `container_of` implementations

**Prompt**: What is the difference between these two macros?

```c
#define container_of(ptr, type, member) \
((type *)((char *)(ptr) - offsetof(type, member)))

#define container_of(ptr, type, member) ({ \
const typeof( ((type *)0)->member ) *__mptr = (ptr); \
(type *)( (char *)__mptr - offsetof(type,member) );}) 
```

Gemini:

You've identified two common forms of the `container_of` macro, and the second one is the more robust and widely used version, especially in the Linux kernel. Let's break down the differences:

### Version 1: Simple (Less Robust)

```c
#define container_of(ptr, type, member) \
    ((type *)((char *)(ptr) - offsetof(type, member)))
```

**How it works:**
This version directly performs the pointer arithmetic.

1.  `(char *)(ptr)`: Casts the member pointer `ptr` to a `char*`. This is crucial because `offsetof` returns a byte offset, and `char*` pointer arithmetic works in terms of bytes. If you didn't cast to `char*`, the subtraction would be in terms of `sizeof(*ptr)`, which would likely be incorrect.
2.  `offsetof(type, member)`: This standard macro (from `<stddef.h>`) calculates the offset in bytes of the `member` within the `type` structure.
3.  `((char *)(ptr) - offsetof(type, member))`: Subtracts the offset from the member's address to get the address of the beginning of the containing structure.
4.  `(type *)`: Casts the resulting byte address back to a pointer to the `type` of the container structure.

**Pros:**

  * Simple and straightforward.
  * Achieves the core functionality.

**Cons:**

  * **No Compile-Time Type Checking:** This is the most significant drawback. If you accidentally pass a `ptr` that is not actually a pointer to the `member` of the specified `type`, the compiler will not warn you. It will happily perform the calculation, resulting in undefined behavior or crashes at runtime. For example, if `ptr` points to an `int` and `member` is supposed to be a `struct list_head`, the compiler won't catch the mismatch.

### Version 2: Linux Kernel Style (Robust with Type Checking)

```c
#define container_of(ptr, type, member) ({ \
    const typeof( ((type *)0)->member ) *__mptr = (ptr); \
    (type *)( (char *)__mptr - offsetof(type,member) );})
```

**How it works:**
This version introduces two key enhancements:

1.  **GNU Statement Expression `({ ... })`**: The entire macro is wrapped in `({ ... })`. This is a GNU C extension (supported by GCC and Clang) that allows you to have a compound statement (with local variable declarations, etc.) act as an expression. The value of the last expression in the block is the value of the overall statement expression. This is essential for declaring a local variable like `__mptr` within the macro without polluting the surrounding scope.

2.  **Compile-Time Type Checking with `typeof`**:

      * `((type *)0)->member`: This is a clever trick. It creates a "null pointer" to the `type` structure and then attempts to access its `member`. Crucially, this expression is **never actually evaluated at runtime** (it doesn't dereference a null pointer). Its sole purpose is to provide a context for `typeof`.

      * `typeof( ((type *)0)->member )`: The `typeof` operator (another GCC/Clang extension) gets the *type* of the expression `((type *)0)->member`. So, if `member` is of type `struct list_head`, `typeof(...)` will yield `struct list_head`.

      * `const typeof( ((type *)0)->member ) *__mptr = (ptr);`: This is the magic line. It declares a temporary local pointer `__mptr` whose type is `const` pointer to the *actual type of the member*. Then, it initializes `__mptr` with `ptr`.

          * **If `ptr` has a different type than `member`**, the compiler will issue a **warning** (or even an error, depending on compiler flags) about incompatible pointer types during this assignment. This catches many potential bugs at compile time that the simpler version would miss.
          * The `const` qualifier is added to `__mptr` to prevent accidental modification of the pointer within the macro.

      * `(type *)( (char *)__mptr - offsetof(type,member) );`: The rest is similar to the simpler version, performing the pointer arithmetic using the now type-checked `__mptr`.

**Pros:**

  * **Compile-Time Type Safety (Major Advantage):** Catches many common programming errors where `ptr` might not actually point to the correct type of `member`. This makes the code much safer and easier to debug.
  * **Encapsulation**: The use of `({ ... })` ensures that any temporary variables (like `__mptr`) used within the macro don't interfere with variables in the code that uses the macro.

**Cons:**

  * **Requires GNU C Extensions:** `typeof` and statement expressions are not part of standard C (C99, C11, C17). While widely supported by GCC and Clang, they are technically non-portable to other compilers. For most Linux kernel development and embedded systems where GCC/Clang are dominant, this is not a practical concern.
  * Slightly more complex to read and understand initially due to the `typeof` trick and statement expression.

### Summary:

| Feature           | Simple Version                      | Linux Kernel Version                |
| :---------------- | :---------------------------------- | :---------------------------------- |
| **Type Safety** | None (runtime errors possible)      | **Yes (compile-time warnings/errors)** |
| **Portability** | Standard C                          | GNU C extension (GCC/Clang specific)|
| **Readability** | Slightly simpler                    | Slightly more complex               |
| **Error Detection**| Poor                                | Excellent                           |
| **Usage** | Fine for quick, internal use, or very simple cases where type is guaranteed | **Preferred for robust, large-scale, or kernel-style development** |

In most real-world scenarios, especially when dealing with low-level C programming or libraries that need to be highly robust (like the Linux kernel), the second version (with `typeof` and statement expressions) is overwhelmingly preferred because of its crucial compile-time type-checking capabilities.

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

[Hacker News - Zig's new LinkedList API (it's time to learn fieldParentPtr)](https://news.ycombinator.com/item?id=43679707)

## Rust

[https://docs.rs/intrusive-collections/latest/intrusive_collections/](https://docs.rs/intrusive-collections/latest/intrusive_collections/)

[Safe Intrusive Collections with Pinning](https://internals.rust-lang.org/t/safe-intrusive-collections-with-pinning/7281)

[What are intrusive linked lists and does Rust really struggle with them?](https://users.rust-lang.org/t/what-are-intrusive-linked-lists-and-does-rust-really-struggle-with-them/117943)

[https://www.kernel.org/doc/rustdoc/next/kernel/list/struct.List.html](https://www.kernel.org/doc/rustdoc/next/kernel/list/struct.List.html)

[https://rust.docs.kernel.org/src/kernel/list.rs.html#255](https://rust.docs.kernel.org/src/kernel/list.rs.html#255)

[https://github.com/bbatha/movecell_graph/blob/master/src/lib.rs](https://github.com/bbatha/movecell_graph/blob/master/src/lib.rs)
