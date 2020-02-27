---
title: c++类型转换
date: 2018-10-28 21:42:48
tags: c++
categories: c++
---

# 四种类型转换
> * static_cast 一种类型转换成另一种兼容类型, 通常用于转换数值类型, 编译时检查
> * dynamic_cast 虚基类转换成派生类, 运行时检查
> * const_cast 修改类型的const或volatile属性
> * reinterpret_cast 比较底层的转换, 在非相关的类型之间转换; 操作结果只  
是简单的从一个指针到别的指针的值的二进制拷贝;在类型之间指向的内容不做任  
何类型的检查和转换

# code example

## static_cast

```
#include <vector>
#include <iostream>
 
struct B {};
struct D : B {};
 
enum class E { ONE, TWO, THREE };
enum EU { ONE, TWO, THREE };
 
int main()
{
    // 1: initializing conversion
    int n = static_cast<int>(3.14); 
    std::cout << "n = " << n << '\n';
    std::vector<int> v = static_cast<std::vector<int>>(10);
    std::cout << "v.size() = " << v.size() << '\n';
 
    // 2: static downcast
    D d;
    B& br = d; // upcast via implicit conversion 隐式转换
    D& another_d = static_cast<D&>(br); // downcast
 
    // 3: lvalue to xvalue
    std::vector<int> v2 = static_cast<std::vector<int>&&>(v);
    std::cout << "after move, v.size() = " << v.size() << '\n';
 
    // 4: discarded-value expression
    static_cast<void>(v2.size());
 
    // 5. inverse of implicit conversion
    // todo
 
    // 6. array-to-pointer followed by upcast
    D a[10];
    B* dp = static_cast<B*>(a);
 
    // 7. scoped enum to int or float
    E e = E::ONE;
    int one = static_cast<int>(e);
 
    // 8. int to enum, enum to another enum
    E e2 = static_cast<E>(one);
    EU eu = static_cast<EU>(e2);
 
    // 9. pointer to member upcast
    // todo
	//std::transform(s.begin(), s.end(), s.begin(), static_cast<int(*)(int)>(std::toupper));
 
    // 10. void* to any type
    void* voidp = &e;
    std::vector<int>* p = static_cast<std::vector<int>*>(voidp);
}

output:
n = 3
v.size() = 10
after move, v.size() = 0

```

## dynamic_cast

```
#include <iostream>
 
struct V {
    virtual void f() {};  // must be polymorphic to use runtime-checked dynamic_cast
};
struct A : virtual V {};
struct B : virtual V {
  B(V* v, A* a) {
    // casts during construction
    dynamic_cast<B*>(v); // well-defined: v of type V*, V base of B, results in B*
    dynamic_cast<B*>(a); // undefined behavior: a has type A*, A not a base of B
  }
};
struct D : A, B {
    D() : B((A*)this, this) { }
};
 
struct Base {
    virtual ~Base() {}
};
 
struct Derived: Base {
    virtual void name() {}
};
 
struct Some {
    virtual ~Some() {}
};
 
int main()
{
    D d; // the most derived object
    A& a = d; // upcast, dynamic_cast may be used, but unnecessary
    D& new_d = dynamic_cast<D&>(a); // downcast
    B& new_b = dynamic_cast<B&>(a); // sidecast
 
 
    Base* b1 = new Base;
    if(Derived* d = dynamic_cast<Derived*>(b1))
    {
        std::cout << "downcast from b1 to d successful\n";
        d->name(); // safe to call
    }
 
    Base* b2 = new Derived;
    if(Derived* d = dynamic_cast<Derived*>(b2))
    {
        std::cout << "downcast from b2 to d successful\n";
        d->name(); // safe to call
    }
 
    if(Some* d = dynamic_cast<Some*>(b1))
    {
        std::cout << "downcast from b1 to Some successful\n";
        d->name(); // safe to call
    }
 
    delete b1;
    delete b2;
}

output:
downcast from b2 to d successful
```

## const_cast

```
#include <iostream>
 
struct type {
    type() :i(3) {}
    void m1(int v) const {
        // this->i = v;                 // compile error: this is a pointer to const
        const_cast<type*>(this)->i = v; // OK
    }
    int i;
};
 
int main() 
{
    int i = 3;                    // i is not declared const
    const int& cref_i = i; 
    const_cast<int&>(cref_i) = 4; // OK: modifies i
    std::cout << "i = " << i << '\n';
 
    type t;
    t.m1(4);
    std::cout << "type::i = " << t.i << '\n';
 
    const int j = 3; // j is declared const
    int* pj = const_cast<int*>(&j);
    *pj = 4;         // undefined behavior!
 
    void (type::*mfp)(int) const = &type::m1; // pointer to member function
//  const_cast<void(type::*)(int)>(mfp); // compiler error: const_cast does not
                                         // work on function pointers
}

output:
i = 4
type::i = 4
```

## reinterpret_cast

```
#include <cstdint>
#include <cassert>
#include <iostream>
int f() { return 42; }
int main()
{
    int i = 7;
 
    // pointer to integer and back
    uintptr_t v1 = reinterpret_cast<uintptr_t>(&i); // static_cast is an error
    std::cout << "The value of &i is 0x" << std::hex << v1 << '\n';
    int* p1 = reinterpret_cast<int*>(v1);
    assert(p1 == &i);
 
    // pointer to function to another and back
    void(*fp1)() = reinterpret_cast<void(*)()>(f);
    // fp1(); undefined behavior
    int(*fp2)() = reinterpret_cast<int(*)()>(fp1);
    std::cout << std::dec << fp2() << '\n'; // safe
 
    // type aliasing through pointer
    char* p2 = reinterpret_cast<char*>(&i);
    if(p2[0] == '\x7')
        std::cout << "This system is little-endian\n";
    else
        std::cout << "This system is big-endian\n";
 
    // type aliasing through reference
    reinterpret_cast<unsigned int&>(i) = 42;
    std::cout << i << '\n';
}

output:
The value of &i is 0x7fff352c3580
42
This system is little-endian
42

```


## lvalue/rvalue/prvalue/glvalue

[参考](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2010/n3055.pdf)

```
An lvalue (so-called, historically, because lvalues could appear on the left-hand side  
of an assignment expression) designates a function or an object. [Example: If E is an expression  
of pointer type, then *E is an lvalue expression referring to the object or function to which E points. 
As another example, the result of calling a function whose return type is an lvalue reference is an lvalue.]

An xvalue (an “eXpiring” value) also refers to an object, usually near the end of  
its lifetime (so that its resources may be moved, for example). An xvalue is the result  
of certain kinds of expressions involving rvalue references. [Example: The result of  
calling a function whose return type is an rvalue reference is an xvalue.]

A glvalue (“generalized” lvalue) is an lvalue or an xvalue.

An rvalue (so-called, historically, because rvalues could appear on the right-hand  
side of an assignment expression) is an xvalue, a temporary object or subobject thereof,  
or a value that is not associated with an object.

A prvalue (“pure” rvalue) is an rvalue that is not an xvalue. [Example: The result  
of calling a function whose return type is not a reference is a prvalue]

The document in question is a great reference for this question, because it shows  
the exact changes in the standard that have happened as a result of the introduction of the new nomenclature.
```

