---
title: thinking in c++ reflection
date: 2019-01-08 10:31:07
tags:
categories: c++
---

# 什么是反射
Java是原生支持反射机制的，通过Class类可以通过名称获得类对象，进一步操作。Python也支持反射机制，  
可以通过globals()获取对象map，也可以通过inspect模块，提供了自省的方法。但是C++呢？C++原生不支  
持反射机制，RTTI（运行时类型识别）也仅仅提供了类型的判断。

开闭原则是设计模式的原则之一，对修改是封闭，对扩展开放。一般来说，需要我们对类进行抽象，针对抽  
象的类进行编程。许多的设计模式中，为了能够满足这一点，我们常常使用一个配置文件，映射字符串与类  
型。然后通过反射机制获得字符串对应的对象，然后自动装配已达到易于扩展的目的。

<!-- more -->
# 反射作用
> 获取类型的信息，包括属性、方法
> 动态调用方法
> 动态构造对象
> 从程序集中获得类型

# 使用场景
> 序列化（Serialization 数据写磁盘）和数据绑定（Data Binding）
> 远程方法调用（RMI）
> 对象/关系数据映射（O/R mapping）

# 实现思路
> 使用map，映射字符串和生产函数
> 每次构造新类型时，将生产函数注册到map中
> 工厂函数通过map获得生产函数，建造不同的对象

# code
```
```

# 参考

[boost](https://svn.boost.org/trac10/wiki/LibrariesUnderConstruction#ReflectiveProgramming)

[qt](https://www.bbsmax.com/A/6pdDvPyXJw/)

[Mirror C++ reflection library](http://www.open-open.com/lib/view/open1326941133530.html)

[RTTR](https://github.com/rttrorg/rttr)
