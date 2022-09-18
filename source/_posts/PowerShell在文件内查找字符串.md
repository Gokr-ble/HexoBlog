---
title: PowerShell在文件内查找字符串
date: 2022-09-15 14:07:47
tags: skills
menu: 
    Go Home: /index.html
categories: 技术分享
cover: /gallery/thumbnails/2022-09-15.jpg
thumbnail: /gallery/thumbnails/2022-09-15.jpg
toc: true
comments: true
---

# 命令1


```powershell
Get-ChildItem -Path .\ -Recurse | Select-String -Pattern 'Hello' -List | Select Path
```
<!--more-->
> - `Get-ChildItem`：获取特定目录及其子项；`-Recurse`递归所有文件、目录、子目录
> - `Select-String`：在指定文件中查找字符串

# 命令2

```powershell
ls -r -Path C:\pc | sls 'Hello' | select -u Path
```

> - `ls`：类似于Linux系统命令，使用`-r`参数列举指定位置的目录及子目录
> - `sls`：是上一个命令里`Select-String`的别名

# 总结

Powershell在功能性上非常丰富且优雅，嗯。
