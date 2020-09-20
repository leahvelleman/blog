.. post:: 4 Aug 2018
   :tags: cloudburst, linguistics
   :excerpt:


.. default-role:: literal



Finite state transducers operate on strings, but often we want to use those
strings to encode some higher type.  In :ref:`cows` I wrote about a successful
example of this --- treating strings as representing `(String, String)` pairs
in order to thread extra information through a derivation. This post is about
an example that *doesn't* succeed.

In descriptive linguistics, it's common to represent words and their morphology
as *interlinear glossed text* --- a 2-D format in which each morpheme has under it
a gloss indicating its meaning, and in which the whole utterance has under it a free
translation.

   E-dorm-i      crapula-m!      (morphemes)
   out-sleep-IMP hangover-ACC    (glosses)
   'Sleep off that hangover!'    (free translation)

The morphemes and glosses, taken together, can be represented as a list of
morpheme–gloss pairs.

   [('E-','out-'), ('dorm','sleep'), ('-i','-IMP'), ('crapula','hangover'), ('-m','-ACC')]

What if we could create morphological FSTs that output such lists of morpheme–gloss
pairs? First I'll explain in more detail why this might be desirable. Then I'll
talk about why it won't work.




The most common way to write morphological FSTs is to have them take a root and a list
of features on their top/input side, and give a fully declined word on their output side.
For instance, here's the input and output for a derivation of the verb
`katinwiloh` 'I see you' in my favorite morphologically interesting language, K'ichee'.

   il+INC+SUBJ1S+OBJ2S+FINAL
   katinwiloh

(The tags on the input side that start with `+` are grammatical features: `+SUBJ1S` means
"first-person singular subject," `+INC` means "incompletive aspect," and so on.)

Tags aren't glosses
...................

It's important to note that the sequence of tags isn't the same as the sequence
of glosses that this form would receive in interlinear glossed text. If you're here for
the computer stuff, you can take my word for this; but if you're interested in the
linguistics, here are some of the reasons why they don't correspond to each other:

* Sometimes, a single features affects multiple morphemes, and so must be reflected
  in multiple glosses. For instance, in K'ichee' the so-called "status suffix" on the
  verb depends on the verb's aspect, its transitivity, and whether or not it's
  the last word in the sentence. The `-VTIF` gloss below --- standing for
  "Verb, Transitive, Independent, Final" --- reflects at least three different
  input features.

     k-at-inw-il-oh
     INC-2s-1s-see-VTIF

* Sometimes, two features that occupy the same 'slot' in the paradigm are realized
  in different places. For instance, in K'ichee', the informal second-person subject
  and object markers come before the root, but the formal ones come after the root.

      k-at-inw-il-oh
      INC-2s-1s-see-SS
      'I see you'

      k-inw-il=lah
      INC-1s-see=2s.FORMAL
      'I see you, sir'

  But for usability, we want to specify our intput features in the same order, so
  that the inputs for the two forms above must still look like this:

      il+INC+SUBJ1S+OBJ2S+FINAL
      il+INC+SUBJ1S+OBJ2SFORMAL+FINAL

* Sometimes, a feature isn't realized at all, *but still needs to be there in
  the input.* In the so-called Agent Focus verb form in K'ichee', only one
  argument is marked, but you have to know the person and number of both
  arguments to know which one it will be. So in the first example below, it's
  the subject that the verb agrees with, but you still need to have known about
  the object in order to determine that.

      in k-in-il-on-ik
      me INC-1s-see-AF-SS
      'it's ME who sees them'

  And in the second example, it's the object that the verb agrees with, but you
  still need to have known about the subject in order to determine that.

      are' k-ee-il-on-ik
      him  INC-3p-see-AF-SS
      'it's HIM who sees them'

  This means that the input for the first example needed to look something like
  this, with a `+OBJ3P` tag that isn't reflected by any gloss

      il+INC+AF+SUBJ1S+OBJ3P+FINAL

  and the input for the second needed to look like this, with a `+SUBJ3S` tag
  that isn't reflected by any gloss.

      il+INC+AF+SUBJ3S+OBJ3P+FINAL




