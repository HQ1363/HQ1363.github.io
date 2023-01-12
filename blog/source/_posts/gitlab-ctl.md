---
title: gitlab维护指南   
date: 2021-08-02 14:36:50   
tags: git   
categories: Git   
excerpt: gitlab部署、日常维护、常见问题解决方式.
---

## 常见命令

### 彻底卸载gitlab
```shell
$ sudo gitlab-ctl stop
$ sudo gitlab-ctl uninstall
$ sudo gitlab-ctl cleanse
$ sudo rm -rf /opt/gitlab
# 可通过`sudo gitlab-ctl help`来获取相关命令
# sudo gitlab-ctl uninstall，关于其的说明是
# Kill all processes and uninstall the process supervisor (data will be preserved).
# 但在实际操作中，只通过uninstall无法彻底卸载gitlab
# sudo gitlab-ctl reconfigure
```

### 重载配置文件
`gitlab-ctl reconfigure`

### 启动所有 gitlab 组件
```shell
$ gitlab-ctl start         # 启动所有服务
$ gitlab-ctl start unicorn # 启动单个服务
```

### 停止所有 gitlab 组件
```shell
$ gitlab-ctl stop          # 停止所有服务
$ gitlab-ctl stop unicorn  # 停止单个服务
```

### 重启所有 gitlab 组件
```shell
$ gitlab-ctl restart         # 重启所有服务
$ gitlab-ctl restart unicorn # 重启单个服务
```

### 查看服务的活动日志
```shell
$ gitlab-ctl tail         # 查看所有服务的活动日志
$ gitlab-ctl tail unicorn # 查看单个服务的活动日志
```

### 查看服务状态
`gitlab-ctl status`
