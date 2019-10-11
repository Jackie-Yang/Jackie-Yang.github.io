---
title: CMake使用入门
date: 2019-02-02 15:30:51
tags:
 - CMake
---
# 概述
>CMake允许开发者编写一种平台无关的配置文件来定制整个编译流程，然后再根据目标用户的平台进一步生成所需的本地化 Makefile 和工程文件，如 Unix 的 Makefile 或 Windows 的 Visual Studio 工程，从而实现“Write once, run everywhere”

<!--more-->

# 配置CMakeLists.txt

## 常用指令

设定变量
```CMake
set(<var> <value>)
```

将源文件文件夹添加到变量
```CMake
aux_source_directory(<dir> SRC)
```

将所选源文件编译成库(默认静态库)
```CMake
add_library(<name> [STATIC | SHARED | MODULE]
            [EXCLUDE_FROM_ALL]
            [source1] [source2 ...])
#eg:
add_library(${base_lib_name} STATIC ${SRC})
```

将所选源文件编译成可执行文件
```CMake
add_executable(<name> [source...])
```

添加头文件包含路径
```CMake
include_directories(<dir>)
```

链接库
```CMake
#target 可以是被add_library或者add_executable的目标
#item 链接的库可以直接使用工程中target生成的库
target_link_libraries(<target> ... <item>... ...)
```

链接库路径
```CMake
link_directories(<dir>)
```

添加目标依赖，确保在生成目标时依赖已经构建
```Cmake
add_dependencies(target depend-target...)
```

添加定义
```CMake
add_definitions(-Dxxxx);
```

打印信息
```CMake
#错误信息
message(FATAL_ERROR "FATAL:xxxxx")

#警告信息
message(WARNING "Warning:xxxxx")
　　
#正常信息
message(STATUS "xxxx")
```

添加子文件夹，CMake会执行其中的CMakeLists.txt
```CMake
add_subdirectory(src)
```

## 常用变量

* CMAKE_SOURCE_DIR：顶层CMakeLists.txt的目录
* PROJECT_SOURCE_DIR：CMakeLists.txt定义了project信息的目录
* CMAKE_BINARY_DIR：CMake编译的目录，一般是build
* PROJECT_BINARY_DIR：项目编译目录，一般同上
* $ENV{NAME}：查看系统环境变量

# 基本编译方法
```sh
mkdir build && cd build
cmake ..
make && make install
```

# 参考文档

[CMake 入门实战](https://www.hahack.com/codes/cmake/)
[CMake官方文档](https://cmake.org/cmake/help/latest/)