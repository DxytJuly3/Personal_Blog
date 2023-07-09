---
layout: '../../layouts/MarkdownPost.astro'
title: '[C++] C++智能指针分析介绍'
pubDate: 2023-07-09
description: '[C++] C++智能指针分析: RAII思想、智能指针原理、auto_ptr、unique_ptr、shared_ptr、weak_ptr...'
author: '七月.cc'
cover:
    url: 'https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307090019836.png'
    square: 'https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307090019836.png'
    alt: 'cover'
tags: ["C++", "指针", "语法", "C++11", "智能指针"]
theme: 'light'
featured: false
---

![|wide](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307090019836.png)

---

一直以来, C++一直最令人诟病的问题之一是什么?

没错, 就是 **`内存泄漏`**

C语言还好一些, 但是 在C++中, 异常的相关处理出现之后, `内存泄漏`的问题简直是防不胜防

> 博主 C++异常相关文章:
>
> [[C++] C++异常处理分析介绍: 异常概念、异常抛出与捕获匹配原则、重新抛出、异常安全、异常体系...](https://www.julysblog.cn/posts/C++-Exception)

而且, C++也不像Java那样带有垃圾自动回收机制, 所以在C++11之前, C++的内存管理一直是一件非常麻烦的事情.

虽然现在也麻烦, 不过已经好了很多.

# 智能指针

C++在引入了异常的概念之后, 内存泄漏的问题就非常有可能出现.

比如这段代码:

```cpp
#include <iostream>
#include <exception>

using std::cin;
using std::cout;
using std::endl;
using std::exception;
using std::invalid_argument;

int div() {
    int a, b;
    cin >> a >> b;
    if (b == 0)
        throw invalid_argument("除0错误");

    return a / b;
}

void Func() {
    // 1、如果p1这里new 抛异常会如何？
    // 2、如果p2这里new 抛异常会如何？
    // 3、如果div调用这里又会抛异常会如何？
    int* p1 = new int;
    int* p2 = new int;

    cout << div() << endl;

    delete p1;
    delete p2;
}

int main() {
    try {
        Func();
    }
    catch (exception& e) {
        cout << e.what() << endl;
    }

    return 0;
}
```

注释的三个问题, 各自都会产生什么结果?

1. 如果 `int* p1 = new int;` 时抛异常, 其实是空间开辟失败了, 所以不会发生内存泄漏.
2. 如果 `int* p2 = new int;` 时抛异常, 那就说明`p1`空间是开辟成功了的, 所以如果这里抛异常, 会导致`p1`指向的空间无法被`delete`, 会发生内存泄漏
3. 而, 如果`div()`执行时抛异常, 那么 就会导致`p1`和`p2`所指向的空间泄露.

不过, 如果是第3种情况, 可以直接这样解决:

```cpp
void Func() {
    int* p1 = new int;
    int* p2 = new int;

    try {
        cout << div() << endl;
    }
    catch (...) {
        delete p1;
        delete p2;
        throw;
    }

    delete p1;
    delete p2;
}
```

但是, 如果类似第2种情况, 就无法相对方便的解决.

虽然看似可以模仿上面的解决方法尝试解决问题. 但实际上是无法解决的.

为什么呢?

`int* p2 = new int;`时抛异常, 我们捕捉到异常之后, 就需要将`p1`的空间释放掉. 

但如果是这种情况呢?

```cpp
void Func() {
    int* p1 = new int;
    int* p2 = new int;
    int* p3 = new int;
    int* p4 = new int;
    int* p5 = new int;

    try {
        cout << div() << endl;
    }
    catch (...) {
        delete p1;
        delete p2;
        delete p3;
        delete p4;
        delete p5;
        throw;
    }

    delete p1;
    delete p2;
    delete p3;
    delete p4;
    delete p5;
}
```

如果是在 `p3`、`p4`或`p5`任意一个位置抛异常了呢? 

如果是`p3`, 那就需要释放 `p1` `p2`; 如果是`p4`, 那就需要释放 `p1` `p2` `p3`; 如果是`p5`, 那就需要释放 `p1` `p2` `p3` `p4`

如果 开辟的操作更多, 那么不同位置抛异常, 要处理的情况就不同. 

这种情况, 就会非常的麻烦. 

所以, C++11正式引入了`智能指针`.

## RAII思想

**`RAII (Resource Acquisition Is Initialization) 资源获取即初始化`** 是一种 **利用对象生命周期来控制程序资源** 的技术.

什么意思呢?

我们知道, 一个对象在生命周期即将结束时 是会自动调用析构函数释放资源的.

我们可以利用这一点, **将我们所需要获取的资源 来托管给对象, 通过对象来进行管理、访问、使用资源**.

即, **实现一个类, 其成员变量包括所需资源, 在实例化对象时 通过构造函数开辟并初始化资源, 在对象生命周期快要结束时 对象会自动调用析构函数, 将开辟出来的空间释放**.

这样做有什么好处呢?

1. 不需要显式地释放资源
2. 所需的资源在对象生命周期内始终保持有效

比如, 这段代码:

```cpp

```

