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

You can crash Python with ``ctypes``.  Specifically, you can crash it in ways
that aren't possible with C.  There's no question that it will be possible to
crash Python by calling into some C library that crashes on a type-valid input
(e.g. a function which takes an ``int`` parameter but has the semantic that it
segfaults when the parameter is odd), if your C compiler wouldn't issue a
warning, there's not much we can do.  On the other hand, ``ctypes`` lets you do
the following, without so much as a warning::

    >>> m = ctypes.CDLL(ctypes.util.find_library("m"))
    >>> m.sqrt.argtypes = [ctypes.c_long]
    >>> m.sqrt.restype = ctypes.c_long
    >>> m.sqrt(34)

This is totally bogus, ``sqrt`` takes a ``double`` parameter, and your C
compiler won't let you pass it a ``long`` without invoking automatic coercion.
``ctypes`` on the other hand, happily uses the calling convention for
``longs``, and so calling ``m.sqrt(34)`` will either return 0 (if the floating
point register or stack location that the argument should be passed in is
empty), or some totally bogus value.  When we bring pointers in to this, it
becomes quite easy to crash your programs.

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
   C-extensions (which are evil).

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
  possible is paramount, these result in segfaults which are not easy
  debuggable from Python.
* The way to signal that a function didn't return an error when using the
  ``errcheck`` feature is to return the arguments tuple from the callback.
  That's just obscure, a) it's totally unguessable, b) even having learned
  about this feature I'd never possibly have guessed it.

Speed
-----

This one can be overcome with some good engineering, but most of it really
shouldn't be necessary.  An FFI needs to be fast, because its competitor is a
C-extension, if it imposes too much overhead, no one will use it.  ``ctypes``
does the following things which lead to slowness

* It uses a different ``type`` for every length of array.  Generally, one of
  the first heuristics an optimizing compiler for Python will use is try to
  generate separate code paths for each type, this defeats this heuristic
  (which is pretty effective on normal Python code).
* The ``argtypes`` and ``restype`` of a function can be changed at any time.
  That's frankly just a bit silly, and means if a ``ctypes`` function call is
  JIT compiled it needs to check that these remain changed.  These should be
  immutable attributes, not things that can change.

I want ``ctypes`` to be fast enough that it could be the core of a ``NumPy``
implementation, both for manipulation of raw memory, and for calling out to
libraries like BLAS.

Conclusion
----------

My goal is that a Python FFI can be fast, robust, and convenient enough to use
that it will be *the* de facto choice of the Python community for interfacing
with C libraries and writing code to manipulate raw memory, I want it to be
considered as safe as writing C code with every warning flag, so safe that
parts of the Python standard library can be written with it.