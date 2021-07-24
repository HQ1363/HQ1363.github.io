---
title: go 一百问   
date: 2021-04-01 16:02:13   
comments: true   
tags: go   
categories: Python   
excerpt: 好的编码风格，能够让你的代码更优雅、更简洁，还能避免很多坑.
---

## Go基础
### Q1 为啥需要私有`goproxy`？
#### 上下文
> 我们知道在大陆的网络环境是无法访问到`http://golang.org`等`google`的网站的。但在开发日常中使用的很多依赖包或系统包依赖都是在`google`的服务器上。为了解决无法加载依赖的问题，国内也有很多种解决方案。一种是使用`http://goproxy.io`或七牛主导的`http://goproxy.cn`。
在企业里，有很多情况是生产网络或测试网络环境是无法正常访问外网的，为了解决这个问题可能需要自己搭建一个`proxy`来管理依赖包。
#### 可选配置
```bash
export GOPROXY=https://mirrors.aliyun.com/goproxy/
export GOPROXY=https://proxy.golang.org,direct
export GOPROXY=https://goproxy.io
export GOPROXY=https://gonexus.dev
export GOPROXY=https://athens.azurefd.net
export GOPROXY=https://gocenter.io
export GOPROXY=https://goproxy.cn
```
### Q2 `Make`和`New`的异同？
