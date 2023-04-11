---
layout: '../../layouts/MarkdownPost.astro'
title: '[Linux] 多线程相关分析'
pubDate: 2023-04-11
description: ''
author: '七月.cc'
cover:
    url: 'https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/CSDN/image-20230411163745002.png'
    square: 'https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/CSDN/image-20230411163745002.png'
    alt: 'cover'
tags: ["Linux", "线程", "系统"]
theme: 'light'
featured: false
---

![ ](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/CSDN/image-20230411163745002.png)

---

我们已经了解了Linux操作系统进程部分相关知识：

> 博主有关Linux进程相关介绍的文章：
>
> - 💥[[Linux\] 系统进程相关概念、系统调用、Linux进程详析、进程查看、fork()初识](https://www.julysblog.cn/posts/Linux-Process-Concept&Processes)
>
> - 💥[[Linux\] 进程状态相关概念、Linux实际进程状态、进程优先级](https://www.julysblog.cn/posts/Linux-Process-States&System-Process-Actual-States)
>
> - 💥[[Linux\] 什么是进程地址空间？父子进程的代码时如何继承的？程序是怎么加载成进程的？为什么要有进程地址空间？](https://www.julysblog.cn/posts/Linux-Process-Addr-Space)
>
> - 💥[[Linux\] 详析进程控制：fork子进程运行规则？怎么回收子进程？什么是进程替换？进程替换怎么操作？](https://www.julysblog.cn/posts/Linux-Process-Control)

通过阅读这几篇文章, 至少可以让我们对Linux系统中的进程 有一个最基本又相对全面的认识.

但是今天这篇文章, 可能又会对之前介绍过的进程多少有一些推翻.

本篇文章的主要内容是：Linux操作系统中, 有关多线程的相关介绍.

# Linux线程概念

线程可以说是实际区别于进程的一个概念, 但也可以说是实际没有区别于进程的一个概念.

而实际区别与否, 其实与平台有关

## 什么是线程

有关线程的概念, 大概可以通过三个要点介绍: 

1. 线程是在进程内部运行的执行流
2. 线程相比进程, 粒度更细, 调用成本更低
3. 线程是CPU调度的基本单位

不过这三个要点只能让你大概的对线程有一个最最最基本的认识：线程比进程要小. 但具体怎么小 是不知道的.

不过可以举个例子来简单的介绍一下, Linux下的线程：

在之前有关进程的介绍中, Linux系统中的进程 = PCB + 被加载到内存中的程序数据, 不过 PCB和内存中的程序数据 并不是直接相映射的.

之间还要通过 进程地址空间和相应的页表, 不过CPU实际只是是通过访问PCB 来实现对进程的调度的：

![ ](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/CSDN/image-20230411175200325.png)

`PCB(task_struct)中描述着进程地址空间, 进程地址空间与物理内存 通过两张页表来相互映射.` 

这是Linux系统中, 单个进程实际在操作系统中的存在形式.

系统创建进程会创建这所有的格式和数据.

不过我们也介绍过, 如果通过fork创建子进程, 在未作数据修改时 子进程与父进程是共享进程的数据和代码的. (子进程也存在自己的进程地址空间和页表, 只不过指向同一块数据和代码)

而且, 我们可以`通过对fork()返回值的判断, 让父子进程执行不同的代码块`.

这, 其实说明了一个 **细节**：**`不同的执行流, 可以做到执行不同的资源, 即 可以做到对特定资源的划分`**

那么, 如果下次创建进程, 操作系统并不创建有关进程的所有结构, 而是`只创建 PCB`. 将新的PCB 指向已经存在的进程.

![不同的PCB指向同一个进程](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/CSDN/image-20230411182232411.png)

然后, 以子进程划分程序资源类似的手段, 将进程划分为不同的区域, 并将不同的PCB分别指向划分出来的区域：



在操作系统中存在, 