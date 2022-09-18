---
title: CTF之pwn入门体验
date: 2019-09-27 21:05:37
tags: skills
menu: 
    Go Home: /index.html
categories: 技术分享
cover: /gallery/thumbnails/2019-9-27.png
thumbnail: /gallery/thumbnails/2019-9-27.png
toc: true
comments: true
---
一入pwn坑深似海，从此头发是路人，，，

<!--more-->

# 一、前言

考虑到协会发展需要，在新生代成员中得至少有一个搞pwn的队员，这样在比赛中才能不吃二进制的亏，，，作为负责人怎么能不担此重任（滑稽

这次的三道题都是超级简单的入门题目，前两道是pwn，最后一道是reverse，权当长见识了。

# 二、题目
## 1.buffOverflow#0

```
#include <stdio.h>
#include <signal.h>

void win()
{
	printf("You win!\n");
	char buf[256];
	FILE* f = fopen("./flag.txt", "r");
	if (f == NULL)
	{
		puts("flag.txt not found - ping us on discord if this is happening on the shell server\n");
	}
	else
	{
		fgets(buf, sizeof(buf), f);
		printf("flag: %s\n", buf);
	}
}

void vuln()
{
	char buf[16];
	printf("Type something>");
	gets(buf);
	printf("You typed %s!\n", buf);
}

int main()
{
	/* Disable buffering on stdout */
	setvbuf(stdout, NULL, _IONBF, 0);

	/* Call win() on SIGSEGV */
	signal(SIGSEGV, win);

	vuln();
	return 0;
}
```

典型的缓冲区溢出<br>
可以看到main函数中首先调用了 **signal(SIGSEGV, win);** 接下来运行的 **vuln()** 用来读取并储存输入的字符串，所以一旦程序产生segmentation fault(段错误)，就会被signal信号捕获，进而执行 **win()** 函数，也就是读取flag文件。

所以用某殷姓学长的话来说，这个程序，进去~~瞎jb~~敲一堆字符串就好了 φ(≧ω≦*)♪

![成功造成缓冲区溢出](/gallery/pictures/2019-9-27/1.png)

此处显示flag.txt not found是因为程序在本地测试，真正题目环境需要nc ip 端口进行连接，就可以正常读取flag.txt了。

## 2.bufOverflow#2

```
#include <stdio.h>

void win()
{
	printf("You win!\n");
	char buf[256];
	FILE* f = fopen("./flag.txt", "r");
	if (f == NULL)
	{
		puts("flag.txt not found - ping us on discord if this is happening on the shell server\n");
	}
	else
	{
		fgets(buf, sizeof(buf), f);
		printf("flag: %s\n", buf);
	}
}

void vuln()
{
	char buf[16];
	printf("Type something>");
	gets(buf);
	printf("You typed %s!\n", buf);
}

int main()
{
	/* Disable buffering on stdout */
	setvbuf(stdout, NULL, _IONBF, 0);

	vuln();
	return 0;//$eip=0x80491b2
}
```

还是缓冲区溢出<br>
仍然是从代码中可以看出，这道题与上一道题的区别在于，main函数里没有了signal()函数进行段错误信号捕获，只好通过某种技术手段跳转到win()函数执行。

打开IDA，查看win()函数的地址：

![win()函数](/gallery/pictures/2019-9-27/2.png)

嗯，，，就是c代码中 *return 0;* 处注释的地址，，，所以我的目的很明显，就是运行到 *return 0;* 时中断，修改$eip寄存器的值为win()函数的地址，这样就成功把程序引入win()函数。

此处插入函数运行基础知识：
>在介绍如何实现溢出攻击之前，让我们先简单温习一下函数调用栈的相关知识。
>函数调用栈是指程序运行时内存一段连续的区域，用来保存函数运行时的状态信息，包括函数参数与局部变量等。称之为“栈”是因为发生函数调用时，调用函数（caller）的状态被保存在栈内，被调用函数（callee）的状态被压入调用栈的栈顶；在函数调用结束时，栈顶的函数（callee）状态被弹出，栈顶恢复到调用函数（caller）的状态。函数调用栈在内存中从高地址向低地址生长，所以栈顶对应的内存地址在压栈时变小，退栈时变大。

>Fig 2. 函数调用发生和结束时调用栈的变化<br>
>函数状态主要涉及三个寄存器－－esp，ebp，eip。esp 用来存储函数调用栈的栈顶地址，在压栈和退栈时发生变化。ebp 用来存储当前函数状态的基地址，在函数运行时不变，可以用来索引确定函数参数或局部变量的位置。eip 用来存储即将执行的程序指令的地址，cpu 依照 eip 的存储内容读取指令并执行，eip 随之指向相邻的下一条指令，如此反复，程序就得以连续执行指令。<br>
>下面让我们来看看发生函数调用时，栈顶函数状态以及上述寄存器的变化。变化的核心任务是将调用函数（caller）的状态保存起来，同时创建被调用函数（callee）的状态。<br>
>首先将被调用函数（callee）的参数按照逆序依次压入栈内。如果被调用函数（callee）不需要参数，则没有这一步骤。这些参数仍会保存在调用函数（caller）的函数状态内，之后压入栈内的数据都会作为被调用函数（callee）的函数状态来保存。

不想copy了直接贴原文章吧https://paper.seebug.org/271/

总之先打开pwndbg调试着：

![在main()下断点](/gallery/pictures/2019-9-27/3.png)

![在return 0下断点](/gallery/pictures/2019-9-27/4.png)

![查看并修改eip](/gallery/pictures/2019-9-27/5.png)

![成功导向win()函数](/gallery/pictures/2019-9-27/6.png)

然而以上只是我验证自己猜想的过程，这时候只好求助于学长~

![殷大佬NB](/gallery/pictures/2019-9-27/7.png)

（鉴于我并没有搞懂函数栈帧结构，payload这部分以后再填）

## 3.parrot(reverse)

这是平台上的一道逆向题，IDA打开

![查一下strings](/gallery/pictures/2019-9-27/9.png)

值得注意的是，这里有一个硬编码近函数的字符串
```
.data:0804A041	00000032	C	jctf{my_b3l0v3d_5qu4wk3r_w0n7_y0u_l34v3_m3_4l0n3}

```
这里是第一个坑，千万别一激动直接就提交了Orz

再来左侧函数窗口选中main，F5反编译

![main函数很简单](/gallery/pictures/2019-9-27/8.png)

还是输入字符串并进行判断的程序，通过分析可知该程序的判断逻辑

1. 判断输入字符串长度是否等于34，否则直接退出；
2. 判断第0~5位是否为 *tjctf{*
3. 判断第6~8位是否为 *3d_*
4. 判断第10~33位是否为 *0n7_y0u_l34v3_m3_4l0n3}*
5. 最后检查第9位是否为 *d*

说明：
- 由于程序硬编码的字符串是jctf{开头，而不是tjctf{，这时就要注意到 0x804A040 这个地址：

![0x804A040](/gallery/pictures/2019-9-27/10.png)

- 这就解释了第一个字符t是哪里来的

- 第5步中提到检查第9位是否为字符'd'，是因为在最开始的变量声明中，v8被声明为char类型，在此处被和100比较，而ASCII=100的字符恰好是'd'。

综上，我们就可以拼出一份完整的flag：
```
tjctf{3d_d0n7_y0u_l34v3_m3_4l0n3}
```

（don't you leave me alone吗？）

# 三、总结

这两个个pwn和一个reverse仅仅是最基础最入门的内容，要想达到丁佬殷佬的高度还远着呢~

o( =•ω•= )m

━━(￣ー￣*|||━━

0:13，睡觉，养生，论当代大学生的个人生活