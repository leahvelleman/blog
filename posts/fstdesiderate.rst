.. post:: 13 Nov 2017
   :tags: cloudburst, linguistics
   :excerpt:


.. default-role:: literal

Desiderata
==========

Two easy pieces
---------------

#1 Script transducer construction
..............................

The first thing I want is to be able to construct and combine transducers using
functions in a scripting language (Python, Lua, Ruby...) rather than by
assembling them using regular expression syntax.  In Python, this might mean
writing
these ::

    fst1.union(fst2, fst3, ... )
    fst1.compose(fst2)
    fst1.star()
    change(fsa1, fsa2).between(fsa3, fsa4)

rather than these::

    fst1 | fst2 | fst3
    fst1 .o. fst2
    fst1*
    [ fsa1 -> fsa2 || fsa3 _ fsa4 ]

The advantage of Python syntax isn't immediately obvious in these short
examples. What makes this feature useful is that it lets you take advantage
of scripting language features like loops, accumulators, and `reduce`, letting
you build up a transducer incrementally. For instance, if `C` is a
list of consonants, you can create a consonant gemination rule like this::

    gemination = reduce(FST.compose, 
                            (change(cons, cons+cons) for cons in C))

This single line of code expresses a simple generalization that, in
regular expression syntax, would require listing out every possible
gemination target::

    [ p -> pp, t -> tt, k -> kk, b -> bb, d -> dd, g -> gg,
      f -> ff, v -> vv, s -> ss, z -> zz ... ]

(Pynini already allows this, though with slightly different Python syntax than
what I've shown. Foma gets partway there, but it forces you back into regular
expression syntax if you want to write a condext-dependent rewrite rule, which
is a pretty serious limitation.)

#2 Script transducer access
........................

The second thing I want to be able to do is *access* the contents of
transducers and acceptors using idiomatic scripting
syntax. In Python, this would mean giving finite state acceptors 
the methods `__len__()` (so the `len()` function works), `__contains__()`
(so `in` works), and `__iter__()` (allowing `for ... in`). ::

    >>> stops = acceptor("p", "t", "k", "b", "d", "g")
    >>> len(stops)
    6
    >>> "k" in stops
    True
    >>> [cons+cons for cons in stops]
    ["pp", "tt", "kk", "bb", "dd", "gg"]

(In fact, if you combine this with #1, acceptors start to look an awful
lot like sets — something we can run with by setting up magic methods
for the other infix operators sets support.) ::

    >>> voiced = acceptor("b", "d", "g", "z", "m", "n")
    >>> "p" in stops | voiced
    True
    >>> "p" in stops & voiced
    False
    >>> "b" in stops - voiced
    False

Finite state transducers are mappings, so we'd want to give them the same
methods as the built-in Python mapping `dict`. ::

    >>> V = acceptor("a", "e", "i", "o", "u")
    >>> lenition = change("t", "d").between(V, V)
    >>> lenition["ata"]
    "ada"

    >>> gemination = reduce(FST.compose, 
                               (change(cons, cons+cons) for cons in C))
    >>> gemination.items()

(There are complications here, since FSTs can be many-to-many instead of
one-to-one; `fsmcontainers <https://leahvelleman.github.io/fsmcontainers/>`_ is
an attempt at handling this in a reasonable way.)

Here's an example showing why this matters. In the example gemination
rule under feature #1 above, I said `C` was a *list* of consonants.  But
if you have feature #2 as well as feature #1, `C` could be an *acceptor*
that accepts single consonants. This change would mean you could loop
over `C` as before::

    gemination = reduce(FST.compose,
                            (change(cons, cons+cons) for cons in C))

*and also* use it in FST operations like concatenation (here written as `+`), Kleene
star, and context-dependent rewrite::

    cluster = C + C + C.star()
    cluster_simplification = change(C, "").before(cluster)

(This time, Foma is the one who gets it right. TK MORE)

Two harder ones
---------------

#3 Abstractions for state
.........................

In my previous post, I talked about the frequent need to maintain *state*
in the course of a derivation. Rather than representing stateless transformations
over strings, we often want transducers to represent stateful transformations.
Ideally, a finite-state morphology toolkit would have a built-in set of abstractions
for handling state in an intuitive and transparent way.

#4 Abstractions for synchronized annotation
...........................................

I talked about this one in the previous post too. Often, rather than having
transducers represent transformations over flat strings, we want them to
represent transformations over some three-dimensional data type representing
strings plus additional annotation. This would be a killer feature for
documentary linguists, since it would let us generate a word and its
interlinear gloss in one pass. A finite-state morphology toolkit should have a
set of abstractions for handling this sort of annotation in an intuitive and
transparent way.

Why they're hard
----------------

Until recently, I thought that we could handle #3 and #4 as serialization
operations — take some complicated object with annotations and whatever else, 
serialize it into a string, then feed that string into an FST that we'd
constructed in the usual way, and finally deserialize the output string to
get another complicated object.

Feelings of impending doom
..........................

I had a nagging feeling there was *something* wrong with this, but it took me
a while to work out exactly what it was. It's easy enough, though, to see that
a totally naïve serialization/deserialization approach is bound to fail.
Suppose we have a function `suffix()` that creates suffixing transducers, and
another one `prefix()` that creates prefixing transducers. ::

    >>> negate = prefix("un")
    >>> negate["interesting"]
    "uninteresting"
    >>> negate["clear"]
    "unclear"

    >>> comparative = suffix("er")
    >>> comparative[negate["clear"]]
    "unclearer"

Now suppose we want to add a piece of state to our derivation --- let's say
one that tracks the part-of-speech category of the word we're building. Now
instead of words we have `(word, state)` pairs. How might we serialize this?
Well, one way, as suggested above, would be to serialize `("clear", "Adj")` into
`"clear <Adj>"`, and deserialize by doing the opposite. But now if we feed this
serialized form into one of our old suffixing transducers, we'll run into
trouble. ::

    >>> comparative["clear <Adj>"]
    "clear <Adj>er"

Ok, so what if we put the state on the left instead of on the right? Well, now
prefixing breaks. ::

    >>> negate["<Adj> clear"]
    "un<Adj> clear"

And... well, shit, *where else could we even put it*? In the middle of the
word?  (This is pretty obviously a terrible idea, but if you're not convinced,
note that it would break infixing just like the other options we've considered
broke prefixing and suffixing.)

What we'd actually need to do, to make this work, is upgrade our `prefix()` and
`suffix()` functions. We need to write new ones --- call them
`state_aware_prefix()` and `state_aware_suffix()` --- that produce transducers
that know how to ignore substrings between brackets. ::

    >>> new_negate = state_aware_prefix("un")
    >>> new_negate["<Adj> clear"]
    "<Adj> unclear"

And even this isn't really enough. After all, all we're doing so far is passing
along an unchanging piece of state. We'll also need ways of getting and setting
that state...

Aw shit, we need monads
.......................

At this point it's pretty clear that serialization and deserialization aren't
enough. No matter how cleverly we construct our serialization and deserialization
scheme, it won't do everything we want it to.

In fact, we shouldn't look at this as serialization at all. I think the way
we should look at it is as a *monad transformation,* or at least as something
akin to one. When we contemplate switching from words to `(word,
state)` pairs --- and writing functions to access and change the state, and
updating all our other functions to be state-aware --- what we're really doing
is awfully similar to putting data into the State monad.

This gives us a nice way of understanding why we need to rewrite `prefix()` and
`suffix()` and so on.  When you put your data in a monad, it's not enough to
change the data itself --- you also need to *lift* all the functions you'll be
using on it into the monad.  What I called
`state_aware_prefix()` and `state_aware_suffix()` above would just be *lifted*
versions of `prefix()` and `suffix()`.

I'm honestly not sure how deep this analogy goes — we're already running up
against the edges of my knowledge. But I think it's safe to say we need one of
two things:

    - A consistent programmatic way of lifting FST-generating functions into
      the State monad --- like Haskell's `lift`, but for 


This feature is the thing on this list that I'm the least sure about, but it's
probably the most important if any of this stuff is going to be of any
practical use. Being able to lift functions into a monad often means being able
to *ignore* the inner workings of the monad. Building fancy representations for
state and annotations isn't much good if every time we try to add a prefix we
have to puzzle out how those representations will interact with it.  Having a
`lift()` function we could apply to `prefix()`, or a hand-written lifted
version of `prefix()`, would mean we could just apply a prefix in a natural way
without thinking about what was under the hood.

#6 Debug mode
-------------

    >>> kichee["b'ihn<walk> (inc subj2s obj3s caus Vintrans)"]
    """ k-  aa- b'iin-isa -aj     (inc subj2s obj3s Vtrans)
        INC-A2s-walk -CAUS-STATUS 
    """

Foo ::

    >>> kichee.debug["(inc subj2s obj3s caus) b'ihn<walk>"]
    """ b'ihn                     (inc subj2s obj3s caus Vintrans)
        walk

        b'ihn-isa                 (inc subj2s obj3s Vtrans)
        walk -CAUS

        b'ihn-isa -aj             (inc subj2s obj3s Vtrans)
        walk -CAUS-STATUS

        a-  b'ihn-isa -aj         (inc subj2s obj3s Vtrans)
        A2s-walk -CAUS-STATUS
       
        k-  a-  b'ihn-isa -aj     (inc subj2s obj3s Vtrans)
        INC-A2s-walk -CAUS-STATUS

        k-  aa- b'ihn-isa -aj     (inc subj2s obj3s Vtrans)
        INC-A2s-walk -CAUS-STATUS

        k-  aa- b'iin-isa -aj     (inc subj2s obj3s Vtrans)
        INC-A2s-walk -CAUS-STATUS
    """
