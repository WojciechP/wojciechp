---
title: "What is the problem with Prolog?"
date: "2016-03-25"
categories: [languages]
tags: [logic, prolog, functional]
excerpt:
  Prolog is a clean, mature programming language based on first order logic.
  Despite its mutliple advantages, hardly anyone uses it to build tools - and
  neither do I. What's wrong with it?
meta:
  description: "Why aren't we using Prolog - the logic-based language?"
  keywords: "programming, logic, Prolog, first-order, functional"
---

> I have a kind of love-hate relationship with Prolog.

From technical point of view, it's a mature programming language with good
support: it might look old, but the major open-source implementation
[SWI-Prolog] includes a good deal of updated libraries, so things like OpenID
support or Google Protobuf serialization just work.

[SWI-Prolog]: http://www.swi-prolog.org/

The point of Prolog, though, it's that it's all about logic.

## Logic is Pro

There are no assignments. Frankly, there are no instructions at all. For a
moment this might sound like functional programming - but there are no
functions, too. The programmers can only define rules of the form *"if this,
then also that"*. The rules are completely declarative, meaning Prolog can apply
transformations and reason about them. "Program" and "data" is the same thing.


## The "Love" Part

One can use terms which are only partially defined. This means that inputs and
outputs are interchangeable for many predicates. Therefore there is no big API
to learn: the whole list processing stuff can be built out of the two built-in
predicates:

 - `member(Elem, List)`, which is true if `List` contains `Elem`
 - `append(A, B, R)`, which is true if `B` appended to `A` gives `R`.

So that's a plus number one. But it doesn't end here. Second neat thing is that
there is also meta-api available, so one can write *predicates that operate on
predicates*. These are like macros, just much more powerful. There are
libraries, then, for HTTP, persistence, and other stuff. Finally, Prolog
natively supports parsing grammars, so its super-easy to write parsers for
custom languages (configuration files? HTTP routes?) without being forced to
drop to JSON or XML or something. Don't get me wrong, I like JSON/YAML in
general, but it does not suit *all* the purposes. I refuse to specify my web app
routing in JSON - thankfully, I'm not aware of a framework which requires it.

## The "Hate" Part

There are only predicates and terms, predicates and terms! So "type safety"
means nothing now and one has to be extremely careful writing/reading contracts
for libraries - especially that usually some arguments for a predicate are
*required* and if you pass a variable there weird things will happen (an
exception is the least painful ending). But the biggest itch for me is the fact
that we usually write programs in order for them to *do* things. Since there are
only predicates in Prolog, you need a way to trigger side-effects - so there are
predicates that have side-effects, like `write(...)`, which takes a single
argument, prints it to the console, and always evaluates to `true`. This is
ugly, even more ugly than the Haskell `IO ()` thing.

# So what's the Problem with Prolog

There clearly *is* a problem, because rather few people use it. I just can't
decide whether Prolog is too abstract for us to grasp or too low-level to be
useful. It kind of feels like both.

