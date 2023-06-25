---
layout: '../../layouts/MarkdownPost.astro'
title: '[Linux] 网络编程-套接字'
pubDate: 2023-06-25
description: ''
author: '七月.cc'
cover:
    url: 'https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202306251756988.png
'
    square: 'https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202306251756988.png'
    alt: 'cover'
tags: ["Linux", "网络"]
theme: 'light'
featured: false

---

![](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202306251756988.png)

---

上一篇文章中, 我们简单的介绍了网络的最基础的部分内容, 没有涉及到编程相关的内容.

从本篇文章开始, 就真正开始涉及到网络编程了.

---

# 网络编程 - 套接字

在正式开始网络编程之前, 实际上还有几个概念没有介绍

## 相关概念介绍

### 1. 源ip地址与目的ip地址

不同局域网的主机之间进行通信, 是通过IP地址进行的.

那么, 其中 **`源IP地址 就是发送主机的IP地址, 目的IP地址 就是接收主机的IP地址.`**

要如何理解这两个ip地址呢? 其实就可以看作我们生活中的两个东西, `始发地和最终目的地`.

假如要从家里自驾去某个地方旅游, 首先 家是不会变的, 其次 即使中途可能经过许多地方, 但是正常情况下 你的最终目的地是不会变的. 

### 2. `端口号`

