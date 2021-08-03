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
【1】$ git add .gitmodules
【2】$ git rm --cached submodule_name
【3】$ rm -rf submodule_name   # 删除submodule目录
【4】编辑.gitmodules文件, 移除对应的submodule信息
【5】编辑.git/modules文件, 移除对应的submodule信息
【6】编辑.git/config 移除对应的submodule信息
===== 上述方式不行，可尝试下述
【1】$ git submodule deinit <submodule-name> # 新版git
===== 上述方式不行，可尝试下述
【1】$ git rm <submodule-name> # 旧版git
```
