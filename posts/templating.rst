.. post:: Aug 24 2017
   :tags: cloudburst, python
   :exclude: 

   In the Python world, the Right Way to render a template is to pass it a
   mapping.

   Python has exactly one built-in mapping type: good old :literal:`dict`. 

   This means there's no consensus on the Right Way for templates to handle
   custom mapping types. Some downcast them to :literal:`dict`, some take
   them as they are. This post looks at which template libraries do which.

Python templates and custom mapping types
=========================================

In the Python world, the Right Way to render a template is to pass it a
mapping --- or some keyword arguments that it can use to build a mapping.

Python has exactly one built-in mapping type: good old :literal:`dict`. 

But what about mapping types other than :literal:`dict`? Duck typing says
templates should accept any type of object, no matter how odd, so long as it
supports :func:`__getitem__` and maybe a few other basic methods. But
this isn't what all of them do.

This becomes especially clear when you pass in a mapping object that
*can't* be converted to a :literal:`dict` --- say, because
the mapping it implements has a large or even infinite domain. In
the examples below, :literal:`infiniteMap` maps strings containing any number
of `a`\ s to strings with the same number of `b`\ s.

.. doctest::

   >>> import jinja2
   >>> from fsmcontainers import FsmMap
   >>> infiniteMap = FsmMap({"a": "b"}).closure()
   >>> infiniteMap["a"]
   'b'
   
.. doctest::

   >>> infiniteMap["aaa"]
   'bbb'

.. doctest::

   >>> infiniteMap["a" * 1000]
   'bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb'

Getting values from the mapping works fine. But passing it as an argument to
Jinja's rendering function doesn't work.

.. doctest::

   >>> jinja2.Template("{{ a }}").render(infiniteMap)
   Traceback (most recent call last):
   ...
   pywrapfst.FstArgError: FST is not acyclic 

The first thing :func:`jinja2.Template.render` does with its arguments is
call :literal:`dict` on them. If it's passed an actual :literal:`dict`, or something
that the :literal:`dict` constructor can be called on, then we have no problem. But
if we pass it a mapping with an infinite number of keys --- one that is meant
to be evaluated lazily on one finitely-long key at a time --- the eager
:literal:`dict` constructor tries to iterate over every key and value in it,
and that doesn't end well. If :class:`FsmMap` hadn't thrown an exception, the
result would have been an infinite loop.

Python 3's built-in :func:`string.format_map`, on the other hand, handles
infinite mappings with ease:

.. doctest::
   
   >>> "{a}".format_map(infiniteMap)
   'b'

.. doctest::
   
   >>> "{a} {aaaa} {aaaaaaa}".format_map(infiniteMap)
   'b bbbb bbbbbbb'

.. doctest::
   
   >>> ("{" + ("a" * 1000) + "}").format_map(infiniteMap) 
   'bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb'

Why would you do that?
----------------------

Let's be clear. Doing this with an infinite mapping is a terrible idea.

Templates aren't supposed to do much logic (or indeed, if you're a purist, any
logic at all). They're *really* not supposed to handle weird Lovecraftian
macro-language-in-disguise "mapping" objects that can expand unlimitedly large
keys in arbitrary ways, even if those objects perversely insist on offering a
dict-like interface.  They're just meant to accept a small --- and *definitely*
finite --- collection of key–value pairs, and a normal finite :literal:`dict`
is a convenient way to pass one of those around.

But there are other, less-Lovecraftian cases where using a template might make
sense, but where downcasting its context to :literal:`dict` is still
undesirable.

In my last post I fantasized about a Fancy Linguistic Resource that would let
you write something like this :: 

   jinja2.Template(
     "The word {{ llegues }} is pronounced {{ llegues.pron }}."
   ).render(resource)

and get back a string like this:: 

   "The word <i>llegues</i> is pronounced /'je.ges/."

In this application, we don't need :literal:`resource` to be an *infinite*
mapping. But it's likely to be a pretty big one, stored as a trie or a finite
state transducer or some other time- and space-efficient structure. Eagerly
iterating over the whole thing to convert it into a :literal:`dict`, while
possible, is still not a great use of resources. 

A purist might say we should separate out logic and presentation and do
something like this::

   word = str(resource['llegues'])
   pronunciation = str(resource['llegues'].pron)
   jinja2.Template(
     "The word <i>{{ word }}</i> is pronounced {{ pronunciation }}"
   ).render(word=word, pronunciation=pronunciation)

But if all the words I want to look up can be found in a single resource — and
if that resource is already efficiently stored, and we're not working in a web
app environment where we need everything to be massively parallelizable — then
this separation of concerns is overkill.

A purist might also say that what I'm doing here really doesn't call for a
*templating* system at all --- that really this is more like text macro
expansion or string interpolation, and I should just use a system for those.
But templating systems have a better ecosystem: lots of available filters
for useful tasks, good integration with Python text-processing tools like Sphinx,
and so on. It sure would be nice if I didn't have to reinvent those wheels.

I can imagine other situations where we might not want our template engine to
convert a custom mapping object to a :literal:`dict`:

    * The mapping's :func:`__getitem__` method has side effects. Maybe it logs
      the fact that a lookup happened, for instance.
    * The mapping has a special behavior for missing keys, and that behavior
      will be lost if we turn it into a vanilla :literal:`dict`.

What works
----------

In these examples, :literal:`finiteMap` is an ordinary dictionary mapping
`a` to `b`, and :literal:`infiniteMap` is a custom mapping object mapping
any number of `a`\ s to any number of `b`\ s. The expected result is for
templates containing the placeholder :literal:`a` to render it to
:literal:`'b'` whether they are given :literal:`finiteMap` or
:literal:`infiniteMap`. These functions do the right thing:

.. testsetup::

   from fsmcontainers import FsmMap
   finiteMap = {"a": "b"}
   infiniteMap = FsmMap({"a": "b"}).closure()

:func:`string.format_map`:

.. doctest::

  >>> '{a}'.format_map(finiteMap)
  'b'

.. doctest::

  >>> '{a}'.format_map(infiniteMap)
  'b'

:func:`string.Template.substitute`:

.. doctest::

  >>> import string
  >>> string.Template("$a").substitute(finiteMap)
  'b'

.. doctest::

  >>> string.Template("$a").substitute(infiniteMap)
  'b'

:func:`pymustache.render`:

.. doctest::

   >>> import pymustache
   >>> pymustache.render("{{ a }}", finiteMap)
   'b'

.. doctest::

   >>> pymustache.render("{{ a }}", infiniteMap)
   'b'

:func:`airspeed.Template.merge`:

.. doctest::

  >>> import airspeed
  >>> airspeed.Template("$a").merge(finiteMap)
  'b'

.. doctest::

  >>> airspeed.Template("$a").merge(infiniteMap)
  'b'

:func:`Cheetah.Template`

.. doctest::

   >>> from Cheetah.Template import Template
   >>> str(Template("$a", finiteMap))
   'b'

.. doctest::

   >>> str(Template("$a", infiniteMap))
   'b'


What doesn't
------------

Internal iteration over mapping argument
........................................

:func:`jinja2.Template.render`:

.. doctest::

   >>> import jinja2
   >>> jinja2.Template("{{ a }}").render(finiteMap)
   'b'

.. doctest::
   
   >>> jinja2.Template("{{ a }}").render(infiniteMap)
   Traceback (most recent call last):
   ...
   pywrapfst.FstArgError: FST is not acyclic 

Silent failure with empty result
................................

:func:`ashes.AshesEnv.render`:

.. doctest::

   >>> from ashes import AshesEnv
   >>> env = AshesEnv()
   >>> env.register_source("test", "{a}")
   <Template name='test'>
   >>> env.render("test", finiteMap)
   'b'

.. doctest::
   
   >>> env.render("test", infiniteMap)
   ''

:func:`pystache.render`:

.. doctest::

   >>> import pystache
   >>> pystache.render("{{ a }}", finiteMap)
   'b'

.. doctest::
   
   >>> pystache.render("{{ a }}", infiniteMap)
   ''

Forced unpacking
................

:func:`string.format`:

.. doctest::

   >>> '{a}'.format(** finiteMap)
   'b'

.. doctest::
   
   >>> '{a}'.format(** infiniteMap)
   Traceback (most recent call last):
   ...
   pywrapfst.FstArgError: FST is not acyclic 

.. doctest::
   
   >>> '{a}'.format(finiteMap)
   Traceback (most recent call last):
   ...
   KeyError: 'a'

.. doctest::
   
   >>> '{a}'.format(infiniteMap)
   Traceback (most recent call last):
   ...
   KeyError: 'a'

:func:`moody.render`:

.. doctest::

   >>> import moody
   >>> moody.render("{{ a }}", ** finiteMap)
   'b'

.. doctest::

   >>> moody.render("{{ a }}", ** infiniteMap)
   Traceback (most recent call last):
   ...
   pywrapfst.FstArgError: FST is not acyclic 

.. doctest::

   >>> moody.render("{{ a }}", finiteMap)
   Traceback (most recent call last):
   ...
   TypeError: render() takes 1 positional argument but 2 were given

.. doctest::

   >>> moody.render("{{ a }}", infiniteMap)
   Traceback (most recent call last):
   ...
   TypeError: render() takes 1 positional argument but 2 were given


:func:`mako.template.Template.render` 

.. doctest::

  >>> from mako.template import Template
  >>> Template("${a}").render(** finiteMap)
  'b'

.. doctest::

  >>> Template("${a}").render(** infiniteMap)
  Traceback (most recent call last):
  ...
  pywrapfst.FstArgError: FST is not acyclic 

.. doctest::

  >>> Template("${a}").render(finiteMap)
  Traceback (most recent call last):
  ...
  TypeError: render_body() takes 1 positional argument but 2 were given

.. doctest::

  >>> Template("${a}").render(infiniteMap)
  Traceback (most recent call last):
  ...
  TypeError: render_body() takes 1 positional argument but 2 were given


:func:`mako.template.Template.render_context`:

.. doctest::

  >>> import sys
  >>> from mako.runtime import Context
  >>> Template("${a}").render_context(Context(sys.stdout, ** finiteMap))
  b

.. doctest::

  >>> Template("${a}").render_context(Context(sys.stdout, ** infiniteMap))
  Traceback (most recent call last):
  ...
  pywrapfst.FstArgError: FST is not acyclic 

.. doctest::

  >>> Template("${a}").render_context(Context(sys.stdout, finiteMap))
  Traceback (most recent call last):
  ...
  TypeError: __init__() takes 2 positional arguments but 3 were given

.. doctest::

  >>> Template("${a}").render_context(Context(sys.stdout, infiniteMap))
  Traceback (most recent call last):
  ...
  TypeError: __init__() takes 2 positional arguments but 3 were given

