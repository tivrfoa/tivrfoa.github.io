---
layout: post
title:  "Should you use Rust unwrap?"
date:   2025-11-20 08:10:00 -0300
categories: rust unwrap errorHandling
---
<style>
h1 {
  font-size: 2.5em;
}
h2 {
  font-size: 2em;
}
</style>

>@alexeiboukirev8357
>Rust is a gift to the developers.  Gifts are meant to be unwrapped.
>
>https://www.youtube.com/watch?v=sJBaMJfxzYk&lc=UgyD5HQg3Ee3GvMOgZ54AaABAg

In interesting discussion started on X, after Richard Feldman said:<br>
[The Cloudflare outage was caused by an unwrap()](https://x.com/rtfeldman/status/1990998613514752383)

This is like saying that a gun killed someone ... It was not the gun, it was
the person that pulled the trigger.

We can't blame a tool that is well documented, because someone misuse it.

From: https://doc.rust-lang.org/std/option/enum.Option.html#method.unwrap

---
Returns the contained [`Some`] value, consuming the `self` value.

Because this function may panic, its use is generally discouraged.
Panics are meant for unrecoverable errors, and
[may abort the entire program][panic-abort].

Instead, prefer to use pattern matching and handle the [`None`]
case explicitly, or call [`unwrap_or`], [`unwrap_or_else`], or
[`unwrap_or_default`]. In functions returning `Option`, you can use
[the `?` (try) operator][try-option].

# Panics

Panics if the self value equals [`None`].
---

The argument that languages and stardard library should avoid having
constructs that enable bad patterns is really valid, though.

So lets investigate:
- use cases for unwrap
- use of unwrap in important Rust crates
- would be better if Rust didn't have it
- how Roc lang handles it
- how the Cloudflare bug could be avoided

# Use cases for unwrap

It is better to read this post by Andrew Gallant first:
[Using unwrap() in Rust is Okay](https://burntsushi.net/unwrap/)

That post is very comprehensive, and I mostly agree with everything.<br>

>Of the different ways to handle errors in Rust, this one is regarded as best
>practice:
>
>One can handle errors as normal values, typically with Result<T, E>. If an
>error bubbles all the way up to the main function, one might print the error
>to stderr and then abort.
>
>One of the most important parts of this approach is the ability to attach
>additional context to error values as they are returned to the caller.
>The anyhow crate makes this effortless.

---

## Points I disagree

### Not using `unwrap` in real code

From reading his post, it's like he doesn't use `unwrap` in the actual code,
and only uses it in tests and sometimes in documentation, but taking a look
at his project ([ripgrep](https://github.com/BurntSushi/ripgrep)), you see
`unwrap` being used in some places, eg:

crates/core/haystack.rs

```rust
impl Haystack {
    pub(crate) fn path(&self) -> &Path {
        if self.strip_dot_prefix && self.dent.path().starts_with("./") {
            self.dent.path().strip_prefix("./").unwrap()
        } else {
            self.dent.path()
        }
    }
```

Valid use case. It wouldn't enter `if` otherwise.

crates/core/main.rs

```rust
        if let Some(ref mut stats) = stats {
            *stats += search_result.stats().unwrap();
        }
        if matched && args.quit_after_match() {
            break;
        }
```

Valid use case. `search_result` is guaranteed to have stats at this point.

Those are valid use cases, and it seems these examples are missing from Andrew's
post.

### “recoverable” vs “unrecoverable”

>I’ve personally never found this particular conceptualization to be helpful.
>The problem, as I see it, is the ambiguity in determining whether a particular
>error is “recoverable” or not. What does it mean, exactly?

I don't see ambiguity there.<br>
It's basically: can the program continue working in a valid state?<br>

It will depend on the program and its use case.<br>


# Use of unwrap in important Rust crates



# References

[Richard Feldman: The Cloudflare outage was caused by an unwrap()](https://x.com/rtfeldman/status/1990998613514752383)

[Using unwrap() in Rust is Okay](https://burntsushi.net/unwrap/)

[ThePrimeTime - Another Internet outage???](https://www.youtube.com/watch?v=sJBaMJfxzYk)
