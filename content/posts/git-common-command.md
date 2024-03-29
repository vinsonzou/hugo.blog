---
title: "Git常用命令备忘"
date: 2016-06-16T12:33:10+08:00
lastmod: 2023-12-09T09:48:00+08:00
description: ""

tags: ["git"]
---

# 配置

Git的设置文件为`.gitconfig`，它可以在用户主目录下（全局配置），也可以在项目目录下（项目配置）。

```sh
# 显示当前的Git配置
$ git config --list

# 文本编辑器
$ git config --global core.editor vim

# 编辑Git配置文件
$ git config -e [--global]

# 设置提交代码时的用户信息
$ git config [--global] user.name "[name]"
$ git config [--global] user.email "[email address]"
```

# FAQ

### 1. 如果在你Fork之后，原始的repo更新了，怎么将原始的更新内容与你当前的合并？

```sh
1. 增加原分支为远程分支，命名为upstream
   git remote add upstream https://github.com/vinsonzou/docker-images.git
2. fetch该远程仓库下的所有分支到remote-tracking分支
   git fetch upstream
3. 确保你当前在master分支
   git checkout master
4. Fork同步
   两种方式，任选其一即可
   a) 如果你已经对当前自己的副本做过更改，并且想要保留，则将更新合并到主分支
      git merge upstream/master
   b) 如果想要保留所有原仓库的历史更新则使用rebase复写当前分支(`本地所有修改丢失`)
      git rebase upstream/master
5. 然后推送
   git push
```

### 2. git 删除错误提交的commit

起因: 不小新把记录了公司服务器IP,账号,密码的文件提交到了git

```sh
#方法
git reset --hard <commit_id>
git push origin HEAD --force

#说明
根据--soft --mixed --hard，会对working tree和index和HEAD进行重置:
git reset --mixed：此为默认方式，不带任何参数的git reset，即时这种方式，它回退到某个版本，只保留源码，回退commit和index信息
git reset --soft：回退到某个版本，只回退了commit的信息，不会恢复到index file一级。如果还要提交，直接commit即可
git reset --hard：彻底回退到某个版本，本地的源码也会变为上一个版本的内容

HEAD 最近一个提交
HEAD^ 上一次
<commit_id>  每次commit的SHA1值. 可以用git log 看到,也可以在页面上commit标签页里找到.
```

### 3. git 空目录无法提交解决方法

空目录下touch .gitkeep

### 4. git 恢复单个文件的历史版本

```sh
#查看该文件的历史版本信息
git log tmp.txt

#记录下需要恢复的commit版本号：如 c2fcb51d554021c0603cbfc8c3b5fc3427d5d9a4

#恢复该文件
git reset c2fcb51d554021c0603cbfc8c3b5fc3427d5d9a4 tmp.txt

#提交
git commit -m "revert old file"
```

### 5. 根据COMMIT生成PATCH

```sh
# 生成patch文件(生成的patch有统计信息和git的版本信息)
git format-patch -1 commit版本号

# 应用patch文件
git apply xxx.patch
```

### 6. 使用 rebase -i 合并提交

**合并最近两个commit**
```sh
git rebase -i HEAD~~
```

默认的文字编辑器会自动开启，将看的HEAD 到 HEAD~~ 的提交

```sh
pick 7118c53 添加commit1的说明
pick ce32af7 添加commit2的说明

#
# Commands:
#  p, pick = use commit
#  r, reword = use commit, but edit the commit message
#  e, edit = use commit, but stop for amending
#  s, squash = use commit, but meld into previous commit
#  f, fixup = like "squash", but discard this commit's log message
#  x, exec = run command (the rest of the line) using shell
#
# If you remove a line here THAT COMMIT WILL BE LOST.
# However, if you remove everything, the rebase will be aborted.
#
```

将第二行的 "pick" 改为 "squash", 保存后退出。由于合并后要提交，所以编辑器会提醒您编辑这个最新的提交信息，请编辑信息后保存并退出。

这样，两个提交就合并成一个提交了。请用 log 命令确认历史记录。

### 7. tag使用
**创建tag**

```sh
git tag -a v0.1 -m 'version 0.1'
```

**查看tag**

```sh
git tag
git show v0.1
```

**tag 远程推送**

```sh
git push origin --tags
```

**获取远程版本部署**

```sh
git fetch origin tag v0.1
```

**删除tag**

```sh
#本地删除
git tag -d v0.1
#远程删除(git >= v1.7.0)
git push origin --delete tag v0.1
```

### 8. 查看具体commit的某个文件的修改内容
```sh
git show commit哈希值 文件名
```

### 9. 覆盖上一次提交记录和注释
```sh
# 本地覆盖记录
git commit --amend -m "Add comment msg"

# 线上覆盖提交
git push origin 分支名称：分支名称 -f
```

### 10. 使用signed-off-by
```sh
git commit -s -m 'commit msg'
```

### 11. 分支相关操作
```sh
# 显示当前分支
git branch --show-current  # Git >= 2.22

# 新建分支
git branch v0.0.0.1

# 切换分支
git checkout v0.0.0.1
git switch v0.0.0.1  # Git >= 2.23
```

### 12. 合并一个分支上某一次的修改到另一个分支上
把a分支的`b5ceeba22213bd05e32f305d79b21fb6048ef840`提交合并至b分支

```sh
# 切换至b分支
git switch b  # Git >= 2.23

git cherry-pick b5ceeba22213bd05e32f305d79b21fb6048ef840
git push
```