.. include:: <isonum.txt>

Architecture
============

When you use ``gdb`` (GNU Project debugger), how does it know what types your
variables are, what fields your structs are, or where anything lives in memory?
When you compile something using the ``-g`` flag with ``gcc`` it inserts a
bunch of extra debugging information into the binary, in a format called DWARF
(a play on the name of the standard file format for binaries on unixes, ELF).
Most people don't compile their production binaries with this debug flag,
because it introduces a lot of bloat into the binary, after all you don't need
do translate your location in the assembler into the corresponding line of C
code in production.  However, what if you could compile a binary using a subset
of this debug data, specifically: the fields a struct has and the argument
types, return types, and calling convention of functions?  Dumping this data
would be small, relative to a full ``-g`` build, and a Sufficiently Smart FFI\
|trade| could use it to introspect all this information.  This would alleviate
the user of the pain of typing it, and give the FFI strong type safety.

The remainder of this architecture is predicated on this idea becoming a
reality, and it becoming sufficiently used, across, all platforms that the
need for a manual fallback is not necessary.

