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













## 参考:
  * [C++14 wiki](https://en.wikipedia.org/wiki/C++14)
