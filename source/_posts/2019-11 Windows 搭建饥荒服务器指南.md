---
title: Windows 搭建饥荒服务器指南
layout: post
date: 2019-11-01 18:47:00
updated: 2026-08-20 18:46:00
comments: true
tags:
- 饥荒联机版
- Windows
- SteamCMD
- 专用服务器
- 洞穴世界
- 模组
categories:
- [游戏, 饥荒]
permalink: /2019/11/windows-dst-server/
---

> 因为有些人不习惯Linux系统，这里我专门写一章Windows版的搭建教程

> 本指南的目的是在Windows上创建带**洞穴**的饥荒服务器，非**多层世界**
> 本指南基于**Windows Server 2016**制作，理论上其他Windows也是兼容的

<!--more--> 

## 下载SteamCMD并安装服务器

为SteamCMD创建一个文件夹

例如：

`C:\steamcmd`

下载适用于Windows的[SteamCMD](https://steamcdn-a.akamaihd.net/client/installer/steamcmd.zip)

将zip的内容提取到该文件夹​​中

- 进入文件夹steamcmd
- 启动软件steamcmd
- 等待有关steamcmd的更新结束
- 输入`login anonymous`
- 输入`force_install_dir C:\DontStarveTogetherServer`
> 将在`C:\DontStarveTogetherServer`安装饥荒服务器
- 输入`app_update 343050 validate`
- 安装完成后输入`安装完成后`退出
> 以后每次更新游戏只要重复上述代码即可

## 启动两个服务器（地面和洞穴）

您需要创建两个bat脚本（`master.sh`和`caves.sh`）来分别运行两个世界

进入目录`C:\DontStarveTogetherServer\bin`

我们先创建`master.bat`，内容为

```
dontstarve_dedicated_server_nullrenderer.exe -persistent_storage_root C:\klei -conf_dir DoNotStarveTogether -cluster Cluster_1 -shard Master
```

再创建`caves.bat`，内容为

```
dontstarve_dedicated_server_nullrenderer.exe -persistent_storage_root C:\klei -conf_dir DoNotStarveTogether -cluster Cluster_1 -shard Caves
```

再创建`start.bat`，内容为

```
start master.bat
start caves.bat
```

## 运行和停止两个服务器<br/>以创建存档文件Master（地面）和Caves（洞穴）

#### 地面的存档

 双击运行`start.bat`，这是会弹出两个命令行窗口，切换到名字为Master的窗口

当你在屏幕上看到`Your Server Will Not Start`字眼时，关闭两个窗口

## 为服务器创建cluster_token.txt

在你PC上打开饥荒联机版游戏 -> 点击`开始游戏` -> 点击`账户信息` -> 点击最上面一排的`游戏` 

-> 点击`Don't Starve Together Servers` -> 拉到最下面，随便输一个名字，点击`添加新服务器`

即可获取一串以`pds-g`开头的字符串，复制

在`C:\klei\DoNotStarveTogether\Cluster_1\`下创建`cluster_token.txt`，内容为上面复制的字符串

## 为两个服务器创建leveldataoverride.lua

leveldataoverride.lua代表生成世界的设置，你可以调整季节、怪物或生物群系以及更多

### 获取leveldataoverride.lua文件配置

1. 打开Windows PC饥荒联机版游戏
2. 按照您的喜好在第一个存档位置创建您的世界（需添加洞穴）
3. 创建成功世界后退出游戏
4. 打开您电脑中的该路径`C:\Users\XXX\Documents\Klei\DoNotStarveTogether`
> `XXX`为您电脑的用户名
5. 打开该路径下的名字为数字的文件夹
6. 打开`Cluster_1`，里面会有`Master`和`Caves`文件夹，这两个文件夹里分别有一个`leveldataoverride.lua`文件

### 创建Linux上的leveldataoverride.lua

#### 地面的leveldataoverride.lua

把上面获取到的`Master`里的`leveldataoverride.lua`复制到`C:\klei\DoNotStarveTogether\Cluster_1\Master\`，如果提示文件存在就选择覆盖替换

#### 洞穴的leveldataoverride.lua

同样，把上面获取到的`Caves`里的`leveldataoverride.lua`复制到`C:\klei\DoNotStarveTogether\Cluster_1\Caves\`，如果提示文件存在就选择覆盖替换

## 创建并配置cluster.ini

编辑`C:\klei\DoNotStarveTogether\Cluster_1\`下的`cluster.ini`

内容为

``` INI
[GAMEPLAY]
game_mode = endless
max_players = 8
pvp = false
pause_when_empty = true


[NETWORK]
lan_only_cluster = false
cluster_intention = cooperative
cluster_password = 123456
cluster_name = 服务器测试
cluster_description = 一起嗨起来
offline_cluster = false
cluster_language = zh

[STEAM]
steam_group_id = 35243410
steam_group_admins = false
steam_group_only = false


[MISC]
max_snapshots = 20
console_enabled = true


[SHARD]
shard_enabled = true
bind_ip=0.0.0.0
master_ip = 127.0.0.1
master_port = 10888
cluster_key = defaultPass
```
> 注：
> game_mode 游戏模式，选项为`survival` `endless` `wilderness`，分别对应`生存` `无尽` `荒野`模式
> max_players 最大人数
> pvp 开启PVP模式，选项为`true` `false`
> cluster_intention 游戏风格，选项为`cooperative` `competitive` `social` `madness`
> cluster_password 服务器密码
> cluster_name 服务器名
> cluster_description 服务器介绍
> steam_group_id steam群组的id，如果不添加群组请留空
> steam_group_admins 群组的管理也变成服务器的管理员，选项为`true` `false`
> steam_group_only 只有群组成员才能加入，选项为`true` `false`
> max_snapshots 最大存档天数，游戏默认只保存5天的存档

## 为两个服务器创建server.ini

#### 地面的server.ini

编辑`C:\klei\DoNotStarveTogether\Cluster_1\Master`下的`server.ini`

内容为

``` INI
[NETWORK]
server_port = 11999


[SHARD]
is_master = true


[ACCOUNT]
encode_user_path = true


[STEAM]
master_server_port = 28018
authentication_port = 9768
```

#### 洞穴的server.ini

编辑`C:\klei\DoNotStarveTogether\Cluster_1\Caves`下的`server.ini`

内容为

``` INI
[NETWORK]
server_port = 12000


[SHARD]
is_master = false
name = Caves
id = 11


[ACCOUNT]
encode_user_path = true


[STEAM]
master_server_port = 28019
authentication_port = 9769
```

## 为两个服务器添加mod

还记得上文的为服务器创建`leveldataoverride.lua`的方法吧？

为服务器添加mod的方法类似

### 获取modoverrides.lua文件配置

1. 打开Windows PC饥荒联机版游戏
2. 按照您的喜好在第一个存档位置创建您的世界（需添加洞穴），这是需添加mod，提前在创意工坊订阅好
3. 创建成功世界后退出游戏
4. 打开您电脑中的该路径`C:\Users\XXX\Documents\Klei\DoNotStarveTogether`
> `XXX`为您电脑的用户名
5. 打开该路径下的名字为数字的文件夹
6. 打开`Cluster_1`，里面会有`Master`和`Caves`文件夹，这两个文件夹里分别有一个`modoverrides.lua`文件

### 创建Linux上的modoverrides.lua

#### 地面的modoverrides.lua

把上面获取到的`Master`里的`modoverrides.lua`复制到`C:\klei\DoNotStarveTogether\Cluster_1\Master\`，如果提示文件存在就选择覆盖替换

#### 洞穴的modoverrides.lua

把上面获取到的`Caves`里的`modoverrides.lua`复制到`C:\klei\DoNotStarveTogether\Cluster_1\Caves\`，如果提示文件存在就选择覆盖替换

### 修改dedicated_server_mods_setup.lua以让服务器下载相应mod

编辑`C:\DontStarveTogetherServer\mods`下的`dedicated_server_mods_setup.lua`

在里面另起一行，输入你要添加的mod，格式如下：

``` Lua
ServerModSetup("375850593")
ServerModSetup("1438233888")
ServerModSetup("850494968")
ServerModSetup("666155465")
ServerModSetup("1185229307")
ServerModSetup("1595631294")
ServerModSetup("462434129")
```

每个mod一行，双引号中间的数字为对应mod的ID

获取mod的ID的方法：

#### 方法一

打开之前获取的`modoverrides.lua`，里面的格式为

``` Lua
["workshop-375850593"]={ configuration_options={  }, enabled=true },
```

其中workshop后面的数字即为mod的ID

#### 方法二

打mod的创意工坊页面，如`https://steamcommunity.com/sharedfiles/filedetails/?id=501385076`

链接中`id=`后面的就是mod的ID

## 测试两个服务器

运行`start.bat`，服务器就启动了
分别查看两个命令行窗口

如果看到`Sim paused`，说明服务器开启成功，打开游戏看看能不能连接上吧

至此服务器就创建完毕了

## 其他

### 添加管理员

如果您用的是自己的cluster_token创建服务器，那么默认您就是游戏管理员

如果您想添加其他的玩家为管理员，在`C:\klei\DoNotStarveTogether\Cluster_1`下创建`adminlist.txt`

在里面添加对应玩家的ID，一行一个玩家

重启服务器后生效

### 添加黑名单

同理，在`C:\klei\DoNotStarveTogether\Cluster_1`下创建`blocklist.txt`

在里面添加对应玩家的ID，一行一个玩家

重启服务器后生效

> 玩家ID可在服务器日志中查看，格式为`KU_`开头

**在服务器创建的过程中如果遇到问题，欢迎在下方留言**
