---
title: Traceroute实验
date: 2019-09-22 18:13:46
tags: 
    - course
    - network
    - python
menu: 
    Go Home: /index.html
categories: 技术分享
cover: /gallery/thumbnails/2019-09-22.png
thumbnail: /gallery/thumbnails/2019-09-22.png
toc: true
comments: true
---

前言：此文章整理于java上机课上，（由于课程安排太满）好惨一大学生在线挤时间做笔记

<!--more-->

# 一、实验背景
专业课 计算机网络 一次路由追踪实验

# 二、实验原理
windows10(64位) cmd tracert命令 
```
用法: tracert [-d] [-h maximum_hops] [-j host-list] [-w timeout]
               [-R] [-S srcaddr] [-4] [-6] target_name

选项:
    -d                 不将地址解析成主机名。
    -h maximum_hops    搜索目标的最大跃点数。
    -j host-list       与主机列表一起的松散源路由(仅适用于 IPv4)。
    -w timeout         等待每个回复的超时时间(以毫秒为单位)。
    -R                 跟踪往返行程路径(仅适用于 IPv6)。
    -S srcaddr         要使用的源地址(仅适用于 IPv6)。
    -4                 强制使用 IPv4。
    -6                 强制使用 IPv6。
```
# 三、实验操作

## 1.准备脚本

由于该实验要求每隔一小时进行一次traceroute实验，共进行三次，所以这种重复性机械操作当然是交给我亲爱的python来完成了~

![没啥技术含量的python代码](/gallery/pictures/2019-9-22/1.png)

（其实time.sleep(3600)才是最sao的）

就这样挂在后台跑吧，我先去睡觉，，，

## 2.处理数据

当我醒来，打开熟悉的txt，看到满屏幕的实验数据，内心无比激动，，，

![第一轮追踪测试](/gallery/pictures/2019-9-22/2.png)

![第二轮追踪测试](/gallery/pictures/2019-9-22/3.png)

![第三轮追踪测试](/gallery/pictures/2019-9-22/4.png)

每个ip地址对应的实际地址已经在图片上了，其中第二三四列分别代表主机发出的三个独立的数据包产生的延迟。

本次实验特地选择了两个代表国内和国外的url<br>
tjublesson.top & www.jd.com

（真的不是在给京东打广告，我也尝试过淘宝，只不过最后被传输到了郑州大学的服务器，咱也不知道）

然后我又尝试了一下某“不存在的网站”：

![www.google.com](/gallery/pictures/2019-9-22/5.png)

好嘛，原来是直接ip阻断了啊，，，

## 3.实验结论

本次实验内容很简单，是作为对网络结构整体理解的一次辅助，可以看到在同级ip网段的连接处会产生较大的延迟，说明跨ISP的延迟相对更大。