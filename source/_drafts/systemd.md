---
title: 使用Systemd进行应用自启守护
date: 2019-07-16 14:21:10
tags:
 - Linux
---

# 简述
<!--more-->

# 执行权限
```bash
# 系统权限
sudo systemctl enable xxx
# 用户权限
systemctl --user enable xxx
# 配置所有用户
sudo systemctl --global enable xxx
```

# 运行日志
systemd 提供了自己的日志系统（logging system），称为 journal。使用 systemd 日志，无需额外安装日志服务（syslog）。使用`journalctl`即可读取日志的命令。


