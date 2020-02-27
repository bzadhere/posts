---
title: corejava_day09
date: 2019-7-2 9:45:44
tags:
categories: java基础
---

# 访问控制修饰符
public			所有可见
protected		同包及子类可见
Default			同包可见
private			仅当前类可见

# Object类中的方法
<!-- more -->
|方法|返回值|说明|
|:-|:-|:-|
| clone()| 对象拷贝| Animal b = a；只是地址赋值；实现对象拷贝：在类中覆盖clone方法，修饰符改为public，并实现Cloneable接口|
| equals()| boolenan| 实现就是"=="|
| finalize()| 资源释放| 垃圾回收前调用这个方法|
| toString()| String| Object中输出包名+类名+@+16进制地址|

# String
对象池（堆、栈空间之外，属于公共空间，由JVM自动管理和维护）, 对象池中存放字符串值（堆中）的引用

String str1 = "hello"; =>  会在对象池中找"hello"的引用
String str2 = new String("hello"); =>  不会在对象池中找，而且也不会存入对象池，该语句创建了两个对象，一个参数"hello"所代表的字符串对象(会在对象池中找)，一个new出来的字符串对象
String s1 = "he"+"llo"; =>编译器先把这条语句优化成String s1 = "hello";
String s2 = "he";
String s3 = s2+"llo";=>编译器认为相加运算中有变量，不会优化语句，故先得到s2，再得到"llo"，再组合这两个字符串成一个新字符串对象
s1==s3 =>false 不是同一个对象
s1==str1=>true
s3==str1=>false

字符串对象的值不可改变
正是由于这种不可改变的特征，如果对于一个具有String类型属性的对象作一个浅拷贝，相当于深拷贝
c++中是写时拷贝

# StringBuffer
线程安全，值可以被修改，默认初始字符长度为16个字符的空间，自动扩容

# StringBuilder
线程不安全，但效率高了

# 静态导入
要使用静态成员（方法和变量）我们必须给出提供这个静态成员的类。
使用静态导入可以使被导入类的静态变量和静态方法在当前类直接可见，使用这些静态成员无需再给出他们的类名。

import java.util.Scanner => Scanner()
import static java.lang.System.out; => out.println()
out为System的静态成员

# Integer
Integer.paresInt("12",8) => 12是八进制的，输出十进制int值 =>10
Integer.toString(10,8) => 10是10进制的，输出八进制字符串=>"12"
Integer.valueOf("12",8) => 12是8进制的，输出十进制Integer对象=>10

自动封箱=>Integer  i =0 <==> Integer i = new Integer(0);
自动解封=>int j = i <==> int j = i.intValue();
相当于类型隐式转换

Integer也有对象池，同String的理解，值也不可改变
Integer的对象池只能存放-128~127的对象，超出范围的只能重新创建对象，不会再到对象池中找
Integer i = 0 ; => 对象池中找
Integer i = Integer.valueOf(0); => 对象池中找
Integer i =new Integer(0); => 创建新对象

# 类型安全的枚举
1)类型不安全
public static final int SPRING = 1;
public static final int SUMMER = 2;
public static final int AUTUMN = 3;
public static final int WINTER = 4;

switch(int){根据int值判断返回值
	case Season.SPRING:return "Spring";
	case Season.SUMMER:return "Summer";
	case Season.AUTUMN:return "Autumn";
	case Season.WINTER:return "Winter";
}

2)外部调用类型安全
class Season{
    private Season(){}
    public static final Season SPRING = new Season();
}


3)枚举
继承自 java.lang.Enum 类, 枚举成员默认都被 final、public, static 修饰，使用时直接使用枚举名称调用成员即可;
枚举的构造方法是私有的，可以添加方法

| 常用方法|  描述|
|:-|:-|
| values()| 以数组形式返回枚举类型的所有成员|
| valueOf()| 将普通字符串转换为枚举实例|
| compareTo()| 比较两个枚举成员在定义时的顺序|
| ordinal()| 获取枚举成员的索引位置|

java.util 中添加了两个新类：EnumMap 和 EnumSet
EnumMap 使用数组来存放与枚举类型对应的值，使得 EnumMap 的效率非常高(比HashMap高)
EnumSet 是枚举类型的高性能 Set 实现，它要求放入它的枚举常量必须属于同一枚举类型
```
//定义数据库类型枚举
public enum DataBaseType
{
    MYSQUORACLE,DB2,SQLSERVER
}
//某类中定义的获取数据库URL的方法以及EnumMap的声明
private EnumMap<DataBaseType,String>urls=new EnumMap<DataBaseType,String>(DataBaseType.class);
public DataBaseInfo()
{
    urls.put(DataBaseType.DB2,"jdbc:db2://localhost:5000/sample");
    urls.put(DataBaseType.MYSQL,"jdbc:mysql://localhost/mydb");
    urls.put(DataBaseType.ORACLE,"jdbc:oracle:thin:@localhost:1521:sample");
    urls.put(DataBaseType.SQLSERVER,"jdbc:microsoft:sqlserver://sql:1433;Database=mydb");
}
//根据不同的数据库类型，返回对应的URL
//@param type DataBaseType 枚举类新实例
//@return
public String getURL(DataBaseType type)
{
    return this.urls.get(type);
}
```






