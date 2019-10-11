---
title: 常见的类型转换
date: 2019-02-27 09:28:58
tags:
 - C++
---

## static_cast
任何具有明确类型之间的转换，不会进行动态类型检查，可以互相转换(void *)类型及其他类型

## dynamic_cast
主要用于类层次间的上行转换和下行转换，还可以用于类之间的交叉转换。dynamic_cast具有类型检查的功能，比static_cast更安全。

## const_cast
改变运算对象的底层const属性。

## reinterpret_cast
比较底层的类型转换，可用于不同指针类型的转换，不安全，要慎用

## C语言风格转换：(type)value

## 函数风格转换：type(value)

## 隐式转换