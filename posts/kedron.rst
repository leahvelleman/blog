.. post:: Sep 7 2014
   :tags: fasola
   :category: music
   :author: me

Kedron
======

Kedron (SH 48) is known to most Sacred Harp singers as a lovely, stark and
simple minor-key tune. So I was startled to see that the `original version,
<http://people.bethel.edu/~rhomar/TunePages/Kedron.html>`_ in the *United States
Sacred Harmony*, is strikingly chromatic in a way unlike anything else I've
encountered in the modern shapenote repertoire.

.. lily:: 
   
   \new StaffGroup <<
   \new Staff = "treble" { 
    \set shapeNoteStyles = ##(la mi fa #f la fa #f)
    \key e \minor \relative c'' {
        b2 b4 b4 | g2 g | a4 c b2| b b4 b | b2 a | g4 e b'2 b b4 e | d2 e | b4
        b d2 | d d4 c | b2 g4.( a8) | b4 b b2 
   }
   }
   \new Staff = "alto" {
    \override Accidental #'color = #red
    \set shapeNoteStyles = ##(la mi fa #f la fa #f)
    \key e \minor \relative c'' {
        e,2 e4 e e2 \once\override NoteHead #'color = #red g fis4 e
        \once\override NoteHead #'color = #red fis2 \once\override NoteHead
        #'color = #red d2 e4 e e2 d2 \once\override NoteHead #'color = #red b4
        \once\override NoteHead #'color = #red cis \once\override NoteHead
        #'color = #red d2 g2 fis4 g d2 e d4 \once\override NoteHead #'color =
        #red d \once\override NoteHead #'color = #red d2 fis2 g4
        \once\override NoteHead #'color = #red fis e2 e \once\override
        NoteHead #'color = #red dis4 \once\override NoteHead #'color = #red
        dis  e2 
    }
   }
   \new Staff = "tenor" {
    \override Accidental #'color = #red 
    \set shapeNoteStyles = ##(la mi fa #f la fa #f)
    \key e \minor \relative c'' {
        g4.( fis8) e4 e | b'2 b | a4 g fis2 | g4.( fis8) e4 e | e'2  f4( e) |
        d c b2 e2 dis4 e | \afterGrace b2 { \once\override NoteHead #'color =
        #red a8 } g2 | d'4 \afterGrace b4 { \once\override NoteHead #'color =
        #red a8 } a2 a b4 e, | \afterGrace g2 { a4 } \afterGrace  b2 { a4 } |
        g4 fis e2
    }
   }
   \new Staff = "bass" {
    \set shapeNoteStyles = ##(la mi fa #f la fa #f)
    \key e \minor \clef bass \relative c {
        e2 e4 b | e2 e | d4 e b2 | b e4 e | g2 d | \once\override NoteHead
        #'color = #red d4 e b2  e b'4 b | g2 b | g4 e d2 | d g4 g8.[ fis16] |
        e2 e | b4 b e2 
    }
   }
   \new Lyrics \lyricsto "alto" {\lyricmode {Thou man of grief re - mem - ber me}}
   >>
   
The tenor line is the same as the familiar one, except for two grace notes and
two accidentals --- but *what accidentals!* The first of the two, the lowered
`mi` in bar 5, is totally unique as far as I know: you occasionally hear a
lowered `mi` in a major-key tune, but I've never seen one in this tradition in
minor.  Technically speaking, the effect is to take us temporarily out of
standard Aeolian minor and into the Phrygian mode.  Aesthetically... well, I
think it's haunting, though also quite unfamiliar-sounding (and hard to sing,
at least for me).

The alto line has quite a few changes, but there are two that jump out. I'm
very fond of the `so` in bar 4, which changes the harmony: it is now a G major
chord rather than an E minor. The raised `fa` in bar 6, on the other hand,
clashes so dramatically with the melody that it is almost unsingable unless
you raise the melody's `fa` as well --- which some singers would do anyway.

It's also interesting to see the extra grace notes in the tenor line --- and
this time it's not because they're unusual, but because they're so ordinary.
At least one of the two, the one in bar 9, I'm fairly certain I've heard sung
before: it is a sort of embellishment that many modern traditional singers add
without thinking, and seeing it written in here makes me wonder if it was
already part of the tradition then.

