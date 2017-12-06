# C++ 11
  Is was approved by IOS on 12 August 2011, replacing **C++03**, superseded by **C++14** on 18 August 2014 and later, by **C++17**. It was formerly named **C++0x** because it was expected to be published before 2010.
  
  Areas of the core language that were significantly improved include **multithread support, generic programming support, uniform initialization, and performance**.
  
## Design goals 
  * Maintain stability and compatibility with **C++98** and possible with **C**
  * Prefer introducing new features via the standard library, rather than extending the core language
  * Prefer changes that can evolve programming technique
  * Improve C++ to facilitate systems and library design, rather than introducing new features usedful only to specific applications
  * Increase type safety by providing safer alternatives to earlier unsafe techniques
  * Increase performance and the ability to work directly with hardware
  * Provide proper solutions for real-world problems
  * Implement zero-overhead principle (further support needed by some utilities must be used only if the utility is used)
  * Make C++ easy to teach and to learn without removing any utility needed by expert programmers
  
## Extensions to the C++ core language  
### Core language runtime performance enhancements
#### Rvalue references and move constructors
  C++11 adds a new non-const reference type called an rvalue reference, identified by **T&&**. This refers a temporaries that are permmitied to be modified after they are initialized, for **purpose of allowing "move semantics"**.
  
  In C++11, a move constructor of std::vector<T> that takes an rvalue reference to a std::vector<T> can copy the pointer to the internal C-style array out of the rvalue into the new std::vector<T>, then set the pointer inside the rvalue to null.

  Rvalue reference can provide performance benefits to existing code without needing to make any changes outside the standard library. **The type of the return value of a function returning a std::vector\<T\> temporary dose not need to be changed explicitly to std::vector\<T\> && to invoke the move constructor, as temporaries are considered rvalues automatically**.
  
  For safety reasons, some restrictions are imposed. A named variable will **never be considered to be an rvalue even if it is declared as such**. To get an rvalue, the **function template std::move()** should be used.
  
  Due to the nature of the wording of rvalue reference, and to some modification to the wording for lvalue reference, rvalue references allow **developers to provide perfect function forwarding**. When combine with variadic templates, this ability allows for function templates that can perfectly forward arguments to another function that takes those particular arguments. **This is useful for forwarding constructor parameters, to create factory functions that will automatically call the correct constructor for those particular arguments**.
  
#### constexpr - Generalized constant expressions
  Constant expressions are optimization oppotunities for compilers, and compilers frequently execute them at **compile time**, and hardcode the results in the program.
  
  However, a constant expression has never been allowed to contain a function call or object constructor.
  ```c++
  int get_five()
  {
      return 5;
  }
  
  int some_value[get_five() + 7]; // Create an array of 12 interger. Ill-format C++
  ```
  C++11 introduced the keyword **constexpr**, which allows user to guarantee that a function or object constructor is a compile-time constant.
  ```c++
  constexpr int get_five()
  {
      return 5;
  }
  
  int some_value[get_five() + 7];
  ```
  
  Using constexpr on a function imposes some limits on what that function can do:
  * The function must have a non-void return type
  * The function body cannot declare variables or define new types
  * The function body may contain old declarations, null statements, and a simple return statement
  * There must exist argument values such that, after argument substitution, the expression in the return statement produces a contant expression
  
  Before C++11, the value of variables could be used in constant expressions only if the variables are declared const, have an initializer which is a constant expression, and are of integral or enumeration type. C++11 removes the restriction that the variables must be of integral or enumeration type if they are define with the constexpr keywork.
  
  
  
  
  
  
  
  
  
  
  
  
  

### Core language build-time performance enhancements
### Core language usability enhancements
### Core language functionality improvements

## C++ standard library changes
### Updates to standard library components
### Threading facilities
### Tuple types
### Hash tables
### Regular expressions
### General-purpose smart pointers
### Extension random number facility
### Wrapper reference
### Polymorphic wrapper for function objects
### Type traits for metaprogramming
### Uniform method for computing the return type of function objects

## Improved C compatibility

## Features originally planned but removed or not include

## Features removed or deprecated

## See also

## References

## External links
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
## 参考:
  * [C++11 wiki](https://en.wikipedia.org/wiki/C++11)
