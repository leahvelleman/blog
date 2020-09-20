.. post:: 13 Nov 2017
   :tags: cloudburst, linguistics
   :excerpt:


.. default-role:: literal

Cows
====

Normally, finite state transducers represent strings --- or sets of strings, or
relations between strings. But in a lot of projects they're used to emulate a
higher data type --- say, `string, string` pairs. 

When you go from "these FSTs represent strings" to "these look like they
represent strings, but they actually represent tuples," there are some changes
you need to make in your existing code.

It would be cool if we could do that in a principled way instead of just
winging it and hoping nothing breaks.

An example
----------

For concreteness, here's a thing I did recently. I wanted to keep track of
some grammatical features while I was deriving words. So I decided, okay, all
my strings are now going to have a `#` in the middle. Everything after the `#`
is a grammatical feature. Everything before the `#` is the word I'm deriving. 
So if I'd been working in English, instead of looking like this, ::

    think
    thinkable
    unthinkable
    unthinkably

my derivations would now look like this. ::

    think # +V
    thinkable # +Adj
    unthinkable # +Adj
    unthinkably # +Adv

Here are some things I needed to change in my existing code after making this
switch:

    - Suffixing rules needed to know how to skip the grammatical features.
      Otherwise, adding the suffix `-ed` to the verb `walk # +V` would give
      this ::

          walk # +Ved
      
      rather than this: ::

          walked # +V
          
    - There needed to be a way to specify suffixing rules that would *change*
      grammatical features.
    - Compounding needed to work differently too: `black # +Adj` plus `bird # +N`
      needed to be ::

          blackbird # +N

      rather than ::

          black # +Adjbird # +N





A partial solution
------------------

Is that everything? *I wasn't sure.* I wasn't even immediately sure how to
*become* sure. 

It turns out there's a nice answer to this. Complicated FSTs are built out of
simple ones using a limited number of operations. Some of these are general
operations from set theory (if `a` and `b` are FSTs representing sets, then `a
∪ b` and `a ∩ b` and so on are all FSTs too; if `f` and `g` are FSTs representing
relations, then `f ∘ g` is a FST). But some are string-specific. Well, when
we start using the strings in our FST to represent some other data type, it is
*precisely the string-specific operations* that need to be changed.

String-specific operations
..........................

.. epigraph ::

   Hildebrand: Mind if we join you?

   Trapper: We were gonna join the nurses.

   Hawkeye: Yeah, we were gonna join them and make one big nurse.

   -- M*A*S*H, Season 2 episode 1

Here's what I mean by "string-specific." The operations of set intersection and
set union can be used on any kind of set at all. If our sets are sets of cows,
sets of functions, sets of strings, sets of other sets --- it doesn't matter,
we can intersect them with one another. Not only that, their type will stay the
same: the intersection of two sets of cows is a (possibly empty) set of cows.

By contrast, concatenation is string-specific. Given two sets of strings, it
makes sense to say "Make a set of larger strings by concatenating one string
from set `a` and one string from set `b`." But given two sets of cows, it
makes no sense to say "Make a set of larger cows by concatenating one cow 
from set `a` and one cow from set `b`." You can't make a larger cow by
concatenating two smaller ones. That's not how cows work.

There are three big categories of string-specific FST operations. The first
are the ones that involve concatenation. In addition to concatenation itself,
this includes Kleene star and plus (which are just repeated concatenation), and
also operations like "`a` repeated exactly 3 times."

The second are the ones that depend on substring or superstring relationships.
Strings have the interesting feature that a part of a string is itself a string.
Not everything that shares this feature; cows don't have subcows or supercows.
There's a whole family of containment operators that fall into this category,
that let you specify things like "the set of all strings that contain `a` as
a substring exactly once." 
