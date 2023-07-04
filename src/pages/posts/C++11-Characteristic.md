---
layout: '../../layouts/MarkdownPost.astro'
title: '[C++] C++11新特性分析介绍：列表初始化、右值引用'
pubDate: 2023-04-21
description: ''
author: '七月.cc'
cover:
    url: 'https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202306251811775.png'
    square: 'https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202306251811775.png'
    alt: 'cover'
tags: ["C++", "C++11", "语法"]
theme: 'light'
featured: false
---

---

![|wide](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202306251811775.png)

---

# C++11

## 介绍

在2003年C++标准委员会曾经提交了一份技术勘误表(简称TC1)，使得C++03这个名字已经取代了C++98称为C++11之前的最新C++标准名称。不过由于`C++03`主要是`对C++98标准`中的漏洞进行`修复`，语言的`核心部分则没有改动`，因此人们习惯性的把`两个标准合并称为C++98/03标准`。

从C++0x到C++11，C++标准10年磨一剑，第二个真正意义上的标准珊珊来迟。相比于C++98/03，`C++11`则带来了`数量可观的变化`，其中包含了约`140个新特性`，以及对`C++03标准中约600个缺陷的修正`，这使得C++11更像是从C++98/03中孕育出的一种新语言。

相比较而言，`C++11能更好地用于系统开发和库开发、语法更加泛华和简单化、更加稳定和安全`，不仅功能更强大，而且能提升程序员的开发效率，公司实际项目开发中也用得比较多。

C++11增加的语法特性非常篇幅非常多，我们这里没办法一一讲解，所以关于C++11只介绍实际中`比较实用的语法` 

## 统一的列表初始化 `{}`

C++11, 为变量、对象、容器提供的一种新的初始化的方式：`{} 初始化`

具体的使用就像这样：

```cpp
struct Point {
    int _x;
    int _y;
};

int main() {
    int a = 1; 			// 之前
    int b = {2};		// C++11 支持
    int c{3};			// C++11 支持

    Point po1 = {1, 2};
    Point po2{1, 2};

    int array1[] = {1, 2, 3, 4, 5};
    int array2[5]{1, 2, 3, 4, 5};

    return 0;
}
```

我们可以在定义变量时, 直接使用 `{}` 对对象进行初始化.

除了简单的对象初始化, 还可以对多成员变量的自定义类进行初始化：

```cpp
class Date {
public:
    Date(int year, int month, int day)
        : _year(year)
        , _month(month)
        , _day(day) {
        cout << "Date(int year, int month, int day)" << endl;
    }

private:
    int _year;
    int _month;
    int _day;
};

int main() {
    Date d1(2022, 1, 1);

    // C++11支持的列表初始化，这里会调用构造函数初始化
    Date d2{2022, 1, 2};
    Date d3 = {2022, 1, 3};

    return 0;
}
```

以前, 对于自定义类实例化对象, 我们一般都会使用 `Date d1(2022, 1, 1);`

C++11 之后, 就也可以使用 `{} 列表初始化`

但是, 这些使用好像有些没用. 

不过下面这样的使用, 就比之前初始化要好用一些：

```cpp
int main() {
    int* pA = new int{1};
    int* pArray = new int[9]{1, 2, 3, 4, 5, 6, 7, 8, 9};
    
    vector<int> v1{1, 2, 3, 4, 5, 6, 7, 8, 9, 0};
    vector<int> v2 = {1, 2, 3, 4, 5, 6, 7, 8, 9, 0};
    vector<int>* v3 = new vector<int>[4]{ {1,2,3,4}, {5,6,7,8}, {9,10,11,12}, {12,13,14,15} };
    
    return 0;
}
```

我们可以直接使用 `{}` 对容器进行初始化, 更可以在 `new` 时使用 `{}` 对数组进行初始化.

即, 列表初始化可以更好地去支持 `new[]` 时 变量的初始化. 使用 `{}` 初始化类, **`会去调用构造函数`** 不仅仅是默认构造函数.

> 所以 `int a = {1};` 这一类就不推荐使用了.

而, C++11 是怎么实现这样的东西的呢？

## `initializer_list`

我们可以这样来查看 `{}` 的类型：

```cpp
int main() {
    auto li = {1,2,3,4,5};
    cout << typeid(li).name() << endl;

    return 0;
}
```

![|inline](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307041000041.png)

可以看到, `auto` 接收 `{}` 的类型是： `initializer_list`

![|inline](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307041013969.png)

其实, `{}` 本身就是一个容器类型. `{1, 2, 3, 4, 5}`  就是通过 `initializer_list<int>` 实例化出的一个对象. 

我们这样初始化：

```cpp
int main() {
	vector<int> v1{1, 2, 3, 4, 5, 6, 7, 8, 9, 0};
    vector<int> v2 = {1, 2, 3, 4, 5, 6, 7, 8, 9, 0};
    vector<int>* v3 = new vector<int>[4]{ {1,2,3,4},{5,6,7,8},{9,10,11,12},{12,13,14,15} };
    
    return 0;
}
```

本质上, 其实就是调用了 以 `{}` 对象为参数的构造函数来实例化对象. 

因为, STL容器中其实定义有 使用 `{}` 对象的构造函数. 

![|inline](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307041021964.png)

其他STL 容器中 也同样如此：

`set:`

![|inline](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307041021169.png)

```cpp
int main() {
 	 set<int> s1{1, 2, 3, 4, 5, 6, 7};
    set<int> s2 = {1, 2, 3, 4, 5, 6, 7};

	 return 0;
}
```

`map:`

![|inline](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307041022546.png)

```cpp
int main() {
	 map<string, string> dict ={ {"apple", "苹果"}, {"banana", "香蕉"}, {"sun", "太阳"} };
    
	 return 0;
}
```

STL容器, 在C++11 之后 都支持了以 `initializer_list` 对象为参数的构造函数.

也就是说, STL容器实现`{}`初始化对象是通过 实现了针对 `initializer_list` 类型的构造函数. 

而我们自己自定义的多成员变量的类是怎么实现的呢？

其实是 `隐式类型转换 + 编译优化`. 比较`类似 C++11 之前, 单个成员变量的类的直接赋值初始化`.

## 新的声明

C++中, 我们除了可以使用各种类型来声明变量、对象、函数之外.

C++11 提供了一些新的声明方式

### `auto`

首先就是 auto. auto我们经常使用, 并且很早就已经介绍过了

auto 会根据对象、变量的赋值类型去自动推导 对象、变量的类型.

```cpp
int main() {
    int b = 1;
    auto c = 3.3;
    std::cout << typeid(b).name() << std::endl;
    std::cout << typeid(c).name() << std::endl;
    
    return 0;
}
```

![|inline](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307041028181.png)

不过, auto 一般用于非常长的容器的迭代器的自动推导

### `decltype`

`decltype` 可以用来推导 表达式的类型：

```cpp
int main() {
    decltype(1 * 1) d;
    decltype(2 * 2.2) e;
    cout << typeid(d).name() << endl;
    cout << typeid(e).name() << endl;
    
    return 0;
}
```

![|inline](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307041030249.png)

### `nullptr`

我们一直在C++中使用 `nullptr` 来表示空指针. 而 `nullptr` 实际上是C++11才提出的

在C语言中, 通常使用NULL作为空指针. 不过 NULL 在C语言中其实就是0. 有时可能会被检测为整型. 

所以 C++11 就是用了nullptr.

## 范围for

范围for, 其实是一种变量容器数据的一种方法. 可以对所有支持 `iterator 迭代器` 的容器使用：

```cpp
int main() {
    vector<int> v = {1, 2, 3, 4, 5, 6, 7, 8, 9};
    for(auto e : v) {
        cout << e << " ";
    }

    return 0;
}
```

<img src="https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/CSDN/image-20230422000634152.png" alt=" |inline" style="zoom:80%; display: block; margin: 0 auto;" />

## 智能指针

C++11 提出一个很重要的概念, 就是智能指针. 

不过本篇文章不做介绍, 会单独写一篇文章介绍 C++11 的智能指针.

## STL 新容器

C++11 为STL 添加了四个新容器：

![|inline](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307041035476.png)

除了哈希表, 另外两个其实没有什么值得介绍的. 

`array`就是静态数组, 在使用上与我们平时 `int arr[10];` 没什么区别. 值得说的就是 `默认支持了越界检查`

`forward_list` 就是单链表. 以方便使用来说, 还是 `list` 好用.

而另外的 哈希表, 博主有专门介绍的文章：

> [[C++-STL] 哈希表以及unordered_set和unordered_set的介绍](https://www.julysblog.cn/posts/C++-Hash-Table)
>
> [[C++-STL] 用哈希表封装unordered_map和unordered_set](https://www.julysblog.cn/posts/C++-Hash-Table-Package-unordered_map&unordered_set)

## **`右值引用 **`**

**`&`** 这个符号, 在 C语言中 表示取地址; 在 C++中 则多了一个功能, 即 引用 用来给变量起别名.

但是, 我们之前使用的 `&` 引用, 在C++11之后 完整的叫法是 `左值引用`

那么问题来了, 什么是左值? 

**`左值`:** 其实就是一个表示数据的表达式, 比如: `变量名` 或 `解引用的指针`, 可以对它 **取地址** 也给它 **赋值**, **`左值可以出现赋值符号的左边, 右值不能出现在赋值符号左边`**. 定义时 `const`修饰符后的左值, 不能给它赋值, 但是可以取它的地址. 

左值引用就是给左值的引用, 给左值取别名。

```cpp
int a = 10;
int* b = &a;
int c = *b;
const int d = *b;

int &e = a;
int &f = d;
```

上面例子中, `a`、`b`、`c`、`d`都是左值, 都可以对其 **取地址**. `int &e` 和 `int &f` 都为 **左值引用**

另外, 左值引用并不单单可以引用左值, const引用 就可以引用右值:

```cpp
#include <iostream>

int main() {
    // 这个是编译不通过的
    //int &g = 10;
    
    // 这个是可以编译通过的
    const int &g = 10;
    
    return 0;
}
```

1. `int &g = 10;` 无法编译通过

    ![int &g = 10 |wide](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307041803172.png)

2. `const int &g = 10; ` 可以编译通过

    ![const int &g = 10 |wide](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307041806267.png)

> 关于 左值引用的小总结:
>
> 1. 普通左值引用只能引用左值, 不能引用右值
> 2. 但是 `const`左值引用既可引用左值, 也可引用右值

而 在C++11之后, 出现了另一种引用: **`右值引用`**

那么, 什么是 **`右值`**? 什么是 **`右值引用`**?

**`右值`:** 右值也是一个表示数据的表达式, 但它是这一类: `字面常量`、`表达式返回值`、`函数返回值`等等, 这些表达式是 **无法对它取地址** 也 **无法给它赋值** 的. **`右值可以出现在赋值符号的右边, 但是不能出现出现在赋值符号的左边`**.

右值引用就是 `对右值的引用`, 给 `右值取别名`

所以, 区分左右值最关键的点是, **`看表达式能否取地址`**.

```cpp
int x = 1, y = 2;
1; 2;
x + y;
min(x, y);

int &&rr1 = 1;
int &&rr2 = x + y;
int &&rr3 = fmin(x, y);
```

上面的例子中, `1`、`2`、`x + y`、`min(x, y)` 都是右值, 无法对其取地址, 也无法给其赋值

而 `int &&rr1`、`int &&rr2` 和 `int &&rr3` 则都是 **`右值引用`**.

**右值引用的符号是 `&&`**

右值引用是无法引用左值的:

```cpp
#include <iostream>

int main() {
    int m = 1;
   	int &&n = m;
    
    return 0;
}
```

![|inline](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307041809978.png)

> 关于 右值引用的小总结
>
> 1. 右值引用只能右值, 不能引用左值
> 2. 但是右值引用可以引用 `move` 以后的左值

> 右值是不能取地址的. 
>
> 但是, 当我们对右值取别名之后, 就会使右值数据被存储到某个特定的位置. 所以可以对其取地址和赋值.
>
> 但这个特性没有什么应用场景.

> 对于区分左值或者右值, 可以根据这一句话和具体情况进行判断:
>
> - 可以取地址的, 有名字的, 非临时的就是 **左值**;
> - 不能取地址的, 没有名字的, 临时的就是 **右值**;

---

介绍了什么是右值引用, 那么 右值引用有什么用呢? 它的使用场景什么呢?

右值, 一般都是表达式? 常量、函数返回值、临时变量(生命周期仅在当前行)

并且, 上面我们提到 右值引用可以引用 `move`以后的左值. 这又是什么意思的?

实际上, 当前右值引用的作用场景是 **`移动拷贝`** 和 **`移动赋值`**. 这两种用法, 可以在 `一定程度上解决深拷贝消耗大的问题`. 

它是如何解决的呢?

先来回顾一下, 在很久之前 模拟实现的 `string类` 时的部分代码:

![string_default |inline](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307042040063.png)

展示的部分代码中, 只包含了几个类的默认函数 和 自定义的 `int类型`转`July::string类型` 的 `to_string()` 函数

当 调用拷贝构造 实例化 `July::string对象` 或者 给 `July::string对象` 赋值时, 会自动的调用拷贝构造函数和赋值重载函数

这里拷贝构造和赋值重载的参数都是左值引用, 然后都要复制传入参数的数据赋给对象, 都是要针对要存储到string的数据 **`进行深拷贝`** 的.

还有 将 `int类型`转换为`July::string类型` 的 `to_string()` 函数. 此函数的返回值是 `July:string` 类型的, 是不能使用左值引用的. 因为返回的是一个临时变量, 临时变量左值引用返回会发生错误. 所以, 为了正确的将转换出的`July::string` 返回, 在返回的过程中一定会发生深拷贝. 

这里只是一个简单的 string 的例子, 如果是更复杂的 类或容器. **深拷贝消耗的资源可能是非常巨大的**. 因为深拷贝涉及到复制对象的所有数据，包括动态分配的资源.

虽然, 左值引用在 传参 或者 某些情况做返回值类型时, 可以节省资源.

> 做返回值节省资源的例子: 
>
> string 的 `+=` 重载实现时, 要实现连续 `+=` 就要将 `+=` 过的string作为返回值返回.
>
> 如果直接传值返回, 还是会造成深拷贝. 
>
> 所以, 可以使用左值引用返回

但还是存在一些可能会发生深拷贝的地方.

而 **右值引用** 则可以更好的解决上面的这类发生深拷贝的问题.

在使用左值引用之后, 深拷贝还可能会发生在 拷贝构造、赋值重载 和 临时对象返回上. 

而右值引用出现之后, 实现了新的临时对象返回的写法 和 两个新类的默认成员函数
