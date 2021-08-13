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
