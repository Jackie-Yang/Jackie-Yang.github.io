---
title: AppImage简介
tags:
 - Appimage
---

# 什么是AppImage？
AppImage 是一种把应用打包成单一文件的格式，允许在各种不同的目标系统（基础系统(Debian、RHEL等)，发行版(Ubuntu、Deepin等)）上运行，无需进一步修改。

* 可在许多发行版上运行（包括Ubuntu，Fedora，openSUSE，CentOS，basicOS，Linux Mint等）
* 一个应用程序 = 一个文件 = 对用户超级简单
* 直接运行，不需要解压或安装，不需要root权限（通常在Debian及其衍生版你需要sudo apt install来安装软件并需要输入密码）
* 基本不依赖、改变系统的依赖库
* 应用运行时挂载到临时目录，源文件不容易被篡改，比较安全

## 适用场景
不需要改变系统框架，不依赖于系统、驱动配置的应用，只需要相应的库、依赖组件集成即可运行的应用。若应用的运行需要对系统进行配置等前置条件才能运行，请考虑deb等软件包格式。

## 注意
虽然AppImage对系统环境的依赖基本可以忽略，但是仍依赖于libc库（GNU C运行时库），因此需要在libc版本较低的编译环境中进行构建，以获得更好的兼容性。

# AppImage构建


# AppImage环境使用