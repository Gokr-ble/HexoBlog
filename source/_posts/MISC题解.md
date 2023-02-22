---
title: MISC题解
date: 2023-02-22 13:41:17
tags: CTF
cover: /gallery/thumbnails/2023-02-22.jpg
thumbnail: /gallery/thumbnails/2023-02-22.jpg
---

## 流量分析

<!--more-->

## 4.3 misc流量分析练习-wireshark-https

解压得到wireshark-https.pcap，从题目中大致可以猜测和https/tls加密有关；Wireshark打开会提示文件大小异常，但仍可正常读取，猜测pcap文件可能存在其他捆绑数据；

![https1](gallery/pictures/2023-02-22/image-20221209150639448.png)

在kali中使用binwalk也能证实文件中存在一个隐藏的压缩包

![https2](gallery/pictures/2023-02-22/image-20221209151006017.png)

以压缩文件形式打开（7-zip，WinRAR），得到`sslkey.log`，该文件记录客户端与服务端双方交换密钥等过程，导入Wireshark即可对https加密流量进行解密。

Wireshark-编辑-首选项-Protocol-TLS-“Master-Secret log filename”，导入sslkey.log

![image-20221209151544397](gallery/pictures/2023-02-22/image-20221209151544397.png)

此时再应用过滤器http即可获得明文流量：

![image-20221209151645211](gallery/pictures/2023-02-22/image-20221209151645211.png)

注意到编号888的数据帧返回的MIME为`application/x-zip-compressed`，存在压缩文件（50 4b 03 04/PK）

![image-20221209151841591](gallery/pictures/2023-02-22/image-20221209151841591.png)

此处可以将数据复制为Hex Stream，使用python脚本将其转换为zip文件：

```python
import binascii

data = '504b0304140000000800b052374fa0ca55373f0000004400000008000000666c61672e747874f3ca34c8f2723630f2f2884a4e7549c94dadb404d3291e15603a2d24282f31cba938d2dda2ca3fdcc22420b7a4d4df3d2a303ccc2d34c223bddc37ac221900504b01021400140000000800b052374fa0ca55373f00000044000000080024000000000000002000000000000000666c61672e7478740a0020000000000001001800c2e8469db571d5019a24aa9db571d501438bc798b571d501504b050600000000010001005a000000650000000000'
hex2 = data.encode('utf-8')
sbin = binascii.unhexlify(hex2)
with open('cap.zip', 'wb') as f:
    f.write(sbin2)
print('done')
```

打开cap.zip得到flag.txt，内容是经过某种编码过后的flag

![image-20221209152834457](gallery/pictures/2023-02-22/image-20221209152834457.png)

### 4.4 misc流量分析练习-FTPPPP

解压后得到FTP.pcap，Wireshark打开，选择“统计”-“协议分级”，可见数据包中大部分为Telnet流量，少部分为TCP流量

![流量分级](gallery/pictures/2023-02-22/image-20221209144813197.png)

在过滤器中输入条件“ftp”并回车，查看FTP请求与响应，可以直接找到登录用户名和密码

![FTP](gallery/pictures/2023-02-22/image-20221209145120865.png)

后续有可能需要实验平台环境，此处暂时分析到这里

### 4.5 misc流量分析练习-数据包中的线索

解压后得到`流量中的线索.pcap`，Wireshark打开，统计-协议分级，可见HTTP流量居多

![协议分级-HTTP](gallery/pictures/2023-02-22/image-20221209145449456.png)

应用过滤器`http`，发现主机`GET /fenxi.php`，存在重大嫌疑，查看HTTP 200响应

![GET /fenxi.php](gallery/pictures/2023-02-22/image-20221209145645384.png)

![明文数据1](gallery/pictures/2023-02-22/image-20221209145747118.png)

![明文数据2](gallery/pictures/2023-02-22/image-20221209145821708.png)

拉到底部发现结尾存在“=”，推测是base64编码；在`Line-based text data: text/html`上右键-复制-...as Printable Text，找一个在线解码二进制base64的网站，复制、解码、下载得到一张图片，flag在图片中：

`flag{209acebf6324a09671abc31c869de72c}`

> https://the-x.cn/base64
>
> http://kylebing.cn/e/#/base64/base64-image

### 4.6 MISC流量分析练习-blueshark

解压得到blueshark.pcap，此题为蓝牙数据包相关；在统计-协议分级中`Bluetooth SBC Codec`占比最大

![image-20221209153056945](gallery/pictures/2023-02-22/image-20221209153056945.png)

尝试ctrl+f搜索zip, 7z, txt, flag等字符串，发现传输了`what_you_want.7z`文件

![image-20221209153516216](gallery/pictures/2023-02-22/image-20221209153516216.png)

可以用python脚本转换，也可直接用7z打开pcap文件，得到加密压缩包，从文件名得知密码为PIN码，回到Wireshark搜索pin，得到PIN为`141854`

![image-20221209154212707](gallery/pictures/2023-02-22/image-20221209154212707.png)

![image-20221209154121070](gallery/pictures/2023-02-22/image-20221209154121070.png)

解压文件得到flag

`flag{6da01c0a419b0b56ca8307fc9ab623eb}`

## 压缩包分析

### 4.8 MISC压缩包分析练习-扫描二维码签到

~~（瞧瞧这像话吗，这是一个签到题该有的难度吗）~~

题目可知和二维码相关，所给文件只有一张图片，强烈怀疑图片里一定有隐藏文件，binwalk证实了这一点

![image-20221209154742506](gallery/pictures/2023-02-22/image-20221209154742506.png)

将后缀名jpg改成zip，解压得到4个加密压缩包，`f.zip` `l.zip` `a.zip` `g.zip`，综合来看大概是将原始图片切割为四份，最后需要拼接得到完整的二维码

根据网上各位大佬的WP分析，四个压缩包采用了四种不同的加密方式

#### f.zip

此文件可以暴力破解，使用ARCHPR，选择payload：A-Z、a-z、0-9，位数4-6位，稍等即可得到密码`61825`

![image-20221209155330059](gallery/pictures/2023-02-22/image-20221209155330059.png)

解压即可获得f.jpg（1/4）

#### l.zip

此文件是伪加密（通过修改文件头特定位置的hex值让压缩软件认为其有加密）

使用010editor打开，发现`ZIP FILE RECORD`区的加密方式标志位（2字节）与正常不同，推断可能是伪加密，将其修改为0（无加密）

![image-20221209161334487](gallery/pictures/2023-02-22/image-20221209161334487.png)

使用同样的方法将`ZIP DIR ENTRY`区的标志位也修改为0，此时可以正常解密l.jpg（2/4）

![image-20221209161610421](gallery/pictures/2023-02-22/image-20221209161610421.png)

#### a.zip

打开压缩包发现有`1.txt` `2.txt` `a.jpg`三个文件，从结构猜测可能是爆破CRC32得到密码

```python
import binascii
import zipfile
import itertools

alph = 'abcdefghijklmnopqrstuvwxyz_'

crc1 = zipfile.ZipFile('a.zip', 'r').getinfo('1.txt').CRC
crc2 = zipfile.ZipFile('a.zip', 'r').getinfo('2.txt').CRC

print("computing all possible CRCs...")

for x in itertools.product(list(alph), repeat=6):
    st = ''.join(x)
    testcrc = binascii.crc32(st.encode('utf-8'))
    if testcrc == crc1:
        print(st, testcrc)
    elif testcrc == crc2:
        print(st, testcrc)

print("Done!")

# 以下为输出结果
# computing all possible CRCs...
# _pass_ 2669927281
# _word_ 3382550259
# Done!
```

得到1.txt的内容为`_pass_`，2.txt的内容为`_word_`，因此压缩包密码即为二者拼合`_pass__word_`

解压得到a.jpg（3/4）

#### g.zip

打开压缩包可以看到`plaintext.jpg(未加密)` `plaintext attack.jpg(加密)`，明示要用明文攻击

>  PS：由于不同压缩软件采用的压缩算法可能不同，建议通过复制副本、删除其他文件的方式制作明文攻击样本

复制g.zip为g2.zip，将g2.zip中的`g.jpg` `plaintext attack.jpg`删除，使用ARCHPR进行攻击

![image-20221209164018756](gallery/pictures/2023-02-22/image-20221209164018756.png)

得到密码`@Fa1se`，解压得到g.jpg（4/4）

最后使用任意软件将四张图片拼接，扫码得到flag（使用PowerPoint效果也是蛮好的）

![image-20221209164304804](gallery/pictures/2023-02-22/image-20221209164304804.png)

`flag{d62427d8f843c98dcfa89f0ba0d1e6fe}`

### 4.9 MISC压缩包分析练习-小明的保险箱

从压缩包中解压得到`小明的保险箱.jpg`，（此处省略binwalk检查捆绑），改后缀名为zip再次解压，得到1.rar（加密压缩包），使用ARCHPR暴力破解得到密码`7869`

![image-20221209164715267](gallery/pictures/2023-02-22/image-20221209164715267.png)

解压得到2.txt<br>`flag{75a3d68bf071ee188c418ea6cf0bb043}`

### 4.10 MISC压缩包分析练习-秘密藏在了哪里？

打开压缩包，文件名为wrongflag.txt，看来不是这个压缩包。同时又看到注释中有一长串奇怪的数字，以`50 4B 03 04`开头，大概率是压缩文件的十六进制串，使用上文的python代码将数字串转换为zip文件，打开得到trueflag.txt

`flag{c75af4251e0afde2fc62e0a76d856774}`

## 图片分析

### 4.12 MISC图片信息隐藏练习-大白

解压得到dabai.png，在windows上有的软件能打开，有的打不开，但有预览，使用kali工具pngcheck检查，提示IHDR节的CRC值错误，因此推测为爆破CRC得到图片正确的长宽值（此题中宽度固定，爆破高度）

![image-20221209165645066](gallery/pictures/2023-02-22/image-20221209165645066.png)

给出python脚本：

```python
import binascii
import struct

f = open('dabai.png', 'rb').read()

for i in range(679):
    data = f[12:20]+struct.pack('>i', i)+f[24:29]
    crc32 = binascii.crc32(data) & 0xffffffff
    if crc32 == 0x6D7C7135:		# 此处为真实的CRC值
        print(i)

# 得到运行结果：479
```

因此在010editor中将图片的高度改为479，即可看到正确的图像，flag隐藏在下半张图片中

`flag{He1l0_d4_ba1}`

### 4.13 MISC图片信息隐藏练习-刷新过的图片

搜索资料得知，有一种图片隐写技术——F5隐写

鉴于本题直接将F5隐写的读取代码给出，直接使用即可`java Extract F5.jpg`

![image-20221209170444697](gallery/pictures/2023-02-22/image-20221209170444697.png)

输出文件output.txt开头是熟悉的PK，因此修改后缀为zip，打开发现有加密，010editor打开，发现`ZIP FILE RECORD`部分加密flag=0但`ZIP DIR ENTRY`部分加密flag=1，考虑伪加密，将下面的1改为0

![image-20221209170923892](gallery/pictures/2023-02-22/image-20221209170923892.png)

解压得到flag.txt

`flag{96efd0a2037d06f34199e921079778ee}`

### 4.14 MISC图片信息隐藏练习-喵喵喵

解压得到mmm.png~~（是猫猫！嘿嘿~猫猫~嘿嘿~）~~

使用Stegsolve.jar逐通道检查，在R通道0号平面发现异常数据，Analyse-Data Extract提取

![image-20221209171326018](gallery/pictures/2023-02-22/image-20221209171326018.png)

测试得到在`位顺序: LSB, 位平面顺序: BGR, 位平面:RGB 000`得到有意义的数据，是另一张png图片

> 注意：这张PNG的头部存在多余字节，需要删除才能正常读取（PNG头部：89 50 4E 47）

打开发现是二维码的一半，且有的软件可读取，有的无法读取，使用上文的代码爆破CRC得到正确图片高度：280

![image-20221209171912490](gallery/pictures/2023-02-22/image-20221209171912490.png)

恢复后可以用Stegsolve反色后解码，总之又得到一个链接`https://pan.baidu.com/s/1pLT2J4f`，下载后是flag.rar文件~~（搁这套娃呢是吧）~~

查资料获知，这里还有NTFS文件隐写，WinRAR提取flag.txt后，用NtfsStreamEditor提取（这个软件会报毒）flag.pyc

> python文件在被import运行的时候会在同目录下编译一个pyc的文件（为了下次快速加载），这个文件可以和py文件一样使用，但无法阅读和修改；
>
> 在线反编译.pyc文件：https://tool.lu/pyc/

```python
import base64

def encode():
    flag = '*************'
    ciphertext = []
    for i in range(len(flag)):
        s = chr(i ^ ord(flag[i]))
        if i % 2 == 0:
            s = ord(s) + 10
        else:
            s = ord(s) - 10
        ciphertext.append(str(s))
    
    return ciphertext[::-1]

ciphertext = ['96','65','93','123','91','97','22','93','70','102','94','132','46','112','64','97','88','80','82','137','90','109','99','112']
```

根据反编译代码写解码脚本：

```python
ciphertext = ['96','65','93','123','91','97','22','93','70','102','94','132','46','112','64','97','88','80','82','137','90','109','99','112']
ciphertext = ciphertext[::-1]

flag = ''
for i in range(len(ciphertext)):
    # s = chr(i^ord(ciphertext[i]))
    if i % 2 == 0:
        s = int(ciphertext[i]) - 10
    else:
        s = int(ciphertext[i]) + 10
    s = i ^ s
    flag += chr(s)

print(flag)
```

运行得到flag

`flag{Y@e_Cl3veR_C1Ever!}`
