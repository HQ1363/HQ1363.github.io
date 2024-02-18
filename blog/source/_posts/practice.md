---
title: 实用技巧指南   
date: 2024-02-10 14:36:50   
tags: ops   
categories: ops   
excerpt: 如果你是理论派，就是缺少实操，不妨来了解下常见的实操技能；如果你是实践派，也可以来查漏补缺下，温故而知新，可以为师矣.
---

## 如何监听go应用程序CPU/Mem情况
> 有时可能并非应用程序占用CPU、Mem过高，应当结合htop一同分析
```shell
$ curl http://localhost:xxxx/debug/pprof/profile?seconds=60 > prof.log
$ brew install graphviz
$ go tool pprof -http=127.0.0.1:8080 prof.log
```
