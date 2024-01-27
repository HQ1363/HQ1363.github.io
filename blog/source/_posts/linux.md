---
title: 运维实践指南   
date: 2024-01-27 14:36:50   
tags: linux   
categories: Linux   
excerpt: 无论你是开发，还是运维，亦或者是测试，免不了要和系统打交道，而linux系统通常也是我们线上服务的部署系统，掌握一些必备的运维知识，能让你如鱼得水.
---

## 远端服务端口探测
```shell
$ nmap -p 80 example.com
$ telnet example.com 22
$ nc -zv example.com 443
```

## 检查ssh是否ready
```shell
$ ssh -T git@example.com
```
