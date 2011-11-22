Why not ``ctypes``?
===================

``ctypes`` has been a part of the Python standard library for quite a while and
thus represents a standard for Python community.  As Thomas Jefferson wrote,
"Prudence, indeed, will dictate that Governments [libraries] long established
should not be changed for light and transient causes;" he then goes on to talk
about despotism.  ``ctypes`` isn't quite that bad, but it has a number of
critical issues, that I feel cannot be fully resolved within the constraints of
backwards compatibility.

Memory safety
-------------

You can crash Python with ``ctypes``.  Specifically, you can crash it in ways that aren't possible with C.  There's no question that it will be possible to crash Python by calling into some C library that crashes on a type-valid input (e.g. a function which takes an ``int`` parameter but has the semantic that it segfaults when the parameter is odd), if your C compiler wouldn't issue a warning, there's not much we can do.  On the other hand, ``ctypes`` lets you do the following, without so much as a warning::

    >>> m = ctypes.CDLL(ctypes.util.find_library("m"))
    >>> m.sqrt.argtypes = [ctypes.c_long]
    >>> m.sqrt.restype = ctypes.c_long
    >>> m.sqrt(34)

This is totally bogus, ``sqrt`` takes a ``double`` parameter, and your C
compiler won't let you pass it a ``long`` with invoking automatic coercion.
``ctypes`` on the other hand, happily uses the calling convention for
``longs``, and so calling ``m.sqrt(34)`` will either return 0 (if the floating
point register at the argument should be passed in is empty), or some totally
bogus value.  When we bring in pointers to this, it becomes quite easy to crash
your programs.

Verbosity
---------

``ctypes`` makes you write out the arguments each of your functions takes, and
the fields each of your structs has.  This is a pain for a few reasons:

1. It's easy to get them wrong, as a result of the aforementioned memory safety
   issues, getting them wrong means your program can pass around bogus data, or
   segfault.
2. It's code duplication, if you're using ``ctypes`` to bind to your own C
   library, it means that every time you change the C library, you need to
   update your ``ctypes`` definitions.  DRY (don't repeat yourself) isn't just
   because we're lazy, it's because it avoids the inevitable train wreck that
   happens when you change only part of your codebase.
3. It just sucks needing to write them out.  In C you just ``#include`` the
   header file and go from there.  This makes writing code to interface with C
   libraries more of a pain in Python, and incentivizes people to write
   C-extensions (which are evil

Poor APIs
---------

Some of the APIs in ``ctypes`` have large issues, a few examples:

* ``obj._as_parameter_``, if an object is passed as an argument to a C
  function, and it has an ``_as_parameter_`` attribute, that's used to
  represent the object in the call, this allows objects to tell the ``ctypes``
  machinery how they coerce to a C type.  Unfortunately, this has a few issues,
  first it uses a poor name, the name should scream "this is used for ctypes",
  second, rather than be an attribute, this should be a function which takes
  the target type, it's eminently possible that an object could be coerced into
  multiple different types.
* ``type * int`` is used to create array types of a specific length.  This API
  isn't limiting in the way the previous one is, it's just silly and
  unintuitive.
* Bad error checking: because ``argtype``, ``restype``, and various other
  properties are set by merely assigning to an attribute, a typo in any of
  these doesn't give a noticeable error, and can lead to crashes (again,
  feeding back into the lack of memory safety).  When interfacing with external
  resources, making sure there is as little chance for an uncaught error as
  possible, these result in segfaults which are not easy debuggable from Python.
*