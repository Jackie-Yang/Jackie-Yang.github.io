---
title: 函数适配器
date: 2019-02-20 20:33:57
tags:
 - C++
---

# 简述
函数适配器以函数作为参数，返回一个函数对象，从而更方便地进行函数调用，甚至绑定生成新的参数列表，实现更多特定情景。
<!--more-->

# 函数对象 std::function
std::function构造函数，能通过函数生成一个函数对象
```c++
bool foo(int bar);
std::function<bool(int)> func{foo};
```
但是对于成员函数，其参数隐含着this指针，则不能使用该方法。

# bind
