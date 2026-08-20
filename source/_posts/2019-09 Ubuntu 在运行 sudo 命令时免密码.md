---
title: Ubuntu 在运行 sudo 命令时免密码
layout: post
date: 2019-09-22 17:33:00
updated: 2028-08-20 19:52:00
comments: true
tags:
- Linux
- Ubuntu
- sudo
- 权限管理
- 系统管理
categories:
- [技术, Linux]
permalink: /2019/09/ubuntu-sudo-no-password/
---

> 本教程基于Ubuntu 18.04 64位，理论上使用于所有Ubuntu系统

<!--more-->

觉得每次超过五分钟后运行sudo命令时都要输入密码很烦吗？

解决办法很简单

输入

``` Bash
sudo visudo
```

到最下面

找到一行`#includedir /etc/sudoers.d`

在其下添加一行

```
ubuntu  ALL=(ALL:ALL) NOPASSWD: ALL
```

> 我的用户名是`ubuntu`，将其替换成您电脑的用户名即可

按组合键`Ctrl`+`X`，再按`Y`，`回车`退出即可
