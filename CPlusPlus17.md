# C++ 17
  C++17 (or informally, C++1z) is the name for the most recent revision of the ISO/IEC standard for the C++ programming language. The C++17 specification reached the Draft International Standard (DIS) in March 2017. This DIS was unanimously approved, with only editorial comments and the final standard was published in December 2017.
  
## New features  
### Language
  * Making the text message for static_assert optional
  * Removal of trigraphs
  * Allow typename (as a alternative to class) in a template template parameter
  * New rules for auto deduction from braced-init-list
  * Nested namespace definitions, e.g., namespace X::Y { ... } instead of namespace X {namespace Y { ... }}
  * Allowing attributes for namespaces and enumerators
  * New standard attributes [[fallthrough]], [[maybe_unused]] and [[nodiscard]]
  * UTF-8 character literals
  * Hexadecimal floating-point literals
  * Constant evaluation for all non-type template arguments
  * Fold expression, for variadic templates
  * A compile-time static if with the form if constexpr(expression)
  * Structured binding declarations, allowing auto [a, b] = getTwoResultValues();
  * Initializers in if and switch statements
  * copy-intialization and direct-initialization of object of type T from prvalue expression of type T (ignoring top-level cvqualifiers) shall result in no copy or move constructors from the prvalue expresion
  * Some extensions on over-aligned memory allocation
  * Template deduction of constructors, allowing std::pair(5.0, flase) instead of std::pair<double, bool>(5.0, false)
  * Inline variables, which allows the definition of variables in header files
  * \_\_has\_include, allowing the available of a header to be checked by preprocessor directives
  * Value of \_\_cplusplus changed to 201703L
  
### Library
  * Most of Library Fundamentals TS I, including:
    * std::string\_view, a read-only non-owning reference to a character sequence or string-slice
    * std::optional, for representing optional objects
    * std::any, for holding single values of any type
  * std::uncaught\_exceptions, as a replacement of std::uncaught\_exception
  * New insertion functions try\_emplace and insert\_or\_assign for std::map and std::unordered\_map
  * Uniform container access: std::size, std::empty, std::data
  * Definition of "contiguous iterators"
  * Removal of some deprecated type and functions, including std::auto\_ptr, std::random\_shuffle, and old function adaptors
  * A file system library based on boost::filesystem
  * Parallel version of STL algorithms
  * Additional mathematical sepcial functions, including elliptic integrals and Bessel functions
  * std::variant, a tagged union container
  * std::byte
  * Logical operator traits: std::conjunction, std::disjunction, std::negation
  
## Next Standard  

## See also

## References

## My References:
  * [C++17 wiki](https://en.wikipedia.org/wiki/C++17)
