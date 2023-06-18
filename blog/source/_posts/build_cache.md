---
title: CI缓存加速实践   
date: 2023-06-18 18:36:50   
tags:
 - cicd
 - cache
categories: cache   
excerpt: 像Java/Go/Node/Python构建都是需要下载依赖的，如何加速构建，减少研发CICD的等待时间显得尤为重要.
---

## Java 缓存
> 我们知道Java的缓存配置，对于Maven构建，是由settings文件指定的，一般是全局共用的本地缓存，当然我们也可以在执行mvn命令的时候，指定`-s`参数来设置settings文件，具体的配置项如下：
```javascript
<localRepository>/root/.m2/repository</localRepository>
```
目前实践下来的缓存思路是按仓库来隔离每一个缓存，这么做的原因是：我们经常遇到不同仓库/应用之间的依赖相互影响，导致经常性出现缓存脏了的构建问题，处理起来很头疼.

## Go 缓存
只缓存`GOCACHE`和`GOMODCACHE`目录，同时按仓库级别做隔离.

## Node 缓存
缓存`NPM_CONFIG_CACHE`和`YARN_CACHE_FOLDER`目录，其次就是`node_modules`目录了，按仓库做隔离，然后动态挂载到对应构建仓库的`node_modules`目录上. 

### 踩过的坑
- node仓库未按仓库做隔离，导致出现奇奇怪怪的缓存问题
- 使用软链的方式做仓库隔离，构建失败，没有用
- 使用cp/mv的方式做仓库隔离，磁盘IO非常慢，小文件又众多，跨磁盘mv/cp，简直是噩梦
- 使用mount的方式，挂载进去，又提示权限问题（内部系统不允许直接mount命令操作，允许的话，此方案可行）
- 基于k8s的磁盘挂载方式，实现动态挂载
- 清理node/go/java等缓存目录，简直是噩梦，速度非常非常慢，建议mv或者格式化操作
