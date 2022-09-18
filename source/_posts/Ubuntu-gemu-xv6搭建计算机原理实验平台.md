---
title: Ubuntu+gemu+xv6搭建计算机原理实验平台
date: 2019-09-11 22:49:41
tags: 
    - course
categories: 技术分享
cover: /gallery/thumbnails/2019-9-11.png
thumbnail: /gallery/thumbnails/2019-9-11.png
toc: true
comments: true
---
----

更新于2019-09-16：<br>
修复了无法成功在虚拟机中加载的bug

----
## 背景：必修课 操作系统原理

# 一、 准备工具：
VMware Workstation 15 Player <br> 
Ubuntu 18.04 <br> 
QEMU虚拟机 <br> 
Xv6实验平台

<!--more-->

>QEMU（quick emulator）是一款由Fabrice Bellard等人编写的免费的可执行硬件虚拟化的（hardware virtualization）开源托管虚拟机（VMM）。<br> *来源：百度百科*


>Xv6是由麻省理工学院(MIT)为操作系统工程的课程（代号6.828）,开发的一个教学目的的操作系统。Xv6是在x86处理器上(x即指x86)用ANSI标准C重新实现的Unix第六版(Unix V6，通常直接被称为V6) 。<br> *来源：维基百科*
# 二、实验原理
在Windows环境中利用VMware虚拟机安装Ubuntu平台，并在Ubuntu上利用QEMU虚拟机安装今天的主角Xv6。

 （大概类似俄罗斯套娃的结构吧）

# 三、实验步骤

## 1.安装VMware Workstation 15 Player    
这个教程就不写了，本专业的应该都会吧，（大概）    
## 2.在VMware上安装Ubuntu 18.04
这个教程随便谷歌一大堆，就不写了吧。
## 3.在Ubuntu里安装QEMU
安装QEMU的方法有很多，可以git clone源码然后自己编译，也可以采用apt-get安装，代码如下：
```
sudo apt-get install qemu
```
安装之后打开终端测试安装是否成功，输入：
```
qemu-system-i386
```
![qemu](/gallery/pictures/2019-9-12/1.png)

注：如果安装的简化版Ubuntu没有git、vim的话，使用以下命令安装：
```
sudo apt-get install git
sudo apt-get install vim
```
## 4.克隆Xv6源码，编译并安装
使用以下命令clone一份Xv6源代码：
```
git clone https://github.com/mit-pdos/xv6-public.git
```
cd进入Xv6源代码的目录，在终端中使用*make*命令编译

注：1. 此时可能会遇到没有安装gcc的情况，使用
```
sudo apt-get update
sudo apt-get install gcc
```
安装gcc，之后就可以编译了。

2. 如果仍然报编译错误，
>lib/printfmt.c:42：对‘__udivdi3’未定义的引用 <br>
lib/printfmt.c:50：对‘__umoddi3’未定义的引用

原因：由于在printfmt.c文件中用了libgcc.a中的库函数，但是我的开发环境是64位的gcc，所以找不到这个库文件。

解决：
```
sudo apt-get install gcc-multilib
sudo apt-get install ia32-libs lib32gcc1 lib32stdc++6
```

**所有步骤完成后，在终端输入make qemu就可以正常启动了**

# 四、结尾
经过一晚上的折腾，总算是成功安装了QEMU和Xv6，但是我遇到了如下问题：

![booting from hard disk](/gallery/pictures/2019-9-12/2.png)

QEMU始终卡在Booting from hard disk...语句，并不断闪烁，希望能找到解决办法。

----
2019-09-16

重新clone了一份Xv6的源代码，再次make编译，然后在根目录使用make qemu，这次就成功了（果然重装解决99.9%的问题）

![How clever I am](/gallery/pictures/2019-9-12/3.png)

![敲一个ls命令试试](/gallery/pictures/2019-9-12/4.png)

Eureka!

![乌拉！](/gallery/pictures/wula.jpg)