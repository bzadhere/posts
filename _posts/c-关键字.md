---
title: c++关键字
date: 2018-11-19 16:11:48
tags:
categories: c++
---

# volatile
> 阻止编译器为了提高速度将一个变量缓存到寄存器内而不写回
> 阻止编译器操作volatile变量的指令顺序

```
volatile int i=10; 
int a = i; 
... 
//其他代码，并未明确告诉编译器，对i进行过操作 
//编译器发现两次从i读数据的代码之间的代码没有对i进行过操作，它会自动把上次读的数据放在b中
int b = i; 
```
<!-- more -->

# inline
对于内联函数, 编译器在符号表里方式函数声明(名字, 参数类型, 返回值类型),
如果编译器没发现错误, 函数代码也被放入符号表里。在调用一个内联函数时, 编译器如果 
检查类型正确, 内联函数代码会直接替换函数调用(宏替换也是一样), 省去了函数调用开销。 

inline 是一种用来实现的关键字, 不是用来声明的关键字; 用户也不要知道函数是否内联, 推荐
风格如下
```
//头文件
class A
{
public:
  void Foo(int i);
};

// 实现文件
inline void A::Foo(int i)
{
  ......
}
```

# explicit
> 防止由构造函数定义的隐式转换

```
class things
{
public:
	explicit things(const std::string&name =""):
		  m_name(name),height(0),weight(0){}
	int CompareTo(const things & other);
	std::string m_name;
	int height;
	int weight;
};

things a;
std::string nm ="book_1";
int result = a.CompareTo(things(nm));//必须显示使用构造函数
```

# extern
> * 被 extern 修饰函数或变量(不能和static同时修饰), 在本模块内全局可见
> * 被 extern "C" 修饰的变量和函数是按照 C 语言方式编译和连接的

```
//file1.c:
    int x=1;
    int f(){do something here}
//file2.c:
    extern int x;
    int f();
    void g(){x=f();}
```

C语言调用C++
```
#ifdef __cplusplus
extern "C" {
#endif

NAMESPACE_BILLING40_FRAME CThread* create_instanceRating();

#ifdef __cplusplus
}
#endif

// C的代码文件c.c中调用c++函数
extern NAMESPACE_BILLING40_FRAME CThread* create_instanceRating();
int main(int argc,char** argv)
{
    print(3);
    return 0;
}


```

C++调用C语言
```
// cHeader.h
#ifndef C_HEADER
#define C_HEADER
 
extern void print(int i);
 
#endif C_HEADER

// cHeader.c
#include <stdio.h>
#include "cHeader.h"
void print(int i)
{
    printf("cHeader %d\n",i);
}

// c++的*.cpp文件中调用
extern "C"{
#include "cHeader.h"
}
 
int main(int argc,char** argv)
{
    print(3);
    return 0;
}

```

# typedef
> 定义类型别名

```
typedef int* PINT; // 一般用大写
PINT pa, pb; // 同时声明了两个指向int变量的指针
typedef float REAL;  // 跨平台编译

typedef struct tagPOINT
{
int x;
int y;
}SPOINT;
SPOINT sa; // 省略了一个struct

int mystrcmp(const pstr, const pstr); 
typedef int (*PF) (const char *, const char *);
PF p = mystrcmp;
```

# const
```
const int *A;       // 修饰指向的对象，A可变，A指向的对象不可变
int const *A; 　    // 修饰指向的对象，A可变，A指向的对象不可变
int *const A; 　    // 修饰指针A，     A不可变，A指向的对象可变 
const int *const A; // 指针A和A指向的对象都不可变
```

# virtual
> 虚表存放的位置一般存放在模块的常量段中，从始至终都只有一份?

和编译器有关, 在gcc编译器的实现中虚函数表vtable存放在可执行文件的只读数据段.rodata中

[问题参考1](https://juejin.im/entry/58576d9c570c3500690b1211)
[问题参考2](https://www.cnblogs.com/chenhuan001/p/6485233.html)


__重载、覆盖、隐藏__
成员函数被重载的特征：（同名不同参）
（1）相同的范围（在同一个类中）；
（2）函数名字相同；
（3）参数不同；
（4）virtual 关键字可有可无。
覆盖是指派生类函数覆盖基类函数，特征是：(虚函数)
（1）不同的范围（分别位于派生类与基类）；
（2）函数名字相同；
（3）参数相同；
（4）基类函数必须有virtual 关键

“隐藏”是指派生类的函数屏蔽了与其同名的基类函数，规则如下：(behavior depends on type of the pointer)
（1）如果派生类的函数与基类的函数同名，但是参数不同。此时，不论有无virtual
关键字，基类的函数将被隐藏（注意别与重载混淆）
（2）如果派生类的函数与基类的函数同名，并且参数也相同，但是基类函数没有virtual
关键字。此时，基类的函数被隐藏（注意别与覆盖混淆）

# assert
void print_number(int* myInt) {
  assert (myInt!=NULL); // 条件表达式为假就退出
  printf ("%d\n",*myInt);
}

# using
1.命名空间
2.继承时, 在子类中使用基类成员
3.指定别名
```
template<typename _Tp, typename _Alloc = std::allocator<_Tp> >
class vector : protected _Vector_base<_Tp, _Alloc>
{
protected:
  using _Base::_M_allocate;
  using _Base::_M_deallocate;
  using _Base::_M_impl;
  using _Base::_M_get_Tp_allocator;
};

template <typename T>
using Vec = MyVector<T, MyAlloc<T>>;
 
// usage
Vec<int> vec;
```
# new
new: 关键字, 先分配内存, 再构造对象
operate new: 只能在类中重载
placement new: 是operator new的一个重载版本, 预先分配好内存中构造对象, 必须显示调用对象析构函数
```
void *operator new( size_t, void *p ) throw()     { return p; }

Widget * p = new Widget; //ordinary new 
pi = new (ptr) int;     //placement new, ptr 指向内存

// --------------
class Test
{
public:
Test(int i): m_i(i) {}
~Test(){cout<<"~Test."<<endl;}
public:
  int m_i;
};

int main()
{
  int* p = new int[1024];
  cout << "p=" << p << endl;

  Test* ptr = new(p) Test(5);
  cout<< "ptr->m_i=" << ptr->m_i << endl;
  cout<< "ptr=" << ptr<< endl;

  ptr = new(p+sizeof(Test)) Test(6);
  cout<< "ptr->m_i=" << ptr->m_i << endl;
  cout<< "ptr=" << ptr<< endl;
  delete[] p;
}
// output, 没有调用~Test.
p=0x1632560
ptr->m_i=5
ptr=0x1632560
ptr->m_i=6
ptr=0x1632560
```

# define/undef

# typename
模板类型在实例化之前, 有三种可能 静态数据成员/静态成员函数/嵌套类型。
typedef创建了存在类型的别名，而typename告诉编译器 std::vector<T>::size_type 是一个类型而不是一个成员。

```
typedef typename std::vector<T>::size_type size_type;
```
