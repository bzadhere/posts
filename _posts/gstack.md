---
title: gstack
date: 2018-10-25 14:23:49
tags:
categories: gdb
---
```
#!bin/sh
while( true )
do 
gstack $1>>gstack_$1.txt
sleep 1
done
```
