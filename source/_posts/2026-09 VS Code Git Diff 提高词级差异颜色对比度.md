---
title: VS Code Git Diff：提高词级差异颜色对比度
layout: post
date: 2026-09-02 00:00:00
updated: 2026-09-02 00:00:00
comments: true
tags:
- VS Code
- Git
- 开发工具
- 代码审查
categories:
- [技术, 开发工具]
permalink: /2026/09/vscode-git-diff-contrast/
---

在 VS Code 中查看 Git Diff 时，修改过的整行通常会显示为浅红色或浅绿色，而行内具体被修改的单词还会使用另一层背景色标记。

如果主题颜色比较柔和，词级背景色与整行背景色的对比度可能不够，尤其是在长文本或医学论文这类内容中，很难快速看清究竟改了哪些字。这个问题可以通过 `workbench.colorCustomizations` 精确调整。

<!--more-->

## 一、词级差异和整行差异是两组颜色

VS Code 的 Diff 编辑器大致使用以下两组颜色：

| 差异范围 | 新增内容 | 删除内容 |
| --- | --- | --- |
| 词级变化 | `diffEditor.insertedTextBackground` | `diffEditor.removedTextBackground` |
| 整行变化 | `diffEditor.insertedLineBackground` | `diffEditor.removedLineBackground` |

因此，想让具体修改的单词更加醒目，主要应该调整 `insertedTextBackground` 和 `removedTextBackground`，而不是只修改整行背景色。

## 二、打开 VS Code 用户设置 JSON

在 VS Code 中按下：

```text
⌘⇧P
```

然后搜索并打开：

```text
首选项：打开用户设置(JSON)
```

在最外层 JSON 对象中加入 `workbench.colorCustomizations`。如果配置文件中已经存在这个对象，只需要把下面的颜色项合并进去，不要重复添加同名键。

## 三、提高词级变化的对比度

可以使用较深、较不透明的颜色标记词级差异，再用较浅的颜色标记整行差异：

```json
"workbench.colorCustomizations": {
  "diffEditor.insertedTextBackground": "#00C853A6",
  "diffEditor.removedTextBackground": "#D50000A6",
  "diffEditor.insertedLineBackground": "#00C8531A",
  "diffEditor.removedLineBackground": "#D500001A"
}
```

这里使用的是 `#RRGGBBAA` 格式，最后两位表示透明度：

- `A6`：词级背景色的不透明度较高，突出具体修改内容；
- `1A`：整行背景色较淡，保留上下文但不抢夺视觉焦点。

如果希望颜色更深或更浅，可以调整最后两位透明度。例如，`FF` 表示完全不透明，`80` 约表示一半透明度。

## 四、仍然看不清时使用边框

对于部分主题，背景色可能会和语法高亮、编辑器配色混在一起。这时可以改用词级边框：

```json
"workbench.colorCustomizations": {
  "diffEditor.insertedTextBorder": "#00FF66",
  "diffEditor.removedTextBorder": "#FF3333"
}
```

边框会沿着词级变化的范围显示，通常比单纯加深背景更容易辨认。

词级变化建议在“背景色”和“边框色”之间选择一种即可。两者同时设置可能使界面过于醒目，反而影响连续阅读。

## 五、如何选择合适的配置

可以按下面的思路调整：

1. 先保留整行背景色较浅，只加深 `insertedTextBackground` 和 `removedTextBackground`。
2. 如果不同主题下仍然不明显，再尝试 `insertedTextBorder` 和 `removedTextBorder`。
3. 如果颜色过于刺眼，优先降低 `#RRGGBBAA` 中最后两位的透明度，而不是立刻更换整套颜色。

调整完成后重新打开 Git Diff，即可更清楚地区分“这一行发生过变化”和“这一行中具体哪些词发生了变化”。

更多可配置的 VS Code 主题颜色可以参考[官方颜色主题文档](https://code.visualstudio.com/api/references/theme-color)。
