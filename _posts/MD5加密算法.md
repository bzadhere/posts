---
title: MD5加密算法
date: 2018-12-01 10:12:34
tags:
categories: 数据结构和算法
---

# 什么是MD5加密算法
MD5消息摘要算法是一种被广泛使用的密码散列函数, 属Hash算法一类, MD5算法对输入  
任意长度的消息进行运行，产生一个128位(16字节)的散列值(hash value)。
<!-- more -->
# 应用场景
## 文件一致性验证
MD5的典型应用是对一段信息（Message）产生信息摘要（Message-Digest），以防止被篡改。  

```
51_zjdev[/data01/zjgrp/zjdev/users/zhangbb]%md5sum test.cpp 
731b8735653acd4c4cdbd05883fe90d1  test.cpp
```

## 数字签名
MD5的典型应用是对一段Message(字节串)产生fingerprint(指纹），以防止被“篡改”。

# 算法原理
以下所描述的消息长度、填充数据都以位(Bit)为单位，字节序为小端字节。
## 数据填充
对消息进行数据填充，使消息的长度对512取模得448，设消息长度为X，即满足X mod 512=448。  
根据此公式得出需要填充的数据长度。  
填充方法：在消息后面进行填充，填充第一位为1，其余为0。

## 添加消息长度
在第一步结果之后再填充上原消息的长度，可用来进行的存储长度为64位。如果消息长度大于264，  
则只使用其低64位的值，即（消息长度 对 264取模）。  
在此步骤进行完毕后，最终消息长度就是512的整数倍

## 数据处理
准备需要用到的数据：

> * 4个常数： A = 0x67452301, B = 0x0EFCDAB89, C = 0x98BADCFE, D = 0x10325476;
> * 4个函数：F(X,Y,Z)=(X & Y) | ((~X) & Z); G(X,Y,Z)=(X & Z) | (Y & (~Z));  H(X,Y,Z)=X ^ Y ^ Z; I(X,Y,Z)=Y ^ (X | (~Z));

把消息分以512位为一分组进行处理，每一个分组进行4轮变换，以上面所说4个常数为起始变量进行　　
计算，重新输出4个变量，以这4个变量再进行下一分组的运算，如果已经是最后一个分组，则这4个　　
变量为最后的结果，即MD5值。

具体计算的实现较为复杂，建议查阅相关书籍，下面给出在C++上的实现代码。
## 代码实现
```
#ifndef MD5H
#define MD5H
#include <math.h>
#include <Windows.h>

void ROL(unsigned int &s, unsigned short cx); //32位数循环左移实现函数
void ltob(unsigned int &i); //B\L互转，接受UINT类型
unsigned int* MD5(const char* mStr); //接口函数，并执行数据填充，计算MD5时调用此函数

#endif

#include "MD5.h"

/*4组计算函数*/
inline unsigned int F(unsigned int X, unsigned int Y, unsigned int Z)
{
    return (X & Y) | ((~X) & Z);
}
inline unsigned int G(unsigned int X, unsigned int Y, unsigned int Z)
{
    return (X & Z) | (Y & (~Z));
}
inline unsigned int H(unsigned int X, unsigned int Y, unsigned int Z)
{
    return X ^ Y ^ Z;
}
inline unsigned int I(unsigned int X, unsigned int Y, unsigned int Z)
{
    return Y ^ (X | (~Z));
}
/*4组计算函数结束*/

/*32位数循环左移实现函数*/
void ROL(unsigned int &s, unsigned short cx)
{
    if (cx > 32)cx %= 32;
    s = (s << cx) | (s >> (32 - cx));
    return;
}

/*B\L互转，接收UINT类型*/
void ltob(unsigned int &i)
{
    unsigned int tmp = i;//保存副本
    byte *psour = (byte*)&tmp, *pdes = (byte*)&i;
    pdes += 3;//调整指针，准备左右调转
    for (short i = 3; i >= 0; --i)
    {
        CopyMemory(pdes - i, psour + i, 1);
    }
    return;
}

/*
MD5循环计算函数，label=第几轮循环（1<=label<=4），lGroup数组=4个种子副本，M=数据（16组32位数指针）
种子数组排列方式: --A--D--C--B--，即 lGroup[0]=A; lGroup[1]=D; lGroup[2]=C; lGroup[3]=B;
*/
void AccLoop(unsigned short label, unsigned int *lGroup, void *M)
{
    unsigned int *i1, *i2, *i3, *i4, TAcc, tmpi = 0; //定义:4个指针； T表累加器； 局部变量
    typedef unsigned int(*clac)(unsigned int X, unsigned int Y, unsigned int Z); //定义函数类型
    const unsigned int rolarray[4][4] = {
        { 7, 12, 17, 22 },
        { 5, 9, 14, 20 },
        { 4, 11, 16, 23 },
        { 6, 10, 15, 21 }
    };//循环左移-位数表
    const unsigned short mN[4][16] = {
        { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15 },
        { 1, 6, 11, 0, 5, 10, 15, 4, 9, 14, 3, 8, 13, 2, 7, 12 },
        { 5, 8, 11, 14, 1, 4, 7, 10, 13, 0, 3, 6, 9, 12, 15, 2 },
        { 0, 7, 14, 5, 12, 3, 10, 1, 8, 15, 6, 13, 4, 11, 2, 9 }
    };//数据坐标表
    const unsigned int *pM = static_cast<unsigned int*>(M);//转换类型为32位的Uint
    TAcc = ((label - 1) * 16) + 1; //根据第几轮循环初始化T表累加器
    clac clacArr[4] = { F, G, H, I }; //定义并初始化计算函数指针数组

    /*一轮循环开始（16组->16次）*/
    for (short i = 0; i < 16; ++i)
    {
        /*进行指针自变换*/
        i1 = lGroup + ((0 + i) % 4);
        i2 = lGroup + ((3 + i) % 4);
        i3 = lGroup + ((2 + i) % 4);
        i4 = lGroup + ((1 + i) % 4);

        /*第一步计算开始: A+F(B,C,D)+M[i]+T[i+1] 注:第一步中直接计算T表*/
        tmpi = (*i1 + clacArr[label - 1](*i2, *i3, *i4) + pM[(mN[label - 1][i])] + (unsigned int)(0x100000000UL * abs(sin((double)(TAcc + i)))));
        ROL(tmpi, rolarray[label - 1][i % 4]);//第二步:循环左移
        *i1 = *i2 + tmpi;//第三步:相加并赋值到种子
    }
    return;
}

/*接口函数，并执行数据填充*/
unsigned int* MD5(const char* mStr)
{
    unsigned int mLen = strlen(mStr); //计算字符串长度
    if (mLen < 0) return 0;
    unsigned int FillSize = 448 - ((mLen * 8) % 512); //计算需填充的bit数
    unsigned int FSbyte = FillSize / 8; //以字节表示的填充数
    unsigned int BuffLen = mLen + 8 + FSbyte; //缓冲区长度或者说填充后的长度
    unsigned char *md5Buff = new unsigned char[BuffLen]; //分配缓冲区
    CopyMemory(md5Buff, mStr, mLen); //复制字符串到缓冲区

    /*数据填充开始*/
    md5Buff[mLen] = 0x80; //第一个bit填充1
    ZeroMemory(&md5Buff[mLen + 1], FSbyte - 1); //其它bit填充0，另一可用函数为FillMemory
    unsigned long long lenBit = mLen * 8ULL; //计算字符串长度，准备填充
    CopyMemory(&md5Buff[mLen + FSbyte], &lenBit, 8); //填充长度
    /*数据填充结束*/

    /*运算开始*/
    unsigned int LoopNumber = BuffLen / 64; //以16个字为一分组，计算分组数量
    unsigned int A = 0x67452301, B = 0x0EFCDAB89, C = 0x98BADCFE, D = 0x10325476;//初始4个种子，小端类型
    unsigned int *lGroup = new unsigned int[4]{ A, D, C, B}; //种子副本数组,并作为返回值返回
    for (unsigned int Bcount = 0; Bcount < LoopNumber; ++Bcount) //分组大循环开始
    {
        /*进入4次计算的小循环*/
        for (unsigned short Lcount = 0; Lcount < 4;)
        {
            AccLoop(++Lcount, lGroup, &md5Buff[Bcount * 64]);
        }
        /*数据相加作为下一轮的种子或者最终输出*/
        A = (lGroup[0] += A);
        B = (lGroup[3] += B);
        C = (lGroup[2] += C);
        D = (lGroup[1] += D);
    }
    /*转换内存中的布局后才能正常显示*/
    ltob(lGroup[0]);
    ltob(lGroup[1]);
    ltob(lGroup[2]);
    ltob(lGroup[3]);
    delete[] md5Buff; //清除内存并返回
    return lGroup;
}
```

```
#include <iostream>
#include <string.h>
#include <stdlib.h>
#include "MD5.h"

int main(int argc, char **argv)
{
    char tmpstr[256], buf[4][10];
    std::cout << "请输入要加密的字符串：";
    std::cin >> tmpstr;
    unsigned int* tmpGroup = MD5(tmpstr);
    sprintf_s(buf[0], "%8X", tmpGroup[0]);
    sprintf_s(buf[1], "%8X", tmpGroup[3]);
    sprintf_s(buf[2], "%8X", tmpGroup[2]);
    sprintf_s(buf[3], "%8X", tmpGroup[1]);
    std::cout <<"MD5:"<< buf[0] << buf[1] << buf[2] << buf[3] << std::endl;
    
    delete[] tmpGroup;
    return 0; //在此下断点才能看到输出的值
}
```



