---
title: mumu模拟器启动时系统蓝屏解决
date: 2020-09-29 08:17:35
tags: skills
menu: 
    Go Home: /index.html
categories: 技术分享
cover: /gallery/thumbnails/2020-9-29.jpg
thumbnail: /gallery/thumbnails/2020-9-29.jpg
toc: true
comments: true
---

本文关键词：WSL2，Hyper-V，安卓模拟器

<!--more-->

### 事件起因

听说WSL（Windows Subsystem for Linux）以及升级版WSL2是最好的Linux发行版（误），遂搞一搞体验一下。按照教程打开`控制面板-程序和功能-添加或删除Windows功能-适用于Linux的Windows子系统`，重启电脑，在Microsoft Store下载Ubuntu 20.04 LTS，等待片刻就可以使用了。

接下来是升级WSL2，由于相比于WSL，WSL2在内核层面进行了全新的升级，两者之间差异巨大。

升级官方教程：

> https://docs.microsoft.com/zh-cn/windows/wsl/install-win10

晚上打开Mumu模拟器想来一把手游，结果尝试多次后发现，启动加载到42%系统必蓝屏崩溃……

![](/gallery/pictures/2020-9-29/1.jpg)

ヽ(｀Д´)ﾉ︵ ┻━┻ ┻━┻ 坑爹呢这是！

### 解决过程

尝试删除Ubuntu20.04并关闭`适用于Linux的Windows子系统`后并没有效果，然后安装另一款模拟器蓝叠，在打开时提示请关闭Hyper-V，遂将问题定位到Hyper-V与模拟器的兼容性上。

打开系统管理-服务，发现所有Hyper-V相关服务都是关闭状态，所以我陷入了沉思……

多方搜索资料后，找到以下解决方案：

1. 以管理员模式运行CMD，依次输入：

   ```bash
   bcdedit /copy {current} /d “Windows10 no Hyper-V”
   bcdedit /set {上一条命令得到的序列号} hypervisorlaunchtype OFF
   ```

2. 此时重启计算机，会发现启动时有两个选项，Windows 10和Windows10 no Hyper-V，选择第二个，进入系统，再次以管理员模式运行CMD，输入以下命令：

   ```bash
   bcdedit									#查看当前启动管理器配置列表
   bcdedit /set description "Windows 10"	#将当前启动配置项的描述改为Windows 10
   bcdedit /delete {default}				#删除原来的default配置
   bcdedit /default {current}				#将当前配置设置为default
   ```

3. 此时再使用`bcdedit`命令查看启动管理列表，发现当前控制项最后一行的`hypervisorlaunchtyper`为off，说明成功了，再打开mumu模拟器也不会崩溃，打开蓝叠也不会提示关闭Hyper-V了。

### 题外话

强推Windows Terminal，美观大气高颜值，可自定义外观字体和背景亚克力效果，中文渲染不会乱码，支持linux命令在windows下的使用，高效整合CMD，PowerShell，Git Bash等多种终端，赶紧到Microsoft Store体验一下吧！

![](/gallery/pictures/2020-9-29/2.png)