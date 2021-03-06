.. post:: Feb 18, 2018
   :tags: cellular automata, python


Lace Automata
=============

.. image:: laceautomata_sample.png

Sometime around fifth grade I either invented or
stole-and-then-forgot-about-stealing a kind of rule-based doodles that I later
realized were cellular automata.

**Update:** Wherever I got them from, it looks like they've been independently
invented at least one other time. `Paige Gulley was posting art made with this
style of automaton a few years ago.
<https://then-there.tumblr.com/post/77739107796>`_ She says she didn't learn
about them anywhere, but came up with the idea herself, the same way I remember
doing it: doodling while bored in class.

.. raw:: html

   <p><strike> I've looked for information about them since and not been able to
   find any --- which makes me think either I did invent them, or else wherever
   I stole them from isn't very well-known. If I did did invent them, I'll call
   them "lace automata," because a lot of them form open, lacy patterns of
   crossing lines and cables.</strike></p>

I've started playing around with them again recently, and trying to work out
some of their properties. As cellular automata, they're similar to, but not the
same as, Wolfram's `elementary cellular
automata <en.wikipedia.org/wiki/Elementary_cellular_automaton>`_.  The
differences mean there are a *lot* more lace automata than ECAs:
:math:`(2^3)^{(2^3)}=16777216` rather than :math:`2^{(2^3)}=256`. And they mean that
lace automata can do some things ECAs can't, like make patterns that grow at
irregular speeds, or at less than the "speed of light," or that alternate
between growing and shrinking.

How they work
-------------

A lace automaton generates a pattern of vertical and diagonal lines drawn against
a grid, like the one at the top of this post. It generates the pattern
row-by-row, following a rule. The rule tells you things like "If two diagonal
lines came together at a point at the end of the last row, draw a vertical line
down from that point in the next row."

For instance, the picture at the top of this post was generated by this rule:

.. image:: laceautomata_rule_1.png

Suppose we start with a single vertical line. Our rule says that it splits in
three, like this --- and that's the end of our second row:

.. image:: laceautomata_1.png

In the third row, we have to work out how each of the new lines evolves.
There is a leftward diagonal, which --- consult the rule above --- splits in two;

.. image:: laceautomata_2.png

a vertical line, which --- as we already know --- splits in three;

.. image:: laceautomata_3.png

and a rightward diagonal, which splits in two. So here's row three
complete:

.. image:: laceautomata_4.png

In the fourth row, we start to see lines meeting each other. Working
from left to right, first we find a single left diagonal, which we know how to
handle:

.. image:: laceautomata_5.png

Then we find a left diagonal coming together with a vertical line. The rule
specifies a different outcome for this than for a left diagonal alone:

.. image:: laceautomata_6.png

And so on for the rest of the row:


.. image:: laceautomata_7.png

Similarly, for the fifth row --- for which I won't work through all the steps
--- at the very center of the pattern, we have to apply for the first time
the part of the rule that says what to do with three lines coming together:


.. image:: laceautomata_8.png




Lace automata are cellular automata
-----------------------------------

Intuitively, this sure seems like a cellular automaton. It's built on a
rectangular grid. It's rule-generated. The rule refers only to a limited
neighborhood --- where the neighborhood consists of the lines converging at a
single point. 

But the business with the lines is unusual. 

In a normal cellular automaton, each state occupies a single spot on the grid,
rather than running from one place to another. So let's see if we can represent
lace automata in that more normal form.

States
......

A normal cellular automaton has a fixed set of states. Each spot on the grid has
a single state assigned to it. 

The counterpart in a lace automaton is *the pattern of lines leaving a single
point*. There are eight such patterns, and so we are looking at an eight-state
automaton. It's convenient to represent each state with three binary bits, where
the low bit answers the question "does this point have a line leaving on the right
diagonal?" and similarly for the middle and high bits.

.. image:: laceautomata_numbering.png


Neighborhoods
.............

In a normal cellular automaton, each cell's state in generation `n+1` is based
on a *neighborhood* of cells in generation `n`. In a lace automaton, a cell's
neighborhood is the set of other cells whose outgoing lines can reach it. Since
lines are always vertical or (on a 45-degree) diagonal, that means we have a 
three-cell neighborhood. 

For instance, the cell marked with a black dot has three neighbors: the one marked 
with a red dot (which can reach it with a rightward diagonal line), the one marked
with a green dot (whch can reach it with a vertical line), and the one marked with
a blue dot (which can reach it with a leftward diagonal line).

.. image:: laceautomata_neighborhood.png

Limited information
...................

But we haven't said anything yet about the directionality of the lines. 

Consider again the picture repeated below. The black dot has the red dot in its
neighborhood. But unlike in a normal cellular automaton, it doesn't know everything
about its neighbor's state. 

.. image:: laceautomata_neighborhood.png

It "can tell" that the red dot has a line coming out on the rightward diagnal. But it
has no idea if the red dot is also emitting vertical or leftward diagonal lines. In other
words, it just knows about the low bit of the red dot's state.

Similarly, the black dot only knows about the middle bit of the green dot's state,
and about the high bit of the blue dot's state.

Implementation 
--------------

Now that we've converted lace automata into something more familiar, it's
easy enough to write code to run them.

Let's say a state is a number from `0b000` through `0b111`
--- or in other words, zero through eight, but as we saw above it's more convenient
to think of them in terms of bits. The leftmost bit represents a line leaving
on the left diagonal, and similarly for the middle and rightmost bit.

We can also represent the *inputs* to our automaton's rule as numbers from
`0b000` through `0b111`. Here, the leftmost bit represents a line *entering* on
the left diagonal, and similarly for the middle and rightmost bits. 

With this implementation, a rule is just a map from three-bit numbers representing
inputs to three-bit numbers representing result states. For instance, the rule
we represented graphically as follows

.. image:: laceautomata_rule_1.png

is, in our Python representation::

   RULE = { 0b000 : 0b000,
            0b001 : 0b110,
            0b010 : 0b111,
            0b011 : 0b001,
            0b100 : 0b011,
            0b101 : 0b010,
            0b110 : 0b100,
            0b111 : 0b000 }

If states are three-bit numbers, then a row of automaton output is a list
of such numbers, and the full output will be a list of rows. Given a specified
width, we'll populate the first row by hand, and then enter a loop where we
calculate new rows until we reach a specified height. ::

    WIDTH = 200
    HEIGHT = 100
    rows = [[0b000]*(WIDTH//2) + [0b010] + [0b000]*(WIDTH//2)]

    for i in range(HEIGHT):
        rows.append(apply_rule(get_neighborhoods(rows[-1])))

To flesh out this skeleton of a program we need two more things: an `apply_rule`
function and a `get_neighborhoods` function.

Actually applying the rule is easy. If we're given a list of neighborhoods, 
we just look each one up in the mapping we've already defined ::

   def apply_rule(neighborhoods):
       return [RULE[n] for n in neighborhoods]

The tricky part turns out to be getting the neighborhoods. Let's start with a
function that looks up just one cell's neighborhood. We give it a list of cell
states and a number `i`, and it looks up the neighborhood for the `i`\ th cell.

.. code-block:: python

   def get_neighborhood(row, i):
       # First, get the complete state of each neighbor.
       left = row[(i-1) % WIDTH]
       mid = row[i]
       right = row[(i+1) % WIDTH]
       # Then, keep only the information we'll use:
       out = ((left & 0b001) << 2) | (mid & 0b010) | ((right & 0b100) >> 2)

That last line is worth a closer look. :code:`(left & 0b001)` gets only the low
bit of :code:`left`; and :code:`(left & 0b001) << 2` says "Take the low bit of
:code:`left` and make it my high bit." Given the conventions we set up for 
representing states and neighborhoods as numbers, this is the same as saying
"If a line leaves my leftward neighbor heading right, it comes to me from
the left." Similarly, the other two parts of that line of code mean "If
a line leaves my middle neighbor heading straight down, it comes to me in the
middle," and "If a line leaves my rightward neighbor heading left, it comes
to me from the right." 

Once we can get a neighborhood for one cell, we can do it for a whole row of
cells. ::

    def get_neighborhoods(prev):
        return [get_neighborhood(prev, i) for i in range(len(prev))]

Now we're done calculating: we've supplied the `apply_rule` and `get_neighborhoods` functions needed to create new generations of output. But our output looks like this::

    [0, 0, 0, 0, 0, 2, 0, 0, 0, 0, 0]
    [0, 0, 0, 0, 0, 5, 0, 0, 0, 0, 0]
    [0, 0, 0, 0, 7, 0, 3, 0, 0, 0, 0]
    [0, 0, 0, 7, 5, 3, 5, 3, 0, 0, 0]
    [0, 0, 7, 7, 3, 2, 3, 2, 3, 0, 0]
    [0, 7, 7, 2, 2, 2, 5, 2, 5, 3, 0]

With a little more code, we can output images instead of lists. ::

    import cairo
    SCALE = 5
    surface = cairo.ImageSurface(cairo.FORMAT_ARGB32, WIDTH*SCALE, HEIGHT*SCALE)
    ctx = cairo.Context(surface)
    ctx.scale(SCALE, SCALE)
    ctx.set_line_width(0.1)

    for row in rows:
        print(row)

    for y, row in enumerate(rows):
        for x, cell in enumerate(row):
            if cell & 0b100:
                ctx.move_to(x,y)
                ctx.line_to(x-1,y+1)
                ctx.stroke()
            if cell & 0b010:
                ctx.move_to(x,y)
                ctx.line_to(x,y+1)
                ctx.stroke()
            if cell & 0b001:
                ctx.move_to(x,y)
                ctx.line_to(x+1,y+1)
                ctx.stroke()

    surface.write_to_png("test.png")

And here is the result!

.. image:: laceautomata_test.png


Gallery
-------

Here are the outputs for a bunch more rules. Click to expand --- they look much better at full size.

.. thumbnail:: laceautomata_irregular_erosion.png
   :group: 1
   
.. thumbnail:: laceautomata_half_c.png
   :group: 1

.. thumbnail:: laceautomata_serpinski.png
   :group: 1

.. thumbnail:: laceautomata_sample1.png
   :group: 1


.. thumbnail:: laceautomata_sample2.png
   :group: 1

.. thumbnail:: laceautomata_sample3.png
   :group: 1

.. thumbnail:: laceautomata_sample4.png
   :group: 1

.. thumbnail:: laceautomata_sample5.png
   :group: 1

.. thumbnail:: laceautomata_sample6.png
   :group: 1

.. thumbnail:: laceautomata_sample7.png
   :group: 1

.. thumbnail:: laceautomata_sample8.png
   :group: 1

.. thumbnail:: laceautomata_sample9.png
   :group: 1
