---
title: BAN掉那个APP(二)——hosts篇
date: 2020-03-08 19:33:15
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
---

书接上文：[https://tjublesson.top/2020/02/01/%E5%B1%80%E5%9F%9F%E7%BD%91kali%E5%AE%9E%E9%AA%8C/](https://tjublesson.top/2020/02/01/局域网kali实验/)

<!--more-->

今日随心情所喜好，再次选择使用路由器后台方式限制家父使用某app，未曾想在发现wifi无法正常使用app后，转而使用了流量……（话说使用流量我还怎么控制啊）

只好研究一种基于Android底层的流量控制方法了

## 修改hosts文件

安卓系统基于linux内核，hosts文件位于`/system/etc/`中，默认情况下该目录处于只读权限。

实验中采用adb+模拟器的配置。

1. 在模拟器中打开该app，此时是可以正常访问的。![avatar](/gallery/pictures/2020-3-8/1.png)
2. 以管理员权限运行cmd，依次输入：![avatar](/gallery/pictures/2020-3-8/2.png)

```
adb connect 127.0.0.1:7555
adb root
adb remount
```

3. 将原有host文件转移到主机中：`adb pull /system/etc/hosts E:/apk-Decompile/`

> 此处最开始我选择了在windows下直接修改好后再转移到android上，但并不起效，查询资料可知，可能是**windows的换行符与android中的并不通用**所致，因此采取以下方法进行hosts的修改

4. ```
   adb shell	//进入adb shell
   echo -e \\n >> /system/etc/hosts	//新建一行
   echo 127.0.0.1 mediaconfig.qutoutiao.net >> /system/etc/hosts
   echo 127.0.0.1 api.1sapp.com >> /system/etc/hosts
   echo 127.0.0.1 cdn.aiclk.com >> /system/etc/hosts
   echo 127.0.0.1 ddd.1sapp.com >> /system/etc/hosts
   echo 127.0.0.1 static.1sapp.com >> /system/etc/hosts
   ```

5. 此时使用`cat /system/etc/hosts`查看是否修改成功![avatar](/gallery/pictures/2020-3-8/3.png)

6. 再次进入app并刷新，可以发现已经无法正常访问了![avatar](/gallery/pictures/2020-3-8/4.png)

然而这种方式有局限，即必须获取到手机的root权限才可以操作，经过实验，在未root过的设备上进行adb root命令并不会起作用，所以该次实验仅做理论上的验证。

> 其实中老年人日常在家也并没有什么事可做，这种用来消磨时间还不搭钱的方式也无可厚非，毕竟，陪伴才是缓解无聊的良药。

>  2020-3-9更新：在模拟器抓包此app并分析其访问的url时曾遇到一个问题，即不开启抓包的情况下，app可以正常连接网络，但开启抓包后，app不能正常访问网络，且各项设置均正常。查阅相关资料后了解到，安卓app在进行https访问时，使用自签名证书+SSL Pinning的方式进行服务器身份验证，而抓包软件的开启使得证书校验失败，无法获取服务器返回信息。
>
> 解决方案：在**已取得root权限**的情况下，安装Xposed框架+JustTrustMe组件，该组件会将apk中所有用于校验SSL证书的API进行hook，从而绕过证书检查。
>
> 参考资料：https://m.imooc.com/article/251500?from=singlemessage
>
> Xposed官方安装地址：https://repo.xposed.info/module/de.robv.android.xposed.installer
>
> JustTrustMe Github地址：https://github.com/Fuzion24/JustTrustMe/releases/tag/v.2

（完）
