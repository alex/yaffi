.. include:: <isonum.txt>

Architecture
============

.. seealso::

    This describes the architecture of ``yaffi``, for the API reference for all
    of these components, see the :doc:`api`.

When you use ``gdb`` (GNU Project debugger), how does it know what types your
variables are, what fields your structs are, or where anything lives in memory?
When you compile something using the ``-g`` flag with ``gcc`` it inserts a
bunch of extra debugging information into the binary, in a format called DWARF
(a play on the name of the standard file format for binaries on unixes, ELF).
Most people don't compile their production binaries with this debug flag,
because it introduces a lot of bloat into the binary, after all you don't need
to translate your location in the assembler into the corresponding line of C
code in production.  However, what if you could compile a binary using a subset
of this debug data, specifically: the fields a struct has and the argument
types, return types, and calling convention of functions?  Dumping this data
would be small, relative to a full ``-g`` build, and a Sufficiently Smart FFI\
|trade| could use this data in order to create bindings with the correct types
automatically.  This would alleviate the user of the pain of typing it, and
give the FFI as strong type safety as the original compiler.

The remainder of this architecture is predicated on this idea becoming a
reality, and it becoming sufficiently used, across, all platforms that the
need for a manual fallback is not necessary.

.. Note:: The Present

    At the moment this doesn't exist, however the information is there in full
    debug builds, so a prototype can be constructed using that.

``yaffi`` has three fundamental components:

* Libraries
* Functions
* Types

Libraries
---------

These correspond to a shared library on a system.  They are the guardians of
all the DWARF data that ``yaffi`` knows about, and provide access to the
functions, structs, and unions defined in it.

Functions
---------

These are callable objects which perform type coercion on their arguments and
return types, based on the types their library knows about.  They also expose
this information at the Python level.

Types
-----

These are responsible for performing the type coercion.  Instances of them
represent the underlying C values.  Structs are simply composite types of
these.  There are also array types.