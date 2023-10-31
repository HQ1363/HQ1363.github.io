---
title: git操作指南   
date: 2021-08-02 14:36:50   
tags: git   
categories: Git   
excerpt: git已然成为开发的必备技能了，来看看有哪些相见恨晚的小技巧吧，可别说我没告诉你哦.
---

## Git SubModule

### 常见操作
```shell
$ git submodule init && git submodule update   # 将子模块下载到本地
$ git clone https://github.com/xx/xx.git --recursive  # 此命令可一次性下载好主库和子库
$ git submodule sync  # 修改.gitmodule文件后，用此命令同步子模块信息
$ git submodule add https://github.com/xxxx/xxxxx.git  # 添加submodule
```

### 批量操作submodule
```shell
$ git submodule foreach <command>
比如:
$ git submodule foreach git checkout master
$ git submodule foreach git submodule update
```

### 删除git submodule
```shell
$ git add .gitmodules        # 第一步
$ git rm --cached submodule_name    # 第二步
$ rm -rf submodule_name   # 第三步：删除submodule目录
# 第四步：编辑.gitmodules文件, 移除对应的submodule信息
# 第五步：编辑.git/modules文件, 移除对应的submodule信息
# 第六步：编辑.git/config 移除对应的submodule信息
# ===== 上述方式不行，可尝试下述
$ git submodule deinit <submodule-name> # 新版git
# ===== 上述方式不行，可尝试下述
$ git rm <submodule-name> # 旧版git
```

### 子仓如何与远端保持同步
```shell
# 子仓的更新，是单独的，需要进入子仓目录，手动与远端同步，例如：
$ cd sub-dir && git fetch origin master && git rebase origin/master
# 完成同步后，需要在主仓下提交子仓的改动，以保存主仓对子仓的最新引用
```

## Git Ops

### 查看last commitId
```shell
$ git log -1 --pretty=format:%H # 完整的
# 输出 7b6b2803d2b7135b239d062847816e55a810371e
$ git log -1 --pretty=format:%h # 前7位
# 输出 7b6b280
```

### 查看某次commit的内容
```shell
$ git show commitId
```

### 查看目录的diff信息
```shell
$ git diff <dir-name>
```

### diff迁移
```shell script
$ git diff > diff.patch
$ git apply --stat diff.patch
$ git apply --check diff.patch
$ git apply diff.patch
```

### 查看log的其他信息
```shell
git --no-pager log -2 --author="HQ" --pretty=format:"%h"
```

### 指定tag/branch克隆仓库
```shell
$ git clone -b <tag-name|branch-name> xxxxxxxxx.git
```

### 获取指定tag下的仓库地址
```shell
# 浏览器地址上，替换对应的tag名称即可
# https://gitlab.com/gitlab-org/gitaly/-/tree/v13.12.8?ref_type=tags  # 官方gitaly仓库地址
```

### 查看分支合并图
```shell
$ git log --graph
```

### 查看标签信息
```shell
$ git show <tag-name>
```

### 查看git命令操作历史
```shell
$ git reflog
```

### git rebase发生冲突怎么办
```shell
$ git rebase --skip  # 抛弃本地的 commit，采用远程的 commit。慎用：因为你本地的修改都会失去
$ git rebase --abort # 终止此次 rebase 操作
$ git rebase --continue # 手动处理冲突的文件：执行git add .，再 git rebase --continue，反复操作直到解决完所有冲突，并合并到分支上。
# 切记，整个rebase解决冲突的过程中，都不需要自己去单独执行commit动作
```

### 删除本地认证
```shell
$ git config --global --unset credential.helper  # --system or --local
$ git config credential.helper store  # window的可能需要手动找到git的凭证删掉，见下图
```
![](git_identity.png)

### HTTP认证方式更改为SSH
```shell
$ git config --global url.ssh://git@github.com/.insteadOf https://github.com/
# 执行上述命令后，可以查看下全局的.gitconfig配置如下：
[url "ssh://git@gitlab.sss.com/"]
	insteadOf = https://gitlab.sss.com/
[url "ssh://git@pkg.sss.com/"]
	insteadOf = https://pkg.sss.com/
[http]
	sslVerify = false
[url "ssh://git@github.com/"]
	insteadOf = https://github.com/
# 完成替换后，记得添加ssh public key到gitlab profile配置里
# 删除Local认证
$ git config --local --unset credential.helper
# 删除全局认证
$ git config --global --unset credential.helper
# 删除系统认证
$ git config --system --unset credential.helper
# 设置记住密码（默认15分钟）：
$ git config --global credential.helper cache
# 如果想自己设置时间，可以这样做
$ git config credential.helper 'cache --timeout=3600'
# 永久存储密码
$ git config --global credential.helper store
# 清理缓存的账号密码
$ git credential-manager uninstall
```

### 如何知道分支是从哪拉出来的
```shell
$ git reflog --date=local | grep 本地分支名       # 仅限本地创建的分支
$ git log --oneline --decorate --graph --all    # 看的迷糊，看不懂
# 可借助sourceTree等工具查看，比较稳
```

### git的两点diff和三点diff区别是啥
```shell
# 三点diff会找到两边的共同祖先，然后拿到祖先到最新版本的差异，可能会有以下应用场景：
# 1、feature分支提交了，很多次，我想知道，我的feature相比于主干分支(master)的所有改动内容，然后codereview
```

### git远端分支被删除后，还可以恢复吗？
```shell
# 1、如果本地或者其他地方，有备份的话，就直接重推一下就好了
# 2、如果存有分支被删除时的，最新sha，那可以继续sha来创建分支恢复   
----   
如果你误删除了 Git 的远程分支，有可能找到该提交的 SHA1 校验值并恢复。
首先，你需要找到该分支最后一个提交的 SHA1 校验值，使用 git reflog 命令可以查看历史的提交记录：
git reflog
在历史记录中找到你误删除的分支的最后一次提交，记下其 SHA1 校验值。
然后，从这个提交创建一个新的分支：
git branch <branch_to_recover> <SHA1>
最后，将恢复的分支推送到远程仓库：
git push origin <branch_to_recover>
这样就实现了误删的远程分支的恢复。
请注意，git reflog 命令仅会记录本地的 Git 操作，如果你在删除远程分支后没有在本地进行过任何 Git 操作，可能无法用这个办法找到该删除的分的 SHA1 校验值。同时，这种方法只能用于恢复误删的最后一次提交，如果在删除分支后做了新的提交，这种方法将无法找回这些新的提交。
```
