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
$ git config --global --unset credential.helper
$ git config credential.helper store  # window的可能需要手动找到git的凭证删掉，见下图
```
![](git_identity.png)

### 测试
```shell
$ bash a.sh
```
