---
title: gorm血泪史   
date: 2021-09-10 12:03:49   
tags:
  - gorm
  - go
categories:
  - gorm
excerpt: 如果你用过gorm，只希望各位大佬不要踩这些坑，每一个都是血和泪的教训。
---

### Rows没有Close
> <strong>上下文</strong>：有个实习生，写了一个需求，结果for循环到第十次的时候，一直卡死，没有任何响应，最终请求超时；不仅如此，后续的接口请求，都是没有处理的，整个程序仿佛处于假死状态。这让当时的我感到十分诧异，我的第一个感觉是存在死循环，为了证实猜想，开始对整个for循环体做单步调试，结果发现，并不存在死循环，程序卡在某个SQL执行处；于是我就想，难道是慢SQL，导致一直从DB拿不到返回结果，但很快我就排除了这种可能性，毕竟问题是发生第十次，而不是第一次；在问题稳定复现后，我开始总结规律，问题复现一定是在for循环执行到第十次的时候，就有这么巧吗，回回都是第十次，也太邪乎了。

<u>针对上述的问题，毫无疑问，肯定是出了DB上的问题，然后又是第十次稳定复现，我开始怀疑DB连接的问题，比如连接被吃满，不够用</u>；于是乎，我开始检查程序连接DB的配置，发现active激活连接数，配置的最大值就是10个，这不刚好就是10嘛，<span style="color:orange">这不是巧了嘛！这不是巧了嘛！</span>那就调大嘛，结果然并软，反而换了一个报错<u>（MySQL error code 1135 (ER_CANT_CREATE_THREAD): Can't create a new thread (errno %d); if you are not out of available memory, you can consult the manual for a possible OS-dependent bug）</u>，这又是什么鬼啊，why！！！
![](question.jpeg)哼，不陪你玩了，哪儿凉快哪儿呆着去吧；哈哈，回到正题，不然要被你们打😄

<strong style="font-size:20px;color:blue">难道不是连接的问题吗，我开始更换思路</strong>；我的SQL请求到底有没有打到MySQL服务器呢，我开始求助DBA，希望帮捞下，第十次的SQL执行日志，得到的反馈是一切正常。what，不对啊；因为程序DB操作，用的是Gorm，我开始查阅官方文档，搜索github上的issue，并为找到任何有用的消息。对新出现的error信息，同向DBA老师求证后，排除了，没办法，我只能把连接配置，改回去，继续排查问题。然后一步步的检查代码，发现DB操作，<u style="color:red">少了一个释放连接的动作</u>，于是我加上，赶紧验证了一下，好在问题得到解决了。
![](good.jpeg)我们来看看代码：
```golang
// 原生 SQL
rows, err := db.Raw("select name, age, email from users where name = ?", "jinzhu").Rows()
defer rows.Close()   // 就是这行代码少了，导致的
for rows.Next() {
  rows.Scan(&name, &age, &email)
  // 业务逻辑...
}
```
查阅资料后，大胆猜测：<u>有可能是mysql每次去查询的时候，获取一个连接，没有空闲的连接，则创建一个新的，查询完成后释放连接到连接池，以便下一个请求使用，而由于没有调用rows.Close()，导致拿了连接之后，没有再放回连接池复用；而我的连接配置最大就是10，所以在第十次执行完后，第十一次，已经无法分配新的连接去执行SQL，最终一直等待，拿不到结果。</u><a href="https://segmentfault.com/a/1190000021493463" target="_blank">感兴趣的朋友，可以看看这篇文章，基于gorm源码解释了问题原因</a>；未执行rows.Close()还可能导致内存泄漏、启动一堆的goroutine不退出等问题。
