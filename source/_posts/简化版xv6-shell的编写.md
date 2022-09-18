---
title: 简化版xv6 shell的编写
date: 2019-10-19 15:04:53
tags: 
    - skills
    - course
menu: 
    Go Home: /index.html
categories: 技术分享
cover: /gallery/thumbnails/2019-10-19.jpg
thumbnail: /gallery/thumbnails/2019-10-19.jpg
toc: true
comments: true
---
操作系统原理课程实验一，要求在给定的xv6 shell框架中补全执行普通命令、重定向命令、管道命令的功能。

注：所有命令执行过程均写在runcmd()中即可，命令的解析在外部已经实现。

<!--more-->

# 一、普通命令

```
case ' ':
    ecmd = (struct execcmd*)cmd;
    if(ecmd->argv[0] == 0)
        _exit(0);
    //fprintf(stderr, "exec not implemented\n");
    execlp(ecmd->argv[0], ecmd->argv[0], ecmd->argv[1], NULL);
    break;
```

已知argv[0]存放的是命令本体，argv[1]存放的是命令参数，根据execlp()的参数表，，，

# 二、重定向命令

```
case '>':
case '<':
    rcmd = (struct redircmd*)cmd;
    //fprintf(stderr, "redir not implemented\n");
    close(rcmd->fd);
    open(rcmd->file, rcmd->flags);
    runcmd(rcmd->cmd);
    break;
```

在Linux中，文件标识符0默认连接stdin，1默认连接stdout，2默认连接stderr<br>
那么，我们需要做的就是首先关闭当前文件的标准输出（或输入），此时fd已根据‘<’或‘>’被设置为0或1；<br>
然后再调用open()函数打开待读/写文件，此时这个文件会被分配当前可用最小文件标识符，即0或1；<br>
这样就起到了覆盖原有的标准输入、输出而实现重定向了。

# 三、管道命令

```
case '|':
    pcmd = (struct pipecmd*)cmd;
    //fprintf(stderr, "pipe not implemented\n");
    pipe(p);
    if(fork1() == 0){
      close(1);
      dup(p[1]);
      close(p[0]);
      close(p[1]);
      runcmd(pcmd->left);
    }
    if(fork1() == 0){
      close(0);
      dup(p[0]);
      close(p[0]);
      close(p[1]);
      runcmd(pcmd->right);
    }
    close(p[0]);
    close(p[1]);
    wait(&r);
    wait(&r);
    break;
```

管道的实现比较复杂。首先，管道是一种特殊的文件，一般不用open()函数进行操作；其次，管道命令的执行需要创建两个子进程分别执行左右命令，然后执行左命令的进程用管道文件覆盖stdout，执行右命令的进程覆盖stdin.这样就可以实现左命令将结果放入管道，右命令从管道读取结果并作为输入执行。而父进程即当前shell进程需要等待两个子进程退出，所以需要两个wait()函数。
