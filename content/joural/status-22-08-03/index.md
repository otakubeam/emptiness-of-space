---
title: "Weekly update: 22-08-03"
customdate: "2022 Aug 03 (水) 08:45AM"
tags: ["status"]
---

## So, what did I manage to accomplish this week?

This week I wanted to maintain focus writing the interpreter. 
So naturally this is where I have spent the bulk of my time.

It proceeded like so:

1. Function declarations, calls; if statements

  The first 2 days I have been busy implementing function calls.

  Relevant commits: from this one 
  [Functions (WIP)](https://git.sr.ht/~orazov_ae/crafting-interpreters/commit/68441a7b03f558034778d9609768ae910bb62802)
  up to this one 
  [Test recursive definitions](https://git.sr.ht/~orazov_ae/crafting-interpreters/commit/e2adc81167f8e3cfe72eb9fc63eafe21f5ef2909)

  {{< figure src="function-test.png#center" alt="Testing functions" >}} 
  Whoa, it almost looks like a programming language.

2. (I) Rework the infrastructure

  At this point I understood my setup to be unsustainable and overly complex.
  So I moved things around and rewrote the CMake files. I have also separated
  tests and added `app/repl` for interactive testing. As a bonus, the
  compilation times reduced considerably.  
  In the past I've been wary of messing with cmake but my current setup seems
  pretty clean.

3. Type system and incremental compiler construction

  - On the fourth day I have spent some time in the morning reading chibicc
    source code and surveying the underlying principles of incremental compiler
    construction. Pretty interesting.

  - Otherwise I decided that `{}` and `if` statements should actually be
    expressions and patched that bit. Sometimes that's a bit more readable so
    why not?

  - Then the hard part happened. I decided it's time to implement some form of
    type-checking. My main learning book is "Crafting interpreters" which
    builds a decidedly untyped language. The kind is which you can freely add
    fileds to structs at runtime. This is not what I want.

    So, instead, I proceeded to read other books on the topic `TAPL` --- Types
    and Programming Languages --- and the source code. As always, the Harec
    source strikes me as being quite simple and beautiful.

    - Type Theory for the Working Rustacean - Dan Pittman - YouTube
    - Type Theory Foundations, Lecture 1 - YouTube
    - ghc-proposals/0378-dependent-type-design.rst
    - https://en.wikipedia.org/wiki/System_F
    - Liquid Haskell

      Although fascinating in their own right. These resources are outside of
      my reach for now. (The idea is that such dead knowledge is but a burden
      is really pronounced in the philosophy. Although it's worth formulating
      it more precisely, as I am in no way advocating for not widening your
      breath of knowledge, rather it's a matter of focusing on the present
      moment) But really --- a lot of insight in type systems.

      If I do decide to try my hand in crafting funtional languages like in
      Simon Peyton Jones' "Implementing functional programming languages" this
      shall come in handy.

    In the end these resources served as references and I have crfated a system
    of my own devise. It did took me a lot of time to wrap my head around it
    though. If everyting goes alright I will be able to finish it do simple
    type checking soon. 

    Some people say that:

    > Having a type system without generics is like having functions which take
    no arugments

    While there's certainly some truth to that. It's still a long way to go for
    me. And honestly, I am not so sure how all that ties with codegen which I
    still have no idea how to do.

    I also have more ambitious plans --- notably HM type inference --- which I
    have detailed in the `docs/todo.md`. Although I am not sure how much of
    that I will be able to accomplish.

Otherwise I should have been populating my wiki and I did somewhat. Although I
am not so sure I would like to interrupt my focus right now.

## Next week

I plan on finishing the type system and starting to think about codegen.

This month has been extremly prolific. That's great news. But the most deepest
deep work is starting right now. It will be a challenge to maintain my focus as
things keep getting harder but I'll try my best.

{{< figure src="closed-tasks.png.png#center" alt="Closed tasks for this month" >}} 


