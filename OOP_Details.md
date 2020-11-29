## Virtual function, Overloading, Inheritate
  * Virtual functions can be overloaded by other virtual functions, as well as non-virtual functions
  * Pure virtual functions can also be overloaded by regular virtual functions or non-virtual functions
  * If multiple overloaded virtual functions (including pure virtual functions) exist in base class, but only one or some of the function signatures are overridden in derived class, all other un-overridden (including non-virtual) function signatures will be hidden and unavailable in derived class, even the inheritance is public
  * However, if none of such virtual functions are overridden in derived class, all overloaded versions are available in derived class if the inheritance is public
  * [Referrence](https://robentan.blogspot.com/2011/10/can-virtual-functions-be-overloaded.html)
  
## Why virtual destructor
  * In the first alternative (delete object), if the static type of the object to be deleted is different from its dynamic type, the static type shall be a base class of the dynamic type of the object to be deleted and the static type shall have a virtual destructor or the behavior is undefined. In the second alternative (delete array) if the dynamic type of the object to be deleted differs from its static type, the behavior is undefined.
  * [OOP52-CPP. Do not delete a polymorphic object without a virtual destructor](https://wiki.sei.cmu.edu/confluence/display/cplusplus/OOP52-CPP.+Do+not+delete+a+polymorphic+object+without+a+virtual+destructor)
  
## Calling virtual function in constructor
  * Member functions, including virtual functions, can be called during construction or destruction. When a virtual function is called directly or indirectly from a constructor or from a destructor, including during the construction or destruction of the class’s non-static data members, and the object to which the call applies is the object (call it x) under construction or destruction, the function called is the final overrider in the constructor’s or destructor’s class and not one overriding it in a more-derived class. If the virtual function call uses an explicit class member access and the object expression refers to the complete object of x or one of that object’s base class subobjects but not x or one of its base class subobjects, the behavior is undefined.
  * [Do not invoke virtual functions from constructors or destructors](https://wiki.sei.cmu.edu/confluence/display/cplusplus/OOP50-CPP.+Do+not+invoke+virtual+functions+from+constructors+or+destructors)
