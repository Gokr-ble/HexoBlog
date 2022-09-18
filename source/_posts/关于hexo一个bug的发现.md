---
title: 关于hexo一个bug的发现
date: 2020-09-10 11:52:19
tags: bug
menu: 
    Go Home: /index.html
categories: 日常划水
cover: /gallery/thumbnails/2020-9-10.jpg
thumbnail: /gallery/thumbnails/2020-9-10.jpg
toc: true
---

昨天我写了一篇《解决CSharp在高分辨率缩放下字体模糊》的文章，今天打开时发现网站提示404，于是各种研究原因。

<!--more-->

1. 首先怀疑和最近git bash、Android Studio莫名乱码有关，但后来发现并不是。
2. 然后怀疑是文件名、标题、tag中含有特殊字符`#`的关系，于是逐一排查。

最后发现，在文章标题（.md文件中）、tag（.md文件中）均含有`#`字符时，网页不会404，唯独在.md文件的文件名中出现则会崩溃，因为网页默认按照文件名创建同名目录，可能是在解析url时`#`产生了影响。

所以，换掉文件名中的`#`就好了。