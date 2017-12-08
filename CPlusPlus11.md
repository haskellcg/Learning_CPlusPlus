# C++ 11
  Is was approved by IOS on 12 August 2011, replacing **_C++03_**, superseded by **_C++14_** on 18 August 2014 and later, by **_C++17_**. It was formerly named **_C++0x_** because it was expected to be published before 2010.
  
  Areas of the core language that were significantly improved include **_multithread support, generic programming support, uniform initialization, and performance_**.
    
## Design goals 
  * Maintain stability and compatibility with **_C++98_** and possible with **_C_**
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
  C++11 adds a new non-const reference type called an rvalue reference, identified by **_T&&_**. This refers a temporaries that are permmitied to be modified after they are initialized, for **_purpose of allowing "move semantics"_**.
  
  In C++11, a move constructor of std::vector<T> that takes an rvalue reference to a std::vector<T> can copy the pointer to the internal C-style array out of the rvalue into the new std::vector<T>, then set the pointer inside the rvalue to null.

  Rvalue reference can provide performance benefits to existing code without needing to make any changes outside the standard library. **_The type of the return value of a function returning a std::vector\<T\> temporary dose not need to be changed explicitly to std::vector\<T\> && to invoke the move constructor, as temporaries are considered rvalues automatically_**.
  
  For safety reasons, some restrictions are imposed. A named variable will **_never be considered to be an rvalue even if it is declared as such_**. To get an rvalue, the **_function template std::move()_** should be used.
  
  Due to the nature of the wording of rvalue reference, and to some modification to the wording for lvalue reference, rvalue references allow **_developers to provide perfect function forwarding_**. When combine with variadic templates, this ability allows for function templates that can perfectly forward arguments to another function that takes those particular arguments. **_This is useful for forwarding constructor parameters, to create factory functions that will automatically call the correct constructor for those particular arguments_**.

---

#### constexpr - Generalized constant expressions
  Constant expressions are optimization oppotunities for compilers, and compilers frequently execute them at **_compile time_**, and hardcode the results in the program.
  
  However, a constant expression has never been allowed to contain a function call or object constructor.
  ```c++
  int get_five()
  {
      return 5;
  }
  
  int some_value[get_five() + 7]; // Create an array of 12 interger. Ill-format C++
  ```
  C++11 introduced the keyword **_constexpr_**, which allows user to guarantee that a function or object constructor is a compile-time constant.
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
  
  To construct constant expression data values from user-defined types, constructors can also be declared with **_constexpr_**. A constexpr constructor's function body can contain only declarations and null statements, and cannot declare variables, or define types, as with constexpr function. There must exist argument values such that, after argument substitution, it initialize the class's members with constant expressions. The destructor for such types must be trival.
  
  The copy constructor for a type with any constexpr constructors should usually **_also be defined as a constexpr constructor_**, to allow objects of the type to be returned by value from a constexpr function. Any member functon of a class, such as copy constructors, operator overload, etc, can be declared as constexpr, so long as they meet the requirements for constexpr functions. **_This allows the compiler to copy objects at the compile time, perform operations on them, etc_**.
  
  **_If a constexpr function or constructors is called with arguments which aren't constant expressions, the call behaves as if the function were not constexpr_**, and the resulting value is not a const expression. Likewise, if the expression in the return statement of a constexpr function does not evaluate to a constant expression for a given invocation, the result is not a contant expression
  
---  
  
#### Modification of the definition of plain old data  
  If someone were to create a C++03 POD type and add a non-vitual member function, this type would **_no longer be a POD type_**, could not be statically initialized, and would be incompatible with C despite no change to memory layout.
  
  C++11 relaxed several of the POD tules, by dividing the POD concept into two separate concepts: **_trivial_** and **_standard-layout_**.
  
  A type that **_trival_** can be **_statically initialized_**. Is also means that it is valid to copy data via **_memcpy_**, rather than having to use a copy constructor. Ths life time of a trivial type **_begins when it storage is defined_**, not when a constructor completes.
  
  A **_trivial class or struct_** is defined as one that:
  * Has a trivial default constructor. This may use the default constructor syntax (SomeConstructor() = default)
  * Has trivial copy and move constructor, which may use the default syntax
  * Has trivial copy and move assignment operators, which may use the default syntax
  * Has a trivial destructor, which **_must not be virtual_**
  
  Constructors are trivial only if there are no virtual member functions of the class and no virtual base class. Copy/move operations also require all non-static data members to be trivial.
  
  A type that is **_standard-layout_** means that **_it orders and packs its members in a way that is compatible with C_**. A class or struct is standard-layout, by definition, provided:
  * It has no virtual functions
  * It has no virtual base classes
  * All its non-static data members have the same access control (public, protected, private)
  * All its non-static data members, including any its base classes, are in the same one class in the hierarchy
  * The above rules also apply to all the base classes and to all non-static data members in the class hierarchy
  * It has no base classes of the same type as the first defined non-static data member
  
  **_A class/struct/union is considered POD if it is trivial, standard-layout, and all of its non-static data members and base classes are PODs_**.
  
### Core language build-time performance enhancements
#### Extern template
  In C++03, the compiler must instantiate a template whenever a fully specified template is emcountered in a translation unit. If the template is **_instantiated with the same types in many translation units_**, this can dramatically increase compile times. There is no way to prevent this in C++03, so C++11 introduced extern template declarations, analogous to extern data declaration.
  
  C++03 has this syntax to oblige the compiler to instantiate a template
  ```c++
  template class std::vector<MyClass>;
  ```
  
  C++11 now provides this syntax:
  ```c++
  extern template class std::vector<MyClass>;
  ```
  **_which tells the compiler not to instantiate the template in this translation unit_**.
    
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
  
  However, C++03 allows intializer-list only on structs and classes that conform to the POD definition; C++11 **_extends intializer-lists_**, so they can be used for classes including standard container like: std::vector:
  
  C++11 binds the concept to a template, called **std::initializer_list**:
  ```c++
  class SequenceClass
  {
      SequenceClass(std::initializer_list<int> list);
  };
  
  SequenceClass some_var = {1, 4, 5, 6};
  ```
  
  This constructor is a special kind of constructor, called **_an intializer-list-constructor_**.

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
  
---  

#### Uniform initializaion
  C++11 provides a syntax that allows for fully uniform type initialization that work on any object.
  ```c++
  struct BasicStruct
  {
      int x;
      double y;
  };
  
  struct AltStruct
  {
      AltStruct(int x, double y):x_(x), y_(y){}
      
      private:
          int x_;
          double y_;
  };
  
  BasicStruct var1{5, 3.2};
  AltStruct var2{2, 4.3};
  ```
  
  **_Implicit type conversion will be used where needed._**
  ```c++
  struct IDString
  {
      std::string name;
      int identifire;
  };
  
  IDString get_string()
  {
      return {"foo", 42};
  }
  ```
  
  Uniform initialization does not replace constructor syntax, which is still needed at times. If a class has an initializer list constructor, then **_it takes priority over other forms of construction_**, provided that the initializer list conforms to the sequence constructor's type.
  
---  
  
#### Type inference  
  In C++03 and C, to use a variable, its type must be specified explicitly. However, with the advent of template types and template metaprogramming techniques, the type of something, particular the **_well-defined return value of a function_**, may not be easily expressed. Thus, strong intermediates in variables is difficult, possibly needing knowledge of the internal of a given metaprogramming library.
  
  C++11 allows this to be mitigate in two ways:
  * the definition of a variable with an explicit initialization can use the **_auto_** keyword, this creates a variable of the specific type of the initializer(The type is easily determined procedurally by the compiler as part of its semantics analysis duties, but is not easy for the user to determine upon inspection):
  ```c++
  auto some_strange_callable_type = std::bind(&some_function, _2, _1, some_object);
  auto other_variable = 5;
  ```    
  * the keyword **_decltype_** can be used to determine the type of expression at compile-time(This is more useful in conjunction with **_auto_**, since the type of auto variable is known only to the compiler. decltype can be also very useful of expressions in code that makes heavy use of operator overloading and specialized types):
  ```c++
  int some_int;
  decltype(some_int) other_integer_variable = 5;
  ```
  
  auto is also useful for reducing the verbosity of the code:
  ```c++
  for (std::vector<int>::const_iterator itr = myvec.begin(); itr != myvec.end(); ++itr)
  
  for (auto itr = myvec.begin(); itr != myvec.end(); ++itr)
  
  for (auto &x: myvec)
  ```
  
  The difference grows as the programmer begins to nest container, though in such cases **_typedef_** are good way to decrease the amount of code. The type denoted by decltype can be different from the type deduced by auto:
  ```c++
  #include <vector>
  
  int main
  {
      const std::vector<int> v(1);
      auto a = v[0];
      decltype(v[0]) b = 1;                // b has type const int&
      
      auto c = 0;
      auto d = c;
      decltype(c) e;
      decltype((c)) f = c;                 // f has type int&
      decltype(0) = g;
  
      return 0;
  }
  ```
  
---  
  
#### Range-based for loop  
  C++11 extends the syntax of the for statement to allow for easy iteration over **_a range of elements_**:
  ```c++
  int my_array[5] = {1, 2, 3, 4, 5};
  
  for (int &x: my_array){
      x *= 2;
  }
  
  for (auto &x: my_array){
      x *= 2;
  }
  ```
  
  This form of for, called the "range-based for", will iterate over each element in the list. It will work for **_C-style arrays, initializer list, any type that has begin() and end() functions defined for it that return iterators_**.
  
---  
  
#### Lambda functions and expressions  
  C++11 provides the ability to create anonymous functions, called **_lambda functions_**:
  ```c++
  [](int x, int y) -> int {return x + y;}
  ```
  The return type (-> int) can be omitted as long as all return expression return the same type.
  
  **_A lambda can optionally be a closure_**.
  
---  
  
#### Alternative function syntax
  For example, in C++03, this is disallowed:
  ```c++
  template <class Lhs, class Rhs>
  Ret adding_func(const Lhs &lhs, const Rhs &rhs)
  {
      return lhs + rhs;
  }
  ```
  
  The type Ret is whatever the addition of types Lhs and Rhs will produce. Even with the aforementioned C++11 functionality of decltype, this is not possible:
  ```c++
  template <class Rhs, class Lhs>
  decltype(lhs+rhs) adding_func(const Lhs &lhs, const Rhd &rhs)
  {
      return lhs + rhs;
  }
  ```
  
  To workaround this, C++11 introduced a new function declaration syntax, with a **trailing-return-type**:
  ```c++
  template <class Rhs, class Lhs>
  auto adding_func(const Lhs &lhs, const Rhs &rhs) -> decltype(lhs+rhs)
  {
      return lhs + rhs;
  }
  ```
  
  This syntax can be used for moew mundane function declarations and definitions:
  ```c++
  struct SomeStruct
  {
      auto func_name(int x, int y) -> int
  };
  
  auto SomeStruct::func_name(int x, int y) -> int
  {
      return x + y;
  }
  ```
  
  **_Use of the keyword "auto" in this case is only part of the syntax and does not perform automatic type definition_**.
  
---  
  
#### Object construction improvement
  **_In C++03, constructors of a class not allowed to call other constructors in an initializer list of that class._** Each constructor must construct all of its class member itself or call a common member function:
  ```c++
  class SomeType
  {
      int number;
      
  private:
      void Constrcutor(int new_number)
      {
          number = new_number;
      }
      
  public:
      SomeType(int new_number)
      {
          Constructor(new_number);
      }
      
      Some_Type()
      {
          Constructor(42);
      }
  };
  ```
  
  Constructor for base classes connot be directly exposed to derived classes; each derived class must implement constructor even if a base class constructor would be appropriate. Non-constant data members of classes cannot be initialized at the side of declaration of those members. They can be initialized only in constructor.
  
  **_C++11 allows constructors to call other peer constructors (termed delegation)_**. This allow constructors to utilize another constructor's behavior with a minimum of added code:
  ```c++
  class SomeType
  {
      int number;
      
  public:
      SomeType(int new_number):number(new_number)
      {
      }
      
      SomeType():SomeType(42)
      {
      }
  };
  ```
  Noice that, in this case, the same effect could have been achieved by making new_number a defaulting parameter. The new syntax, however, allows the default value (42) to be expressed in the **implementation rather than the interface** -- a benefite to maintainers of library code since default values for function parameters are "baked in" to call sites, whereas constructor delegation allows the value to be changed without recompilation the code using the library. [头文件依赖问题？就是说实现方改变编译的时候变动小]
  
  For base-class constructors, C++11 allows a class to **specify that base class constructors will be inherited**. Thus, the C++11 compiler will generate code to perform the inheritance and the following of derived class to the base class. This is an all-or-nothing feature: either all of that base class's constructors are forwarded or none of then are. Also, an inherited constructor will be shadowed if it matches the signature of a constructor of the derived class, and restrictions exist for multiple inheritance: class constructors cannot be inherited from two classes that use constructors with the same signature.
  ```c++
  class BaseClass
  {
  public:
      BaseClass(int value);
  };
  
  class DerivedClass: public BaseClass
  {
  public:
      using BaseClass::BaseClass;
  };    
  ```
  
  for member initialization, C++11 allows this syntax:
  ```c++
  class SomeClass
  {
  public:
      SomeClass()
      {
      }
      
      explicit SomeClass(int new_value):value(new_value)
      {
      }
  
  private:
      int value = 5;  // default initialization
  };
  ```

  **_Any constructor of the class will initialize value with 5, if the constructor does not override the initialization with its own_**.
  
---  
  
#### Explicit override and final
  ```c++
  struct Base
  {
      virtual void some_func(float);
  };
  
  struct Derived: Base
  {
      virtual void some_func(int);
  };
  ```
  
  Because it has a diferent signature, it create a second virtual function. This is a common problem, particular when a user goes to modify the base class. C++11 provides syntax to solve this problem:
  ```c++
  struct Base
  {
      virtual void some_func(float);
  };
  
  struct Derived: Base
  {
      virtual void some_func(int) override;  // ill-formed - doesn't override a base class method
  };  
  ```
  
  **_The override special identifier means the compiler will check the base class(es) to see if there is a virtual function with this exact signature. And if there is not, the compiler will indicate an error__**.
  
  C++11 also adds the ability to prevent inheriting from classes or simplt preventing overriding methods in derived class. This is done with special identifier **_final_**:
  ```c++
  struct Base final
  {
  };
  
  struct Derived: Base  // ill-formed because the class Base has been marked final
  {
  };
  
  struct Base_1     
  {
      virtual void f() final;
  };
  
  struct Derived_1: Base_1
  {
      void f();  // ill-formed because the virtual function Base_1::f has been marked final
  };
  ```
  
  **_Note that neither override nor final are language keywords_**. They are technically indetifier for declarator attributes
  * They gain special meaning as attributes only when used in thoes specific trailing context (after all type specifier, access specifiers. member declarations (for struct, class, enum types) and declaration specifiers, but before initialization or code implementation of each declaration of each declaration in a comma-separated list of declarations)
  * They do not **_alter the declared type signature and do not declare or override any new identifier in any scope_**
  * The recognized and accepted declarator attribute may be extended in future versions of C++
  * In any other location, they can be valid identifier for new declarations (and later use if they are accessible)
  
---  

#### Null pointer constant
   Since the dawn of C in 1972, the constant 0 has had the double role of constant integer and null pointer constant. The ambiguity inherent in the double meaning of 0 was dealt with in C by using the preprocessor macro NULL, which commonly expands to either **_((void \*)0) or 0_**. C++ forbids implicit conversion conversion from void \* to other pointer types, thus removing the benefit of casting 0 to void \*. As a consequence, only **_0 is allowed as a null pointer constant_**. This interacts poorly with function overloading:
   ```c++
   void foo(char *);
   void foo(int);
   ```
  If NULL is defined as 0 (which is usually the case in C++), the statement **_foo(NULL)_** will call foo(int), which is almost centainly not what the programmer intended, and not what a superficial reading of the code suggests.
  
  C++11 corrects this by introducing a new **_keyword_** to serve as a distinguished null pointer constant: **_nullptr_**. It is of type **_nullptr\_t_**, which is implicitly convertible and comparable to any pointer type or pointer-to-member type. It is not implicitly convertible or comparable to integral types, **_except for bool_**. **_0 remains a valid null pointer contant_**.
  ```c++
  char *pc = nullptr;  // OK
  int *pi = nullptr;   // OK
  bool b = nullptr;    // OK, b is false
  int i = nullptr;     // error
  
  foo(nullptr);        // calls foo(nullptr_t), not foo(int)
  /*
    Note that foo(nullptr_t) will actually call the foo(char *) in the above example.
    Only if no other functions are overloading with compatible pointer types in scope.
    If multiple overloading exist, the resolution will fail as it is ambiguous, unless
  there is an explicit declaration of foo(nullptr_t)
  
    In standard types headres for C++11, the nullptr_t type should be declared as:
        typedef decltype(nullptr) nullptr_t;
    but not as:
        typedef int nullptr;
        typedef void * nullptr;
  */
  ```
  
---
  
#### Strongly typed enumerations  
  In C++03, enumerations are not type-safe. The only safety that C++03 provides is that an integer or a value of one enum type does not convert implicitlt to another enum type. Further, the underlying intergal type is implementation defined; code that depends on the size of the enumeration is thus non-portable. Lastly, enumeration values are scoped to the enclosing scope. Thus it is not possible for two separate enumeration in the same scope to have maching member names.
  
  C++11 allows a special classification of enumeration that has none of these issues. This is expressed using the **_enum class_** (enum struct is also accepted as a synonym) declaration:
  ```c++
  enum class Enumeration
  {
      Val1,
      Val2,
      Val3 = 100,
      Val4
  };
  ```
  
  This enumeraton is type-safe. Enum class values are not implicitly converted to integer. Thus, they cannot be compare to integers either (**_the expression Enumeration::Val4 == 101 gives a compile error_**).
  
  The underlying type of enum classes is always known. The deafult is **_int_**, this can be overridden to a different integal type as can be seen in this example:
  ```c++
  enum class Enum2: unsigned int 
  {
      Val1, 
      Val2
  };   
  ```
   **_With old-style enumeration the values are placed in the outer scope. With new-style enumerations they are placed within the scope of the enum class name. So in the above example, Val1 is undefined, but Enum2::Val1 is defined._**
   
   There is also a transitional syntax to allow old-style enumerations to provide explicit scoping, and the definition of the underlying type:
   ```c++
   enum Enum2: unsigned long
   {
       Val1 = 1,
       Val2
   };
   ```
   
   Forward-declaring enums are also possible in C++11. **_Formly, enum types could not be forward-declared because the size of the enumeration depends on the definition of its members. As long as the size of the enumeration is specified either implicitly or explicitly, it can be forward-declared_**:
   ```c++
   enum Enum1;    // Invalid in C++03 and C++11; the underlying type cannot be determined
   enum Enum2: unsigned int;    // Valid in C++11, the underlying type is specified explicitly
   enum class Enum3;    //Valid in C++11, the underlying type is int
   enum class Enum4: unsigned int;    // Valid in C++11
   enum Enum2: unsigned short;    //Invalid in C++11, because Enum2 was formerly declard with a different underlying type
   ```
   
---   
   
#### Right angle bracket
  C++03's parser defines ">>" as right shift operator or stream extraction operator in all cases. However, with nested template declarations, there is tendency for the programmer to neglect to place a space between the two right angle brackets, thus causing a compiler syntax error.
  
  C++11 improves the specification of the parser so that multiple right angle brackets will be interpreted as closing the template argument list where it is reasonable.
  ```c++
  template <bool Test> class SomeType;
  
  std::vector<SomeType<1>2>> x1;    
      // Interpreted as a std::vector of SomeType<true>, followed by "2 >> x1", which is not valid syntax for declaration
  
  std::vector<SomeType<(1>2)>> x2;
      // Interpreted as a std::vector of SomeType<false>, followed by the declarator "x2", which is valid C++11 syntax
  ```
  
---  
  
#### Explicit conversion operators  
  C++98 added the **_explicit keyword as modifier on constructors ro prevent single-argument constructor from being used as implicit type conversion operators_**. However, this does nothing for actual conversion operators. For example, a smart pointer class may have an **_operator bool()_** to allow it to act more like primitive pointer: if it include this conversion, it can be tested with **_if (smart_ptr_variable)_** . However, this allows other, unintended conversions as well. Because C++ bool is defined as an arithmetic type, it can implicitly converted to integral ot even floating-point types, which allows for mathematical operations that not intended by the user.
  
  In C++11, the **_explicit keyword can now be applied to conversion operators_**. As with constrcutor, it prevents using those conversion functions in implicit conversion. however, language contexts that specifically need a boolean value count as explicit conversions and can thus use a bool conversion operator.
  
---  
  
#### Template aliases  
  ```c++
  template <typename First, typename Second, int third>
  class SomeType;
  
  template <typename Second>
  typedef SomeType<OtherType, Second, 5> TypedefName;    //Invalid in C++03
  ```
  
  C++11 adds this ability with this syntax:
  ```c++
  template <typename First, typename Second, int Third>
  class SomeType;
  
  template <typename Second>
  using TypedefName = SomeType<OtherType, Second, 5>
  ```
  
  The **_using_** syntax can be also used as type alias in C++11:
  ```c++
  typedef void (*FunctionType)(double);    //old style
  
  using FunctionType = void (*) (double);    // New introduced syntax
  ```
  
---  

#### Unrestricted unions
  In C++03, **_there are restrictions on what types of objects can be members of a union_**. For example, unions cannot contain any objects that define a non-trivial constructor or destructor. C++11 lifts some of these restrictions.
  
  If a union member has a non-trivial special member function, the compiler will not generate the equivalent member function for the union and it must be manually defined:
  ```c++
  #include <new>    // Needed for placement 'new'
  
  struct Point
  {
      Point()
      {
      }
      
      Point(int x, int y):x_(x), y_(y)
      {
      }
      
      int x_;
      int y_;
  };
  
  union U
  {
      int z;
      double w;
      Point p;    // Invalid in C++03, valid in C++11
      
      U()
      {
      }
      
      U(const Point &pt):p(pt)
      {
      }
      
      U &operator=(const Point &pt)
      {
          new(&p) Point(pt);
          return *this;
      }
  };
  ```
 
 
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
