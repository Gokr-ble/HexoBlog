---
title: pikachu靶场系列——XSS
date: 2020-03-06 22:55:33
tags: CTF
menu: 
    Go Home: /index.html
categories: 技术分享
cover: /gallery/thumbnails/2020-3-6.jpg
thumbnail: /gallery/thumbnails/2020-3-6.jpg
toc: true
comments: true
---

https://www.cnblogs.com/dogecheng/p/11556221.html

概述什么的自己看吧

<!--more-->

## 反射型XSS（get）

随便输入并提交后可以发现，是使用get方法提交的参数，所以可以直接插入JS代码：

`<script>alert('xss')</script>`

![get](/gallery/pictures/2020-3-6/1.png)

此处前端会有输入长度限制，在F12控制台中取消即可。![删掉20](/gallery/pictures/2020-3-6/2.png)

![成功弹窗](/gallery/pictures/2020-3-6/3.png)

并且此时可以在源代码中看到插入的JS代码被直接嵌到标签中。

## 反射型XSS（post）

打开burpsuite，在网页中输入username和password后提交，可以看到post的具体信息

![post](/gallery/pictures/2020-3-6/4.png)

登录成功后可以像上一个get一样操作，并且这个没有最大长度限制。

（但是实际应用中需要自己搭一个恶意站点，详细参考开头的文章）

（搭站点这个技能树还没点亮）

## 存储型XSS

存储型XSS的特点就是恶意代码会被存放在网站后台，每当有用户访问到该处时便会触发。

插入代码和上面相同。

## DOM型XSS

查看网页源代码：![dom](/gallery/pictures/2020-3-6/5.png)

可以看到官方是给了提示的。

通过字符串拼接构造闭合payload

`' onclick="alert('xss')">`

![成功弹窗](/gallery/pictures/2020-3-6/6.png)

## XSS之过滤

由于过滤存在多种可能性，所以需要经验和尝试

通过测试可发现过滤了“<script”字符，因此尝试改用大写payload

`<SCRIPT>alert('xss')</SCRIPT>`

通常情况下还有其他绕过策略，如：

- 双写（拼凑），`<scri<script>pt>alert(111)</scri</script>pt>`。后台可能把<script>标签去掉换，但可能只去掉一次。
- 注释干扰，`<scri<!--test-->pt>alert(111)</sc<!--test-->ript>`。加上注释后可能可以绕过后台过滤机制。

（To be continued）