Tutorial
========

.. currentmodule:: yaffi

This is an introduction to using ``yaffi``.  It will describe using ``yaffi``
to build your own replacement for the builtin :py:mod:`math` module, which
largely wraps the functions found in the C ``math.h`` header.

Before we get started using ``yaffi``, we need to install it::

    $ pip install yaffi

It's highly reccomended to do this inside of a `virtualenv`_.

Now it's time to get coding.  We need to create a :py:class:`Library`.  On
Unixes, the library containing mathematical functions is called ``m``::

    import yaffi

    c_math = yaffi.Library("m")

This will try to find a library on our system named ``m``.

.. note::

    If this gives you an exception, it's likely that you're on a weird
    platform, please file a bug.

The simplest function in the :py:mod:`math` module is probably
:py:func:`~math.isinf`, so let's start there::

    def isinf(value):
        res = c_math.getfunc("isinf")(value)
        return bool(res)

This is pretty simple, first we get the :c:func:`isinf` function from the
``c_math`` library, and then we call it with our value.  :c:func:`isinf` is
defined to return an ``int``, but we want our ``isinf`` function to return a
``bool``, so we coerce that before returning the value.

Let's try it out::

    >>> import my_math
    >>> my_math.isinf(3.0)
    False
    >>> my_math.isinf(float("inf"))
    True
    >>> my_math.isinf(2)
    False

As you can see our ``isinf`` function appears to work, and it properly handles turning ``ints`` into :c:type:`doubles <double>`.  Let's see what happens if we give it something that can't be turned into a ``float``::

    >>> my_math.isinf([])
    Traceback (most recent call last):
    ...
    CoercionError: Can't coerce list to double.

It raised a :py:exc:`CoercionError`, a subclass of ``TypeError``.

Let's try doing something slightly more complicate now, let's implement the ``asin``.  It's got an interesting catch though, as you may (or, far more likely, may not) remember from your high school trig class, the arcsin of a value is only defined on the interval ``[-1, 1]``, so we want to report an error, just like the builtin :py:func:`math.asin` implementation does::

    import errno

    @c_math.register_error_handler("asin")
    def math_errno_handler(func, result, args):
        if yaffi.posix.get_errno() == errno.EDOM:
            raise ValueError("math domain error")

    def asin(value):
        return c_math.getfunc("asin")(value)

There's a few new things here, so let's go through them:

1. We define an error handler, which is a function that takes the
   ``(func, result, args)`` as its parameters.  ``func`` is the function object
   that was called, ``result`` is what will be returned if no error is raised,
   and ``args`` is a tuple of the arguments that the function was called with.
   Error handlers should raise an exception if one occured, they have no return
   value.
2. We register our error handler, using
   :py:meth:`Library.register_error_handler`, providing the name of the C
   function for which we're registering the handler.
3. We use :py:func:`yaffi.posix.get_errno` to get the ``errno``, and compare it
   with a constant from the :py:mod:`errno` module.
4. Finally we define our ``asin`` function normally, the error will be raised
   automatically if it occurs.

Using an error handler, rather than manually doing the error checking after
each call allows us to ensure that no matter where we call this function an
error will always be correctly propagated.

.. _virtualenv: http://www.virtualenv.org/