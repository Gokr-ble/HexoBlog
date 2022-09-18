---
title: CTF系列——Misc之PNG的CRC32校验
date: 2020-03-13 19:15:22
tags:
	- CTF
	- misc
menu: 
    Go Home: /index.html
categories: 技术分享
cover: /gallery/thumbnails/2020-3-13.jpg
thumbnail: /gallery/thumbnails/2020-3-13.jpg
toc: true
comments: true
---

参考资料：https://wiki.x10sec.org/misc/picture/png/

<!--more-->

题目是一张名为crc.png的PNG图片，在windows下并不能正常打开，怀疑是该PNG的宽度、高度被修改，导致CRC校验未通过。

使用010Editor打开该图片，看到如下信息：

![Hex信息](/gallery/pictures/2020-3-13/1.png)

- 前八个字节`89 50 4E 47 0D 0A 1A 0A`是PNG格式固定的文件头；
- `00 00 00 0D`代表图片长宽的数据块长度为13，也是固定值；
- 第一个方框中`49 48 44 52`代表IHDR，固定值；
- **第二行开始的`00 00 01 00`为图片宽度，`00 00 00 00`为图片高度；**
- 由于数据块长度为13，所以`08 02 00 00 00`为剩余部分；
- **`12 9C B6 9B`为CRC32校验和。**

> 正常情况下，windows系统的图片查看器是可以忽略CRC校验失败而直接打开图片的，但由于这张图片的高度是0，所以，，无论如何也不能正常打开。而Linux下不可以直接打开CRC校验失败的图片。

先将高度改为01 00试试：

![。。。](/gallery/pictures/2020-3-13/2.png)

这辣眼睛的色彩。。。看起来是不对的

参考网络上的相关资料，需要考虑爆破CRC校验，所以上python

## 版本一

```python
import binascii
import struct

# \x49\x48\x44\x52\x00\x00\x01\x00\x00\x00\x00\x00\x08\x02\x00\x00\x00
crc32key = 0x129CB69B

for i in range(0, 65535):
    for j in range(0, 65535):
        width = struct.pack('>i', j)
        height = struct.pack('>i', i)
        data = b'\x49\x48\x44\x52' + width + height + b'\x08\x02\x00\x00\x00'
        crc32result = binascii.crc32(data) & 0xffffffff
        if crc32result == crc32key:
            print(''.join(map(lambda c: "%02X" % c, width)))
            print(''.join(map(lambda c: "%02X" % c, height)))


```

该版本在python 3.8运行通过。

得到答案如下：

![width x height](/gallery/pictures/2020-3-13/3.jpg)

将图片的宽度和高度修改为以上结果，可以还原初始图片

![answer](/gallery/pictures/2020-3-13/4.png)

## 版本二

该代码来源于CTFwiki官方：

```python
import os
import binascii
import struct


misc = open("此处选择图片","rb").read()

for i in range(1024):
    data = misc[12:16] + struct.pack('>i',i)+ misc[20:29]
    crc32 = binascii.crc32(data) & 0xffffffff
    if crc32 == "此处填写CRC校验和":
        print i
```

声明：该代码未经测试，且对于此题需要同时爆破高度和宽度，这里只给出一个维度的循环，需仿照上面的代码进行修改。（嘛，懒得再测试一遍了，反正结果一样~

总结：以前的CRC经常是给出宽度或者高度，另一个值需要自己破解，但是这次居然同时需要两个值，算是一个小坑吧（我太菜了，逃

特别鸣谢：手动@谢大佬