---
title: Git子模块
tags:
 - Git
---
```bash
# 添加子模块
git submodule add <repository> [path]
```
于是，便会生成子模块


更新子模块信息
```bash
git submodule update --init --recursive
```

子模块信息是写在当前git仓库中的，若子模块有更新，则要使用git add 更新子模块版本

```bash
git add <path>
# 查看修改如下
@@ -1 +1 @@
-Subproject commit 14872c2b6f4b22d070ace58a18e35db6bdc02472
+Subproject commit d544110b702e4726db4763b61a799b5a7bcb6a82
```