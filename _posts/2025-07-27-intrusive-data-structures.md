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

I didn't know about *intrusive data structures*, then I saw it mentioned in this X thread:

[CedarDB on why they picked C++ over Rust](https://x.com/iavins/status/1946598328533430702)

Let's learn about it.

- [What is an Intrusive Data Structure?](#why-use-intrusive-data-structure)
- [Why use them?](#why-use-them)
- [Who is responsible for freeing the object?](#who-is-responsible-for-freeing-the-object)
- [Examples](#examples)
  - [C](#c)
  - [C Generic List](#c-generic-list)
  - [C Intrusive List and Tree](#c-intrusive-list-and-tree)
  - [C Task Scheduler](#c-task-scheduler)
  - [C LRU Cache](#c-lru-cache)
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

## Who is responsible for freeing the object?

**Prompt**: In an intrusive data structure, who should be responsible for freeing the actual object?

what if I just want to free the list, but still want to use the tree? Are there use cases for that?

**Gemini**:

Excellent questions. They get to the very heart of why and how intrusive data structures are used.

### Who is Responsible for Freeing the Object?

The short answer is: **the code that allocated the object is responsible for freeing it.**

Intrusive containers (like the list or tree) **do not own** the objects they contain. They are simply mechanisms for organizing pointers that live *inside* the objects. Think of the containers as different "views" or "indexes" into a set of data that you manage separately.

This is a fundamental shift from non-intrusive (e.g., `std::list` in C++) or standard library containers, where the container allocates its own nodes and is therefore responsible for freeing them.

Here are the common ownership patterns:

1.  **A "Master" Container:** One of your intrusive structures is designated as the "owner." In the example code, we decided the linked list was the master. We used the list to iterate through and `free` every single object. This is a simple and common approach when the objects' lifetime is tied to one primary organization.
2.  **External Logic:** The application's main logic owns the objects, perhaps in a simple array or by some other means. The intrusive list and tree are just secondary ways to access them. When the application decides an object is no longer needed, it's responsible for removing it from all intrusive containers it's a part of and *then* freeing it.
3.  **Reference Counting:** For maximum flexibility, you could add a reference count to `MyIntData`. When you add an object to a container, you increment the count. When you remove it, you decrement the count. The object is only freed when its count reaches zero. This is the most robust method but adds complexity.

-----

### Can You Free the List but Keep the Tree?

**Yes, absolutely.** This is one of the most powerful use cases for intrusive data structures.

However, you need to be precise about what "free the list" means. It does **not** mean iterating through the list and calling `free()` on each object, because that would deallocate the objects that the tree also points to, leaving you with a tree full of dangling pointers.

Instead, "freeing the list" means **dismantling the list structure**, leaving the objects intact. You would simply traverse the list and reset the `next` pointers, then set the list's `head` pointer to `NULL`. The `MyIntData` objects themselves would remain allocated and valid, and the tree structure would be completely unaffected.

A function to do this would look like this:

```c
// Dismantles the list structure WITHOUT freeing the underlying objects.
void intrusive_dismantle_list(IntrusiveLink** headRef) {
    IntrusiveLink* current_link = *headRef;
    while (current_link != NULL) {
        IntrusiveLink* next_link = current_link->next;
        current_link->next = NULL; // Unlink the node
        current_link = next_link;
    }
    *headRef = NULL; // The list is now empty
}
```

#### Use Cases for This Behavior

This decoupling of the container's lifetime from the data's lifetime is extremely useful.

  * **Temporary Re-ordering:** Imagine your tree is the primary, canonical storage for your objects, sorted by value. For a specific task, you need to process these objects in the reverse order of their creation. You can temporarily build an intrusive list for this task, process it, and then simply **dismantle the list** when you're done, leaving your tree untouched.

  * **Filtering and Subsets:** You have thousands of objects in your tree. You want to run an operation on a small subset that meets a certain criterion (e.g., `value > 100`). You can iterate through your tree and add only the matching objects to a temporary intrusive list. You then operate on that much smaller list, which is very efficient. Once finished, you dismantle the list.

  * **State Management (e.g., Task Schedulers):** An object might represent a task. It could live in a main tree of all known tasks. When a task is ready to run, it's added to an intrusive `run_queue` list. When it's waiting for I/O, it's moved to a `wait_queue` list. The object itself is never created or destroyed; it's just linked into different lists that represent its current state. This is a core pattern in OS kernel development.

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

## From mgaunard

[https://news.ycombinator.com/item?id=43680655](https://news.ycombinator.com/item?id=43680655)

There are several advantages to intrusive node-based data structures...:

- the same object can move between containers with no allocation and no need for a dedicated complex API
- the same object can be part of multiple containers at once; particularly useful for intrusive binary trees, for indexing data with different criteria.
- the container can be fully polymorphic, no need for all elements to be the same dynamic type.
- no need for complex allocators, you can just store the objects as you see fit.

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

It shows how different structures can use the same intrusive link definition.

Here we have `Person` and `IntegerWrapper`:

```c
typedef struct Person {
    IntrusiveNode node;
    char *name;
    int age;
} Person;

typedef struct IntegerWrapper {
    IntrusiveNode node;
    int value;
} IntegerWrapper;
```

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

## C Intrusive List and Tree

```c
#include <stdio.h>
#include <stdlib.h> // For malloc, free
#include <stddef.h> // For offsetof

// A macro to get the containing structure from a pointer to its member
// This is crucial for intrusive data structures.
// ptr: pointer to the member (e.g., &myIntDataInstance->node)
// type: the type of the containing struct (e.g., MyIntData)
// member: the name of the member (e.g., node)
#define container_of(ptr, type, member) ({ \
    const typeof( ((type *)0)->member ) *__mptr = (ptr); \
    (type *)( (char *)__mptr - offsetof(type,member) );})

//====================================================================
// Intrusive Linked List Structures and Functions
//====================================================================

// Define the "list hook" that will be embedded in our data structures
typedef struct IntrusiveLink {
    struct IntrusiveLink *next;
} IntrusiveLink;


//====================================================================
// Intrusive Binary Tree Structures and Functions
//====================================================================

// Define the "tree hook" for an intrusive binary tree
typedef struct IntrusiveTreeNode {
    struct IntrusiveTreeNode *left;
    struct IntrusiveTreeNode *right;
} IntrusiveTreeNode;


//====================================================================
// Our Main Data Structure with Multiple Intrusive Hooks
//====================================================================

// Our data structure now contains hooks for both a list and a tree.
typedef struct MyIntData {
    int value;
    IntrusiveLink node;          // The embedded list hook
    IntrusiveTreeNode tree_node; // The embedded binary tree hook
} MyIntData;

// Factory function to create and initialize a new data object
MyIntData* newIntData(int value) {
    MyIntData* data = (MyIntData*)malloc(sizeof(MyIntData));
    if (!data) {
        perror("Failed to allocate memory for MyIntData");
        exit(EXIT_FAILURE);
    }
    data->value = value;
    // Initialize both intrusive hooks
    data->node.next = NULL;
    data->tree_node.left = NULL;
    data->tree_node.right = NULL;
    return data;
}

//====================================================================
// Functions for Managing the Intrusive List
//====================================================================

// Function to insert a new data structure at the beginning of the intrusive list
void intrusive_insert_at_beginning(IntrusiveLink** headRef, MyIntData* new_data_ptr) {
    new_data_ptr->node.next = *headRef;
    *headRef = &(new_data_ptr->node);
}

// Function to print the intrusive linked list
void intrusive_print_list(IntrusiveLink* head) {
    IntrusiveLink* current_link = head;
    printf("Intrusive Linked List (insertion order): ");
    while (current_link != NULL) {
        MyIntData* data_item = container_of(current_link, MyIntData, node);
        printf("%d -> ", data_item->value);
        current_link = current_link->next;
    }
    printf("NULL\n");
}

// Function to free the memory allocated for the intrusive linked list
void intrusive_free_list(IntrusiveLink** headRef) {
    IntrusiveLink* current_link = *headRef;
    IntrusiveLink* next_link;
    while (current_link != NULL) {
        next_link = current_link->next;
        MyIntData* data_item = container_of(current_link, MyIntData, node);
        free(data_item); // Free the entire MyIntData object
        current_link = next_link;
    }
    *headRef = NULL;
    printf("Intrusive List freed. All underlying MyIntData objects have been deallocated.\n");
}

// Dismantles the list structure WITHOUT freeing the underlying objects.
void intrusive_dismantle_list(IntrusiveLink** headRef) {
    IntrusiveLink* current_link = *headRef;
    while (current_link != NULL) {
        IntrusiveLink* next_link = current_link->next;
        current_link->next = NULL; // Unlink the node
        current_link = next_link;
    }
    *headRef = NULL;
}


//====================================================================
// Functions for Managing the Intrusive Binary Tree
//====================================================================

// Function to insert a new data structure into the intrusive binary search tree
void intrusive_tree_insert(IntrusiveTreeNode** rootRef, MyIntData* new_data_ptr) {
    if (*rootRef == NULL) {
        *rootRef = &(new_data_ptr->tree_node);
        return;
    }
    
    IntrusiveTreeNode* current = *rootRef;
    while (1) {
        MyIntData* current_data = container_of(current, MyIntData, tree_node);
        if (new_data_ptr->value < current_data->value) {
            if (current->left == NULL) {
                current->left = &(new_data_ptr->tree_node);
                break;
            }
            current = current->left;
        } else {
            if (current->right == NULL) {
                current->right = &(new_data_ptr->tree_node);
                break;
            }
            current = current->right;
        }
    }
}

// Recursive function to print the tree using in-order traversal
void intrusive_print_tree_in_order_recursive(IntrusiveTreeNode* root) {
    if (root == NULL) {
        return;
    }
    intrusive_print_tree_in_order_recursive(root->left);
    MyIntData* data_item = container_of(root, MyIntData, tree_node);
    printf("%d ", data_item->value);
    intrusive_print_tree_in_order_recursive(root->right);
}

// Wrapper function to print the entire tree
void intrusive_print_tree(IntrusiveTreeNode* root) {
    printf("Intrusive Binary Tree (sorted order):  ");
    intrusive_print_tree_in_order_recursive(root);
    printf("\n");
}


//====================================================================
// Main Demo
//====================================================================

int main() {
    // Heads for our two intrusive data structures
    IntrusiveLink* list_head = NULL;
    IntrusiveTreeNode* tree_root = NULL;

    printf("Creating and inserting data into both list and tree...\n");

    // Create several data objects
    MyIntData* data1 = newIntData(30);
    MyIntData* data2 = newIntData(10);
    MyIntData* data3 = newIntData(40);
    MyIntData* data4 = newIntData(20);
    MyIntData* data5 = newIntData(50);

    // Insert each object into BOTH data structures
    intrusive_insert_at_beginning(&list_head, data1);
    intrusive_tree_insert(&tree_root, data1);

    intrusive_insert_at_beginning(&list_head, data2);
    intrusive_tree_insert(&tree_root, data2);

    intrusive_insert_at_beginning(&list_head, data3);
    intrusive_tree_insert(&tree_root, data3);

    intrusive_insert_at_beginning(&list_head, data4);
    intrusive_tree_insert(&tree_root, data4);

    intrusive_insert_at_beginning(&list_head, data5);
    intrusive_tree_insert(&tree_root, data5);

    printf("\n--- Data Structures Populated ---\n");
    // Print both structures to show the same data is organized differently
    intrusive_print_list(list_head);
    intrusive_print_tree(tree_root);
    printf("----------------------------------\n\n");

    // Free the memory. We only need to do this once.
    // We iterate through the list to free each MyIntData object.
    // After this, the pointers in the tree (tree_root) are dangling.
    intrusive_free_list(&list_head);

    // The tree's pointers are now invalid. We must set its root to NULL
    // to avoid using dangling pointers.
    tree_root = NULL;
    printf("Tree root set to NULL as its memory has been freed via the list.\n\n");
    
    printf("--- Data Structures After Freeing ---\n");
    intrusive_print_list(list_head);
    intrusive_print_tree(tree_root);
    printf("-----------------------------------\n");

    return 0;
}
```

## C Task Scheduler

**Prompt**: Create a C code example for the task scheduler use case:

State Management (e.g., Task Schedulers): An object might represent a task. It could live in a main tree of all known tasks. When a task is ready to run, it's added to an intrusive run_queue list. When it's waiting for I/O, it's moved to a wait_queue list. The object itself is never created or destroyed; it's just linked into different lists that represent its current state. This is a core pattern in OS kernel development. 

**Gemini**:

Here is a C code example that implements a simple task scheduler using intrusive data structures.

A single `Task` object can exist in the main `all_tasks_tree` for its entire lifetime while being moved between the `run_queue` and `wait_queue` lists without any additional memory allocations. The lists and the tree simply manage pointers embedded within the `Task` object itself.

```c
#include <stdio.h>
#include <stdlib.h>
#include <stddef.h>
#include <string.h>

// Macro to get the containing structure from a pointer to its member
#define container_of(ptr, type, member) \
    ((type *)((char *)(ptr) - offsetof(type, member)))

//====================================================================
// Intrusive Data Structure "Hooks"
//====================================================================

// A link for a doubly-linked list, useful for efficient queue removal
typedef struct IntrusiveLink {
    struct IntrusiveLink *next;
    struct IntrusiveLink *prev;
} IntrusiveLink;

// A node for a binary search tree
typedef struct IntrusiveTreeNode {
    struct IntrusiveTreeNode *left;
    struct IntrusiveTreeNode *right;
} IntrusiveTreeNode;

//====================================================================
// The Task Object
//====================================================================

typedef enum {
    TASK_STATE_NEW,
    TASK_STATE_READY,
    TASK_STATE_WAITING,
    TASK_STATE_DONE
} TaskState;

// The core object containing data and intrusive hooks
typedef struct Task {
    int task_id;
    const char* name;
    TaskState state;

    IntrusiveLink queue_link;     // Hook for run/wait queues
    IntrusiveTreeNode tree_node;  // Hook for the master task tree
} Task;

//====================================================================
// "Scheduler" Global State
//====================================================================

// Master tree of all tasks that have ever been created, indexed by ID
IntrusiveTreeNode* all_tasks_tree = NULL;

// Queues representing the current state of tasks
IntrusiveLink run_queue  = { &run_queue, &run_queue }; // Sentinels for an empty list
IntrusiveLink wait_queue = { &wait_queue, &wait_queue };

//====================================================================
// Intrusive Queue (List) Management
//====================================================================

// Removes a link from whichever queue it is currently in
void queue_unlink(IntrusiveLink* link) {
    link->prev->next = link->next;
    link->next->prev = link->prev;
    link->next = link->prev = NULL; // Isolate the link
}

// Adds a link to the back of a queue
void queue_push_back(IntrusiveLink* head, IntrusiveLink* new_link) {
    new_link->prev = head->prev;
    new_link->next = head;
    head->prev->next = new_link;
    head->prev = new_link;
}

// Removes and returns the link at the front of a queue
IntrusiveLink* queue_pop_front(IntrusiveLink* head) {
    if (head->next == head) return NULL; // Queue is empty
    IntrusiveLink* link = head->next;
    queue_unlink(link);
    return link;
}

//====================================================================
// Core Task Management & Scheduler Logic
//====================================================================

// Factory to create a new task
Task* create_task(int id, const char* name) {
    Task* task = (Task*)malloc(sizeof(Task));
    task->task_id = id;
    task->name = name;
    task->state = TASK_STATE_NEW;
    task->queue_link.next = task->queue_link.prev = NULL;
    task->tree_node.left = task->tree_node.right = NULL;

    // Add to the master tree (simple BST insertion)
    IntrusiveTreeNode** current = &all_tasks_tree;
    while (*current) {
        Task* existing_task = container_of(*current, Task, tree_node);
        if (task->task_id < existing_task->task_id) {
            current = &(*current)->left;
        } else {
            current = &(*current)->right;
        }
    }
    *current = &task->tree_node;
    printf("CREATED:  Task %d ('%s')\n", id, name);
    return task;
}

// Moves a task to the ready queue to be run
void schedule_task(Task* task) {
    if (task->queue_link.next) queue_unlink(&task->queue_link); // Unlink from old queue if any
    task->state = TASK_STATE_READY;
    queue_push_back(&run_queue, &task->queue_link);
    printf("SCHEDULED: Task %d ('%s') moved to RUN queue.\n", task->task_id, task->name);
}

// Simulates running a task. Returns 1 if task is done, 0 otherwise.
int execute_task(Task* task) {
    printf("EXECUTING: Task %d ('%s')...\n", task->task_id, task->name);
    
    // Simple logic: tasks with ID > 100 need I/O, others finish
    if (task->task_id > 100) {
        task->state = TASK_STATE_WAITING;
        queue_push_back(&wait_queue, &task->queue_link);
        printf("WAITING:  Task %d ('%s') moved to WAIT queue for I/O.\n", task->task_id, task->name);
        return 0;
    } else {
        task->state = TASK_STATE_DONE;
        printf("DONE:     Task %d ('%s') has finished.\n", task->task_id, task->name);
        return 1;
    }
}

// Prints the current state of the scheduler queues
void print_scheduler_state() {
    printf("\n--- Scheduler State ---\n");
    printf("Run Queue:  ");
    for (IntrusiveLink* l = run_queue.next; l != &run_queue; l = l->next) {
        printf("%d -> ", container_of(l, Task, queue_link)->task_id);
    }
    printf("END\nWait Queue: ");
    for (IntrusiveLink* l = wait_queue.next; l != &wait_queue; l = l->next) {
        printf("%d -> ", container_of(l, Task, queue_link)->task_id);
    }
    printf("END\n-----------------------\n\n");
}

// Free all tasks using a post-order traversal of the master tree
void free_all_tasks(IntrusiveTreeNode* node) {
    if (!node) return;
    free_all_tasks(node->left);
    free_all_tasks(node->right);
    free(container_of(node, Task, tree_node));
}

//====================================================================
// Main Demo
//====================================================================

int main() {
    // 1. Create all tasks. They start in the NEW state.
    Task* task1 = create_task(10, "Calculate report");
    Task* task2 = create_task(205, "Fetch network data");
    Task* task3 = create_task(30, "Update UI");
    Task* task4 = create_task(155, "Write to disk");

    // 2. Schedule initial tasks. They move to the READY state and enter the run queue.
    schedule_task(task1);
    schedule_task(task2);
    schedule_task(task3);
    
    print_scheduler_state();

    // 3. Run a scheduler cycle: execute one task from the run queue.
    IntrusiveLink* link = queue_pop_front(&run_queue);
    execute_task(container_of(link, Task, queue_link)); // Task 10 will finish.

    print_scheduler_state();

    // 4. Run another cycle. This task will need I/O and move to the wait queue.
    link = queue_pop_front(&run_queue);
    execute_task(container_of(link, Task, queue_link)); // Task 205 will wait.
    
    print_scheduler_state();

    // 5. Schedule the last task.
    schedule_task(task4);

    print_scheduler_state();

    // 6. Simulate I/O completion for task 205. It's found in the wait queue
    //    and moved back to the run queue.
    printf("...I/O for task 205 has completed...\n");
    schedule_task(task2); // Re-scheduling moves it from wait_queue to run_queue.
    
    print_scheduler_state();
    
    // 7. Run all remaining tasks until the run queue is empty.
    printf("...Running all remaining tasks...\n");
    while ((link = queue_pop_front(&run_queue))) {
        execute_task(container_of(link, Task, queue_link));
    }

    print_scheduler_state();

    // 8. Clean up all memory by freeing the master tree.
    printf("All tasks processed. Freeing memory.\n");
    free_all_tasks(all_tasks_tree);

    return 0;
}
```

## C LRU Cache

**Gemini**:

Here is a complete C implementation of an LRU (Least Recently Used) cache using intrusive data structures.

This example demonstrates a key strength of this pattern: a single allocated object (`CacheEntry`) is simultaneously part of two different data structures:

1. A **Hash Table** (implemented as an array of singly-linked lists) for O(1) average time complexity lookups.
2. A **Doubly-Linked List** to maintain the usage order (from most- to least-recently-used), allowing for O(1) updates and evictions.

The CacheEntry object itself contains the "hooks" (hash_link and lru_link) that allow it to be managed by these external structures without any additional memory allocations per operation.

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <stddef.h>

// A macro to get the containing structure from a pointer to its member.
#define container_of(ptr, type, member) \
    ((type *)((char *)(ptr) - offsetof(type, member)))

//====================================================================
// Intrusive Data Structure "Hooks"
//====================================================================

// Hook for a singly-linked list (for hash table collision chains).
typedef struct HashLink {
    struct HashLink *next;
} HashLink;

// Hook for a doubly-linked list (for LRU ordering).
typedef struct LruLink {
    struct LruLink *next;
    struct LruLink *prev;
} LruLink;

//====================================================================
// The Cache Entry Object
//====================================================================

// This is the core object we store. It contains the actual data
// and the hooks for both the hash table and the LRU list.
typedef struct CacheEntry {
    int key;
    char value[256];

    HashLink hash_link; // Hook for hash collision chaining.
    LruLink lru_link;   // Hook for LRU ordering.
} CacheEntry;

//====================================================================
// The LRU Cache Structure
//====================================================================

typedef struct LRUCache {
    size_t capacity;
    size_t size;
    HashLink **hash_table; // Array of pointers to collision chains.
    size_t hash_table_size;
    LruLink lru_list_head; // Sentinel node for the LRU list.
} LRUCache;

//====================================================================
// Doubly-Linked List (LRU) Management
//====================================================================

// Initializes a sentinel node for a doubly-linked list.
static void lru_list_init(LruLink *head) {
    head->next = head;
    head->prev = head;
}

// Removes a link from its current position in the list.
static void lru_list_unlink(LruLink *link) {
    link->prev->next = link->next;
    link->next->prev = link->prev;
}

// Adds a link to the front of the list (most recently used).
static void lru_list_push_front(LruLink *head, LruLink *new_link) {
    new_link->next = head->next;
    new_link->prev = head;
    head->next->prev = new_link;
    head->next = new_link;
}

//====================================================================
// Cache Internals
//====================================================================

// A simple hash function for integer keys.
static size_t hash_key(int key, size_t table_size) {
    return (size_t)key % table_size;
}

// Finds a cache entry by key. Does NOT change LRU order.
static CacheEntry* find_entry(LRUCache* cache, int key) {
    size_t index = hash_key(key, cache->hash_table_size);
    HashLink* current = cache->hash_table[index];

    while (current) {
        CacheEntry* entry = container_of(current, CacheEntry, hash_link);
        if (entry->key == key) {
            return entry;
        }
        current = current->next;
    }
    return NULL;
}

// Removes an entry from the hash table.
static void remove_from_hash_table(LRUCache* cache, CacheEntry* entry) {
    size_t index = hash_key(entry->key, cache->hash_table_size);
    HashLink** indirect = &cache->hash_table[index];

    while (*indirect && *indirect != &entry->hash_link) {
        indirect = &(*indirect)->next;
    }

    if (*indirect) {
        *indirect = entry->hash_link.next;
    }
}

// Evicts the least recently used item from the cache.
static void evict_one(LRUCache* cache) {
    // The LRU item is at the back of the list (before the sentinel).
    LruLink* lru_link_to_evict = cache->lru_list_head.prev;
    if (lru_link_to_evict == &cache->lru_list_head) {
        // Should not happen if size > 0, but good practice.
        return;
    }

    CacheEntry* entry_to_evict = container_of(lru_link_to_evict, CacheEntry, lru_link);
    printf("EVICTING: Key %d ('%s')\n", entry_to_evict->key, entry_to_evict->value);

    // Unlink from both data structures.
    lru_list_unlink(&entry_to_evict->lru_link);
    remove_from_hash_table(cache, entry_to_evict);

    // Free the actual object and decrement cache size.
    free(entry_to_evict);
    cache->size--;
}


//====================================================================
// Public LRU Cache API
//====================================================================

// Creates and initializes an LRU cache.
LRUCache* lru_create(size_t capacity) {
    LRUCache* cache = (LRUCache*)malloc(sizeof(LRUCache));
    if (!cache) return NULL;

    cache->capacity = capacity;
    cache->size = 0;
    // For simplicity, hash table size is twice the capacity.
    cache->hash_table_size = capacity * 2; 
    cache->hash_table = (HashLink**)calloc(cache->hash_table_size, sizeof(HashLink*));
    if (!cache->hash_table) {
        free(cache);
        return NULL;
    }
    
    lru_list_init(&cache->lru_list_head);
    return cache;
}

// Puts a key-value pair into the cache.
void lru_put(LRUCache* cache, int key, const char* value) {
    printf("PUT:      Key %d, Value '%s'\n", key, value);

    CacheEntry* entry = find_entry(cache, key);

    if (entry) {
        // --- Key exists: Update value and move to front ---
        strncpy(entry->value, value, sizeof(entry->value) - 1);
        // Unlink from current position and move to front of LRU list.
        lru_list_unlink(&entry->lru_link);
        lru_list_push_front(&cache->lru_list_head, &entry->lru_link);
    } else {
        // --- Key does not exist: Create new entry ---
        if (cache->size >= cache->capacity) {
            evict_one(cache);
        }

        // Create the new entry.
        entry = (CacheEntry*)malloc(sizeof(CacheEntry));
        entry->key = key;
        strncpy(entry->value, value, sizeof(entry->value) - 1);

        // Add to hash table.
        size_t index = hash_key(key, cache->hash_table_size);
        entry->hash_link.next = cache->hash_table[index];
        cache->hash_table[index] = &entry->hash_link;

        // Add to front of LRU list.
        lru_list_push_front(&cache->lru_list_head, &entry->lru_link);
        cache->size++;
    }
}

// Gets a value by key from the cache.
const char* lru_get(LRUCache* cache, int key) {
    CacheEntry* entry = find_entry(cache, key);

    if (entry) {
        printf("GET:      Key %d -> Found '%s'. Moving to front.\n", key, entry->value);
        // Move to front of LRU list to mark as recently used.
        lru_list_unlink(&entry->lru_link);
        lru_list_push_front(&cache->lru_list_head, &entry->lru_link);
        return entry->value;
    }

    printf("GET:      Key %d -> Not found.\n", key);
    return NULL;
}

// Frees all resources used by the cache.
void lru_destroy(LRUCache* cache) {
    LruLink* current = cache->lru_list_head.next;
    while (current != &cache->lru_list_head) {
        LruLink* next = current->next;
        CacheEntry* entry = container_of(current, CacheEntry, lru_link);
        free(entry);
        current = next;
    }
    free(cache->hash_table);
    free(cache);
}

// Utility to print the cache contents in LRU order.
void lru_print(LRUCache* cache) {
    printf("\n--- Cache State (MRU to LRU) ---\n");
    printf("Size: %zu, Capacity: %zu\n", cache->size, cache->capacity);
    printf("Order: ");
    LruLink* current = cache->lru_list_head.next;
    while (current != &cache->lru_list_head) {
        CacheEntry* entry = container_of(current, CacheEntry, lru_link);
        printf("%d -> ", entry->key);
        current = current->next;
    }
    printf("END\n--------------------------------\n\n");
}

//====================================================================
// Main Demo
//====================================================================

int main() {
    LRUCache* cache = lru_create(3);

    lru_put(cache, 10, "apple");
    lru_put(cache, 20, "banana");
    lru_put(cache, 30, "cherry");
    lru_print(cache); // Order: 30 -> 20 -> 10

    lru_get(cache, 10); // Access 10, making it the most recent.
    lru_print(cache); // Order: 10 -> 30 -> 20

    // This next put will cause an eviction. '20' is the LRU item.
    lru_put(cache, 40, "date");
    lru_print(cache); // Order: 40 -> 10 -> 30

    lru_get(cache, 20); // '20' was evicted, so this is a miss.
    lru_print(cache);

    lru_put(cache, 30, "cantaloupe"); // Update existing key '30'.
    lru_print(cache); // Order: 30 -> 40 -> 10
    lru_get(cache, 30);

    lru_destroy(cache);
    printf("Cache destroyed.\n");

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
    ($ptr:expr, $Type:ty, $member:ident) => {
        {
            // Ensure the pointer is mutable. `*mut u8` is the generic byte pointer.
            let member_ptr = $ptr as *mut u8;
            // Calculate the offset of the member within the struct.
            // This requires the actual type name ($Type) and member name ($member).
            let offset = mem::offset_of!($Type, $member);
            // Subtract the offset from the member's address to get the struct's base address.
            (member_ptr.sub(offset)) as *mut $Type
        }
    };
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

---

### Protty @kingprotty Jul 21, 2025

>intrusive-collection requires the nodes to outlive the linked-list (static hierarchical lifetimes) or use box/arena (heap allocation).
>
>By intrusive, I mean not assuming where memory must live
>
>[https://x.com/kingprotty/status/1947288844094603443](https://x.com/kingprotty/status/1947288844094603443)

>[https://github.com/mitchellh/libxev/blob/main/src/ThreadPool.zig](https://github.com/mitchellh/libxev/blob/main/src/ThreadPool.zig)
>Pool.schedule() takes Task ptr that can live anywhere until its callback is invoked. Store it on stack for fork/join structured concurrency pattern. Store in heap & free itself in callback for spawn(detached)/join pattern. Store in array for pool pattern.
>
>[https://x.com/kingprotty/status/1947363773217419707](https://x.com/kingprotty/status/1947363773217419707)

[https://github.com/mitchellh/libxev/blob/main/src/ThreadPool.zig#L377](https://github.com/mitchellh/libxev/blob/main/src/ThreadPool.zig#L377)

```zig
const task: *Task = @fieldParentPtr("node", result.node);
(task.callback)(task);
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

**Gemini**:

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

[https://www.gamedeveloper.com/programming/in-depth-intrusive-lists](https://www.gamedeveloper.com/programming/in-depth-intrusive-lists)

[Adopt intrusive data structures for better performance](https://github.com/doitsujin/dxvk/issues/3796)

## Zig

[Zig's @fieldParentPtr for dumbos like me](https://www.ryanliptak.com/blog/zig-fieldparentptr-for-dumbos/)

[Hacker News - Zig's new LinkedList API (it's time to learn fieldParentPtr)](https://news.ycombinator.com/item?id=43679707)

[Resource efficient Thread Pools with Zig](https://zig.news/kprotty/resource-efficient-thread-pools-with-zig-3291)

## Rust

[https://docs.rs/intrusive-collections/latest/intrusive_collections/](https://docs.rs/intrusive-collections/latest/intrusive_collections/)

[Safe Intrusive Collections with Pinning](https://internals.rust-lang.org/t/safe-intrusive-collections-with-pinning/7281)

[What are intrusive linked lists and does Rust really struggle with them?](https://users.rust-lang.org/t/what-are-intrusive-linked-lists-and-does-rust-really-struggle-with-them/117943)

[https://www.kernel.org/doc/rustdoc/next/kernel/list/struct.List.html](https://www.kernel.org/doc/rustdoc/next/kernel/list/struct.List.html)

[https://rust.docs.kernel.org/src/kernel/list.rs.html#255](https://rust.docs.kernel.org/src/kernel/list.rs.html#255)

[https://github.com/bbatha/movecell_graph/blob/master/src/lib.rs](https://github.com/bbatha/movecell_graph/blob/master/src/lib.rs)
