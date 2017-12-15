# C++ 14
  C++14 is a version of the standard for the programming language C++. It is intended to be small extension over C++11, featuring mainly bug fixes and small improvements. Its approval was announced on August 18, 2014. C++14 was released on December 15, 2014.
  
## New language features  
### Function return type dedution
  C++11 allowed lambda functions to deduce the return type based on the type of the expression given to the return statement. C++14 **_provides this ability to all functions_**. It also extends these facilities to lambda functions, allowing return type dedution for functions that are not of the form **_return expression_**.
  
  In order to **_induce return type dedution_**, the function msut be declared with auto as the return type, but without the trailing return type specifier in C++11:
  ```c++
  auto DeduceReturnType();
  ```
  
  **_If multiple return expression are used in te function's implementation, then they must all deduce the same type_**.
  
  Functions that deduce their return types can be forward declared, but they cannot be used until they have been defined. Their definitions must be available to the translation unit that uses them.
  
  Recursion can be used with a function of this type, but recursion call must happen afetr at least one return statement in the definition of the function:
  ```c++
  auto Correct(int i)
  {
      if (i == 1){
          return i;    // return type deduced as int
      } else {
          return Correct(i - 1) + i;    // ok to call it now
      }
  }
  
  auto Wrong(int i)
  {
      if (i != 1){
          return Wrong(i - 1) + i;    // Too soon to call this. No prior return statement
      } else {
          return i;    // return type deduced as int
      }
  }
  ```
  
### Alternate type dedution on declaration  
  In C++11, two methods of type deduction were added:
  * auto was a way to create a variable of the appropriate type, base on a given expresion
  * decltype was a way to compute the type of a given expression
  
  However, decltype and auto deduce types in different ways. In particular, **_auto always deduces a non-reference type_**, as though by using std::decay, while auto&& always deduces a reference type.
  
  However, decltype can be prodded into deducing a reference or non-reference type, based on the value category of the expression and the nature of the expression it is deducing:
  ```c++
  int i;
  int &&f();
  
  auto x3a = i;    // decltype(x3a) is int
  decltype(i) x3d = i;    // decltype(x3d) is int
  auto x4a = (i);    // decltype(x4a) is int
  decltype((i)) x4d = (i);    // decltype(x4d) is int &
  auto x5a = f();    // decltype(x5a) is int
  decltype(f()) x5d = f();    // decltype(x5d) is int &&
  ```
  
  **_C++14 adds the decltype(auto) syntax. This allows auto declarations to use the decltype rules on the given expression_**. The decltype(auto) syntax can also be used with return type dedution, by using decltype(auto) syntax instead of auto for the function's return type dedution.
  
### Relaxed constexpr restrictions
  C++11 introduced the concept of a constexpr-declared function, a function which could be executed at compile time. Their return value cound be consumed by operations that required constant expressions, such as an integer template argument. **_However, C++11 constexpr functions could only contains a single expression that is returned (as well as static_assert and a small number of other declarations)_**.
  
  C++14 relaxes these restrictions. Constexpt-declared functions may now contain the following:
  * Any declarations except:
    * static or thread_local variables
    * Variable declarations without initializers
  * The condition braching statements if and switch
  * Any looping statement, including range-based for
  * Expressions which change the value of an object if the lifetime of that object began within the constant expression function. This includes calls to any non-const constexpr-declared non-static member functions
  
  goto statements are forbidden in C++14 relaxed constexpr-declared functions.
  
  Also, C++11 stated that **_all non-static member functions that were declared constexpr were also implicitly declared const_**, with respet to this. That has since been removed; non-static member functions may be non-const. However, per the above restrictions, a non-const constexpr member function can only modify a class member if that object's lifttime began within the constant expression evaluation.
  
### Variable templates  
  In prior version of C++, only functions, classes or type alias could be templated. **_C++14 now allows the creation of variables that are templated_**. An example given in the proposal is a variable pi that can be read to get the value pi for various types (e.g., 3 when read as integral type; the closest value possible with float, double, long long precision when read as float, double, long double, respectively).
  
  The usual rules of templates apply to such declarations and definitions, including specialization:
  ```c++
  template<typename T>
  constexpr T pi = T(3.141592653589793238462643383);
  
  // Usual specialization rules apply
  template<>
  constexpr const char * pi<const char *> = "pi";
  ```
  
### Aggregate member initialization
  C++11 added member initializer, expressions to be applied to members at class scope if a constructor did not initializer the member itself. **_The definition of aggregates was changed to explicitly exclude any class with member initializer_**, therefore, they are not allowed to use aggregate initialization.
  
  C++14 relaxes this restriction, allowing aggregate initialization on such types. If the braced init list does not provide a value for that argument, the member initializer takes care of it.
  
### Binary literals  
  Numeric literals in C++14 can be specified in binary form. Thx syntax uses the prefixes **_0b or 0B_**. The syntax is also used in other language e.g. Java, C#, Swift, Go, Scala, Ruby, Python, OCaml, and as an unofficial extension in some C compilers since at least 2017.
  
### Digit separators
  In C++14, the single-quote character may be used arbitrarily as a digit separator in numeric literals, both integer literals and floating point literals. This can make it easier for human readers to parse large numbers through subitizing:
  ```c++
  auto integer_literal = 1'000'000
  auto floating_point_literal = 0.000'015'3
  auto binary_literal = 0b0100'1100'0110
  auto silly_example = 1'0'0'000'00
  ```
  
### Generic lambdas  
  In C++11 lambda function parameters need **_to be declared with concrete types_**. C++14 relaxes this requirement allowing lambda function parameters to **_be declared with the auto type specifier_**.
  ```c++
  auto lambda = [](auto x, auto y) {return x + y;};
  ```
  
  As for auto type deduction, generic lambdas follow the rules of template argument deduction (which are similar, but not identical in all respect). The above code is equivelant to this:
  ```c++
  struct unnamed_lambda
  {
      template<typename T, typename U>
      auto operator()(T x, U y) const
      {
          return x + y;
      }
  };
  
  auto lambda = unnamed_lambda();
  ```
  
  **_Generic lambdas are essentially templated functor lambdas_**.
  
### Lambda capture expressions  
  C++11 lambda functions capture variables declared in their outer scope by value-copy or by reference. This means that value members of a lambda **_cannot be move-only types_**. C++14 allows captured members to be initialized with arbitrary expressions. This allow both capture by value-move and declaring arbitrary members of the lambda, without having a corresponding  named variable in an outer scope.
  
  This is done via the use of an initializer expression:
  ```c++
  auto lambda = [value = 1] {return value};
  ```
  
  The lambda function lambda returns 1, which is what value was initialized with. 
  
  This can be used to capture by move, via the use of the standard std::move:
  ```c++
  std::unique_ptr<int> ptr(new int(10));
  auto lambda = [value = std::move(ptr)] {return *value};
  ```
  
### The attribute [[deprecated]]
  The deprecated attribute allows marking an entity deprecated, which **_make it still legal to use but puts users on notice that use is discouraged and may cause a warning message to be printed during compilation_**. An option string literal can appear as the argument of deprecated, to explain the rationate for deprecation and/or to suggest a replacement.
  ```c++
  [[deprecated]] int f();
  
  [[deprecated("g() is thread-safe. Use h() instead")]]
  void g(int &x);
  
  void h(int &x);
  
  void test()
  {
      int a = f();
      g();
  }
  ```

## New standard library features
### Shared mutexes and locking










## 参考:
  * [C++14 wiki](https://en.wikipedia.org/wiki/C++14)
