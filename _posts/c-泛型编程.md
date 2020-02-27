---
title: c++泛型编程
date: 2019-06-11 17:15:21
tags:
categories: c++
---

泛型编程就是与类型无关的编程, 可以提高软件重用
# 函数模板
关键字
template &lt; class T &gt;
template &lt; typename &gt;

```
#include <iostream>
using namespace std;

template <class T>
inline T square(T x)
{
   T result;
   result = x * x;
   return result;
};

template <typename T, typename U>
void squareAndPrint(T x, U y)
{
   T result;
   U otherVar;
   cout << "X: " << x << " " <<  x * x << endl;
   cout << "Y: " << y << " " <<  y * y << endl;
};

template <typename T, int count>
void loopIt(T x)
{
   T val[count];

   for(int ii=0; ii<count; ii++)
   { 
       val[ii] = x++;
       cout <<  val[ii] << endl;
   }
};

template <typename T=float, int count=3>
T multIt(T x)
{
   for(int ii=0; ii<count; ii++)
   {
       x = x * x;
   }
   return x;
};

// template specialization
template <>
string square<string>(string ss)
{
   return (ss+ss);
};

main()
{
   int    i, ii;
   float  x, xx;
   double y, yy;

   i = 2;
   x = 2.2;
   y = 2.2;

   ii = square<int>(i);
   cout << i << ": " << ii << endl;

   xx = square<float>(x);
   cout << x << ": " << xx << endl;

   // Explicit use of template
   yy = square<double>(y);
   cout << y << ": " << yy << endl;

   // Implicit use of template
   yy = square(y);
   cout << y << ": " << yy << endl;
}
```

# 类模板
```
// Matrix2x2.hpp----
#ifndef MATRIX_2X2_HPP__
#define MATRIX_2X2_HPP__

using namespace std;

/**
    m(11)  m(12)
    m(21)  m(22)
*/

template <class T>
class Matrix2x2
{
public:
   Matrix2x2(T m11, T m12, T m21, T m22);    //constructor
   Matrix2x2(T m[2][2]);
   Matrix2x2();

   int Add(Matrix2x2 x)
   int Multiply(Matrix2x2 x)
   void Print();
   T m[2][2];
};

template <class T>
Matrix2x2<T>::Matrix2x2(T _m11, T _m12, T _m21, T _m22)
{
   m[0][0] = _m11;
   m[0][1] = _m12;
   m[1][0] = _m21;
   m[1][1] = _m22;
}

template <class T>
Matrix2x2<T>::Matrix2x2(T _m)
{
   m[0][0] = _m[0][0];
   m[0][1] = _m[0][1];
   m[1][0] = _m[1][0];
   m[1][1] = _m[1][1];
}

template <class T>
Matrix2x2<T>::Matrix2x2()
{
   m[0][0] = 0;
   m[0][1] = 0;
   m[1][0] = 0;
   m[1][1] = 0;
}
.......
#endif

// file2.cpp----
#include <iostream>

#include "Matrix2x2.hpp"

using namespace std;

int main(int argc, char* argv[])
{
    Matrix2x2<int> X(1,2,3,4);
    Matrix2x2<int> Y(5,6,7,8);

    cout << "X:" << endl;
    X.Print();
}
```

## 静态成员
```
#include <iostream>

using namespace std;

template <class T> 
class XYZ
{
public:
    void putPri();
    static T ipub;
private:
    static T ipri;
};

template <class T> 
void XYZ<T>::putPri()
{
    cout << ipri++ << endl;
}

// Static variable initialization:
template <class T> T XYZ<T>::ipub = 1;
template <class T> T XYZ<T>::ipri = 1.2;

main()
{
    XYZ<int> aaa;
    XYZ<float> bbb;

    aaa.putPri();
    cout << aaa.ipub << endl;
    bbb.putPri();
}
```

## 继承
```
class Clor
{};

// 泛型类继承非泛型类
template <typename T>
class Circle : public Color
{};

// 非泛型类继承泛型类
class Sphere : public Circle<float>
{};

// 泛型类继承泛型类
template <typename T>
class Sphere : public Circle<T>
{};

```

# 模板参数
```

```

# 全特化和偏特化
```

```

# 类型萃取

# 模板的分离编译
因为编译和连接是分开的, 将声明和定义单独编译, 链接会出错; 所以声明和定义放在头文件中.



