---
title: 解决c#在高分辨率缩放下字体模糊
date: 2020-09-09 09:07:26
tags: c#
menu: 
    Go Home: /index.html
categories: 技术分享
cover: /gallery/thumbnails/2020-9-9.jpg
thumbnail: /gallery/thumbnails/2020-9-9.jpg
toc: true
---

众所周知在Windows笔记本上大多启用文本缩放125%，150%等，导致很多应用字体模糊。

<!--more-->

# 一、现象描述

在上一次写过网易云音乐缓存转换器后，我发现生成的.exe文件字体十分模糊，字体边缘不够锐利：

![](/gallery/pictures/2020-9-9/1.png)

# 二、解决方案

在参考了许多资料后，找到了有效方案，微软已经在C#（Winform）开发中内置了对于高分辨率的适配

1. 在`解决方案资源管理器`中右键项目名称，点击`添加-类`

![](/gallery/pictures/2020-9-9/2.png)

2. 选择`应用程序清单文件`

![](/gallery/pictures/2020-9-9/3.png)

3. 打开清单文件，找到以下代码，并将除中文注释外的部分解除注释。

```c#
<!-- 指示该应用程序可以感知 DPI 且 Windows 在 DPI 较高时将不会对其进行
       自动缩放。Windows Presentation Foundation (WPF)应用程序自动感知 DPI，无需
       选择加入。选择加入此设置的 Windows 窗体应用程序(目标设定为 .NET Framework 4.6 )还应
       在其 app.config 中将 "EnableWindowsFormsHighDpiAutoResizing" 设置设置为 "true"。-->
  
  <application xmlns="urn:schemas-microsoft-com:asm.v3">
    <windowsSettings>
      <dpiAware xmlns="http://schemas.microsoft.com/SMI/2005/WindowsSettings">true</dpiAware>
    </windowsSettings>
  </application>
```

使用此配置生成的Winform程序就不会有字体模糊问题了。

# 三、效果展示

![](/gallery/pictures/2020-9-9/4.png)