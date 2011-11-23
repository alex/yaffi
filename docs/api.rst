API Reference
=============

.. py:module:: yaffi

.. py:class:: Library(path)

    These represent shared objects, and all the data contained within them.
    If the ``path`` provided does not exist a :py:exc:`LibraryDoesNotExist` is
    raised, if the ``path`` exists but is not a valid shared object a
    :py:exc:`InvalidLibrary` exception is raised.

    .. py:method:: getfunc(self, name)

        Returns the :py:class:`Function` instance corrosponding to this
        function within the library.  Raises a :py:exc:`FunctionDoesNotExist`
        if the function doesn't exist within the library.

    .. py:method:: getstruct(self, name)

        Returns the :py:class:`Struct` subclass corresponding to this struct
        within the library.  Raises a :py:exc:`StructDoesNotExist` if the
        struct doesn't exist within the library.

    .. py:method:: register_error_handler(self, error_handler, *funcs)

        Registers an error handler for each named function.  ``error_handler``
        is a callable which takes ``(function, result, args)`` as arguments.
        It should raise an exception if one has occurred, it's return value is
        ignored.  If any of the function already has an error handler
        registered, an ``ErrorHandlerAlreadyRegistered`` exception is raised,
        and none of the functions will be modified.


.. py:class:: Function

    Represents a callable function from a :py:class:`Library`.

    .. py:method:: __call__(self, *args)

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
        :py:exc:`CantCoerce` if the ``param`` can't be coerced into the desired
        type.

.. py:class:: PrimitiveType

    .. py:function:: __init__(self [, value])

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

.. py:exception:: CantCoerce