---
title: corejava_day03
date: 2019-7-1 9:49:44
tags:
categories: java基础
---

# 流程控制
```
switch(byte/short/int/char/enum(精确存储，可以匹配))
{
 case byte/short/int/char/enum:... ... ...;break(如果进入该句，而没有break，则以下语句将执行完);
 case byte/short/int/char/enum:... ... ...;
 case x1 case x2 case x3:...;
  ... ... ...
  default :...;
}	
```
<!-- more -->
# 数组
```
一维数组
	int[] a; -->数组声明
	a = new int[5] --> 数组实例化，分配空间，5*(4 byte)，把空间首地址赋给a
	int[] a = new int[5] ; 
	int[] a =new int[5]{1,2,3,4,5} <==> int[] a = {1,2,3,4,5}
	a.length	
	ArrayIndexOutOfBoundsException --> 数组下标越界异常
	for循环赋值
	
二维数组
	int[][] a = new int[3][5] --> 规则
	int[][] a = new int[3][] --> 可以不规则 , 如
		a[0] = new int[3]
		a[1] = new int[5]
		a[2] = new int[7]
	a[i] --> 存地址，指向a[i][0].a[i][1]...的首地址
	两层for循环赋值 a.length , a[i].length
```
# 方法
方法：完成某个功能的程序的一个封装体
基本语法：
修饰符 返回值类型 方法名 ( 形参类型 形参名字 ，... ...){
	方法体
}
public(公开的) static(静态的) void(空类型) main(String[] args)

方法调用过程
	1，在栈中给被调用方法的形参分配空间
	2，将调用方法实参的值传递给形参
	3，断开原方法的执行，跳到被调用方法的执行过程
	4，直至被调用方法执行完毕，程序调回原方法继续执行
	5，如果有返回值也带回值
	6，释放被调用方法的临时空间
	
# 空间
代码空间：存放字节码
数据空间：
	栈 --> 给方法中的变量分配的临时空间，特点：先进后出
	堆 --> 给创建的对象分配的空间
栈：
	1,从代码空间装载字节码，因为要运行，最先找到主方法，在栈空间给主方法分配空间，变量分配在主方法的栈空间里
	2,如果主方法中调用了m1方法，JVM又去寻找m1方法的字节码，再在栈空间给m1方法分配空间(一样包括变量的分配)
	3,如果m1中还有调用m2方法... ... ...
	4,如果m2执行完了，释放m2空间
	5,如果m1执行完了，释放m1空间
	6,如果主方法也执行完了，释放主方法空间
堆：
	存放创建的对象(如主方法中的对象的引用，则引用变量存放在栈，创建的对象存放在堆)

# Scanner
Scanner scan = new Scanner(System.in);

scan.next(); --> return String (a word)
scan.nextLine(); --> return String (a line)
scan.nextInt(); --> return int
scan.nextFloat();
scan.nextDouble();

