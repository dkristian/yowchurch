Church encoding workshop for YOW Lambda Jam 2016
=======================

First, install [cabal](https://www.haskell.org/cabal/), from the site, or with `brew install cabal` or whatever.

Then, clone this repo and you should be ready to go.  

`cabal run` will run the tests, which assess your answers. 


Ex 1: Church booleans
--------------------------
A Church boolean is a function with two possible answers: one representing "true" and one representing "false".  It is of course equivalent to the good ol' if/else statement that is in pretty much every language. Here we rebuild boolean logic out of functions -- and, or, not.

Ex 2a: Peano numbers
---------------------------
Peano numbers represent non-negative integers with a simple recursive definition; they can be Zero, or a Successor to another Peano number. By induction this covers all of the natural numbers, but doing anything with them is very inefficient -- at least proportional to the size of the integer!  Here we recreate addition, multiplication and exponentiation, using the Peano representation.

Ex 2b: Church numerals
---------------------------
We can derive Church numerals by applying the usual Church-encoding technique to Peano numbers. In essence, we take a function and compose it 0-N times. 

The resulting function takes two arguments: 
* a function `r -> r`: "Perform this action N times..."
* a zero value `r` "...starting from here."  

Once again we can rebuild addition, multiplication and exponentiation, now from nothing but functions! This time though, we should be able to add and multiply in constant time.

Ex 3: Church lists
-------------------------
Lists are very similar to the natural numbers, differing only in that they annotate each number with an element of some other type.  The Church encoding of lists gives us a familiar function -- the usual right fold on lists! (`foldr` in Haskell) 
Here we rebuild the empty list, cons, and a constant-time list append.

Ex 4: Church free monads
-------------------------
Functors are an important concept in FP; in programming they allow us to map over values in some larger computation or structure. If we make a "Church numeral" out of a functor, composing it with itself 0-N times, then something interesting happens -- the result is a monad! This is called the _free monad_ generated by the functor.

For example, the free monad of the List functor might contain:
* 22  (just a value)
* [1, 22, -5]   (a list)
* [[3,3,5], [44, 71, 0], []]   (a list of lists)
* [[[1,2,4],[8],[9,0,3,5]], [[2,3],[5,6],[1]], [[], []]]   (a list of list of lists)
* etc

The free monad of a pair (a, r), polymorphic in `r`, might contain:
* ()    (just a value)
* ("bob", ())   (a pair)
* ("bob", ("anna", ()))   (a pair of a pair)
* ("bob", ("anna", ("liz", ())))   (a pair of a pair of a pair)

Why, it seems that we have re-invented the linked list! Many familiar recursive structures can be deconstructed into a free monad of some functor.

Free monads are particularly interesting in everyday FP, as they are a powerful tool for separating the representation of logic from its execution. Taking an "instruction set" as the functor, the resulting free monad is essentially an abstract syntax tree. 

To enable all these nested layers to inhabit the same type, the representation needs to wrap a recursive layer, and terminate with a pure value. This usually takes the form of a simple inductive data structure, similar to cons lists and Peano numbers:
```
data NaiveFree f a 
  = Wrap (f (NaiveFree f a))
  | Pure a
```

Unfortunately, it has the same performance problems as cons lists and Peano numbers, multiplied by whatever branching factor the functor has!  Each >>= has to bubble down from the top of the tree down to the leaves, finally substituting in the next layer at the bottom.

Fortunately, Church encoding can save the day! When we Church-encode this structure, its behaviour changes: now the underlying functor is never called!  This means that >>= runs in constant time, just composing continuations. Functions for the win!

Here we reimplement free monad functionality, using Church encoding:
* pure
* wrap
* fmap
* bind
* foldFree

Be sure to have a look at the naive implementation in NaiveFree.hs.

