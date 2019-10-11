---
title: C++包管理器：Conan
date: 2019-02-16 15:20:21
tags:
 - Conan
---
```bash
查看本地库
conan search 
conan search Poco* --remote=conan-cente
conan search xxxx --remote=xxxx
查看库信息
conan inspect xxxx
安装库
conan install zlib/1.2.8@lasote/stable
删除库
conan remove zlib/1.2.11@conan/stable
```

2. 配置conanfile.txt
```
[requires]
Poco/1.9.0@pocoproject/stable

[generators]
cmake
 ```

3. conan install
```bash
$ mkdir build && cd build
$ conan install ..
```

4. CMakeLists.txt
```cmake
conan_basic_setup()
target_link_libraries(target ${CONAN_LIBS})
```

5. build
* **win**
```powershell
$ cmake .. -G "Visual Studio 15 Win64"
$ cmake --build . --config Release
```
* **linux, mac**
```bash
$ cmake .. -G "Unix Makefiles" -DCMAKE_BUILD_TYPE=Release
$ cmake --build .
```
# 参考文档
[conan 官方文档](http://docs.conan.io/en/latest/)