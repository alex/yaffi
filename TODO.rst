To Do
=====

Resolve issues with the type system, namely:

* Are the arrays in `void x(char z[])` and `struct { char x[10];}` the same
  types?
* What type is a `const char *` is it the same as a `char *`?
* Can we fulfill the invariant that `type(x(*args)) is x.result_type` and still
  have a sane type heirarchy?
