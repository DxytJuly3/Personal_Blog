---
layout: '../../layouts/MarkdownPost.astro'
title: '[C++] C++新的类型转换方式介绍'
pubDate: 2023-07-10
description: ''
author: '七月.cc'
cover:
    url: 'https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307101801517.png'
    square: 'https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307101801517.png'
    alt: 'cover'
tags: ["C++", "语法", "继承"]
theme: 'light'
featured: false
---

![|wide](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307101801517.png)

---

# C语言中的类型转换

在C语言中, 有一些情况需要发生类型转化:

1. 赋值运算符左右两侧类型不同
2. 比较运算符左右两侧类型不同
3. 形参与实参类型不匹配
4. 返回值类型与接收返回值类型不一致
5. ...

C语言中总共有两种形式的类型转换：**隐式类型转换** 和 **显式类型转换**

## **隐式类型转化**

1. 编译器会在编译阶段自动进行, 比如像这样:

    ```cpp
    #include <iostream>
    
    int main() {
        size_t pos = 0;
        int i = 10;
        while (i >= pos) {
            i--;
            std::cout << i << std::endl;
        }
    
        return 0;
    }
    ```

    这段代码的执行结果是什么?

    第一眼看去: 好像很简单啊, 不就是 打印 `9->0` 吗?

    但是, 当程序执行之后:

    ![|inline](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307102117182.gif)

    很神奇, 明明 `i < 0` 了 但是循环还在继续.

    原因是: `while (i >= pos)` 这里发生了隐式类型转换:

    前10次循环, 是没有出什么问题的 也就是 i从10->0的过程.

    不过, 当i==0进入循环时 会执行i--, i成为了负数-1. 按正常思维, 循环就要跳出了.

    但是, 当 `i==-1`时, `while (i >= pos)` 的这个条件判断, 在作比较时 i会被隐式类型转换为 `size_t` 无符号整型

    这意味着什么? 这意味着, 即使 实际上`i==-1`, 但在比较时 它就是 **二进制32位全1的无符号整数**, 所以 `while (i >= pos)` 也就永远不会结束.

2. 如果允许隐式类型转换, 那编译器就会发生. 如果此处不允许, 那 编译就会报错

    ```cpp
    #include <iostream>
    
    int main() {
        char a = 'c';
        int b = a;
        std::cout << a << " : " << b << std::endl;
    
        int* c= a;
    
        return 0;
    }
    ```

    这段代码, `int* c = a;` 会报错. 因为不允许发生隐式类型转换.

    ![|inline](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307102319439.png)

    不过, 前面的可以编译通过:

    ![|inline](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307102320096.png)

这是C语言的隐式类型转换

## 显式强制转换

除了上面介绍的 编译器自动做的一些类型的转换. 用户还可以显式的对变量的类型做转换

```cpp
#include <cstdlib>
#include <iostream>

int main() {
    int* a = (int*)std::malloc(sizeof(int));
    *a = 20;

    std::cout << *a << std::endl;

    std::free(a);

    return 0;
}
```

将 `malloc()`的返回值, 使用`(int*)`强制类型转换为 `int*`

即, C语言中 可以使用 `(类型)` 强制转换变量的类型. 但是 必须是相近的类型, 不然会报错.

---

C语言中 就只有这两种类型转换的方式, 用户可以使用的也就只有 `(类型)` 

并且还具有明显的缺陷:

可视性比较差, 所有的转换形式都是以一种相同形式书写, 难以跟踪错误的转换

C++则实现了 4种不同的类型转换

# C++的4种类型转换

虽然C语言的类型转换很简单, 但是也存在一些问题:

1. 隐式类型转换可能会导致精度丢失或者判断错误
2. 显式类型转换则所有情况同一书写方式, 比较混乱 代码不够清晰

所以, C++增添设计了4种新的类型转换: `static_cast` `reinterpret_cast` `const_cast` `dynamic_cast`

## `static_cast`

`static_cast` 静态转换, 常用于相关的内置类型之间、以及相关自定义类型之间的转换.

用法很简单, `static_cast<需要转换为什么类型>(需要转换类型的变量)` 下面三种也是这种用法

```cpp
#include <iostream>

int main() {
    double d = 123.12398;
    int a = static_cast<int>(d);
    std::cout << d << " : " << a << std::endl;

    return 0;
}
```

![|inline](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307102347545.png)

## `reinterpret_cast`

`reinterpret_cast` 则常用于指针类型的转换. 