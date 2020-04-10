---
title: c++模版入门
date: 2019-01-30 20:09:17
tags:
 - C++
 - Template
---

# 函数模版

```c++
template <typename type>
void function(vector<type>& parm)
{
    for (int i = 0; i < parm.size(); ++i)
    {
        type t = parm[i];
        std::cout << t << std::endl;
    }
}

```
其中，type相当于占位符，相当于一个暂定的类型，在执行时才会与实际类型绑定，产生函数实例。

```c++
vector<int> var;
vector<std::string> str;
function(var);
function(str);
```

# 类模版
```c++
template<typename T>
class Vector
{
private:
    T* elem;
    int size;
public:
    explicit Vector(int s);
};
```
与函数模版一致，T只是该声明的形参，代表对所有T类型

类模版实例化：
```c++
Vector<char> vc(200);
```

# 特化
函数模板不建议特化，可以通过重载实现