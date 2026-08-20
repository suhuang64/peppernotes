---
title: GitHub Pull Request 完整教程：从 Fork、提交 PR 到同步上游仓库
layout: post
date: 2026-08-20 17:02:00
updated: 2026-08-20 17:02:00
comments: true
tags:
- GitHub
- Git
- Pull Request
- Fork
- 开源协作
- 版本控制
categories:
- [技术, Git]
permalink: /2026/08/github-pull-request/
---

在参与 GitHub 开源项目时，一个非常常见的工作流是：

> **Fork 别人的项目 → 修改代码 → 提交 Pull Request → 作者 Review → Merge 或 Reject → 继续同步原项目**

<!--more-->

第一次接触这套流程时，很容易遇到一些问题：

* Fork 之后到底应该在哪个分支修改？
* Pull Request 被拒绝了，我写的代码会怎么样？
* 作者只接受我的部分修改怎么办？
* PR 合并以后，我自己的 Fork 怎么处理？
* 原作者继续更新项目后，我的 Fork 怎么同步？
* 如果我的 Fork 已经有自己的修改，还能不能继续跟原项目保持同步？
* `merge` 和 `rebase` 到底应该怎么选？

这篇文章从头梳理一遍 GitHub Fork + Pull Request 的完整工作流。

---

## 一、先理解三个仓库：upstream、origin 和 local

假设 GitHub 上有一个开源项目：

```text
alice/project
```

我希望参与这个项目，于是在 GitHub 上点击 **Fork**。

GitHub 会在我的账号下创建一份副本：

```text
alice/project
     │
     │ Fork
     ▼
pepper/project
```

然后，我再把自己的 Fork 克隆到电脑：

```text
alice/project
     │
     │ Fork
     ▼
pepper/project
     │
     │ git clone
     ▼
本地 project
```

因此整个开发过程中，其实存在三个仓库：

```text
① upstream
   alice/project
   原作者的仓库

② origin
   pepper/project
   我 GitHub 账号下的 Fork

③ local
   我电脑上的本地 Git 仓库
```

这是理解整个 Pull Request 工作流最重要的基础。

通常约定：

```text
origin   = 自己的 Fork
upstream = 原作者的仓库
```

可以简单记成：

> **origin 是我的，upstream 是上游作者的。**

---

## 二、Fork 项目后的第一次配置

首先，在 GitHub 页面点击：

```text
Fork
```

得到自己的仓库：

```text
pepper/project
```

然后克隆自己的 Fork：

```bash
git clone git@github.com:pepper/project.git
cd project
```

此时执行：

```bash
git remote -v
```

通常只能看到：

```text
origin  git@github.com:pepper/project.git (fetch)
origin  git@github.com:pepper/project.git (push)
```

因为 Git 目前只知道自己的 Fork 在哪里，并不知道原作者仓库在哪里。

因此需要添加一个 `upstream`：

```bash
git remote add upstream https://github.com/alice/project.git
```

再次检查：

```bash
git remote -v
```

应该类似：

```text
origin    git@github.com:pepper/project.git (fetch)
origin    git@github.com:pepper/project.git (push)

upstream  https://github.com/alice/project.git (fetch)
upstream  https://github.com/alice/project.git (push)
```

以后只需要记住：

```text
origin   → 我的 Fork
upstream → 原作者仓库
```

---

## 三、最重要的原则：不要直接在 main 上开发

这是整个工作流中最值得养成的习惯：

> **自己的 `main` 尽量永远与原作者的 `main` 保持一致。自己的功能修改全部放到单独的 branch。**

假设准备增加：

```text
CSV 导出功能
```

不推荐：

```text
main
 ↓
修改代码
 ↓
commit
 ↓
提交 PR
```

更推荐：

```text
main
 │
 └── feature/csv-export
          │
          ├── 修改
          ├── commit
          └── Pull Request
```

首先切换到 `main`：

```bash
git switch main
```

获取原作者最新代码：

```bash
git fetch upstream
```

同步到本地：

```bash
git merge upstream/main
```

然后创建功能分支：

```bash
git switch -c feature/csv-export
```

此时提交历史可以理解成：

```text
A──B──C
       \
        D──E
```

其中：

```text
A B C = 原作者代码
D E   = 我的修改
```

而：

```text
main                → C
feature/csv-export  → E
```

这样 `main` 是干净的，而自己的功能独立存在。

---

## 四、修改完成后提交代码

修改完成后：

```bash
git status
```

查看发生了哪些变化。

然后：

```bash
git add .
```

提交：

```bash
git commit -m "feat: add CSV export support"
```

接下来把功能分支推送到自己的 GitHub Fork：

```bash
git push -u origin feature/csv-export
```

此时 GitHub 上自己的仓库就有：

```text
pepper/project

├── main
└── feature/csv-export
```

注意，这一步只是：

```text
本地 feature/csv-export
        │
        │ git push
        ▼
自己的 GitHub Fork
```

代码还没有进入原作者的仓库。

---

## 五、创建 Pull Request

Push 完成后，GitHub 通常会提示：

```text
Compare & pull request
```

点击以后，需要确认两个方向：

```text
base repository:
alice/project

base:
main

←

head repository:
pepper/project

compare:
feature/csv-export
```

它表达的意思是：

```text
pepper/project:feature/csv-export
                │
                │ Pull Request
                ▼
alice/project:main
```

也就是：

> **请求原作者把我的 `feature/csv-export` 修改合并进他的 `main`。**

这就是 Pull Request，简称 PR。

---

## 六、Pull Request 提交之后会发生什么？

作者可以对 PR 进行 Review。

常见结果包括：

```text
Approve
Request changes
Comment
Close
Merge
```

实际开发中，大致可以分成下面几种情况。

---

## 七、情况一：作者完全接受 PR

假设原作者原来的代码是：

```text
A──B──C
```

我的功能分支增加：

```text
A──B──C──D──E──F
```

作者认为这些修改没有问题，于是：

```text
Approve
   ↓
Merge
```

那么这些修改就会进入原作者的 `main`。

概念上可以理解成：

```text
原来：

A──B──C

合并后：

A──B──C──D──E──F
```

实际 Git 历史可能有所不同，因为 GitHub 支持：

* Create a merge commit
* Squash and merge
* Rebase and merge

但从代码结果来看，你的贡献已经进入 upstream。

---

## 八、情况二：PR 被拒绝会怎么样？

假设：

```text
upstream/main

A──B──C
```

而我的分支：

```text
feature/csv-export

A──B──C──D──E
```

提交 PR 后，作者认为这个功能不适合项目，于是关闭 PR：

```text
Close Pull Request
```

这时候：

> **你的代码不会消失。**

原作者还是：

```text
upstream/main

A──B──C
```

而你的 Fork 仍然可以是：

```text
origin/main

A──B──C
```

同时：

```text
origin/feature/csv-export

A──B──C──D──E
```

所以：

> **PR 被拒绝，只代表作者没有把你的代码合并进他的项目，并不代表你的代码被删除。**

你的 branch、commit 和代码都还属于你。

---

## 九、PR 被拒绝以后，不想要这些代码怎么办？

如果这个功能以后也不需要了，可以直接删除 branch。

首先回到：

```bash
git switch main
```

删除本地分支：

```bash
git branch -d feature/csv-export
```

如果 Git 因为分支没有被合并而拒绝删除，可以强制删除：

```bash
git branch -D feature/csv-export
```

然后删除自己 GitHub Fork 上的远程分支：

```bash
git push origin --delete feature/csv-export
```

这样就恢复干净了。

---

## 十、如果 PR 被拒绝，但我自己还想继续用这个功能呢？

那就什么都不用做。

继续保留：

```text
feature/csv-export
```

即可。

这也是 Fork 非常重要的意义之一。

原作者可以维护：

```text
alice/project
```

而你完全可以维护自己的魔改版本：

```text
pepper/project
```

两者不完全一致没有任何问题。

---

## 十一、作者只接受部分修改怎么办？

假设一个 PR 里包含：

```text
A 功能
B 功能
C 功能
```

作者 Review 后认为：

```text
A 👍
B 👍
C ❌
```

一般不是简单地在 GitHub 上把 C “切掉”，更常见的工作流是作者：

```text
Request changes
```

然后告诉你：

> A、B 可以，但是请删除 C。

这时候不需要关闭 PR，也不需要重新创建 PR。

直接继续修改当前：

```text
feature/xxx
```

把 C 删除。

然后：

```bash
git add .
git commit -m "refactor: remove feature C based on review"
git push
```

因为 PR 跟踪的是这个 branch，所以：

> **继续向这个 branch push，原来的 PR 会自动更新。**

于是 PR 最终可能只剩：

```text
A
B
```

作者重新 Review：

```text
Approve
↓
Merge
```

即可。

---

## 十二、原作者更新项目以后，我的 Fork 怎么同步？

这是 Fork 项目以后最常见的操作之一。

假设最初：

```text
upstream/main

A──B──C
```

自己的 Fork：

```text
origin/main

A──B──C
```

两边完全一致。

后来作者继续开发：

```text
upstream/main

A──B──C──D──E──F
```

但是自己的 Fork 还是：

```text
origin/main

A──B──C
```

GitHub 此时可能显示：

```text
This branch is 3 commits behind
```

也就是说自己的 Fork 落后于 upstream。

---

## 十三、通过 Git 命令同步 upstream

首先切换到 `main`：

```bash
git switch main
```

获取原作者最新信息：

```bash
git fetch upstream
```

这里要特别注意：

```bash
git fetch upstream
```

并不会直接修改自己的 `main`。

它只是告诉 Git：

> 去 upstream 看一下有没有新东西，有的话下载回来。

此时 Git 知道：

```text
upstream/main

A──B──C──D──E──F
```

但是本地：

```text
main

A──B──C
```

仍然没有变化。

接下来：

```bash
git merge upstream/main
```

本地 `main` 就变成：

```text
A──B──C──D──E──F
```

最后：

```bash
git push origin main
```

把本地最新版同步到自己的 GitHub Fork。

于是：

```text
upstream/main
      =
local main
      =
origin/main
```

重新保持一致。

所以，日常同步 upstream 最值得记住的是这四条：

```bash
git switch main
git fetch upstream
git merge upstream/main
git push origin main
```

---

## 十四、也可以直接在 GitHub 网页同步 Fork

如果自己的 Fork 落后于原仓库，GitHub 通常会显示：

```text
This branch is X commits behind xxx:main
```

并提供：

```text
Sync fork
```

点击：

```text
Sync fork
→
Update branch
```

GitHub 就会帮助同步。

如果只是单纯希望自己的 `main` 与原作者保持一致，而且自己的 `main` 没有什么魔改，那么这种方式非常方便。

---

## 十五、为什么不要直接修改自己的 main？

现在就可以理解为什么前面一直强调：

> **main 尽量只用来同步 upstream。**

假设：

```text
upstream/main

A──B──C
```

你直接在自己的 `main` 修改：

```text
origin/main

A──B──C──X──Y
```

后来作者又提交：

```text
upstream/main

A──B──C──D──E
```

此时历史就出现分叉：

```text
          X──Y    ← 我的修改
         /
A──B──C
         \
          D──E    ← 作者的新修改
```

这时候再同步 upstream，就需要处理：

```text
merge
```

如果双方恰好修改了同一部分代码，还可能出现：

```text
merge conflict
```

随着时间越来越长，维护成本也会越来越高。

---

## 十六、正确方式：main 跟踪 upstream，功能放 branch

如果从一开始就创建独立分支：

```text
                 X──Y
                /
A──B──C────────
       \
        D──E
```

那么：

```text
X Y = 我的 feature
D E = upstream 新更新
```

此时可以直接更新 `main`：

```bash
git switch main
git fetch upstream
git merge upstream/main
git push origin main
```

于是：

```text
main

A──B──C──D──E
```

而自己的：

```text
feature/my-feature
```

仍然独立存在。

两件事情互不干扰。

---

## 十七、正在开发 Feature 时，upstream 更新了怎么办？

这是另一个非常常见的情况。

假设开始开发时：

```text
A──B──C
       \
        X──Y
```

其中：

```text
X Y = 我的修改
```

但是在我开发过程中，原作者继续提交：

```text
A──B──C──D──E──F
```

现在可以理解成：

```text
              X──Y
             /
A──B──C
       \
        D──E──F
```

我希望自己的 Feature 也基于作者最新版开发。

最终希望达到：

```text
A──B──C──D──E──F──X──Y
```

这时候主要有两种方法：

```text
merge
```

或者：

```text
rebase
```

---

## 十八、方法一：merge upstream/main

首先：

```bash
git switch feature/my-feature
```

获取 upstream：

```bash
git fetch upstream
```

然后：

```bash
git merge upstream/main
```

历史可能变成：

```text
          X──Y────M
         /       /
A──B──C──D──E──F
```

其中：

```text
M = merge commit
```

这种方式的优点是：

* 容易理解
* 不改写已有 commit 历史
* 相对安全

缺点是：

* commit graph 可能比较复杂
* feature branch 可能出现很多 merge commit

对于 Git 初学者，这通常是比较容易理解的方式。

---

## 十九、方法二：rebase upstream/main

另一种方式是：

```bash
git switch feature/my-feature
git fetch upstream
git rebase upstream/main
```

原来的：

```text
              X──Y
             /
A──B──C
       \
        D──E──F
```

经过 rebase 后，可以理解成：

```text
A──B──C──D──E──F──X'──Y'
```

也就是说 Git 把自己的：

```text
X
Y
```

重新放到了最新版：

```text
F
```

后面。

可以把 rebase 理解成：

> **假装我当初就是从 upstream 最新版本开始写 X 和 Y 的。**

因此历史会非常干净。

---

## 二十、为什么 rebase 后有时候要 Force Push？

因为 rebase 会重新生成 commit。

原来：

```text
X
Y
```

rebase 后实际上变成：

```text
X'
Y'
```

它们虽然可能包含类似的代码，但 Git commit hash 已经不同。

所以如果之前已经：

```bash
git push origin feature/my-feature
```

rebase 后普通：

```bash
git push
```

可能会被 Git 拒绝。

此时通常需要：

```bash
git push --force-with-lease
```

推荐使用：

```bash
--force-with-lease
```

而不是直接：

```bash
--force
```

因为前者会多做一层检查，相对更加安全。

---

## 二十一、完整 Pull Request 工作流

整个过程可以总结为：

```text
                   GitHub
                      │
            ┌─────────┴─────────┐
            │                   │
        upstream             origin
       原作者仓库            我的 Fork
            │                   ↑
            │                   │
            └──────→ local ─────┘
                       │
                      main
                       │
            ┌──────────┼──────────┐
            │          │          │
            ▼          ▼          ▼
        feature/A  feature/B   fix/bug
            │
            │
            ▼
       Pull Request
            │
      ┌─────┴─────┐
      │           │
      ▼           ▼
    Merge       Reject
```

其中最重要的是：

```text
upstream/main
      │
      ▼
local main
      │
      ▼
origin/main
```

尽量保持一致。

而自己的工作：

```text
main
 ├── feature/A
 ├── feature/B
 ├── fix/bug
 └── docs/readme
```

全部独立开发。

---

## 二十二、一个完整的实战例子

假设我要给：

```text
github.com/alice/medical-ai
```

贡献代码。

Fork 后，我自己的仓库是：

```text
github.com/pepper/medical-ai
```

### Step 1：Clone 自己的 Fork

```bash
git clone git@github.com:pepper/medical-ai.git
cd medical-ai
```

### Step 2：添加 upstream

```bash
git remote add upstream https://github.com/alice/medical-ai.git
```

检查：

```bash
git remote -v
```

### Step 3：同步 main

```bash
git switch main
git fetch upstream
git merge upstream/main
git push origin main
```

### Step 4：创建功能分支

```bash
git switch -c feature/add-dicom-support
```

### Step 5：修改代码并 Commit

```bash
git add .
git commit -m "feat: add DICOM support"
```

### Step 6：Push 到自己的 Fork

```bash
git push -u origin feature/add-dicom-support
```

### Step 7：创建 Pull Request

在 GitHub 创建：

```text
pepper/medical-ai:feature/add-dicom-support
                    │
                    ▼
alice/medical-ai:main
```

### Step 8：根据 Review 修改

如果作者提出修改意见：

```bash
# 修改代码

git add .
git commit -m "fix: address PR review comments"
git push
```

原来的 PR 会自动更新。

### Step 9：作者 Merge

作者：

```text
Approve
↓
Merge
```

代码进入 upstream。

### Step 10：重新同步 main

```bash
git switch main
git fetch upstream
git merge upstream/main
git push origin main
```

### Step 11：删除已经完成的 Feature Branch

```bash
git branch -d feature/add-dicom-support
git push origin --delete feature/add-dicom-support
```

最终重新回到：

```text
upstream/main
      =
local main
      =
origin/main
```

整个工作区重新变得干净。

---

## 二十三、如果 PR 被拒绝，完整处理流程

如果作者关闭 PR：

```text
Pull Request
     ↓
   Closed
```

原作者的代码没有变化。

首先同步自己的 `main`：

```bash
git switch main
git fetch upstream
git merge upstream/main
git push origin main
```

如果这个 Feature 自己也不要了：

```bash
git branch -D feature/add-dicom-support
git push origin --delete feature/add-dicom-support
```

如果自己还需要这个功能，那么直接保留：

```text
feature/add-dicom-support
```

即可。

它不会影响 `main` 继续跟踪 upstream。

---

## 二十四、常用命令速查

### 查看远程仓库

```bash
git remote -v
```

### 添加 upstream

```bash
git remote add upstream https://github.com/作者/项目.git
```

### 获取 upstream 最新代码

```bash
git fetch upstream
```

### 同步 main

```bash
git switch main
git fetch upstream
git merge upstream/main
git push origin main
```

### 创建 Feature Branch

```bash
git switch -c feature/xxx
```

### 提交代码

```bash
git add .
git commit -m "feat: xxx"
```

### 第一次 Push Feature Branch

```bash
git push -u origin feature/xxx
```

### 后续更新 PR

```bash
git add .
git commit -m "fix: address review comments"
git push
```

### Feature 同步 upstream：Merge

```bash
git switch feature/xxx
git fetch upstream
git merge upstream/main
```

### Feature 同步 upstream：Rebase

```bash
git switch feature/xxx
git fetch upstream
git rebase upstream/main
```

如果已经 Push：

```bash
git push --force-with-lease
```

### 删除本地 Feature Branch

```bash
git branch -d feature/xxx
```

强制删除：

```bash
git branch -D feature/xxx
```

### 删除远程 Feature Branch

```bash
git push origin --delete feature/xxx
```

---

## 二十五、最终需要建立的 Git 思维

如果只记住一条原则，那么就是：

> **main 用来追踪原作者，branch 用来保存自己的工作，Pull Request 用来把 branch 的修改贡献给原作者。**

理想状态下：

```text
upstream/main
      │
      │ sync
      ▼
local main
      │
      │ push
      ▼
origin/main
```

而开发过程则是：

```text
main
 │
 ├── feature/A ──→ PR
 │
 ├── feature/B ──→ PR
 │
 ├── fix/bug ────→ PR
 │
 └── docs/readme ─→ PR
```

这样做最大的好处是：

1. `main` 始终保持干净；
2. upstream 更新后很容易同步；
3. 每一个功能互不干扰；
4. PR 被拒绝不会影响其他工作；
5. PR 被接受后可以直接删除对应 branch；
6. 可以同时维护多个 PR；
7. 即使自己的 Fork 长期存在，也不容易和 upstream 搞乱。

因此，一个比较成熟的 GitHub Fork 工作流可以概括成：

```text
Fork
  ↓
Clone
  ↓
添加 upstream
  ↓
同步 main
  ↓
创建 feature branch
  ↓
修改代码
  ↓
Commit
  ↓
Push 到 origin
  ↓
创建 Pull Request
  ↓
根据 Review 继续修改
  ↓
Merge / Reject
  ↓
重新同步 main
  ↓
删除或保留 feature branch
  ↓
下一轮开发
```

一旦把 **upstream、origin、main、feature branch、Pull Request** 这五个概念理清楚，GitHub 上绝大多数开源项目的日常贡献流程就已经基本掌握了。
