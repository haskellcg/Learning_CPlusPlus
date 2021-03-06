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
  These features allow the language to do things that were formerly impossible, exceedingly verbose, or needed non-portable libraries.

#### Variadic template
  In C++11, templates can take variable numbers of template parameters. This also allows the definition of type-safe variadic functions.
  
---

#### New string literals
  **_C++03 offers two kinds of string literals (const char: ""; const w_char: L"")_**. Neither literal type offers support for string literals with UTF-8, UTF-16, or any other kind of Unicode encodings.
  
  The definition of the type char has been **_modified to explicitly express that it's at least the size needed to store an eight-bit coding of UTF-8_**, and large enough to contain any member of the compiler's basic execution character set. It was formerly defined as only the latter in the C++ standard itself, then relying on the C standard yo guarantee at least 8 bits.
  
  C++11 supports 3 Unicode encodings: UTF-8(char), UTF16(char16_t), UTF-32(char32_t):
  ```c++
  u8"I'm a UTF-8 string"    // const char[]
  u"This is a UTF-16 string"    // const char16_t[]
  U"This is a UTF-32 string"    // const char32_t[]  
  ```
  
  When building Unicode string literals, it is often useful to insert **_Unicode codepoints directly into the string_**. To do this, C++11 allows this syntax:
  ```c++
  u8"This is a Unicode Character: \u2018"
  u"This is a bigger Unicode Character: \u2018"
  U"This is a Unicode Character: \U00002018"
  ```
  Only valid Unicode codepoints can be entered. For example, codepoints on the range U+D800--U+DFFF are forbidden, as they are reserved for surrogate pairs in UTF-16 encodings.
  
  It is also sometimes useful to avoid escaping strings manually, C++ provides a raw string literal:
  ```c++
  R"(The String Data \ Stuff " )"
  R"delimiter(The String Data \ Stuff " )delimiter"
  ```
  In second case, the "delimiter(" starts the string, and it ends only when ")delimiter" is reached. The string **_delimiter_** can be any string up to 16 characters in length, including the empty string. This string cannot contain spaces, control characters, (, ), ot the \\ character. **_Using this delimiter string allows the user to have ) characters within raw string literals_**.
  ```c++
  R"delimiter((a-z))delimiter"    // (a-z)
  ```
  
  Raw string literals can be combined with the wide literal or any of the Unicode literal prefixes:
  ```c++
  u8R"XXX(I'm a "raw UTF-8" string.)XXX"
  uR"*(This is a "raw UTF-16" string.)*"
  UR"(this is a "raw UTF-32" string.)"
  ```
  
---

#### User-defined literals
  C++03 provides a number of literals. "12.5 -> double, 12.5 -> float", the suffix modifiers for literals are fixed by the C++ specification, and C++03 code cannot create new literal modifiers.

  **_By contrast, C++11 enables the user to define new kinds of literal modifiers that will construct objects based on the string of characters that the literal modifier_**.

  Transformation of literals is redefined into two distinct phases:raw and cooked.
  
  raw literal|cooked literal
  -----------|--------------
  "1234"|1234
  "0xA"|10
  
  literals can be extended in both raw and cooked forms, **_with the exception of string literals, which can be processed only in cooked form_**. This exception is due to the fact that strings have prefixs that affect the specific meaning and type of the characters in question.
  
  All user-defined literals are suffixes; defining prefix literal is not possible. All suffixes starting with any character except underscore (\_) are reserved by the standard. Thus, all user-defined literals must have suffixes starting with an underscore (\_).
  
  User-defined literals processing the raw from of the literal are defined via a literal operator, which is written as **_operator ""_**:
  ```c++
  OutputType operator "" _mysuffix(const char *literal_string)
  {
      // assumes that OutputType has a constructor that takes a const char *
      OutputType ret(literal_string);
      return ret;
  }
  
  // This function is passed "1234" as a C-style string, so it has a null terminator
  OutputType some_variable = 1234_musuffix;
  // assumes that OutputType has a get_value() method that returns a double
  assert(some_variable.get_value() == 1234.0)
  ```

  An alternative mechanism for processing integer and floating point raw literals is via a variadic templte:
  ```c++
  template <char...>
  OutputType operator "" _tuffix();
  
  OutputType some_variable = 1234_tuffix;
  OutputType another_variable = 2.17_tuffix;
  ```
  
  This instantiate the literal processing function as **_operator "" \_tuffix<'1', '2', '3', '4'>_**. In thsi form, there is no null character terminating the string. The main purpose for doing this is to use C++11's constexpr keyword to ensure that the compiler will transform the literal entirely at compile time, assuming OutputType is a constexpr-constructible and copyable type, and the literal processing function is a constexpr function.
  
  For numeric literals, the type of the cooked literal is either **_unsigned long long_** for integral literals or **_long double_** for floating point literals. (Note: There is no need for signed integral types because a sign-prefixed literal is parsed as an expression containning the sign as a unary prefix operator and the unsigned number.) There is no alternative template form:
  ```c++
  OutputType operator "" _suffix(unsigned long long);
  OutputType operator "" _suffix(long double);
  
  OutputType some_variable = 1234_suffix;    // Uses the "unsigned long long" overload
  OutputType another_variable = 3.1416_suffix;    // Uses the "long double" pverload
  ```
  
  In accord with the formerly mentioned new string prefix, for string literals, these are used:
  ```c++
  OutputType operator "" _ssuffix(const char *string_values, size_t num_chars);
  OutputType operator "" _ssuffix(const wchar_t *string_values, size_t num_chars);
  OutputType operator "" _ssuffix(const char16_t *string_values, size_t num_chars);
  OutputType operator "" _ssuffix(const char32_t *string_values, size_t num_chars);
  
  OutputType some_variable = "1234"_ssuffix;    // Uses the "const char *" overload
  OutputType some_variable = u8"1234"_ssuffix;    // Uses the "const char *" overload
  OutputType some_variable = L"1234"_ssuffix;    // Uses the "const wchar_t *" overload
  OutputType some_variable = u"1234"_ssuffix;    // Uses the "const char16_t *" overload
  OutputType some_variable = U"1234"_ssuffix;    // Uses the "const char32_t *" overload
  ```
  
---

#### Multithreading memory model
  C++11 standardlizes support for multithreaded programming.
  
  There are two parts involved:
  * A memory model which allows multiple threads to co-exist in a program
  * Library support for interaction between threads
  
  The memory model defines when multiple threads may access the same memory localtion, and specifies when updates by one thread become visible to other threads.
  
---

#### Thread-local storage
  In a multi-threaded environment, it is common for every thread to have some unique variables. This already happends for **_the local variables of a function, but it does not happen for global and static variables_**.
  
  A new thread-local storage duration (in addition to the existing static, dynamic, automatic) is indicated by the storage specifier **_thread\_local_**.
  
  Any object which could have static storage duration (i.e., lifetime spanning the entire execution of the program) may be given thread-local duration instead. The intend is that like any other static-duration variable, **_a thread-local object can be initialized using a constructor and destroyed using a destructor_**.
  
---

#### Explicitly defaulted and deleted special member functions
  In C++03, making a class inheritedly non-copyable, for example, requires declaring a private copy constructor and copy assignment operator and not defining them. Attempting to use these functions is **_a violation of the One Definition Rule (ODR)_**. While a disgnostic message is not required, violations may result in a linker error.
  
  C++11 allows the explicit defaulting and deleting of these special member functions:
  ```c++
  struct SomeType
  {
      SomeType() = default;    // The default constructor is explicitly stated
      SomeType(OtherType value);
  };
  
  struct NonCopyable
  {
      NonCopyable() = default;
      NonCopyable(const NonCopyable &) = delete;
      NonCopyable &operator=(const NonCopyable &) = delete;
  };
  ```

  The **_= delete_** specifier can be used to prohibit calling any function, which can be used to disallow calling a member function with particular parameters:
  ```c++
  struct NoInt
  {
      void f(double);
      void f(int) = delete;
  };
  
  // To disallow calling the function with any type other than double
  struct OnlyDouble
  {
      void f(double d);
      
      template <class T>
      void f(T) = delete;
  };
  ```
  
---  

#### Type long long int
  In C++03, ther largest integer type is **_long int_**. It is guaranteed to have at least as many useable bits as int. **_This resulted in long int having size of 64 bits on some popular implementations and 32 bits on others_**.

  C++11 adds a new integer type **_long long int_** to address this issue. It is guaranteed to be at least as large as a **_long int_**, and have no fewer than 64 bits. The type was originally introduced by C99 to standard C, and most C++ compilers supported it as an extension already.
  
---

#### Static assertions
  **_C++03 provides two methods to test assertions_**:
  * the macro assert
  * the preprocessor directive #error
  
  However, neither is appropriate for use in template: **_the macro tests the assertion at execution-time, while the preprocessor directive tests the assertion during preprocessor, which happens before instantiation of templates_**.
  
  
  The new utility introduces a new way to test assertions at compile-time, using keyword **_static_assert_**. The declaration assumes this form:
  ```c++
  static_assert(constant-expression, error-message)
  ```
  
  Example:
  ```c++
  static_assert((GREEKPI > 3.14) && (GREEKPI < 3.15), "GREEKPI is inaccurate");    
  ```
  ```c++
  template <class T>
  struct Check
  {
      static_assert(sizeof(int) <= sizeof(T), "T is not big enough");
  };
  ```
  ```c++
  template <class Integral>
  Integral foo(Integral x, Integral y)
  {
      static_assert(std::is_integral<Integral>::value, "foo() parameter must be an integral type");
  }
  ```
  
  Static assertions are useful outside of templates also. For instance, a given implementation of an algorithm might depend on the size of a long long being larger than an int, something the stanard does not guarantee. Such an assumption is valid on most systems and compilers, but not all.
  
---

#### Allow sizeof to work on members of classes without an explicit object
  In C++03, the sizeof operator can be used on types and pbjects. But it cannot be used to do this:
  ```c++
  struct SomeType
  {
      OtherType member;
  };
  
  sizeof(SomeType::member);    // Does not work on C++03. Okay with C++11
  ```
  
  This should return size of OtherType.
  
  **_C++03 disallow this, so it is a compiler error. C++11 allow it_**.
  
  It is also allowed for the **_alignof_** operator introduced in C++11.
  
---

#### Control and query object alignment
  C++11 allows variable alignment to be queried and controlled with **_alignof_** and **_alignas_**.
  
  The alignof operator takes the type and returns the power of 2 byte boundary on which the type instances must be allocated (as a std::size_t). When given a reference type alignof returns **_the reference type's alignment_**; **_for arrays it returns the element type's alignment_**.
  
  The alignas specifier controls the memory alignment for a available. The specifier takes a contant or a type; when supplied a type **_alignas(T)_** is shorthand for **_alignas(alignof(T))_**. 
  
  For example, to specify that a clear array should be properlt aligned to hold a float:
  ```c++
  alignas(float) unsigned char c[sizeof(float)];
  ```
  
#### Allow garbage collected implementations
  Prior C++ standard provided for programmer-driven garbage collection via **_set_new_handler_**, but gave no definition of object reachability for the purpose of automatic garbage collection.
  
  C++11 defines conditions under which pointers values are "safely derived" from other values. An implementation may specify that it operats under **_strict pointer satety_**, in which case pointers that are not derived according to these rules can become invalid.
  
---

#### Attributes
  C++11 provides a standardized syntax for compiler/tool extensions to the language. **_Such extensions were traditionally specified using #pragma directive or vendor-specific keywords (like __attribute__ for GNU and __declspec for Microsoft)_**. With new syntax, added information can be **_specified in a form of an attribute enclose in double square brackets_**:
  ```c++
  int [[attr1]] i [[attr2, attr3]];
  
  [[attr4(arg1, arg2)]] if (coud){
      [[vendor::attr5]] return i;
  }
  ```
  
  In general (but with some exceptions), an attribute specified for a named entity is placed after the name, and before the entity otherwise, as shown above, several attributes may be listed inside one pair of double square brackets, added arguments may be provided for an attribute, and attribute may be scoped by vendor-specific attribute namespace.
  
  **_It is recommended that attributes have no language semantic meaning and do not change the sense of a program when ignored_**. Attribute can be useful for providing information that, for example, helps the compiler to issue better diagnostic or optimize the generated code.
  
  C++ provides two standard attributes itself:
  * noreturn, to specifiy that a function does not return
  * carries_dependency, to help optimizing multi-threaded code by indicating that function arguments or return value carry a denpendency

## C++ standard library changes
  A large part of the new library in the document **_C++ Standards Committee's Library Technical Report (TR1)_**, which was published in 2005. Various full and partial implementations of TR1 are currently available using the namespace std::tr1. 

### Updates to standard library components
  Most standard library containers can benefit **_from Rvalue reference based move constructor support_**, both for quickly moving heavy containers around and for moving the contents of those containers to new memory locations. The standard library components were upgraded with new C++11 language features where appropriate. These include, but are not necessarily limited to:
  * Rvalue references and the associated move support
  * Support for the UTF-16 encoding unit, and UTF-32 encoding unit Unicode character types
  * Variadic templates (couple with Rvalue references to allow for perfect forwarding)
  * Compile-time constant expressions
  * decltype
  * explicit conversion operators
  * default/delete functions
  
  Further, much time has passed since the prior C++ standard. Much code using the standard library has been writen. This has revealed parts of the standard libraries that could use some improving.
  
  Among the many areas of improvements considered were standard library allocators. A new scope-based model of allocators was included in C++11 to supplement the prior model.

### Threading facilities
  While the C++03 language provides a memory model that supports threading, the primary support for actually using threading comes with the C++11 standard library:  
  * A thread class: std::thread
  * Thread join support: std::thread::join
  * Access is provided: std::thread::native_handle
  * For synchronization: std::mutex, std::recursive_mutex, std::condition_variable, std::condition_variable_any
  * Resource Acquisition is Initialization (RAII) locks: std::lock_guard, std::unique_lock
  * For high-performance: atomic operations on memory locations, memory barries
  * The C++11 library also includes futures and promises for passing asynchronous results between threads
  * The new std::async facility provides a convenient method of running tasks and tying them to a std::future
  
### Tuple types
  Tuples are collections composed of heterogeneous objects of pre-arranged dimensions. A tuple can be considered a generalization of a struct's member variables.

  The C++11 version of the TR1 tuple type benefit from C++11 features like variadic templates.
  
  Using variadic templates, the declaration of the tuple class looks as follows:
  ```c++
  template<class ...Types>
  tuple;
  ```
  
  An example of definition and use of the tuple type:
  ```c++
  typedef std::tuple<int, double, long &, const char *> test_tuple;
  long lengthy = 12;
  test_tuple proof(18, 6.5, lengthy, "Ciao!");
  
  lengthy = std::get<0>(proof);
  std::get<3>(proof) = " Beautiful!";
  ```
  
  It is possible to assign a tuple to another tuple: if the 2 tuples' type are the same, each element type must prossess a copy constructor, otherwise, each element type of the right-ide tuple must be convertible to that of the corresponding elenment type of the left-side tuple or that the corresponding element type of the left-side tuple has a suitable constructor:
  ```c++
  typedef std::tuple<int, double, string> tuple_1 t1;
  typedef std::tuple<char, double, const char *> tuple_2 t2('X', 2, "Hola");
  
  t1 = t2;    // OK, first two elements can be converted
              // the third one can be constructed from a 'const char *'
  ```
  
  **_std::make\_tuple to automatically create std::tuple using type dedution and auto helps to declare such a tuple. std::tie creates tuples of lvalue reference to help unpack tuples, std::ignore also helps here_**:
  ```c++
  auto record = std::make_tuple("Hari Ram", "New Delhi", 3.5, 'A');
  std::string name;
  float gpa;
  char grade;
  std::tie(name, std::ignore, gpa, garde) = record;
      // std::ignore helps drop the place name
  std::cout << name << ' ' << gpa << ' ' << grade << std:endl;
  ```
  
  Two expressions are available to check a tuple's characteristics (only during compilation):
  * std::tuple_size<T>::value -- returns the number of elements in the tuple T
  * std::tuple_element<I, T>::type -- returns the type of the object number I of the tuple T
  
### Hash tables
  Including hash tables (unordered associative contains) in the C++ standard library is one of the most recurring requests. **_It was not adopted in C++03 due to time contraints only_**.
  
  Collisions are managed only via **_linear chaining_** because the committee didnt consider it to be opportune to standardize solutions of **_open addressing_** that introduce quite a lot of intrinsic problems (above all when erasure of elements is admitted). 
  
  The new library has four types of hash tables. They correspond to the four existing binary-search-tree based associative contianers, with an unordered_ prefix:
  
  Type of Hash table|Associated values|Equivalent keys
  ------------------|-----------------|---------------
  std::unordered_set|No|No
  std::unordered_multiset|No|Yes
  std::unordered_map|Yes|No
  std::unordered_multimap|Yes|Yes
  
  This new feature didn't need any C++ language core extensions (though implementations will take advantages of various C++11 language features), only a small extension of the header <functional> and the introduction of header <unordered_set> and <unordered_map>.
  
### Regular expressions
  The new library, defined in the new header <regex>, is made of a couple of new classes:
  * regular expression are represented by instance of the template class std::regex
  * occurrences are represents by instance of the template class std::match_results
  
  The algorithms std::regex_search and std::regex_replace tak a regular expression and a string and write the occurrences found in the struct std::match_results:
  ```c++
  const char *reg_esp = "[ ,.\\t\\n;:]";
  
  // This is can be done using raw string literals:
  // const char *reg_esp = R"regex([ ,.\t\n;:])regex"
  
  // regex is an instance of the template calss 'base_regex' with argument of type 'char'
  std::regex rgx(reg_esp);
  
  // cmatch is an instance of the tamplate class 'match_results' with argument of type 'const char *'
  std::cmatch match;
  
  const char *target = "Unseen University - Ankh-Morpork"
  
  if (std::regex_search(target, match, rgx)){
      const size_t n = match.size();
      for (size_t a = 0; a < n; a++){
          std::string str(match[a].first, match[a].second);
          std::count << str << "\n";
      }
  }
  ```
  
  The library <regex> requires neither alternation of any existing header (though it will use them where appropriate) nor an extension of the core language. In POSIX C, regular expression are also available the C POSIX library#regex.h.  

### General-purpose smart pointers
  C++11 provides std::unique_ptr, and improvements to std::shared_ptr and std::weak_ptr from TR1, std::auto_ptr is deprecated.

### Extension random number facility
  The C standard library provides the ability to generate pseudorandom number via the function **_rand_**. However, the algorithm is delegated entirely to the library vendor. **_C++ inherited this functionality with no changes, but C++11 provides a new method for generating pseudorandom numbers_**.
  
  C++11's random number functionality is split into 2 parts:
  * A generator engine that contains the random number generator's state and produces the pseudorandom numbers
  * A distribution, which determines the range and mathematical distribution of the outcome
  
  C++11 mechanism will come with 3 base generator engine algorithms:
  * linear_congruential_engine
  * subtract_with_carry_engine
  * mersenne_twister_engine
  
  C++11 also provides a number of standard distributions:
  * uniform_int_distribution
  * uniform_real_distribution
  * bernoulli_distribution
  * binomial_distribution
  * geometric_distribution
  * negative_binomial_distribution
  * poisson_distribution
  * exponential_distribution
  * gamma_distribution
  * weibull_distribution
  * extreme_value_distribution
  * normal_distribution
  * lognormal_distribution
  * chi_squared_distribution
  * cauchy_distribution
  * fisher_f_distribution
  * student_t_distribution
  * discrete_distribution
  * piecewise_contant_distribution
  * piecewise_linear_distribution
    
  The generator and distributions are combined as in this example:
  ```c++
  #include <random>
  #include <functional>
  
  std::uniform_int_distribution<int> distribution(0, 99);
  std::mt19937 engine;
  auto generator = std::bind(distribution, engine);
  
  //1
  int random = generator();
  
  //2
  int random2 = distribution(engine);
  ```
    
### Wrapper reference
  A wrapper reference is obtained form an interface of the template class **_reference_wrapper_**. Wrapper references are similar to normal reference ('&') of the C++ language. To obtain a wrapper reference from any object the function template **_ref_** is used (for a constant reference cref is used).
  
  Wrapper references are useful above all for function template, where references to parameters rather than copies are needed:
  ```c++
  void func(int &r)
  {
      r++;
  }
  
  template<class F, class P>
  void g(F f, P t)
  {
      f(t);
  }
  
  int main()
  {
      int i = 0;
      g(func, i);
      std::cout << i << std::endl;
      
      g(func, std::ref(i));
      std::cout << i << std::endl;
  
      return 0;
  }
  ```
  
  This new utility was added to existing <utility> header and didn't need further extension of the C++ language.
  
### Polymorphic wrapper for function objects
  Polymorphic wrappers for function objects are similar to function pointers in semantics and syntax, but are less tightly bound and **_can indiscriminately refer to anything which can be called (function pointer, member function pointer, or functors) whose arguments are compatible with those of the wrapper_**:
  ```c++
  std::function<int (int, int)> func;
  std::plus<int> add;
  
  func = add;
  int a = func(1, 2);
  
  std::function<bool (short, short)> func2;
  if (!func2){
      bool adjacent(long x, long y);
      func2 = &adjacent;
      
      struct Test
      {
          bool operator()(short x, short y);
      }:
      Test car;
      func = std::ref(car);
  }
  
  func = func2;
  ```  
  The template class function was defined inside the header <functional>, without needing any change to the C++ language.
  
### Type traits for metaprogramming
  Metaprogramming consists of creating a program that creates or modifies another program (or itself). This can happen during compilation or during executing. The C++ standard Committee has decided to introduce a library that allows metaprogramming during compiling via templates.
  
  Here is an example of a meta-program, using the C++03 standard: a recursion of template instances for calculating integer exponents:
  ```c++
  template<int B, int N>
  struct Pow
  {
      enum
      {
          value = B * Pow<B, N-1>::Value;
      };
  };
  
  template<int B>
  struct Pow<B, 0>
  {
      enum
      {
          value = 1;
      };
  };
  
  int quartic_of_three = Pow<3, 4>::value;
  ```
  
  Many algorithms can operate on different types of data; C++'s templates support generic programming and make code more compact and useful. Nevertheless, **_it is common for algorithms to need information on data types being used_**. This information can be extracted during instantiation of a template class using type traits.
  
  Type traits can identify the category of an object and all the characteristics of a class (or of a struct). They are defined in the new header <type_traits>:
  ```c++
  template<bool B>
  struct Algorithm
  {
      template<class T1, class T2>
      static int do_it(T1 &, T2 &)
      {
      }
  };
  
  template<>
  struct Algorithm<true>
  {
      template<class T1, class T2>
      static int do_it(T1, T2)
      {
      }
  };
  
  template<class T1, class T2>
  int elaborate(T1 A, T2 B)
  {
      return Algorithm<std::is_integral<T1>::value && std::is_floating_point<T2>::value>::do_it(A, B);
  }
  ```
  
  Via type traits, defined in header **_type_traits_**, it is also possible to create type transformation operations (static_cast and const_cast are insufficient inside a template).
  
  This type of programming produces elegant and concise code; however the weak point of these techniques is the debugging: uncomfortable during compilation and very difficult program execution.
  
### Uniform method for computing the return type of function objects
  Determining the return type of a template function object at compile-time is not intuitive, particularly if the return value depends on the parameters of the function. As an example:
  ```c++
  struct Clear
  {
      int operator()(int) const;
      double operator()(double) const;
  };
  
  template<class Obj>
  class Calculus
  {
  public:
      template<class Arg>
      Arg operator()(Arg &a) const
      {
          return member(a);
      }
  
  private:
      Obj member;
  };
  ```
  
  Instantiating the class template **_Calculus<Clear>_**, the function object of calculus will have always the same return type as the function object of Clear, however, given class Confused below: 
  
  ```c++
  struct Confused
  {
      double operator()(int) const;
      int operator()(double) const;
  };
  ```
  The compiler may generate warnings about the conversion from int to double and vice versa.
  
  TR1 introduces, and C++11 adopts, the template class **_std:result\_of_** that allows one to determine and use the return type of a function object for every declaration:
  ```c++
  template<class Obj>
  class CalculusVer2
  {
  public:
      template<class Arg>
      typename std::result_of<Obj(Arg)>::type operator()(Arg &a) const
      {
          return member(a);
      }
  
  private:
      Obj member;  
  };
  ```
  
  The only change from the TR1 version of std::result_of is that the TR1 version allowed an implementation to fail to be able to determine the result type of a function call. **_Due to changes to C++11 for support decltype_**, the C++11 version of std::result_of no longer needs these special cases; implementation are required to compute a type in all cases.
  
## Improved C compatibility
  For compatibility with C, from C99, these were added:
  * Preprocessor:
    * variadic marocs
    * concatenation of adjacent narrow/wide string literals
    * _Pragma() -- equivalent of #pragma
  * long long  -- integer type that is at least 64 bits long
  * \_\_func\_\_ -- macro evaluating to the name of the function it is in
  * headers:
    * cstdbool
    * cstdint
    * cinttypes

## Features originally planned but removed or not include
  Heading for a separate TR:
  * Modules
  * Decimal types
  * Math special function
  
  Postponed:
  * Concepts
  * More complete or required garbage collection support
  * Reflection
  * Macro scopes

## Features removed or deprecated
  The term **_sequence point_** was removed, being replaced by specifying that either one operation is sequenced before another, or that two operations are unsequenced.
  
  The former use of the keyword export was removed, the keyword itself remains, being reserved for potential future use.
  
  Dynamic exception sepcification are deprecated. Compile-time specification of non-exception-throwing functions is available with noexcept keyword, which is useful for optimization.
  
  std::auto_ptr is deprecated, having been superseded by std::unique_ptr
  
  Function object base class (std::unary_function, std::binary_function), adapters to pointers to functions and adapters to pointers to members, and binder classes are all deprecated.
  
## See also

## References

## External links
  * [Online C++11 compiler](http://coliru.stacked-crooked.com)
  
## 参考:
  * [C++11 wiki](https://en.wikipedia.org/wiki/C++11)
