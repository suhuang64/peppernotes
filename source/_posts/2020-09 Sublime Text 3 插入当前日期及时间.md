---
title: Sublime Text 3 插入当前日期及时间
layout: post
date: 2020-09-09 00:09:04
updated: 2020-09-09 00:22:51
comments: true
tags:
- Sublime Text
- Python
- 插件开发
- 编辑器
- 时间处理
categories:
- [技术, 开发工具]
permalink: /2020/09/sublime-text-date-time/
---

Sublime Text 3 是我平时最喜欢的 Coding 工具，没有之一，可以通过 Python 来 DIY 编写很多插件

<!--more-->

打开 Sublime Text 3，点击菜单栏中 `Preferences` -> `Browser Packages...`，会弹出资源管理器，进入 `User` 文件，新建一个名为 `addCurrentTime.py` 的文件

添加以下内容：

```Python
import datetime
import sublime_plugin

class AddCurrentTimeCommand(sublime_plugin.TextCommand):
    def run(self, edit):
        self.view.run_command("insert_snippet",
            {
                "contents": "%s" % datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
            }
        )
```

在 Sublime Text 3 的菜单栏中点击 `Preferences` -> `Key Bindings`，在弹出窗口**右边栏**添加以下内容：

```JSON
[
    { "keys": ["ctrl+shift+,"], "command": "add_current_time" },
]
```

以后要输入当前日期及时间，按组合键 `Ctrl` + `Shift` + `,`即可

参考文献：

[Sublime Text 3 添加插入当前时间_sshfl_csdn的专栏-CSDN博客_sublime新建文件自动导入时间](https://blog.csdn.net/sshfl_csdn/article/details/46415551)
