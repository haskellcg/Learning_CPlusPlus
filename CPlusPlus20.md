# C++ 20
  C++20 is the informal name for the revision of the ISO/IEC standard for the C++ programming language except to follow C++17. The C++ Standards Committee began panning C++20 in july of 2017, the curent draft is N4713.
  
## Possible new language features  
  * The following were voted into the C++20 draft in 2017:
    * Concepts
    * Designated initializers
    * [=, this] as a lambda capture
    * Template parameter lists on lambdas
  * Features voted into C++20 in the fall meeting in November 2017 include:
    * Three-way comparison using the "spaceship operator", operator<=>
    * Initialisation of an additional veriable within a range-based for statement
    * Bit-casting of object representations, with less verbosity than memcpy() and more ability to exploit compile internals
    * A specialisation of std::atomic for std::shared\_ptr
    * Lambdas in unevaluated context. This removes the long-standing restriction against having a lambda-expression inside a decltype in many contexts
    * Default constructible and assignable stateless lambdas
    * Allow pack expansions in lambda init-capture. There was no compelling to disallow this, and the workaround of constructing a tuple to store the arguments and then unpacking it is inefficient
    * String literals as template parameters. The syntax is simple: template<auto &string>. You can't write template<size_t N, const char (&string)[N]> and have both N and String deduced from a single string literal template argument. This is not big deal, though: using auto form, you can easily recover N via traits, and even constrain the length or the character type using a requires-clause
  * Other possible new language features
    * Coroutines
    * Modules -- experimenrally support in Clang 5 and Visual Studio 2015 Update 1
    * Transactional memory
    * Reflection
    * Metaclasses

## Possible library changes
  * Atomic smart pointers (std::atomic_shared_ptr and std::atomic_weak_ptr)
  * Extended futures
  * Latches and barries
  * Networking extensions, including async, basic I/O services, timers, buffers and buffer-oreinted streams, sockets, and Internet protocols
  * Ranges
  * Task blocks

## See also

## References

## External links

## My References
  * [C++ 20](https://en.wikipedia.org/wiki/C++20)
  * [C++ Standards Committee Papers](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/)