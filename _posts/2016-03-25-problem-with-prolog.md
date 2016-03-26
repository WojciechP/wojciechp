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


## Do Logic like a Pro

Remember some logic classes back at school? Like, predicates, atoms, and terms?
That's all you get in Prolog. There are no loops, no methods, no assignments (so
it smells of functional programming), but also no constants, no return values
and even **no functions**.

So let's write a predicate that checks whether one list is a prefix of another:

```prolog
list_prefix([], _). 
list_prefix([H|PT], [H|T]) :-
  list_prefix(PT, T).
```

There are two entries there. The first one says that an empty list is a prefix
of anything. That's not entirely true, the *anything* in question should still
be a list, but let's leave that. The second one says that a list with head `H`
and some tail `PT` is a prefix of another list with the same head `H` and tail
`T` if `PT` is a prefix of `T`.


Having this we can fire up the interactive console and do some tests:

```prolog
?- list_prefix([], []).
true.

?- list_prefix([a], [a,b,c]).
true.

?- list_prefix([a,c], [a,b,c]).
false.
```

You get the idea. But now, the magic happens if instead of using lowercase atoms
we supply an uppercase variable:

```prolog
?- list_prefix(X, [])
X = [] .

?- list_prefix(X, [a])
X = [] ;
X = [a].


?- list_prefix([a, b], X).
X = [a, b] ;
X = [a, b,  _G1190|_G1191].
```

So what happened, exactly? By supplying variables in predicate arguments, we
asked the Prolog runtime to find possible valuations of these values so that the
predicate holds. In the first example, Prolog found one possible valuation - an
empty list, which makes sense, as it's the only possible prefix of another empty
list. In the second example, two prefixes were found - that's cool. The third
example is more interesting: first found solution is `[a, b]`, which is fine.
The second one is in fact a whole family of solutions - `[a, b, _G1190|_G1191]`
should be read as "Any list that starts with elements `a`, `b`, and `_G1190`,
and then has any tail `_G1191`", where these `_G1190` and `_G1191` are any
values you wish (recall that if something starts with an uppercase, it's a
variable in Prolog. Well, these start with `_`, because they were not introduced
by the user).. So it also makes sense.

## What Happened to the Control Flow?

Let's continue our inquiry:

```prolog
?- list_prefix([a, X, b], Y), list_prefix(Y, [a, b, Z, T]).
X = Z, Z = b,
Y = [a, b, b] ;
X = Z, Z = b,
Y = [a, b, b, T].
```

This time we're introducing more variables - `X`, `Y`, `Z`, and `T`. Moreover,
we're specifying two comma-separated goals at once - this will force Prolog to
search for valuations which satisfy both of them (so it's kind of an `&&`). Our
query yields two results.

The first one is `X=Z, Z=b, Y=[a,b,b]`. The `X=Z` and `Z=b` is quite obvious;
`Y` needs `b` on both second and third position. Then there is `Y=[a,b,b]` and
no constraints on `T` whatsoever. Looks legit.

The second result also requires `X=Z, Z=b` - which is nothing weird - and also
sets `Y = [a, b, b, T]` without any restrictions on `T`. So the only difference
is that the `Y` list is one element longer. That works too.

Now, if you pause for a moment and zoom out on what exactly happened, there is
something really interesting going on, because despite having no functions in
the language we managed to perform a sort of calculation. And not an easy one,
because there is no clear input and output in our task. In fact the single
`list_prefix` predicate, which is 4 lines of code, can be used to a wide variety
of tasks, which are even hard to name (what did we just do? "Merged" two lists?)
- and this is how the whole Prolog standard library is constructed.

## The "Love" Part

Since input and outputs are interchangeable, there is no big API
to learn. You can do the whole list processing stuff using the two built-in
predicates:

 - `member(Elem, List)`, which is true if `List` contains `Elem`
 - `append(A, B, R)`, which is true if `B` appended to `A` gives `R`.

So that's a plus number one. But it doesn't end here. Second neat thing is that
there is also meta-api available, so you can write *predicates that operate on
predicates*. These are like macros, just much more powerful. There are
libraries, then, for HTTP, persistence, and other stuff. Finally, Prolog
natively supports parsing grammars, so its super-easy to write parsers for
custom languages (configuration files? HTTP routes?) without being forced to
drop to JSON or XML or something. Don't get me wrong, Ilike JSON/YAML in
general, but it does not suit *all* the purposes. I refuse to specify my web app
routing in JSON - thankfully, I'm not aware of a framework which requires it.

## The "Hate" Part

You only have predicates and terms, predicates and terms! So "type safety"
means nothing now and you have to be extremely careful writing/reading contracts
for libraries - especially that usually some arguments for a predicate are
*required* and if you pass a variable there weird things will happen (an
exception is the least painful ending).
But the biggest itch for me is the fact that we usually write programs in order
for them to *do* things. Since there are only predicates in Prolog, you need a
way to trigger side-effects - so there are predicates that have side-effects,
like `write(...)`, which takes a single argument, prints it to the console, and
always evaluates to `true`. This is ugly, even more ugly than the Haskell `IO
()` thing.

# So what's the Problem with Prolog

There clearly *is* a problem, because rather few people use it. I just can't
decide whether Prolog is too abstract for us to grasp or too low-level to be
useful. It kind of feels like both.

