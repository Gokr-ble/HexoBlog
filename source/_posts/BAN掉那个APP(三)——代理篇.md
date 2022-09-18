---
title: BAN掉那个APP(三)——代理篇
date: 2020-03-09 20:51:39
tags: 
	- skills
	- android
menu: 
    Go Home: /index.html
categories: 技术分享
cover: /gallery/thumbnails/404.jfif
thumbnail: /gallery/thumbnails/404.jfif
toc: true
comments: true
updated: 2020-03-09 20:58
---

通过不断努力查找，这可能是迄今最接近可行解的方案（之一）……

<!--more-->

众所周知在一台Android设备没有root权限的情况下，是不可以随意修改只读目录`/system/etc/`下的hosts文件的……所以我转换一个思路，可以从网络代理方向下手，通过第三方app建立底层的V\*N连接，接管系统所有的网络流量，这样在V\*N的配置中就可以进行数据包的过滤。

## Postern

通过搜索引擎得到一款支持定制Hosts文件的代理app，postern。不得不说这款软件在国内十分小众，各大应用商店均无下载（也有可能存在V\*N的关系被下架了）链接会放在文末。

- 安装并打开该app后，第一次会询问是否允许建立V\*N连接，需要提前允许。

- 点击侧边栏，进入“配置Hosts文件”选项，

![配置Hosts文件](/gallery/pictures/2020-3-9/1.png)

- 点击“添加Host-IP”，将以下键值对输入进去：

| Host                      | IP        |
| ------------------------- | --------- |
| cdn-qukan.1sapp.com       | 127.0.0.1 |
| mediaconfig.qutoutiao.net | 127.0.0.1 |
| ddd.1sapp.com             | 127.0.0.1 |
| api.1sapp.com             | 127.0.0.1 |

![Host-IP](/gallery/pictures/2020-3-9/2.png)

经过测试，大概只有先打开代理app，再打开受害者app，才会生效。

![已经无法访问了](/gallery/pictures/2020-3-9/3.png)

只是这样的话，通知栏会常驻一个钥匙图标，还是挺容易发现的。



Postern下载地址：https://apktrending.com/apk-android/app-tunnelworkshop-postern.html

有点儿深，得到文中找