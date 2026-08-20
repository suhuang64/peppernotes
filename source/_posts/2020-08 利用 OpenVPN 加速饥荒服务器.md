---
title: 利用 OpenVPN 加速饥荒服务器
layout: post
date: 2020-08-17 18:52:58
updated: 2020-08-27 17:50:02
comments: true
tags:
- OpenVPN
- VPN
- 网络加速
- 饥荒联机版
- 服务器
- Linux
categories:
- [技术, 网络]
permalink: /2020/08/openvpn
---

**郑重声明：本文仅用于搭建饥荒游戏服务器用，禁止用于任何违反国家法律的用途！**

<!--more-->

开饥荒服务器1年多以来，在服务器的开销上已有 ￥3000 多，足以我买一台物理主机来做饥荒服务器了。接下来几个月，手头上的服务器也要陆续到期了，就趁着这个机会买个物理主机搭服务器吧。

但是，物理主机有个致命的缺点就是网络，不同运营商网络的玩家或者不同地区的玩家连接延迟参差不齐。

在封锁和时光档服主小水的指点下，我摸索了一套利用 OpenVPN 加速本地物理主机的方法，让小伙伴们连接不再卡顿。

## 准备工作

- 云服务器
    云服务器是用来加速你的物理主机的，配置不重要，重要的是带宽，这决定了你的玩家数。一般1m带宽可以带4-6个玩家。
- 物理主机
- OpenVPN一键脚本
    [openvpn-install](https://github.com/Nyr/openvpn-install)

云服务器和物理主机我安装的都是 Ubuntu Server 18.04 系统。

## 云服务器部分

### 1. 安装 Git

``` bash
sudo apt update
sudo apt install git
```

### 2. 克隆 OpenVPN 一键脚本的仓库

``` bash
git clone https://github.com/Nyr/openvpn-install.git
```

### 3.运行一键脚本

执行下面三行：

``` bash
cd openvpn-install
chmod +x openvpn-install.sh
sudo ./openvpn-install.sh
```

运行脚本后会看到：

```
Welcome to this OpenVPN road warrior installer!

This server is behind NAT. What is the public IPv4 address or hostname?
Public IPv4 address / hostname [49.235.180.43]:
```

直接回车

```
Which protocol should OpenVPN use?
   1) UDP (recommended)
   2) TCP
Protocol [1]:
```

我们按默认的 UDP 模式，也直接回车

```
What port should OpenVPN listen to?
Port [1194]:
```

端口也默认，直接回车

```
Select a DNS server for the clients:
   1) Current system resolvers
   2) Google
   3) 1.1.1.1
   4) OpenDNS
   5) Quad9
   6) AdGuard
DNS server [1]:
```

DNS 也默认，直接回车

```
Enter a name for the first client:
Name [client]:
```

客户端配置文件名也默认，直接回车

```
OpenVPN installation is ready to begin.
Press any key to continue...
```

直接回车安装，看到下面的提示，说明安装完成，配置文件在 `/home/ubuntu/client.ovpn`，将它下载物理主机上

```
Finished!

The client configuration is available in: /home/ubuntu/client.ovpn
New clients can be added by running this script again.
```

## 物理主机部分

> Windows 主机的话方法很简单，官网下载 OpenVPN 客户端，傻瓜式操作

将 `client.ovpn` 放到你主文件夹内，即 `/home/用户名/`

### 1. 安装 OpenVPN 和 screen

``` bash
sudo apt update
sudo apt install openvpn screen
```

### 2. 修改配置文件

``` bash
nano ~/client.ovpn
```

在最下面添加下面三行内容

```
script-security 2
up /etc/openvpn/update-resolv-conf
down /etc/openvpn/update-resolv-conf
```

保存退出

### 2.开启 OpenVPN

``` bash
screen -S openvpn
openvpn --config ~/client.ovpn
```

然后按 `Ctrl` + `A`，再按 `D` 退出 screen 即可

## 运行饥荒服务器

这部分具体看我博客的其他文章

## 后续工作

详见 [利用 frp 进行内网穿透](/2020/08/frp)

## 参考文献

[Nyr/openvpn-install: OpenVPN road warrior installer for Ubuntu, Debian, CentOS and Fedora](https://github.com/Nyr/openvpn-install)
[Linux Connection Guide For OpenVPN Access Server | OpenVPN](https://openvpn.net/vpn-server-resources/connecting-to-access-server-with-linux/)
[Linux下OpenVPN客户端配置 | 治部少辅](https://www.codewoody.com/posts/38823/)
