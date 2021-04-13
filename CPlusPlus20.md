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
  * [C++协程](https://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666553202&idx=2&sn=bbde0de4ae0c12c06021d7adbd5e9e6f&chksm=80dc9bd9b7ab12cf767906343cd0a921e6d5066b068d4e1e4b75a5be50f0427a6517e71253b6&xtrack=1&scene=0&subscene=91&sessionid=1618288935&clicktime=1618289021&enterid=1618289021&ascene=7&devicetype=android-29&version=2800023d&nettype=WIFI&abtest_cookie=AAACAA%3D%3D&lang=en&exportkey=A1O4NST5MhT2FKZM9ivF7%2BM%3D&pass_ticket=ntCHoJHsVL9uAifMjtFX%2BqOaQe3n1MtJYgiiBqn2mQGzMNdmyA2Y0yCAHwHFovfK&wx_header=1)
  * [C++ Coroutine in FB](https://github.com/facebook/folly/tree/master/folly/experimental/coro)
  * [C++ 20新特性](https://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666553211&idx=2&sn=cf1564f1c58f63879478b7797e964379&chksm=80dc9bd0b7ab12c61374297927789cea953f7538ce58585a9c287eadff6b58c12ab381952ed0&&xtrack=1&scene=0&subscene=91&sessionid=1618288935&clicktime=1618288941&enterid=1618288941#rd)
