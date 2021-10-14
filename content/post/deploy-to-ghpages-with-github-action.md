+++
description = ""
date = "2021-10-14T16:35:08+08:00"
title = "Hugo使用Github Action自动部署到Github Pages"
tags = ["Hugo"]

+++

## 目的

hugo手动部署到github pages流程:
- 使用hugo或hugo --minify生成静态网页
- 将静态网页push到建好的github pages repo中，一般是<username>/<username>.github.io

本文的目的就是使用github action自动部署github pages。

## 流程及原理

- 本地新建文章，push到Github仓库的master分支。master分支存放博客文章的源码。
- push 操作自动触发预先配置的Actions。
- GitHub Action自动执行yml文件中的"action"，构建打包，推送至gh-pages repo。

## 详细步骤

首先点击github头像在下拉栏里进入`Settings - Developer Settings - Personal access tokens`
选择`Generate new token`
![action-01](/img/github-action-01.png)

在上方填入名字，并勾选repo和action选项，如下图
![action-02](/img/github-action-02.png)

然后执行以下几步：

在源码repo里
- 在repo根目录新建嵌套的两个文件夹.github/workflows
- 在workflows里新建一个后缀为.yml的配置文件，名字自取
- 写进去以下配置（从hugo官方文档修改而来）

```yml
name: github pages # 名字自取

on:
  push:
    branches:
      - master  # 这里的意思是当 main分支发生push的时候，运行下面的jobs，这里先改为github-actions

jobs:
  build-deploy: # 任务名自取
    runs-on: ubuntu-latest	# 在什么环境运行任务

    steps:
      - uses: actions/checkout@v2	# 引用actions/checkout这个action，与所在的github仓库同名

      - name: Setup Hugo	# 步骤名自取
        uses: peaceiris/actions-hugo@v2	# hugo官方提供的action，用于在任务环境中获取hugo
        with:
          hugo-version: 'latest'	# 获取最新版本的hugo

      - name: Build
        run: hugo --minify	# 使用hugo构建静态网页

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3	# 一个自动发布github pages的action
        with:
          # github_token: ${{ secrets.GITHUB_TOKEN }} 该项适用于发布到源码相同repo的情况，不能用于发布到其他repo
          external_repository: vinsonzou/vinsonzou.github.com	# 发布到哪个repo
          personal_token: ${{ secrets.ACTION_ACCESS_TOKEN }}	# 发布到其他repo需要提供上面生成的personal access token
          cname: ops.m114.org   # 添加你的网站域名作CNAME以便解析
          publish_dir: ./public	# 注意这里指的是要发布哪个文件夹的内容，而不是指发布到目的仓库的什么位置，因为hugo默认生成静态网页到public文件夹，所以这里发布public文件夹里的内容
          publish_branch: master # 发布到哪个branch
          commit_message: ${{ github.event.head_commit.message }}
```
