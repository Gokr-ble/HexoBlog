---
title: 安卓手游解包笔记
date: 2020-02-09 09:56:51
tags: skills
menu:
	Go Home: /index.html
categories: 技术分享
cover: /gallery/thumbnails/2020-2-9.jpg
thumbnail: /gallery/thumbnails/2020-2-9.jpg
toc: true
comments: true
---

# 使用unity制作的安卓手游解包

注：本文仅限技术讨论，请勿进行任何侵犯他人知识产权的行为。

<!--more-->

示例包名：com.hypergryph.arknights

## 解包游戏apk

由于apk是一种压缩文件格式，可以使用任何解压缩软件进行解包。

![avatar](/gallery/pictures/2020-2-9/1.png)

通过查阅资料可知，各个文件（夹）的作用如下：

|   文件（夹）名称    |           内容           |
| :-----------------: | :----------------------: |
|       assets        |  unity游戏所需资源文件   |
|         lib         | arm和x86所需要的.so文件  |
|      META-INF       |          信息包          |
|         res         |    存放图标等资源文件    |
| AndroidManifest.xml |     Android清单文件      |
|     classes.dex     | Android Dalvik可执行文件 |
|   resources.arsc    |  编译后的二进制资源文件  |

由于是要关注立绘、语音等资源，所以需要重点浏览assets目录。

如果对游戏机制和具体实现感兴趣，可以反编译classes.dex文件，那就是另外一个故事了。

## 查看资源文件

**使用工具：UnityStudio**（下载地址放在文末）

经努力查找，游戏中使用的美术资源、语音资源都放在assets/AB/Android中，推测AB为AssetBundle的缩写。

以下列出各文件的具体位置：



|        资源类别         |             位置              |
| :---------------------: | :---------------------------: |
|        人物语音         |   /audio/sound_beta_2/voice   |
|     攻击、技能音效      |  /audio/sound_beta_2/player   |
|         游戏BGM         |   /audio/sound_beta_2/music   |
|     立绘、游戏背景      |             /avg              |
|      其他美术资源       |             /arts             |
|    **主线剧情文本**     |    /gamedata/story/obt.ab     |
|      活动剧情文本       | /gamedata/story/activities.ab |
|  游戏内其他文本及提示   |      /i18n/string_map.ab      |
|      每章标题图片       |          /spritepace          |
| 历次寻访图片（存在bug） |           /ui/gacha           |
|   每章内部关卡背景图    |         /ui/stage.ab          |

来发个刀子

![avatar](/gallery/pictures/2020-2-9/2.png)

~~白兔子不要走！~~

~~最后希望yj快出第七章吧~~

参考资料：https://blog.csdn.net/u011611902/article/details/104154072

UnityStudio：

​	源码：https://github.com/Perfare/AssetStudio

​	latest release：https://ci.appveyor.com/project/Perfare/assetstudio/branch/master/artifacts