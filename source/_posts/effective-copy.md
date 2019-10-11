---
title: Effective C++笔记：赋值及拷贝
date: 2019-03-23 09:22:33
tags:
 - C++
---
拷贝构造函数和赋值操作符，统称为copying函数。编译器会在必要的时候替我们创建默认的版本：将对象的所有成员做一份拷贝（memwise)。如果用户声明自己的copying函数，编译器便不会进行干预，因此要确保资源被正确拷贝。

<!--more-->
# 条款 10. 令operator= 返回一个 reference to *this

关于赋值，你可以写成以下连锁的形式：
```c++
int a, b, c;
a = b = c = 8;
```
由于赋值运算符采用右结合律，上述操作可被解析为：
```c++
a = (b = (c = 8));
```
若赋值表达式返回一个值，赋值能正确执行

但是对于以下形式（虽然一般不会这么写）：
```c++
int a;
int b = 2;
int c = 3;
(a = b) = c;
```
若赋值表达式以值的形式返回，则对a的第二次赋值是无效的。

因此，对于赋值表达式，大家约定以引用的形式返回，大家共同遵守。即使不遵循它并不会导致编译错误。

# 条款 11. 在operator= 中处理 “自我赋值”

说到自我复制，大家可能想到的是这样的情景；
```c++
class Widget {
    ...
    Widget &operator=(const Widget &wdg);
    ...
};

Widget w;
... 
w = w;
```

看起来很蠢，但是是可行的，在更多情况下，自我赋值会以以下形式出现。
```c++
widgets[i] = widgets[j];
*p_widget1 = *p_widget2;
```
以上都是“别名”带来的结果，如果代码涉及指针或者引用，也可能是指向的对象是同一继承体系的，就要考虑指向的的对象是否为同一个。

## 自我赋值安全
若你的对象自行进行资源的分配，则需要考虑在自我赋值的时候，是否会出现错误释放资源的情况。
以下例子就不具备自我赋值安全性
```c++
class Widget {
public:
    ... 
    Widget &operator=(const Widget &wdg) {
        delete pBitmap;
        pBitmap = new Bitmap(*wdg.Bitmap);
        return *this;
    }

private:
    Bitmap *pBitmap;
} 

```

这样的情况下，Bitmap就会在自我复制前被销毁，防止这种异常的方法便是加入`证同测试`：
```c++
if(this == &wdg) {
    return *this;
}
```
## 异常安全
另一个潜在的问题是，这样的做法不具备`异常安全性`，当在new Bitmap的时候出现异常，指针将成为野指针，保证异常安全的一个承诺是：`不允许数据败坏`，因此在成功复制数据之前，不要删除原有数据:
```c++
Bitmap *pOrig = pBitmap;
pBitmap = new Bitmap(*wdg.Bitmap);
delete pOrig;
return *this;
```
可见，这种保证`异常安全`的方法，也同时具备`自我赋值安全`，而不需要进行`证同测试`。

## Copy and Swap
一种替代这种精心排列的方法是`Copy and Swap`，这是一种常见且安全的赋值方法，
```c++
class Widget {
    ...
    void swap (Widget& wdg)
    {
        ... //交换*this和wdg数据
    }

    Widget& operator=(const Widget& wdg)
    {
        Widget temp(wdg);   //假设已经实现拷贝构造
        swap(temp);
        return *this;
    }
    ...
}
```

# 条款 12. 复制对象的每一部分

若用户自行定义拷贝构造，赋值函数，编译器并不会检查用户是否复制了对象的每一部分，因此当你添加了成员变量，记得同时修改copying函数。

然而一旦出现继承，便会有另一个隐藏的风险:`自定义copying函数不会为你复制基类成员`。对于拷贝构造函数，如果基类没有不带参的构造函数，则编译器会报错提示基类部分无法构造，否则，基类部分会执行默认构造，不会进行拷贝。因此，你需要手动调用基类的构造函数：

```c++
class Button : public Widget {
    ...
    Button(const Button &button) : Widget(button){
        ...
    };

    Button &operator=(const Button &button) {
        Widget::operator=(button); 
    ...
    }
...
};
```

* 不要尝试拷贝构造及赋值函数的互相调用，对于拷贝构造，对象并没有初始化完成，调用赋值函数没有意义，而对于赋值函数，去调用拷贝构造构造已经完成初始化的对象是很荒谬的。

