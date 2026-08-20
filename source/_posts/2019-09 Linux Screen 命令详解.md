---
title: Linux Screen 命令详解
layout: post
date: 2019-09-20 19:16:00
updated: 2026-08-20 19:44:00
comments: true
tags:
- Linux
- Screen
- SSH
- 终端
- 服务器运维
categories:
- [技术, Linux]
permalink: /2019/09/linux-screen/
---

> Linux系统管理员经常需要SSH远程登录到Linux服务器，经常运行一些需要很长时间才能完成的任务，比如系统备份、传输文件等等。<br/>
> 通常情况下我们都是为每一个这样的任务开一个远程终端窗口，因为它们执行的时间太长了，必须等待它们执行完毕，在此期间不能关掉窗口或者断开连接，否则这个任务就会被杀掉。<br/>
> 我们这里就是需要长期开饥荒服务器。

<!--more-->

## Screen介绍

- Linux screen命令用于多重视窗管理程序。
- 用户可以通过该软件同时连接多个本地或远程的命令行会话，并在其间自由切换。
- 只要Screen本身没有终止，在其内部运行的会话都可以恢复。这一点对于远程登录的用户特别有用——即使网络连接中断，用户也不会失去对已经打开的命令行会话的控制。只要再次登录到主机上执行screen -r就可以恢复会话的运行。同样在暂时离开的时候，也可以执行分离命令detach，在保证里面的程序正常运行的情况下让Screen挂起（切换到后台）。

## 语法

``` Bash
screen [-AmRvx -ls -wipe][-d <作业名称>][-h <行数>][-r <作业名称>][-s ][-S <作业名称>]
```

**参数说明**

-d <作业名称> 　将指定的screen作业离线。

-m 　即使目前已在作业中的screen作业，仍强制建立新的screen作业。

-r <作业名称> 　恢复离线的screen作业。

-S <作业名称> 　指定screen作业的名称。

-ls或--list 　显示目前所有的screen作业。

## 常用screen参数

screen -S yourname -> 新建一个叫yourname的session

screen -ls -> 列出当前所有的session

screen -r yourname -> 回到yourname这个session

screen -d yourname -> 远程detach某个session

**在每个screen session 下，所有命令都以 ctrl+a(C-a) 开始。**

C-a d -> detach，暂时离开当前session，将目前的 screen session (可能含有多个 windows) 丢到后台执行，并会回到还没进 screen 时的状态，此时在 screen session 里，每个 window 内运行的 process (无论是前台/后台)都在继续执行，即使 logout 也不影响。

C-a k -> kill window，强行关闭当前的 window

## 饥荒服务器运用screen实战

### 安装screen

```Bash
sudo apt-get install screen
```

### 用screen来启动饥荒服务器

**启动地面服务器** 

``` Bash
screen -dmS m ~/dst/bin/master.sh
```

> 释义：
> 参数`d`让screen在后台启动
> 参数`m`指创建一个新的screen
> 参数`S`指定新创建的screen的名字
> `-dmS`后面的那个m是新创建的screen的名字
>`~/dst/bin/master.sh`是在新创建的screen中运行的代码

**启动洞穴服务器** 

``` Bash
screen -dmS c ~/dst/bin/caves.sh
```

> 同理，该代码的意思是在后台创建一个名字为c的screen，并在screen中运行代码`~/dst/bin/caves.sh`

在[Linux搭建饥荒服务器指南](https://jupitersh.github.io/dst-server-guide) 这篇教程教程中，不需要执行该命令，因为我已经将该命令写入`startmaster.sh`和`startcaves.sh`这两个脚本中，并通过`launch.sh`来运行这两个脚本，详见[教程](https://jupitersh.github.io/dst-server-guide#%E5%90%AF%E5%8A%A8%E8%84%9A%E6%9C%AC)。

### 恢复后台运行的screen

饥荒服务器运行后，查看日志可以用[tail命令](https://jupitersh.github.io/dst-server-guide#%E6%B5%8B%E8%AF%95%E4%B8%A4%E4%B8%AA%E6%9C%8D%E5%8A%A1%E5%99%A8)的办法，也可以用screen命令恢复之前后台运行的两个screen。

两个screen的名字分别是m和c。

#### 恢复名字为m的screen

名字为m的screen代表地面，代码为

``` Bash
screen -dr m
```

#### 恢复名字为c的screen

名字为c的screen代表洞穴，代码为

``` Bash
screen -dr c
```

### 使当前前台的screen重新进入后台

按组合键`Ctrl`+`A`，再按`D`即可使screen进入后台
