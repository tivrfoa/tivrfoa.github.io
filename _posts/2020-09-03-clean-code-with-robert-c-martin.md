---
layout: post
title:  "Clean Code with Robert C. Martin - Uncle Bob"
date:   2020-09-03 10:26:44 -0300
categories: clean code uncle Bob interview
---

# Clean Code with Robert C. Martin, the Uncle Bob

I watched a fantastic interview by Rodrigo Branas with Uncle Bob:
[Clean Code with Robert C. Martin // Live #58](https://www.youtube.com/watch?v=DaRpFF-di4w)

Bob said so many important things that I had to write it down some
of the highlights, but the whole interview is a must watch! =)

Bob's sense of humor makes it very pleasant to watch.


### What does it mean to be a professional?
  
A professional is someone who professes something and typically what they
professes is a set of standards and disciplines and ethics, **primarily
the ethics**, along with a set of practices or disciplines that they
promise to follow.

*Now we as programmers don't do any of that.*

There is no ethics that we adhere to, there's no rules, there's hardly
any disciplines, there's no standards. **Mostly we just kind of write
code**.


### What is your personal definition of clean code?

My favorite definition was Ward Cunningham's:
> You know you're reading clean code when each routine you read is pretty
much what you expected.

His statement made it clear that clean code is code that when you read it,
it's not surprising. It communicates to you.

Michael Feathers said it very well too:
> Clean code looks like it was written by someone who cares.

Most programmers believe their first obligation is to make the code work
and that's not right.

**The programmers' first obligation is to communicate with other
programmers.** Make sure their code communicates to everybody else because
if I can read your code, I can make it work even if you can't.

**It's much more important that the code communicate than that the code
work!**


### What is the point for you about Test-Driven Development (TDD)?

TDD is a difficult discipline, but once you have mastered it, it pays back
enormously.

The goal of TDD of course is your idea of courage. **What we want is some
way to be able to courageously change the code and we cannot do that if
we fear that the code is being broken.**

But if we have a suite of tests, and if we trust that suite of tests with
our life, and if that suite of tests *runs in matter of seconds*, then
we can clean the code.

If we don't have the tests we lose control of the code and then
**the code control us**.

How do you get the suite of tests? I don't really care, but getting that
suite of tests is the real important thing. TDD is one of the disciplines
that can get you that suite of tests.

### Do you follow the three laws of TDD?

I follow them, but I also take a certain amount of liberty with them.

My rule of thumb is that I can relax the rules until I wind up in a debugger.
As soon as I'm in a debugger I've relaxed them too much and I have to
tighten them back up.


### Debuggers or print statements?

It is much easier for me to use a print statement than to do a debugging
session, because the bugs that I create are vanishingly small and stupidly
simple. If you're doing test-driven development that's just kind of the
way that works.


### What is the most dangerous cold smell?

Well, the most prevalent code smell is `long method`.<br>
It's not the most dangerous, but it does the most damage because it's
the most prevalent.

**The most dangerous has to do with writing functions that serve more
than one master.** That gets back to principles called the single
responsibility and so on, one of the SOLID principles.

When you have a function and that function serves these people over here
but it also serves these people over there, then it becomes possible to
make a change in this function that answers a question or a desire that
these folks over here had, but it breaks those guys.

That's the symptom of code fragility.

**Fragility is when you touch the code in an obvious and simple way to
solve a problem for someone, and you do, you solve the problem for them
but you break it for someone else.**

You know you have failed when your boss says **nobody touches that code
anymore!**


### Why did you come up with the clean architecture concept?

I thought there needs to be a book that explains what the real goal of an
architecture is. **The goal of architecture is separation, separating the
things that are not related to each other.**

The fact that they are not related needs to be maintained. You have to
have boundaries that separate these things that are not related, so that
when changes occur in one, they don't break things in the other, they
don't propagate to the other.

The end result of that should be a set of business rules at the core of
your system that anybody could look at, at the highest level, and know
what the system does.
