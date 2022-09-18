---
title: BAN掉那个APP(一)——Kali篇
date: 2020-02-01 18:39:43
tags: 
	- skills
	- kali
	- android
menu: 
    Go Home: /index.html
categories: 技术分享
cover: /gallery/thumbnails/404.jfif
thumbnail: /gallery/thumbnails/404.jfif
toc: true
comments: true
---

## Kali的正确使用——父慈子孝

前情提要：回家一个月，不巧赶上疫情爆发，只好在家久宅。家父鉴于生活习惯，勤于节俭，锱铢之事必较于细。曾几何时陷入“趣*条”挣钱无法自拔，殊不知自已委身广告商，每日辛苦所得不过一二元余，令人甚是叹惋。

<!--more-->



## 一、ettercap搭配arpspoof

打开kali linux，虚拟机网络模式选择桥接模式，以确保kali与目标主机在同一网段。使用带有图形界面的ettercap，

1. 在菜单栏选择`Sniff-Unified sniffing-OK`开始嗅探；

2. 在菜单栏选择`Hosts-Scan for hosts`扫描局域网所有主机，此处可能扫描不到，建议多尝试几次。

   ![hosts](/gallery/pictures/2020-2-1/1.png)

3. 使用某种手段确定家父的手机IP为192.168.1.101。

4. 在命令行中键入`arpspoof -i eth0 -t 192.168.1.105 192.168.1.1`开启转发，此时来自目标主机的流量都会被转发至网关（大概），从而使其得不到真正的DNS解析。

5. 在家父不断抱怨为什么网络质量这么差的时候忍住不笑。



## 二、咱不是有后台嘛

打开安卓模拟器，安装HTTP Canary、抓包精灵和某受害者app，开始抓包。

![抓包记录](/gallery/pictures/2020-2-1/2.png)

可知该app访问的URL主要有：

> http://api.1sapp.com/
>
> http://apk.1sapp.com/
>
> http://api.weibo.com/
>
> http://mediaconfig.qutoutiao.net/
>
> http://rcv.aiclk.com/

2020-2-24新增：

> https://message-push.1sapp.com/
>
> https://html2.qktoutiao.com/
>
> http://cdn-qukan.1sapp.com/
>
> https://ddd.1sapp.com/
>
> 以及云服务提供商：腾讯云 119.29.29.29

浏览器键入192.168.1.1进入路由器后台管理界面

![路由器](/gallery/pictures/2020-2-1/3.png)

点击管理按钮，添加禁止访问记录

![禁止访问](/gallery/pictures/2020-2-1/4.png)

将以上URL添加到黑名单中。



> 注：按以上内容操作，不排除被发现的可能，以及不提供如何面对父慈子孝的经典场面的教程。

