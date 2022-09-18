---
title: Linux多线程编程计算pi值
date: 2019-10-28 22:28:38
tags: 
    - course
menu: 
    Go Home: /index.html
categories: 技术分享
cover: /gallery/thumbnails/2019-10-28.jpg
thumbnail: /gallery/thumbnails/2019-10-28.jpg
toc: true
comments: true
---
操作系统原理课程LAB2<br>
使用pthread系列函数进行多线程编程计算pi的近似值。

<!--more-->

# 一、实验原理

## 1. 计算公式

![计算公式](/gallery/pictures/2019-10-28/1.png)

## 2. pthread.h 包含pthread系列函数的头文件

- pthread_create() 函数原型：<br>
``` c
int pthread_create(pthread_t * thread, const pthread_arrt_t* attr,void*(*start_routine)(void *), void* arg);
```
>各参数含义：<br>
> (1) thread是线程标识符，类型为*int*；<br>
> (2) attr用于设置线程属性，如果为*NULL*则表示为默认属性；<br>
> (3) start_routine和arg表示新线程运行的函数和参数；<br>
> (4) 该函数返回值：成功返回0，失败返回错误号。

- pthread_join() 函数原型：<br>
``` c
void pthread_join(pthread_t thread,void ** retval);
```
>各参数含义：<br>
> (1) thread为新线程的标识符；<br>
> (2) retval有如下情况：
> 1. 如果thread线程通过return返回,value_ptr所指向的单元里存放的是thread线程函数的返回值。
> 2. 如果thread线程被别的线程调用pthread_cancel异常终掉,value_ptr所指向的单元里存放的是常数PTHREAD_CANCELED。
> 3. 如果thread线程是自己调用pthread_exit终止的,value_ptr所指向的单元存放的是传给pthread_exit的参数。 如果对thread线程的终止状态不感兴趣,可以传NULL给value_ptr参数。<br>

函数功能：调用该函数的线程将被**挂起等待**，直到标识符为*thread*的线程结束运行。

- pthread_exit() 函数原型：<br>
``` c
void pthread_exit(void * retval);
```
>各参数含义：<br>
> （1）retval是void *类型,其它线程可以调用pthread_join获得这个指针。需要注意,pthread_exit或者return返回的指针所指向的内存单元必须是全局的或者是由malloc分 配的,不能在线程函数的栈上分配,因为当其它线程得到这个返回指针时线程函数已经退出了。<br>
>（2）pthread_exit函数通过retval参数向线程的回收者传递其退出信息。它执行之后不会返回到调用者，且永远不会失败。

## 3. 计算过程与线程互斥操作

假设线程数量为t，则可将上述公式中的N平均分为t份，每份交给一个线程运行，最后再将计算得到的结果进行累加，在这里需要注意，累加的过程中涉及到了对同一变量的修改，出现临界区的问题，需要加锁进行解决，可使用*pthread_mutex_lock(&lock)* 和 *pthread_mutex_unlock(&lock)*。

## 4. 编译命令示例
``` 
gcc test.c –o test –lpthread
```


# 二、代码

``` c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <time.h>

int N,t;//声明全局变量
double *pret;//指向计算结果的指针
double ret;//每个线程的返回值
pthread_mutex_t lock;//锁变量

double total_time[12];//存放程序运行时间
//此处是为了存放下面注释掉的12组对比实验结果

void* thread(void *ID);

double multiThread(int t, int N, double *p_time);

int main(){
    printf("input N: ");
    scanf("%d", &N);
    printf("input t: ");
    scanf("%d", &t);

    double res = multiThread(t, N, &total_time[0]);
    printf("t=%d, N=%d, pi=%.15lf, total_time=%lf\n", t, N, res, total_time[0]);
    
    //以下代码分别测试线程数为2，4，6，8，
    //计算量分别为100000，1000000，10000000时的结果和运行时间。
    /*for(t = 2; t <= 8; t += 2){
        N = 100000;
        double res = multiThread(t, N, &total_time[(t/2)-1]);
        printf("t=%d, N=%d, pi=%.15lf, total_time=%lf\n", t, N, res, total_time[(t/2)-1]);
    }
    printf("\n");

    for(t = 2; t <= 8; t += 2){
        N = 1000000;
        double res = multiThread(t, N, &total_time[(t/2)-1]);
        printf("t=%d, N=%d, pi=%.15lf, total_time=%lf\n", t, N, res, total_time[(t/2)-1]);
    }
    printf("\n");

    for(t = 2; t <= 8; t += 2){
        N = 10000000;
        double res = multiThread(t, N, &total_time[(t/2)-1]);
        printf("t=%d, N=%d, pi=%.15lf, total_time=%lf\n", t, N, res, total_time[(t/2)-1]);
    }
    printf("\n");*/

    
    return 0;
}

double multiThread(int t, int N, double *p_time){
    int check_create;//检查线程是否成功创建
    int check_exit;//检查线程是否成功退出
    double sum = 0.0;

    clock_t start, finish;//程序运行的起止时间
    //double total_time = 0.0;

    pthread_t *threads;//线程标识数组
    void** threads_ret;//存放每个线程返回结果的数组

    threads = (pthread_t*)malloc(sizeof(pthread_t)*(t+1));//为每一个数组分配内存空间
    threads_ret = (void*)malloc(sizeof(void*)*(t+1));

    start = clock();//开始计时

    for(int i = 0; i < t; i++){
        check_create = pthread_create(&threads[i], NULL, thread, (void*)&i);//循环创建线程
        if(check_create != 0){
            printf("Thread creates failed\n");
            exit(1);
        }

        check_exit = pthread_join(threads[i], &threads_ret[i]);//等待线程结束并携带计算结果返回
        if(check_exit != 0){
            printf("Thread exit failed\n");
            exit(1);
        }

        pthread_mutex_lock(&lock);//执行累加操作前上锁
        sum += (*(double*)threads_ret[i]);
        pthread_mutex_unlock(&lock);//解锁
    }

    finish = clock();
    *p_time = (double)(finish - start) / CLOCKS_PER_SEC;//计算总时间，单位ms


    return sum;
}

void *thread(void *ID){
    int id = *(int*)ID;//使用ID判断当前线程计算哪一组
    ret = 0.0;//初始化结果

        for(int i = (N/t)*id; i < (N/t)*(id+1); i++){
            double tmp = (4 / (1+ ((i+0.5)/N)*((i+0.5)/N))) * (1.0/N);
            ret += tmp; 
        }
        pret = &ret;
        pthread_exit((void*)pret);//退出线程并返回计算结果 
    
}
```