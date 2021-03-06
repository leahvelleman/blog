.. post:: 11 Nov 2017
   :tags: cloudburst, linguistics
   :excerpt: 4


.. default-role:: literal

Finite State Transducers and the Two Kinds of Suck
==================================================

Over the summer I set out to make :doc:`a language description plugin for rST </posts/redundancy>` that
would let you call on Fancy Linguistic Resources — like, say, finite state
morphological transducers. 

Along the way, I've learned one thing:

   Writing finite state transducers by hand *really sucks.*

I think it might not have to.


What FSTs are
-------------

I'm not going to go into a real detailed explanation here, but a finite state
transducer is basically a slightly broken Turing machine. It
works in a similar way, with symbols on tape. But unlike a Turing machine, *it
isn't Turing-complete:* there are limits on what it can calculate. [1]_

This is actually a selling point in linguistic applications: it turns out that
some NLP tasks, like morphology (constructing words from their parts, or breaking
them back down into parts), don't *need* Turing-completeness, and so using FSTs
for these tasks is effective — and often more convenient than using more powerful
formalisms. 

What we talk about when we talk about suck
------------------------------------------

So ok. Why does writing FSTs suck?

Intuitively you might think it's because they're Turing-incomplete. And if
that is why, then there's no fixing it — the reason people use them at all is
that they're underpowered enough to have some convenient properties, so adding
power is out of the question. Let's call this Type I Suck.

But there's another possibility, which is that they suck for the same boring
reasons that a lot of Turing-complete languages suck — let's call this Type II Suck.
And actually, the most widely used FST libraries (XFST, HFST, Foma) clearly have
quite a lot of Type II Suck. Using them means working in a domain-specific
language that has...

	* Syntax from an era when bits of disk space were expensive
	* A tiny-to-nonexistent "standard library"
	* No data types
	* No debugger
	* No namespaces or modules
	* No build system

Fixing *these* problems is actually a thing we can do without adding computational
complexity. Adding types to a language, for instance, doesn't actually increase
the number of things that it can handle in theory. (A Turing-complete language is
Turing-complete with or without a good type system.) But it might well increase the
number of things that a normal human would bother doing in it.

Analogy: regular expressions
----------------------------

Here's an analogy. A lot of people have pointed out that "regular expression"
really means two things. 

One thing it can mean is "an expression that describes a `regular language
<en.wikipedia.org/wiki/Regular_language>`_," meaning a language that can be
recognized by a finite state automaton [2]_ or a read-only Turing machine. If you
wanted to be irritatingly pedantic, you could argue that this line of code was
a regular expression:

.. code:: python

   Trie(word for word in open("/usr/bin/words").readlines())

After all, tries represent (a subset of) regular languages, and this expression
describes one. 

But when most people say "regular expression," they're not talking about
computational power — they're talking about restricted syntax. What they mean
is "one of those irritating strings with a million slashes and asterisks that
you have to use as an argument for `grep`." In fact, regular expressions in the
second sense `don't have to be regular expressions in the first sense
<https://nikic.github.io/2012/06/15/The-true-power-of-regular-expressions.html>`_.
This one, as the previous link explains, isn't:

.. code:: perl

   "/^(a(?1)?b)$/"

It matches strings consisting of an arbitrary number of `a`\ s followed by the 
same number of `b`\ s. That's beyond the computational power of a finite
state automaton. In the computational-power sense, it isn't a regular expresion.
But in the restricted-syntax sense, it is.

So when people say "regular expressions suck," they can mean one of two things:

    1. "It sucks to work in a non-Turing-complete language when you're used to
       Turing-completeness" — which is a complaint about Type I Suck.

    2. "It sucks to try to write real programs to do real tasks when you're
       using a garbage syntax that looks like line noise" — which is a complaint
       about Type II Suck.

And these two problems have different solutions. The first would technically be
solved by switching to Befunge, `doing everything with the S and K combinators
<https://en.wikipedia.org/wiki/Combinatory_logic#Completeness_of_the_S-K_basis>`_,
or writing binary executables bit-by-bit by hand. But none of those would get
you anywhere on the second one.

Back to FSTs
------------

My hunch right now is that FSTs have a lot more Type II Suck than Type I.
The issue isn't they're underpowered. It's that the languages we use to create
them are missing important design features.

I listed some of the problems with them up above:

	* Syntax from an era when bits of disk space were expensive
	* A tiny-to-nonexistent "standard library"
	* No data types
	* No debugger
	* No namespaces or modules
	* No build system

For what it's worth, recent FST libraries have fixed a lot of these.  In
particular, Pynini is a pretty okay Python module that lets you write FSTs
using reasonably-Pythonic syntax, and using Python's development ecosystem.

But one thing Pynini *doesn't* get you is data types --- and my experience with
it makes me think that this is a serious unacknowledged problem, a major source
of Type II Suck that it shares with oldschool FST libraries.

Data types
..........

Because here's the thing. FSTs are defined as mapping strings to strings. But
every finite-state morphology project I've ever seen ends up implementing some
more complicated data type. 

"So the input is a string, but I needed to deal with some long-distance
dependencies, so everything after the `$` symbol gets interpreted as an
agreement feature" --- Ok, what you've really got there is a tuple of a string
and some Booleans. 

"…oh, and the features can be specified in any order. But then there's this bug
where if you specify two different values for the same feature then things get
really weird, so don't do that. And actually I decided a few weeks ago that one
of them was going to be a three-way feature instead of binary." Right,
wonderful, then it's a tuple of a string and a set of enums (with a bug in your
:literal:`set` implementation that you're working around for now).

"…and also, in the part before the `$`, each syllable is followed by a tone
letter, but the tone letter has to be either `H` or `M` or `L` or stuff breaks.
But then a bunch of other rules started mistaking the tone letters for normal
letters, so I put them between angle brackets and set the other rules only to
work outside angle brackets." Cool, great, ok, you just refactored and now
you've got a list of string–enum pairs and then a set of enums.

Except now instead of writing your sound change rules as::

   a b -> c d || e _ f;

you have to write them as::

   a (<[toneletter]>)? b -> c (<[toneletter]>)? d || e _ f [letter]* $

It's like that old joke about every programming language eventually including
a half-assed implementation of most of Common Lisp.  What's cool about FSTs is
that they can support all these types — though only with limitations, and the
limitations aren't always obvious, especially when you're used to
Turing-complete languages that aren't limited in the same way.  What's less
cool is that everyone rolls their own buggy ad-hoc implementation of the types,
and then has to be responsible for maintaining type discipline on their own.

Or it's like you're living in a world where all anyone gets is Javascript
strings. You can get by, but it's going to be pretty ugly. 

What is to be done?
...................

The way FSTs implement all these ad-hoc types is by serializing them to strings
--- or more often, by implicitly serializing them, because the designers have
decided to interpret bare strings using a convention that sneakily captures
extra information.

Because FSTs are Turing-incomplete, not every type can be serialized and still
work in the way you'd expect *and* give results that can be unserialized at the
other end. And the limitations are sometimes subtle. Tuples of strings are easy
if they're guaranteed to all be the same length: you rewrite :literal:`{'abcd',
'1234', '!@#$'}` as :literal:`'a1!b2@c3#d4$'`, the rewrite is trivial to reverse,
and everything's great; they're harder but still possible if the difference in
string length is guaranteed to be under some specific finite number; but tuples
of arbitrary-length strings fall apart. 

So: Imagine a FST library with a serialization plugin architecture. A plugin
specifies how to turn some type --- Python `namedtuples`, say --- into strings
and back, and how to perform a few other operations over them. In particular,
for linguistic applications we'd want plugins to specify how to create
*context-dependent rewrite* rules for our new type. (I'm glossing over technical
details, here — I'll come back in another post, but for now trust me that this
kind of type-raising is possible.)

Now, if we've got `Syl` as a namedtuple representing a syllable, we might write
something like this:

.. code:: python

   nasal = Syl(segments=startswith('m')) \
         | Syl(segments=startswith('n')) \
         | Syl(segments=startswith('ŋ'))
   high = Syl(tone='H')
   low = Syl(tone='L')

   rewrite(high, low, rightEnvironment=nasal)

In English, that's "high-toned syllables become low-toned before another
syllable that starts with a nasal consonant." 

Design patterns
---------------

Thinking about the kinds of data types that people shoehorn into FSTs, I've
started seeing two design patterns that come up over and over.

State
.....

Typically, a FST takes an input string and applies some transformations to it. In
NLP, these transformations can include things like "add a prefix" or "apply a 
sound change." One thing we often want to do is to threat some kind of *state*
through these transformations.

For instance, maybe a word can belong to several classes. Depending on our
application, these might be genders, declensions, part-of-speech categories,
etc. Sometimes an affix might change the class of a word: Italian *–ina* turns
nouns of any gender into feminine ones; English *–ize* turns words from other
part-of-speech categories into verbs . And sometimes an affix might depend on
the class of a word: Italian nouns form plurals in *–e* if they're feminine, in
*–i* if they're masculine; English verbs can take *–ing*, but nouns can't
unless you turn them into a verb first. To handle things like this, we need to
thread a piece of state — the current class of the word — through a derivation.

Or for instance, maybe the form of a word depends on agreement features that
can show up in multiple places. In K'ichee', a Mayan language, most agreement
markers are prefixes, but the formal second-person agreement markers are 
clitics that come after the verb. Importantly, the two can't coexist — if you have
a subject agreement prefix, you can't have a subject agreement enclitic too; and
the same goes for object agreement. So we need to "remember" whether we've added
one of the prefixes, no matter how many other affixes have come between, so that
when we get to the end of the word we know if we're "allowed" to add an agreement
enclitic. Again, this means threading state through the derivation.

The important thing about state is that it applies to a particular *moment in
the derivation* of a word, not to a particular part of the word::

            capital       # N
            capital-ize   # V
            capital-ize-d # Adj
         un-capital-ize-d # Adj

            power         # N
         em-power         # V
         em-power-ing     # Adj
         em-power-ing-ly  # Adv 

We might be tempted to label *–ize* as a "verb suffix" or *em–* as a "verb prefix,"
but really verbhood is a property of a whole word — note that words starting with
*em–* aren't always verbs at all, and that the class of a word really depends on
which affix was added most recently.

Tracking state in an FST is simple. The way I've most often done it is precisely
the way I did it in the example above: by setting off the state from the word
using a separator character (here `#`). Better support for this pattern would mean
handling the separator character in a friendly and automatic way, and adding
functions to query and change state directly, so that instead of this ::

    cdrewrite(acceptor(" ").star() + "# N", "-ize # V")

we could — more verbosely but more legibly — write this::

    when(has_state("N")).then(suffix("-ize"), change_state("V"))

Synchronized notation
.....................

I said a moment ago that state doesn't apply to a particular part of the word.
The other common design case is wanting to add extra information that *does*
apply to a particular part of the word. There are lots of examples of this:

    - Division into morphemes, or into larger morphological domains (like the
      conjunct versus disjunct domains of the verb in Athabaskan, or the word
      itself versus the clitics attached to it in Greek or Romance)
    - Morpheme-by-morpheme glosses
    - Potentially, glosses for domains larger than a morpheme — as in SIL-style
      transcription, where each word gets a word-level gloss
    - Division into syllables, feet, etc.
    - Suprasegmental features like tone or stress that apply to a whole
      syllable or more at a time

I don't know if there's a generic term that refers to all of these. I'll call them
"synchronized annotations" — "synchronized" because they need to be lined up with
the word in a particular way.

If we think of state as being written "off to the right" in a derivation, we
can think of synchronized notation as being written "above" or "below" the word —
as in this Spanish example where morpheme-by-morpheme glosses are written below
the word and stress is written above it::

              !
        soportar            # V Inf
        tolerate 

              !
        soporta -ble        # Adj Sg
        tolerate-able

              !
     in-soporta -ble        # Adj Sg
     un-tolerate-able
     
                       !
     in-soporta -ble -mente # Adv
     un-tolerate-able-ly

The three-dimensionality of this kind of annotation makes it harder to handle than
state, because FSTs operate on strings and strings are basically two-dimensional.
Straightforward examples like the one above can be handled by `zip`\ ing the levels
together, so that the last stage of the derivation would be written as::

     iu nn -- st oo pl oe rr ta at  e -- ba lb el  e -- ml!ey n  t  e     #     A  d  v

And we could of course write an API that would handle the `zip`\ ing and un\
`zip`\ ing transparently. But in the fully general case [3]_ this sort of
representation amounts to a *multi-tape transducer*, and multi-tape transducers
aren't finite-state. So there are decisions that need to be made here about how
much generality we can afford to give up, and what *kinds* of generality we can
afford to give up, while still properly handling the linguistic use cases listed
up above.


Where does this leave us?
-------------------------

So, ok.

We started by noting that hand-writing FSTs sucks. We narrowed in on one
particular *kind* of suck — I ventured that the issue isn't their
Turing-incompleteness, it's the prehistoric programming ecosystem around them.

A lot of the ecosystem issues have been solved. One that *hasn't* — at least,
not in a way that makes it available to non-researchers — is the issue of type.

FSTs fundamentally represent transformations of strings. But often we end up
using them as if they represented some higher-typed thing: transformations of
strings-with-associated-state, or of
sequences-of-tuples-of-strings-with-associated-state, or of
strings-with-synchronized-annotations-and-state. 

This happens often enough that we might as well stop treating it as an edge
case and start treating it as a design pattern. Rather than rigging together
a brand new ad-hoc buggy facsimile of those higher types every time we need one,
what we really should be doing is putting together a library that supports them
in a consistent, well-tested way and then hides them under a nice abstraction
so we can forget about them. In order to do that, we'll need to make some tough
decisions about where we can afford loss of generality without sacrificing
usefulness for linguistic applications.

More on that soon, hopefully.


Footnotes
---------

.. [1] The usual way to think about FSTs is as Turing machines that have two
   tapes, but can't rewind either one of them. It's common to think of one tape
   as the input and the other as the output. It turns out you don't have to
   think of them in input/output terms --- but if you do take the input/output
   point of view, then it works like this: an FST can always read the next
   symbol from its input tape, and can always write another symbol on its
   output tape, but can never go back to reread or to overwrite.

   This inability to rewind is enough to make them substantially weaker than
   Turing machines, which for a lot of appliations is actually a good thing.
   It means that it's trivial to check whether a FST runs forever or halts; it's
   always decidable whether it maps a specified input to a specified candidate
   output; and if `f` and `g` are FSTs then `f∘g` is a FST that runs almost as
   quickly. This makes them useful for things like speech recognition, autocorrect,
   and morphological analysis, where you have a lot of possible inputs and a lot
   of possible outputs and need to match them up very quickly but don't need to
   handle recursively nested structures.


.. [2] Yup. Regular expressions in this first sense of the word are equivalent
   in their expressiveness to FSTs, and so both are Type I Sucky in exactly
   the same ways. 
   
   Their Type II Suckiness is also historically connected —
   the most widely used languages for describing FSTs are based on regular
   expression syntax.

.. [3] The issue, as I understand it, is that this `zip`-style representation
   is too fine-grained: it treats all of these as distinct, when really they
   ought to be interchangeable, since the spaces aren't meaningful and are
   just there to get the levels to line up::

       in-soporta -ble -mente
       un-tolerate-able-ly

       in- soporta -ble-mente
       un-tolerate-able-ly

       in-   soporta   -ble   -mente
       un-   tolerate  -able  -ly

   If we think of each line of the representation as a tape, then the `zip`-style
   representation amounts to pasting those tapes together,
   allowing us to treat them as a single tape. But really we want to allow
   some "slippage" between the tapes, so that different ways of aligning them
   can be treated as equivalent — and the trouble is that if we allow
   unlimited slippage, then we've got a multi-tape transducer and we lose our
   convenient finite-state properties. (This is analogous in an interesting
   way to adding backreferences to regular expressions: both multitape
   transducers and regexps with backreferences can generate the language a\
   :sup:`n` b\ :sup:`n`\ , which isn't a regular language, bumping them both
   up into a higher complexity class.)

   Anyway, all this means there are tradeoffs to be made.
