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
  
  To construct constant expression data values from user-defined types, constructors can also be declared with **constexpr**. A constexpr constructor's function body can contain only declarations and null statements, and cannot declare variables, or define types, as with constexpr function. There must exist argument values such that, after argument substitution, it initialize the class's members with constant expressions. The destructor for such types must be trival.
  
  The copy constructor for a type with any constexpr constructors should usually **also be defined as a constexpr constructor**, to allow objects of the type to be returned by value from a constexpr function. Any member functon of a class, such as copy constructors, operator overload, etc, can be declared as constexpr, so long as they meet the requirements for constexpr functions. **This allows the compiler to copy objects at the compile time, perform operations on them, etc**.
  
  **If a constexpr function or constructors is called with arguments which aren't constant expressions, the call behaves as if the function were not constexpr**, and the resulting value is not a const expression. Likewise, if the expression in the return statement of a constexpr function does not evaluate to a constant expression for a given invocation, the result is not a contant expression
  
#### Modification of the definition of plain old data  
  If someone were to create a C++03 POD type and add a non-vitual member function, this type would **no longer be a POD type**, could not be statically initialized, and would be incompatible with C despite no change to memory layout.
  
  C++11 relaxed several of the POD tules, by dividing the POD concept into two separate concepts: **trivial** and **standard-layout**.
  
  A type that **trival** can be **statically initialized**. Is also means that it is valid to copy data via **memcpy**, rather than having to use a copy constructor. Ths life time of a trivial type **begins when it storage is defined**, not when a constructor completes.
  
  A **trivial class or struct** is defined as one that:
  * Has a trivial default constructor. This may use the default constructor syntax (SomeConstructor() = default)
  * Has trivial copy and move constructor, which may use the default syntax
  * Has trivial copy and move assignment operators, which may use the default syntax
  * Has a trivial destructor, which **must not be virtual**
  
  Constructors are trivial only if there are no virtual member functions of the class and no virtual base class. Copy/move operations also require all non-static data members to be trivial.
  
  A type that is **standard-layout** means that **it orders and packs its members in a way that is compatible with C**. A class or struct is standard-layout, by definition, provided:
  * It has no virtual functions
  * It has no virtual base classes
  * All its non-static data members have the same access control (public, protected, private)
  * All its non-static data members, including any its base classes, are in the same one class in the hierarchy
  * The above rules also apply to all the base classes and to all non-static data members in the class hierarchy
  * It has no base classes of the same type as the first defined non-static data member
  
  **A class/struct/union is considered POD if it is trivial, standard-layout, and all of its non-static data members and base classes are PODs**.
  
### Core language build-time performance enhancements
#### Extern template
  In C++03, the compiler must instantiate a template whenever a fully specified template is emcountered in a translation unit. If the template is **instantiated with the same types in many translation units**, this can dramatically increase compile times. There is no way to prevent this in C++03, so C++11 introduced extern template declarations, analogous to extern data declaration.
  
  C++03 has this syntax to oblige the compiler to instantiate a template
  ```c++
  template class std::vector<MyClass>;
  ```
  
  C++11 now provides this syntax:
  ```c++
  extern template class std::vector<MyClass>;
  ```
  **which tells the compiler not to instantiate the template in this translation unit**.
    
### Core language usability enhancements
#### Initializer lists  
  C++03 inherited the initializer-list feature from C:
  ```c++
  struct Object
  {
      float first;
      int second;
  };
  
  Object scalar = {0.43f, 10};
  Object anArray[] = {{13.4f, 3}, {43.28f, 29}, {5.934f, 17}};
  ```
  
  However, C++03 allows intializer-list only on structs and classes that conform to the POD definition; C++11 **extends intializer-lists**, so they can be used for classes including standard container like: std::vector:
  
  C++11 binds the concept to a template, called **std::initializer_list**:
  ```c++
  class SequenceClass
  {
      SequenceClass(std::initializer_list<int> list);
  };
  
  SequenceClass some_var = {1, 4, 5, 6};
  ```
  
  This constructor is a special kind of constructor, called **an intializer-list-constructor**.

  The class std::initializer_list<> is a first-class C++11 standard library type. However, they can be initially constructed statically by the C++11 compiler only via use of the {} syntax. The list can be copied once constructed, though this is only a copy-by-reference. An initializer list is constant, its members cannot be changed once the initialzer list is created, nor can the data in those member be changed.
  
  Because initializer_list is a real type, it can be used in other places besides class constructors.
  ```c++
  void function_name(std::initializer_list<float> list);
  
  function_name({1.0f, -3.45f, -0.4f});
  ```
  
  Standard containers can also be intialized in these ways:
  ```c++
  std::vector<std::string> v = {"x", "y", "z"};
  std::vector<std::string> v({"x", "y", "z"});
  std::vector<std::string> v{"x", "y", "z"};
  ```

#### Uniform initializaion
  

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
