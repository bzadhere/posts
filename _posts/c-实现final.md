---
title: c++实现final
date: 2018-11-19 16:04:47
tags:
categories: c++
---

有时候我们希望一个类不能被继承，这种类称为final类，一个类如果有一个虚拟私有继承的基类，那么该类  
不能被继承。在C++11标准之前要实现这种技术，需要巧妙地利用一些细节
<!-- more -->
首先我们要明确以下几点：
> 类的构造函数或析析构函数声明为私有的，那么该类不能被继承，但同时该类也不能使用
> 派生类只能访问基类的公有成员和保护成员，如果是私有继承，基类中所有成员到子类中  
将成为私有的，子类的派生类也即子类的子类只能访问其直接父类的公有成员或保护成员，  
不能访问最原始基类的任何成员
> 虚继承时, 由最终子类构造基类

```
class FinalBase
{
protected:
	//FinalBase(){}
	//~FinalBase(){}
};

class Filal : virtual private FinalBase
{
public:
	Filal() { cout<<"final class."<<endl; }
};

```

Final 就是一个final类, 不能被继承。gcc4.7以前有些版本编译器需要声明保护的基类构造或析构。

在C++11标准中，引入了final关键字，实现就简单多了。

```
class Test final
{
}
```
