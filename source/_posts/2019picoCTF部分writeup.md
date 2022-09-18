---
title: 2019picoCTF部分writeup
date: 2019-10-01 22:27:56
tags: 
    - skills
    - CTF
menu: 
    Go Home: /index.html
categories: 技术分享
cover: /gallery/thumbnails/2019-10-01.jpg
thumbnail: /gallery/thumbnails/2019-10-01.jpg
toc: true
comments: true
---

# 一、写在前面的话

七十周年，岿然屹立，海晏河清，国泰民安。<br>
继先烈之遗志，泓炎黄之气概。<br>
这盛世，定如您所愿！<br>
**伟大的祖国母亲生日快乐！**

<!--more-->

# 二、picoCTF介绍

美国CMU主办，面向初高中生，难度偏简单，综合性强。<br>
~~唯一的缺点是我太菜了~~

# 三、部分题目wp

## 1. like 1000 | Forensics

一个似曾相识的层层嵌套压缩包，只不过我打包了100层，它打包了1000层。

![python代码](/gallery/pictures/2019-10-1/1.png)

最后里面有一个flag.png文件就是明文密码。

## 2. Flag | Crypto

一个信号旗组成的密码，使用的是国际通用信号旗。

![flag](/gallery/pictures/2019-10-1/2.png)

![信号旗](/gallery/pictures/2019-10-1/3.png)

这里有一点需要注意，大括号 { 右面第二个字符由于无法直接对应，所以经过尝试确定为数字 1


<font color = white> *手动感谢小白同学为我提供的思路支持* </font>

## 3. What Lies Within | Forensics

一张图片，猜测是图片隐写。

![buildings.png](/gallery/pictures/2019-10-1/4.png)

用Stegsolve打开，选择选择Data Extract, 按如图所示勾选，注意flag是不包括里面出现的空格的

![Stegsolve](/gallery/pictures/2019-10-1/5.png)


其他的可能会持续更新。。。