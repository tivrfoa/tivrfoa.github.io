---
layout: post
title:  "Rust for Linux 2021-09 pull requests"
date:   2021-10-02 09:15:44 -0300
categories: linux rust
---

## Merged Pull Requests

1. rust: Remove is-builtin from target configs #490
2. rust: remove usage of const_raw_ptr_deref #492
3. rust: move to stable rustc release (1.55.0) #497
4. rust: port liballoc to release 1.55.0 #498
5. rust: docs: update to 1.55.0 #499
6. Sync with v5.15-rc3 #500
7. rust: add try_pin under no_global_oom_handling #501
8. [RFC] Fix the support of larger symbol name #503

### [RFC] Fix the support of larger symbol name #503

[https://github.com/Rust-for-Linux/linux/pull/503](https://github.com/Rust-for-Linux/linux/pull/503)

Some interesting C code with macros!

```c
// main.c

#include <stdio.h>

#define TIMES_10_(x) x##x##x##x##x##x##x##x##x##x
#define TIMES_10(x)  TIMES_10_(x)

#define ADD_11_(x)   x ## abcde ## abcde ## a
#define ADD_11(x)    ADD_11_(x)

int
#include "longest_symbol"
;

int main() {
  printf("%ld\n", TIMES_10(2)); // 2222222222
  printf("%ld\n", TIMES_10_(2)); // 2222222222

  char *aabcdeabcdea = "Hello";
  printf("%s\n", ADD_11(a)); // Hello

  return 0;
}

// longest_symbol

longest = 3
``` 
