---
title: corejava_day07
date: 2019-7-1 15:49:44
tags:
categories: java基础
---

# instanceof
instanceof ：判断给定的某个对象是不是某一个类型的实例
boolean : A instanceof Animal
ClassCastException 类强转异常

# static
__static修饰属性（类变量/成员变量）__
静态属性与非静态属性区别
<!-- more -->
|区别点| 静态属性| 实例态属性|
|:-|:-|:-|
| 空间分配时机| 类加载时分配| 生成对象时|
| 空间分配方式| 整个类只有一份| 同对象的数量|
| 访问方式| 类名.属性名| 对象名.属性名|

__static修饰方法（静态方法）__
1 用" 类名.方法名 "访问
2 能直接访问本类中的非静态成员, 但本类的非静态方法可以访问本类的静态成员
3 在静态方法中不能出现this关键字
4 父类中是静态方法, 子类中不能覆盖为非静态方法
5 父子类中, 父类中的静态方法可以被子类中的静态方法覆盖, 但是没有多态！（在使用对象调用静态方法时其实是调用编译时类型的静态方法）
6 java中的main方法必须写成static的原因：在类加载时无法创建对象, 而静态方法可以不通过对象调用, 所以在类加载时就可以通过main方法入口来运行程序。

__static修饰初始代码块__
这个代码块只在类加载时被执行一次, 可以用静态初始代码块初始化一个类
```
public class TestStatic {
    public static String str;

    static {
        str = "i am here";
    }

    public static void main(String[] args) {
        System.out.println(TestStatic.str);
    }
}
```
# final 
1 修饰类：类不能被继承
2 修饰属性：属性的值不可改变(外部不能改变, 所以一般定义成公开）
			final static int a = 0 ; 静态常量, 一般这里要赋初始值, 否则就没有机会赋值（只可以这样赋值或则通过构造方法赋值, 而通过构造方法给静态属性赋值没有意义）, 代表整个类的特征
			final int a ; 非静态常量, 一般不赋初值, 通过构造方法赋值, 每创建一个对象就有这么一个常量, 属于对象的不可改变的特征 
3 修饰方法：方法不能被覆盖
4 修饰局部变量(or 形参)：值不可变, 为常量

# abstract
1 修饰类：抽象类 --> 有构造方法, 给子类调用, 但不能实例化 , 必须有子类！
2 修饰方法：抽象方法(只有方法的声明没有方法的实现)
有抽象方法的类必须是抽象类, 抽象类未必有抽象方法

# 接口(interface,implements)
将服务的提供者和服务的使用者分开
1, 接口的所有方法必须是抽象方法abstract, 被隐式的指定为 public abstract
2, 接口的所有属性都默认是静态常量static final
3, 接口不能实例化, 没有构造方法
4, 一个类可以同时实现多个接口
5, 一个接口可以同时继承自多个接口(普通类只能单继承)

接口编程的好处：
   1）降低系统的耦合度；
       六字真言：高内聚, 低耦合；
       内聚：一个类独立完成某项功能的能力；
       耦合：类和类之间, 模块与模块之间关联关系的复杂度；一个类改了不影响别的类；
   2）将标准的制定者和标准的实现者分离；
   3）接口应该尽量简单和单一；
   4）基于接口的编程（基于抽象的编程）；
