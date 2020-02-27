---
title: c++函数指针
date: 2018-11-19 16:11:07
tags:
categories: c++
---
函数名到底是什么东西呢？

<!-- more -->
# 常用函数调用
```
#include <iostream>
using namespace std;

void Func(int a); // void Func(int); 声明

int main()
{
  Func(5);
  return 0;
}
void Func(int a)
{
  cout<< "a=" << a <<endl;
}
```

# 函数指针变量的声明和使用
```
#include <iostream>
using namespace std;

void Func(int a); // 或void Func(int);
void (*FuncP)(int); // 或void (*FuncP)(int a);

int main()
{
  Func(5);
  (*Func)(5); // 一般不这样写
  
  FuncP = &Func; 
  (*FuncP)(5);   
  FuncP(5);
  
  FuncP = Func; 
  FuncP(5);     
  
  
  
  return 0;
}
void Func(int a)
{
  cout<< "a=" << a <<endl;
}
```

为了书写与数学函数形式一样方便, C语言的设计者允许如下操作:
> * FuncP 和 Func是一样的，都是函数指针
> * (*Func)(5) = Func(5), 与数学函数形式一样
> * FunP函数指针变量也可以FunP(10)的形式来调用
> * 赋值时，即可FunP=&Func形式，也可FunP=MyFun

# 定义函数的指针类型
```
#include <iostream>
using namespace std;
typedef int* INTP;             // 定义一个类型
void Func(int a);              // 声明一个函数
typedef void (*FuncType)(int); // 定义一个函数指针类型
FuncType FuncP;                // 声明一个函数指针变量

int main()
{
  FuncP = Func;
  FuncP(5);
  return 0;
}
void Func(int a)
{
  cout<< "a=" << a <<endl;
}
```

# 函数指针作为参数
```
#include <iostream>
using namespace std;
       
void Func(int a);              
typedef void (*FuncType)(int); 

int int CallFunc(FuncType p, int);(FuncType p, int a);
int main()
{
  CallFunc(Func, 5);
  return 0;
}
void Func(int a)
{
  cout<< "a=" << a <<endl;
}
int CallFunc(FuncType p, int)
{
  p(a);
}
```

# 地址跳转
```
(*(void (*)(void))(0x30700000))();
```
(void (*)(void)) 转化为一个函数指针fp, 上面表达式同 (*fp)()

