---
title: shell脚本实用指令
date: 2019-05-06 15:16:29
tags:
 - shell
---


# getopts
```bash
getopts optstring name [args]
```
* optstring: 选项列表，将会逐个匹配其中的字母作为选项，若选项后面加上冒号: ，则选项必须跟额外参数，可通过OPTARG获取。
    * 细则1: 若optstring开头加上冒号: 
        * 输入选项不存在时，选项会判断为 \\? ，OPTARG为该选项。
        * 选项缺失参数时，则选项会判断为 : ，OPTARG为该选项。
    * 细则2: 若optstring开头不加冒号:
        * 选项不存在或者缺失参数都会判断为 \\? ，OPTARG应该为空（不确定不同平台的实现），getopts会输出错误提示

    * 注意: 选项\\?不要漏掉反斜杠，否则会当做通配符？处理，将合法但不在其上方case中的选项拦截，导致下方的case无法判断


* name: 匹配成功的选项
* args: 一个或多个由空格隔开的字符串，输入给getopts，若不指定该参数，getopts从命令行获取输入参数即可

*example:*
```bash
while getopts ":o:fa" OPT
do
    case "$OPT" in
       o) 
            echo "option -$OPT"
            echo "argument $OPTARG"
            ;;
       f) echo "option -$OPT" ;;
       \?) echo "Invalid option: -$OPTARG" ;;
       :) echo "option -$OPTARG requires argument" ;;
       *) echo "Warning: Option -$OPT $OPTARG not handle" ;;
    esac
done
```

```bash
./test.sh -f
option -f

./test.sh -o "Hello World"
option -o
argument Hello World

$ ./test.sh -o
option -o requires argument

./test.sh -d
Invalid option: -d

./test.sh -a
Warning: option not handle
```

```bash
# 若optstring开头不加冒号：
while getopts "o:fa" OPT
...
```

```bash
./test.sh -o
./test.sh: option requires an argument -- o
Invalid option: -

./test.sh -d
./test.sh: illegal option -- d
Invalid option: -

```