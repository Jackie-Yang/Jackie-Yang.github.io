---
title: Markdown基本使用
date: 2017-06-15 17:03:30
tags:
 - Markdown
---

Markdown是一种纯文本标记语言，通过简单的语法，即可实现一定的文本格式。本博客即是基于Markdown编写的。

<!--more-->

# 标题1
## 二级标题
### 三级标题

```Markdown
# 标题1
## 二级标题
### 三级标题
```




标题2
===
二级标题
-----

```Markdown
标题2
===
二级标题
-----
```


___


# 字体
* 使用*修饰加粗及斜体
* 使用_也可以，不过可能有兼容性问题*

## **加粗**
```Markdown
**加粗**
__加粗__
```

## *斜体*
```Markdown
*斜体*
_斜体_
```

## ~~删除~~
```Markdown
~~删除~~
```

___


# 引用

>引用
>>二级引用
>>>三级引用

```Markdown
>引用
>>二级引用
>>>三级引用
```


# 分割线
* 使用星号*、减号-、下划线_，中间有空格也可以
```Markdown
---
___
***
```
---
___
***

* *Tip:建议使用下划线作为分割，减号也有标题的作用，如果分割线上一行有文字，则可能会作为标题*

___

# 列表
## 无序列表（三种符号相同）

- item1
+ item2
* item3

```Markdown
- item1
+ item2
* item3
```

## 列表分级（缩进或者3个以上空格）

- item1
  + item2
    * item3

```Markdown
- item1
  + item2
    * item3
```

## 有序列表

1. item1
2. item2
3. item3
   1. item3-1

```Markdown
1. item1
2. item2
3. item3
   1. item3-1
```
___

# [超链接]()
方括号内为链接文字，圆括号内为链接，!引用图片
```Markdown
[名字](链接)
![图片名](路径)
```
eg.
```Markdown
[前往标题](#标题1)
[百度网址](http://www.baidu.com)
![图片](baidu_logo.png)
```

[前往标题](#标题1)

[百度网址](http://www.baidu.com)

![图片](baidu_logo.png)



参数链接：
```Markdown
[名字]: 链接
[调用]
```
eg.
```Markdown
[Google链接]: http://www.google.com
[Google链接]

[Google图片]: google_logo.png
![Google图片]
```


[Google链接]: http://www.google.com

[Google链接]

[Google图片]: google_logo.png
![Google图片]
___


# 表格
表头|表头|表头
:--|:-:|--:
左对齐|居中|右对齐
内容|内容|内容
内容|内容|内容

```Markdown
表头|表头|表头
:--|:-:|--:
左对齐|居中|右对齐
内容|内容|内容
内容|内容|内容
```
___


# 代码
## 行内代码段
Code: `This is code`
```Markdown
Code: `This is code`
```


## 代码块
>\`\`\`
>code
>\`\`\`


```
code
```

## 代码语法高亮

>\`\`\`c
>printf("This is c code\n");
>\`\`\`

```c
printf("This is c code\n");
```


___

