.. post:: Feb 18, 2018
   :tags: cellular automata, python


Lace Automata
=============



Starting in synch: skipping
---------------------------


I've never seen a skipping fugue, in the Sacred Harp or elsewhere, that skipped
words from the middles or ends of lines. You could do it, but I don't think
singers would find it natural. The desire to end in unison is too strong.


Ending out of synch: trailing off
---------------------------------



Getting in synch
----------------

Aside from the ones that skip at the beginning or trail off at the end, all fugues
face the same basic problem: they start out of sync, and they must arrange to end
in sync. The rest of this post discusses ways of making that happen, which fall into
two broad categories: playing with the *pace,* usually by making some voices stall
for time; and playing with the *words,* usually by repeating lines or phrases.

Stalling
........

Suppose that all four voices are given exactly the same words.  And suppose too
that all four voices proceed at the same baseline pace of one syllable per
quarter note. If the voices are to come into synch by the end of the fugue
instead of trailing off, then some of them will need to deviate from that
baseline, either by speeding up, or by slowing down or pausing.

Interestingly, speeding up is very rare. I don't know of any logical reason for
that --- as far as I know, it's just a matter of style that we tend not to do it.
(My impression is that it's a bit more common in modern fugues with
Stamps-Baxter influence.)

Much more often, then, fugues that play with pacing do it by slowing down or
stopping --- which I will group together as *stalling,* since they have
basically the same effect. 


2-stalls
''''''''

Stalling is almost always done in even numbers of syllables. The most common
kind stalls by two syllables, by taking a measure that would normally fit four
syllables and leaving out two of them. If a pair of voices enter a full measure
apart, it takes two of these stalls in the earlier voice to let the later one
catch up.

.. lily::

    \new ChoirStaff <<
        \new Voice = "tenor" {
            \set Staff.instrumentName = "T"
            \set shapeNoteStyles = ##(la mi fa #f la fa #f)
            \key a \minor
            \autoBeamOff
            \relative c'' {
                r1 | r2 r4 a4 | e' e e8[ d] c4 | d d d8[ c] b4 | c c c8[ b] a[ c]
                   | b4 b b8[ a] g4 | a8[ b] c[ d] e4 d8[ c] | b2. b4 |   a1
            }
        }
        \new Lyrics \lyricsto "tenor" \lyricmode {In all my fears, in all my
            straits, My soul on his sal -- va -- tion waits, My soul on his
            sal -- va -- tion waits.}
        \new Voice = "bass" {
            \set Staff.instrumentName = "B"
            \set shapeNoteStyles = ##(la mi fa #f la fa #f)
            \key a \minor
            \autoBeamOff
            \clef bass
            \relative c {
                r2 r4 e4 | a a a8[ g] f4 | e e e f8[ e] | d4 d d e8[ d] 
                | c2. d4 | e2. e4 | a a g a | e2. e4 | a,1
            }
        }
        \new Lyrics \lyricsto "bass" {\lyricmode {In all my fears, in all my
            straits, My soul on his sal -- va -- tion waits, My soul on his
            sal -- va -- tion waits.}}
    >>

GREENWICH has a fugue where all four parts synch using 2-stalls. Not
counting the last two measures, where all parts are already synched and slow
down together, there are six 2-stalls in the first voice to enter, four in the
second, two in the third, and none in the last. The stalls are arranged so that
each voice gets a satisfying rhythm to sing; and so that at any given moment
there is at least one voice not stalling, preserving the steady quarter-note
rhythm that drives the piece forward.

.. thumbnail:: greenwich.jpg
   :group: 1

A dotted-half/quarter rhythm is more common, but two half notes have the same
effect, such that I think of them as a kind of 2-stall. (I suppose if you
insisted you could call this a pair of 1-stalls, and a violation of my even
syllable rule --- but 

.. lily::

    \new ChoirStaff <<
        \new Voice = "tenor" {
            \set Staff.instrumentName = "T"
            \set shapeNoteStyles = ##(la mi fa #f la fa #f)
            \key g \minor
            \autoBeamOff
            \relative c'' {
                \partial 4
                r4 | r2 r4 bes | c bes a g | d' d d g, | bes c d c | bes bes bes
            }
        }
        \new Lyrics \lyricsto "tenor" \lyricmode {It means Thy praise, how --
        ev -- er poor, It means Thy praise, how -- ev -- er poor.}
        \new Voice = "bass" {
            \set Staff.instrumentName = "B"
            \set shapeNoteStyles = ##(la mi fa #f la fa #f)
            \key g \minor
            \autoBeamOff
            \clef bass
            \relative c' {
                \partial 4
                bes | c bes a g | f f f g | d2 d | f2. f4 | bes, bes bes
            }
        }
        \new Lyrics \lyricsto "bass" {\lyricmode {It means Thy praise, how --
        ev -- er poor, It means Thy praise, how -- ev -- er poor.}}
    >>

And you can also do a 2-stall that crosses a bar line.

.. lily::

    \new ChoirStaff <<
        \new Voice = "tenor" {
            \set Staff.instrumentName = "T"
            \set shapeNoteStyles = ##(la mi fa #f la fa #f)
            \key fis \minor
            \autoBeamOff
            \relative c'' {
                r1 | r2 r4 a | cis cis cis cis8[ d] | e4 e e cis | d d
            }
        }
        \new Lyrics \lyricsto "tenor" \lyricmode {And let His praise
            from ev -- 'ry hill Rise tune -- ful}
        \new Voice = "bass" {
            \set Staff.instrumentName = "B"
            \set shapeNoteStyles = ##(la mi fa #f la fa #f)
            \key fis \minor
            \autoBeamOff
            \clef bass
            \relative c {
                r2 r4 e | a a a2~ | a4 e fis e | a2. a4 | b gis
            }
        }
        \new Lyrics \lyricsto "bass" {\lyricmode {And let His praise
            from ev -- 'ry hill Rise tune -- ful}}
    >>

Stalls also work when they are filled in with passing notes. The musical effect
is different, but the word-setting effect is the same whether you write a
dotted half note or three beats' worth of melisma. 

[TKTKTK]

4-stalls and longer
'''''''''''''''''''

You could look at two consecutive 2-stalls as a 4-stall. Other than that, the
most common kind of 4-stall covers the last beat of one measure and the first
three of the next. One of these is sufficient to synch adjacent parts.

[SABBATH MORNING treble and tenor?] [SWEET MAJESTY treble and alto?]
[ARBACOOCHEE alto and tenor?]


.. lily::

    \new ChoirStaff <<
        \new Voice = "tenor" {
            \set Staff.instrumentName = "Tr"
            \set shapeNoteStyles = ##(la mi fa #f la fa #f)
            \key e \minor
            \autoBeamOff
            \relative c'' {
                \partial 4
                r4 | r2 r4 b8[ d] | e4 e g8[ fis] e[ d] | b4 b b 
                b | d d 
            }
        }
        \new Lyrics \lyricsto "tenor" \lyricmode {Where Je -- sus sheds 
            the bright -- est beams, Where Je -- sus sheds the bright -- est
            beams}
        \new Voice = "bass" {
            \set Staff.instrumentName = "A"
            \set shapeNoteStyles = ##(la mi fa #f la fa #f)
            \key e \minor
            \autoBeamOff
            \relative c'' {
                \partial 4
                g8[ fis] | e4 d g g | e e g2~ | g4 r4 r4 
                g | fis fis 
            }
        }
        \new Lyrics \lyricsto "bass" \lyricmode {Where Je -- sus sheds 
            the bright -- est beams, Where Je -- sus sheds the bright -- est
            beams}
    >>


Longer stalls are possible, but they take up at least an entire measure. When I
write these, I feel less like I'm DOING anything in particular with that voice,
and more like I'm just SETTING IT DOWN until I need to pick it up again. So I
rarely find myself thinking "I need a 6-stall" here. But they exist and
sometimes it's useful to think about them. For instance, the bass part in
SHERBURNE has two 6-stalls in a row that form a sort of rhythmic theme --- the
second feels like an echo of the first. 

[SHERBURNE tenor and bass]

.. lily::

    \new ChoirStaff <<
        \new Voice = "tenor" {
            \set Staff.instrumentName = "T"
            \set shapeNoteStyles = ##(la mi fa #f la fa #f)
            \key b \minor
            \autoBeamOff
            \relative c'' {
                \partial 4
                r4 | r2 r4 a4 | d d fis d | b b b a | d2. a4 | fis fis fis
                a | b2. g4 | e e e2 | r2.
            }
        }
        \new Lyrics \lyricsto "tenor" \lyricmode {The an -- gel of the Lord
        came down And glo -- ry shone a -- round, And glo -- ry shone a -- around.}
        \new Voice = "bass" {
            \set Staff.instrumentName = "B"
            \set shapeNoteStyles = ##(la mi fa #f la fa #f)
            \key b \minor
            \autoBeamOff
            \clef bass
            \relative c {
                \partial 4
                a4 | d d fis d | b b b a | d1~ | d2. fis4 | b, b b d
                e1~ | e2. e4 | a a a2 \bar""
            }
        }
        \new Lyrics \lyricsto "bass" \lyricmode {The an -- gel of the Lord
        came down And glo -- ry shone a -- round, And glo -- ry shone a -- around.}
    >>



Inefficient stalling
''''''''''''''''''''

Those 6-stalls in SHERBURNE make an interesting contrast with some of the other
songs we've looked at. GREENWICH, for instance, is as efficient as possible
with its stalls: it makes the four voices synch with as little stalling as
possible. But SHERBURNE is much more liberal with them: if the goal was just to
make the voices synch, it could make do with fewer; but it includes "extras"
for musical effect. Specifically, it uses them to emphasize the word "glory"
every time it comes around in the text --- and to make that happen, every
voice, even the last to enter, must stall at least a few times. (And then, for
the bass to catch up with the tenor when both parts are stalling, the bass must
stall LONGER. The 6-stalls, in addition to being catchy, serve the practical
purpose of making that synch happen.) 

SHERBURNE is not unusual in this --- lots of fugues have extra stalling, either
for text-painting, to make more interesting rhythms, to give solos to specific
voices, or for whatever other reason. 

Similarly, if fugues were always maximally efficient, then any time two voices
got synched they would STAY synched. But not all fugues work this way. In
SHERBURNE the tenor starts behind the bass, catches up to synch with it, then
gets ahead of it, and finally does an extra text repeat (LINK) to let the bass
catch up. Another interesting example shows up in YE HEEDLESS ONES: the fugue
here starts as the word-skipping variety, meaning that everyone is synched as
soon as they enter --- but then they begin to stall, fall out of synch, and
trail off individually. (TK WORD PAINTING)



