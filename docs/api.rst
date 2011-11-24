API Reference
=============

This documents the public APIs of all these classes, exceptions, and functions.
It may be that they have other attributes or methods, however nothing about
their behavior or stability is guaranteed.

.. py:module:: yaffi

``yaffi``
---------

.. py:class:: Library(name_or_path)

    These represent shared objects, and all the data contained within them. If
    the ``path`` provided refers to a file it will attempt to load the library
    from that file, else it will treat it as the name of a library and attempt
    to find a shared object with that name on the system.  If no library
    matching the name can be found, and it is not a file on the system
    :py:exc:`LibraryDoesNotExist` is raised, if the ``path`` exists but is not
    a valid shared object a :py:exc:`InvalidLibrary` exception is raised.

    .. py:method:: getfunc(name)

        Returns the :py:class:`Function` instance corrosponding to this
        function within the library.  Raises a :py:exc:`FunctionDoesNotExist`
        if the function doesn't exist within the library.

    .. py:method:: getstruct(name)

        Returns the :py:class:`Struct` subclass corresponding to this struct
        within the library.  Raises a :py:exc:`StructDoesNotExist` if the
        struct doesn't exist within the library.

    .. py:decoratormethod:: register_error_handler(*funcs)

        Registers the decoratored function as an error handler for each named
        function.  The error handler is a callable which takes
        ``(function, result, args)`` as arguments.  It should raise an
        exception if one has occurred, it's return value is ignored.  If any of
        the function already has an error handler registered, an
        :py:exc:`ErrorHandlerAlreadyRegistered` exception is raised, if any of
        the functions do not exist within the module
        :py:exc:`FunctionDoesNotExist` exception is raised, in either case none
        of the functions will be modified.  An error handler that checks that
        the return value of a function is ``0`` looks like::

            @my_library.register_error_handler("func1", "func2", "func3")
            def my_error_handler(func, result, args):
                if result != 0:
                    raise SomeException


.. py:class:: Function

    Represents a callable function from a :py:class:`Library`.

    .. py:method:: __call__(*args)

        Calls the underlying function from the shared library, and returns the
        result from it.  All arguments are coerced to the appropriate
        :py:class:`Type` based on the type information the :py:class:`Library`
        has, as is the return type.  It will also invoke the error handler
        registered with it's library if one exists.

    .. py:attribute:: argument_types

        A tuple of the argument :py:class:`types <Type>` this function takes.

    .. py:attribute:: return_type

        The :py:class:`Type` this function returns.

    .. py:attribute:: error_handler

        The error handler registered for this function.

.. py:class:: Type

    A base class for all types, including both primitives and composite types.

    .. py:classmethod:: from_param(cls, param)

        Coerces the ``param`` into an instance of ``cls``.  Subclasses should
        override this to provide their own custom coercion logic.  Should raise
        :py:exc:`CoercionError` if the ``param`` can't be coerced into the
        desired type.

.. py:class:: PrimitiveType

    .. py:function:: __init__([value])

        Instantiates the primitive with the provided value, doing only basic
        coercion (i.e. this does not invoke :py:func:`from_param`).  If
        ``value`` is not provided the type's zero value is used.

.. py:exception:: DoesNotExist

    A base class for all exceptions that indicate that something doesn't exist.

    .. py:attribute:: name

        The name of the thing which doesn't exist.

.. py:exception:: LibraryDoesNotExist

.. py:exception:: FunctionDoesNotExist

.. py:exception:: StructDoesNotExist

.. py:exception:: CoercionError

    This is a subclass of :py:exc:`TypeError`.

.. py:exception:: ErrorHandlerAlreadyRegistered


.. py:module:: yaffi.posix

``yaffi.posix``
---------------

This module contains helpers for working with the various POSIX functionality.

    .. py:function:: get_errno()

        Returns the POSIX ``errno`` set by the last ``yaffi`` call in this
        thread.

        .. note::

            Due to way ``errno`` works, this is not the same as
            ``return errno`` in C, the ``errno`` value is cached after each
            call and this the last cached value.