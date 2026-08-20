---
title: 利用 GitHub Action 部署 Hexo 博客
layout: post
date: 2020-05-24 18:44:00
updated: 2020-05-24 18:44:00
comments: true
tags:
- Hexo
- GitHub Actions
- CI/CD
- 自动部署
- GitHub
- Gitee
categories:
- [技术, Hexo]
permalink: /2020/05/github-action-hexo/
---

> 自动部署博客到 GitHub 和码云，并实现码云的自动更新，不需要再登录网页点击按钮
> 在使用本教程前，请确保本地部署 Hexo 博客到 GitHub 及码云已成功
> 参考文献：[**利用 GitHub Actions 自动部署 Hexo 博客**](https://hexo.fluid-dev.com/posts/actions-deploy/)

<!--more-->

## 创建 Workflow 文件

在博客目录下创建 `.github/workflows/main.yml` 文件，文件内容为：

``` YAML
name: Blog                                                      # Actions 显示的名字，随意设置

on: [push]

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
    - name: Checkout
      uses: actions/checkout@v2
      with:
        ref: master

    - name: Setup Node
      uses: actions/setup-node@v1
      with:
        node-version: "10.x"

    - name: Hexo Generate
      run: |
        rm -f .yarnclean
        yarn --frozen-lockfile --ignore-engines --ignore-optional --non-interactive --silent --ignore-scripts --production=false
        rm -rf ./public
        yarn run hexo clean
        yarn run hexo generate

    - name: Hexo Deploy
      env:
        SSH_PRIVATE: ${{ secrets.SSH_PRIVATE }}
        GIT_NAME: XXX                                           # 你 GitHub 用户名  
        GIT_EMAIL: XXX@email.com                                # 你 GitHub 邮箱
      run: |
        mkdir -p ~/.ssh/
        echo "$SSH_PRIVATE" | tr -d '\r' > ~/.ssh/id_rsa
        chmod 600 ~/.ssh/id_rsa
        ssh-keyscan gitee.com >> ~/.ssh/known_hosts
        ssh-keyscan github.com >> ~/.ssh/known_hosts
        git config --global user.name "$GIT_NAME"
        git config --global user.email "$GIT_EMAIL"
        yarn run hexo deploy

    - name: Build Gitee Pages
      uses: yanglbme/gitee-pages-action@master
      with:
          gitee-username: arxhd.love@qq.com                     # 你的 Gitee 用户名
          gitee-password: ${{ secrets.GITEE_PASSWORD }}
          gitee-repo: jupitersh/jupitersh                       # 你的 Gitee 仓库
```

有 `#` 注释的部分修改为你自己的，其他保持默认即可

## 新建 GitHub 仓库

新建一个 GitHub 仓库，在下图的位置的添加两个 `Secrets` ，分别名为 `GITEE_PASSWORD` 和 `SSH_PRIVATE` ，其中 `GITEE_PASSWORD` 是你码云的登陆密码，`SSH_PRIVATE` 是你之前部署用的私钥

将本地的博客文件夹提交到该仓库中，该仓库至少包括：`.github`、`source`、`themes`、`_config.yml`、`package.json` 这五个文件夹或文件

## 部署成功

以后每次有新内容只要更改后提交到刚刚新建的仓库即可，或者可以在 GitHub 网页版直接写博客

进入Action选项卡，出现一个勾勾就代表成功了，同时打开自己的 GitHub 和码云博客看看是否成功更新内容吧
