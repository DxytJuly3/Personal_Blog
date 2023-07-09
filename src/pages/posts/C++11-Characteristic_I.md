---
layout: '../../layouts/MarkdownPost.astro'
title: '[C++] C++11新特性介绍 分析(1): 列表初始化、右值引用、完美转发、移动语义...'
pubDate: 2023-04-21
description: '本篇文章是关于C++11标准 一些常用的新特性的介绍, 比如: 列表初始化、右值引用、万能引用、完美转发、类的新默认成员函数 和 可变参数列表等'
author: '七月.cc'
cover:
    url: 'https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202306251811775.png'
    square: 'https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202306251811775.png'
    alt: 'cover'
tags: ["C++", "C++11", "语法"]
theme: 'light'
featured: false
---

![|wide](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307061929155.png)

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

**`右值`:** 右值也是一个表示数据的表达式, 但它是这一类: `字面常量`、`表达式返回值`、`函数返回值`等等, 都是一些临时的表达式, 这些表达式是 **无法对它取地址** 也 **无法给它赋值** 的. 虽然, 这些临时表达式存在自己的地址, 但它的 **`地址也是临时的`**, 无法被普通指针指向, 当表达式失效时 地址会相应的失效, 空间资源会被释放.

 **`右值可以出现在赋值符号的右边, 但是不能出现出现在赋值符号的左边`**.

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

上面的例子中, `1`、`2`、`x + y`、`min(x, y)` 都是右值, 无法对其取地址, 也无法给其赋值, 并且 **`生命周期只在本行`**. 本行执行完毕, 右值的临时地址空间资源就会被释放. 

而 `int &&rr1`、`int &&rr2` 和 `int &&rr3` 则都是 **`右值引用`**.

需要注意的是, 右值引用并 **`不是将右值变成一个变量存储起来`**, 而是起了别名. 

可以看作 **`用别名将右值绑定`** 了起来, 将右值的地址生命周期延长了, 其临时地址空间不会在除了本行马上释放, 进而使其 **`可以直接当作变量`** 使用.

**右值引用的符号是 `&&`**

右值引用是 **`无法引用左值`** 的:

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
> 2. 但是右值引用可以引用 `move` 以后的左值 (后面在解释)

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

当 使用其他`July::string`对象 实例化 `July::string对象` 或者 给 `July::string对象` 赋值时, 会自动的调用拷贝构造函数和赋值重载函数

这里拷贝构造和赋值重载的参数都是左值引用, 然后都要复制传入参数的数据赋给对象, 都是要针对要存储到string的数据 **`进行深拷贝`** 的.

还有 将 `int类型`转换为`July::string类型` 的 `to_string()` 函数. 此函数的返回值是 `July:string` 类型的, 是不能使用左值引用的. 因为返回的是一个临时变量, 临时变量左值引用返回会发生错误. 所以, 为了正确的将转换出的`July::string` 返回, 在返回的过程中一定会发生深拷贝. 

这里只是一个简单的 `string` 的例子, 如果是更复杂的 类或容器. **深拷贝消耗的资源可能是非常巨大的**. 因为深拷贝涉及到复制对象的所有数据，包括动态分配的资源.

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

---

在使用左值引用之后, 很大程度上解决了传参时深拷贝的问题, 但深拷贝还可能会发生在 拷贝构造、赋值重载 和 临时对象返回上. 

而右值引用出现之后, 实现了`新的临时对象返回的方式` 和 `两个新类的默认成员函数`

1. 新的临时对象返回方式

    这里的新的返回方式并 **不是指编写方式发生了改变**, 即 返回值类型不做变化的, 依旧是 **传值返回**. 而是指, C++11之后 当一个函数返回的数据是一个临时对象 或 直接返回一个右值时(出了函数就要销毁的数据做返回值), 编译器会将返回值类型 **识别为右值引用类型**.

2. 两个新的 类的默认成员函数

    1. 默认移动构造函数

        什么是移动构造函数?

        还是以 `July::string` 为例, 移动拷贝构造的函数名是这样的:
    
        ```cpp
        string(string &&str) 
        	: _str(nullptr)
        	, _size(0)
        	, _capacity(0) {
            ...
        }
        ```
    
        并且, 移动拷贝构造函数的实现方法, 通常是 **`直接将传入的参数拥有的数据 与 对象的成员数据进行交换`**.
    
        那么, `July::string` 之中, 实现应该是这样的:
    
        ```cpp
        void swap(string& str) {
            std::swap(_str, str._str);
            std::swap(_size, str._size);
            std::swap(_capacity, str._capacity);
        }
        
        string(string &&str) 
        	: _str(nullptr)
        	, _size(0)
        	, _capacity(0) {
            swap(str);
        }
        ```
    
        这样传入右值引用参数, 并直接交换 传参数据 和 对象成员数据 来实例化对象的函数 就叫 **`移动构造函数`**
    
    2. 默认移动赋值重载函数
    
        根据移动构造函数的实现, 可以很快的推断出 移动赋值重载函数的实现:
    
        ```cpp
        void swap(string& str) {
            std::swap(_str, str._str);
            std::swap(_size, str._size);
            std::swap(_capacity, str._capacity);
        }
        
        string& operator=(string &&str) {
            swap(str);
            
            return *this;
        }
        ```
        
        这样传入右值引用参数, 并直接交换 传参数据 和 对象成员数据 来给对象赋值的函数 就叫 **`移动赋值重载函数`**

C++11之后, 类添加了这两个默认成员函数之后, 可以解决很大一部分的深拷贝问题. 因为, 当使用右值来给对象赋值或实例化对象时, 类会直接调用 **`移动构造函数`** 和 **`移动赋值重载函数`**. 这两个函数不会深拷贝, 而是会 **交换数据资源**. 即, `移动构造` 和 `移动赋值重载` 的思想是 **将临时对象的数据(所需数据) 与 源对象数据做交换, 从而避免因数据拷贝消耗资源**.

并且, C++11之后, 当一个函数的返回值类型为传值返回 且返回的是一个函数内的临时变量 或 其他类型的右值时, 编译器会默认将**返回类型方式识别为 右值引用返回**, 从而让临时变量或者右值 不会在出函数作用域时被销毁. 从而避免深拷贝的发生.

> 之前已经介绍过, 针对类的各种构造函数 一些编译器对其进行一些优化
>
> 相关文章: 
>
> 之前介绍的都是 拷贝构造 的 一些优化.
>
> 而C++11引入了 右值引用, 引入了 移动构造 之后. 编译其又会做什么优化呢?
>
> 以 `to_string()` 为例:
>
> ```cpp
> July::string to_string(int value) {
>     bool flag = true;
>     if (value < 0) {
>         flag = false;
>         value = 0 - value;
>     }
> 
>     bit::string str;
>     while (value > 0) {
>         int x = value % 10;
>         value /= 10;
> 
>         str += ('0' + x);
>     }
> 
>     if (flag == false) {
>         str += '-';
>     }
>     std::reverse(str.begin(), str.end());
>     
>     return str;
> }
> ```
>
> 在C++11之前, 当我们使用 `to_string()` 的返回值 去实例化 `July::string` 对象时. 
>
> 如果编译器不优化:
>
> 1. 传值返回, 需要生成临时对象, 会发生一次拷贝构造
> 2. 使用 `July:string` 对象 实例化 `July::string` 又会发生一次拷贝构造
>
> 而由于 生成临时对象这一步, 非常的多余且消耗资源. 函数传值返回, 已经是临时对象了, 还要再拷贝构造一个临时对象, 太多余了
>
> 所以 编译器会优化掉第一次的拷贝构造, **`直接使用函数内 return时的临时对象`** 返回值 去拷贝构造实例化 `July::string`对象
>
> 而在C++11之后, 编译器会将传值返回 识别为传右值引用返回, 所以回去调用移动构造函数. 
>
> 如果编译器不优化:
>
> 1. 传值返回, 需要生成临时对象, 但编译器会识别出右值引用返回, 所以 发生一次移动构造
> 2. 使用 一个右值 `July:string` 对象 实例化 `July::string` 又会发生一次 移动构造
>
> 还是有一步多余的临时对象的移动构造, 所以编译器会优化掉.
>
> 编译器会 直接使用函数内 **`return时的临时对象`** 返回值 去移动构造实例化 `July::string` 对象

C++11之后, 所有的STL容器也增加了这两个成员函数:

**`vector`:**

![|inline](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307051418871.png)

**`string`:**

![|inline](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307051421175.png)

所有的STL容器都增加了这两个成员函数. 

---

除了这两个成员函数之外, 前面还提到了一句话: **右值引用只能右值, 不能引用左值. 但是右值引用可以引用 `move` 以后的左值**

这句话中的 `move` 是什么意思呢?

其实, C++11 不仅提出了右值引用, 还增添了一个新的函数: `std::move()`, 这个函数的功能很简单 就是 **将左值转换为右值引用, 并返回.**

![|inline](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307051430262.png)

也就是说, 某些情况我们需要将左值右值引用时, 就可以使用 `move()` 来将指定的左值变为右值引用.

但是, 由于右值引用的特点和设计 实际上是为了更好的支持 **移动语义**. 所以, 当一个左值通过 `move()` 变为右值引用之后, 这个**左值就默认被编译器认为 此左值支持 移动语义了**. 也就意味着, 左值被 `move()` 之后 此左值的数据就可能被置换走. 

> 什么是移动语义? 
>
> 支持移动语义可以理解为, 可以通过那两个类的成员函数: `移动构造` 和 `移动赋值重载`, 实例化新对象 或 给其他对象赋值.

在 **左值被move() 之前**, 用此左值实例化对象 或 给其他对象赋值 编译器会调用 **`拷贝构造`** 或 **`普通赋值重载`**, **不会使此左值失去它的原数据**. 

在 **左值被move() 之后**, 再用此左值实例化对象 或 给其他对象赋值 编译器就会调用 **`移动构造`** 和 **`移动赋值重载`**, 之后 **此左值的原数据会被置换走, 此左值会拥有另一个对象的原数据**.

所以, **`move() 的使用需要谨慎, 因为他可能会导致左值对象随时失去原数据或被销毁, 所以 被move之后的左值 又常被称为 将亡值.`** 

### 容器中 另外的右值引用

上面介绍了, 函数传值返回 可能会被编译器识别为右值引用返回, 并且也介绍了两个右值引用传参的类默认成员函数

除此之外, STL容器的其他地方也通过右值引用, 减少了深拷贝的出现.

那就是, **`push_back()`** 、 **`insert()`** 等一系列向容器中插入数据的接口. 

![|inline](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307051505735.png)

![|inline](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307051507705.png)

不仅 `vector`, 其他容器也同样实现了 **数据添加接口的右值引用参数版本**.

虽然 之前的版本使用的左值引用 已经避免了传参时可能发生的深拷贝.

但 STL在实际的数据插入实现中, 还是会 **对传入的数据进行深拷贝** 实例化对象 进行存储. 所以 可以直接使用右值引用, 表示传入的参数的数据可以进行置换, 就会直接置换数据 防止发生深拷贝.

## **`万能引用与完美转发 **`**

C++11引入了右值引用, 用 `&&` 表示. 并且 类中也新增了两个使用右值引用的默认成员函数. 

因此, C++11之后 就会有些场景, 就需要使用 **`右值引用类型的参数作为模板参数`**.

但是 却存在着一些问题.

我们还是使用上面的 `July::string` 类. 但是执行这段代码:

```cpp
void Fun(int &x) {
    cout << "左值引用" << endl; 
}
void Fun(const int &x) {
    cout << "const 左值引用" << endl; 
}
void Fun(int &&x) {
    cout << "右值引用" << endl; 
}
void Fun(const int &&x) {
    cout << "const 右值引用" << endl; 
}

template<typename T>
void PerfectForward(T&& t) {
    Fun(t);
}

int main() {
    PerfectForward(10); 			// 右值

    int a;
    PerfectForward(a); 				// 左值
    PerfectForward(std::move(a)); 	// 右值

    const int b = 8;
    PerfectForward(b); 				// const 左值
    PerfectForward(std::move(b)); 	// const 右值

    return 0;
}
```

首先适当的分析一下代码: 

我们定义了重载了4个函数`Fun()`, 会根据传入的参数类型, 判断const左值或右值. 

然后定义了一个函数模板, 并且 **函数的参数类型设置为 模板参数的`&&`**. 

然后我们在主函数内, 分别依次向 调用模板函数 并传入: `右值` `左值` `右值` `const 左值` `const 右值`

按理来说, 函数的执行结果应该是 按照传入顺序输出 相应的参数类型. 

而实际的结果是:

![|inline](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307051719360.png)

不管是 `左值` 还是 `右值`. 传入模板函数之后, 识别出的类型都是 `左值` 相关的.

这是为什么呢?

首先, 先介绍 函数模板的参数里的 `&&`.

当 `&&` 用在模板中, `&&` 就不再是右值引用了, 而是 **`万能引用`**.

万能引用, 顾名思义. 即, **`左值 和 右值都可以传入`** (原本 右值引用是不能直接引用左值的), 并且 **左值传入 实参就为左值引用, 右值传入 实参就为右值引用**.

但, 实际我们用过之后, 发现 确实 **左值右值都可以传入, 但 参数统统为左值**.

出现这种现象, 涉及到一个概念, **`引用折叠`**. 

> 什么是 `引用折叠` :
>
> 当一个函数的形参为引用类型时, 一下这些情况会发生引用折叠:
>
> 1. 假如存在函数 `Fun(int& f)`, 且存在 变量 `int &a = b`
>
>     当 `a` 传入 `Fun()`, `f` 的类型可以看作 `int& &`
>
>     此时, 会发生引用折叠 `f` 会折叠为 `&左值引用`
>
>     > 折叠规则: `T& &` --> `T&`
>
> 2. 假如存在函数 `Fun(int& f)`, 且存在 变量 `int &&a = 10`
>
>     当 `a` 传入 `Fun()`, `f` 的类型可以看作 `int& &&`
>
>     此时, 会发生引用折叠 `f` 会折叠为 `&左值引用`
>
>     > 折叠规则: `T& &&` --> `T&`
>
> 3. 假如存在函数 `Fun(int&& f)`, 且存在 变量 `int &a = b`
>
>     当 `a` 传入 `Fun()`, `f` 的类型可以看作 `int&& &`
>
>     此时, 会发生引用折叠 `f` 会折叠为 `&左值引用`
>
>     > 折叠规则: `T&& &` --> `T&`
>
> 4. 假如存在函数 `Fun(int&& f)`, 且存在 变量 `int &&a = 10`
>
>     当 `a` 传入 `Fun()`, `f` 的类型可以看作 `int&& &&`
>
>     此时, 会发生引用折叠 `f` 会折叠为 `&&右值引用`
>
>     > 折叠规则: `T&& &&` --> `T&&`
>
> 总结就是, **只有出现 `T&& &&` 引用折叠才会折叠为 `&&右值引用`**.

还涉及到一个特性, 即 **右值引用的`变量`在`直接用于作表达式`(使用变量)时，会被认为是左值变量**.

这个特性可以从 简单的代码表现出来:

```cpp
int &&a = 10;
int* b = &a;
a = 7;
```

`右值引用变量a, 可以被取地址, 也可以被赋值. `

也可以通过一下代码 表现出来:

```cpp
void fun(int&& f) {}

int main() {
	int &&d = 10;
    fun(d);
    
    return 0;
}
```

这段代码 编译是不通过的:

![](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307051953061.png)

**其实就可以说明, 当右值引用的`变量`被 `直接用于作表达式`(使用变量) 时, 会被认为是左值.**

当然, 还有一种验证方式, 依旧是使用 July::string 类. 不过此时要在 默认构造、移动构造 和 拷贝构造里各添加一句话:

```cpp
// 默认构造
string(const char* str = "")
    : _size(strlen(str))
    , _capacity(_size)
    {
        // 添加提示语句
        cout << "默认构造" << endl;
        _str = new char[_capacity + 1];
        strcpy(_str, str);
    }

string(string&& str) {
    // 暂不实现功能
    cout << "移动构造" << endl;
}

// 拷贝构造函数 传统
string(const string& s)  
    : _size(s._size)
    , _capacity(s._capacity)
    {
        // 添加提示语句
        cout << "拷贝构造" << endl;
        _str = new char[_capacity + 1];
        strcpy(_str, s._str);
    } 
```

然后执行下面的代码:

```cpp
int main() {
    July::string &&str = "12345";
    July::string s = str;
    
    return 0;
}
```

执行结果是:

![|inline](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307052006779.png)

可以看到, 最终的执行结果是 执行了默认构造 和 拷贝构造. 都没有执行移动构造.

这也可以说明, **当右值引用的`变量`被 `直接用于作表达式`(使用变量) 时, 会被认为是左值**

而 最上面模板函数使用万能引用的例子中. 尽管调用 `PerfectForward()` 函数时传入的是右值引用. 

但是在 此函数内部 再通过实参调用 `Fun()` 函数, 依旧会被识别为左值, 就是因为这两个原因.

1. 引用折叠了, 虽然 `T&& &&` 折叠之后依旧 表示右值引用
2. `Fun(t)` 调用时, `t` 直接用作表达式(直接当变量使用), 会被认为是左值

而 使用模板、万能引用的目的并不是这样的. 我们的目的是 传入左值, 就以左值引用使用, 传入右值 就以右值引用使用.

这时候, 就要用到C++11的一个新接口: **`std::forward()完美转发`**

![|inline](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307052020225.png)

这个接口看起来非常的复杂, 但是实际使用并没有那么复杂:

```cpp
void Fun(int &x) {
    cout << "左值引用" << endl; 
}
void Fun(const int &x) {
    cout << "const 左值引用" << endl; 
}
void Fun(int &&x) {
    cout << "右值引用" << endl; 
}
void Fun(const int &&x) {
    cout << "const 右值引用" << endl; 
}

template<typename T>
void PerfectForward(T&& t) {
    cout << "非完美转发: ";
    Fun(t);
    
    cout << "完美转发: ";
    Fun(std::forward<T>(t));
    cout << endl;
}

int main() {
    PerfectForward(10); 			// 右值

    int a;
    PerfectForward(a); 				// 左值
    PerfectForward(std::move(a)); 	// 右值

    const int b = 8;
    PerfectForward(b); 				// const 左值
    PerfectForward(std::move(b)); 	// const 右值

    return 0;
}
```

这段代码的执行结果是:

![|inline](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307052028288.png)

可以发现, **`经过完美转发的引用变量 会被识别为原本的类型`**.

> `std:move()` 和 `std::forward()` 都是转换变量用的. 
>
> 不过 `move()` 是将左值转换为右值引用
>
> `forward()` 则是将 变量原本表示的类型还给它.

不过, 完美转发的使用场景是下边这样:

我们介绍右值引用时, 提到过 STL容器在各方面支持了右值引用.

并且, STL容器都是类模板, 肯定需要使用到完美转发. 就像这样:

![|inline](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307052040700.png)

此例中, 由于 `List` 是一个模板类. 所以要想 针对不同类型 在各方面实现对右值引用的支持, 就需要用到 完美转发

比如, 形参有右值引用的 `Insert()` 接口. 调用此接口时需要传入 右值, 应该变为右值引用. 但进入函数之后 `x` 就会变为左值形式. 所以要想实现 `node` 插入, 就要使用 `forward()` 将 `x` 恢复为右值引用, 才能调用 `Node` 结构体的移动赋值.

还有另外两个调用 `Insert()` 接口的 `PushBack()` 和 `PushFront()`, 也是如此.

## 新的类功能

### 新默认成员函数

在C++11之前, 类一共有6个默认成员函数:

1. 构造函数
2. 析构函数
3. 拷贝构造函数
4. 拷贝赋值重载
5. 取地址重载函数
6. const 取地址重载函数

C++11之后, 又有2个新增的默认成员函数 我们已经介绍过了:

1. 移动构造函数
2. 移动赋值重载函数

既然是默认成员函数, 那么他们是可以由编译器自动生成的.

但是, 这2个默认成员函数与其他的默认成员函数有一些不同. 他们的自动生成的条件有一些严苛, 不过功能的实现与其他默认成员函数类似:

1. 如果没有自己实现 `移动构造函数`, 且 **`没有实现析构函数、拷贝构造、拷贝赋值重载中的任意一个`**. 那么编译器会自动生成一个默认移动构造. 

    默认生成的移动构造函数, 对于内置类型成员会执行逐成员按字节拷贝(深拷贝). 对于自定义类型成员, 则需要看这个成员是否实现移动构造, 如果实现了就调用移动构造, 没有实现就调用拷贝构造.

2. 如果没有自己实现 `移动赋值重载函数`，且 **`没有实现析构函数 、拷贝构造、拷贝赋值重载中的任意一个`**. 那么编译器会自动生成一个默认移动赋值. 

    默认生成的移动构造函数, 对于内置类型成员会执行逐成员按字节拷贝. 对于自定义类型成员, 则需要看这个成员是否实现移动赋值, 如果实现了就调用移动赋值, 没有实现就调用拷贝赋值. 

3. **`如果你提供了移动构造或者移动赋值，编译器不会自动提供拷贝构造和拷贝赋值`**

> **`没有实现析构函数、拷贝构造、拷贝赋值重载中的任意一个`** 的意思是, 三个函数都没有实现

### 强制生成默认函数的关键字 `default`

这个关键字的使用很简单:

```cpp
class MyClass {
public:
    MyClass();  // 默认构造函数
    MyClass(const MyClass& other) = default;  // 强制生成默认拷贝构造函数
    MyClass& operator=(const MyClass& other) = default;  // 强制生成默认拷贝赋值运算符

    MyClass(MyClass&& other);  // 移动构造函数
    MyClass& operator=(MyClass&& other);  // 移动赋值运算符
};
```

只需要 函数定义时 在函数后加上 `= default` 就可以强制编译器生成相应的默认成员函数

### 禁止生成默认函数的关键字 `delete`

此关键字的用法 与 `default` 相同. 功能相反

## 可变参数模板

在C语言中, 经常使用的两个函数 具有可变参数: `printf()` 和 `scanf()`

这两个函数的参数数量是可变的. 即可以根据需要传入不同数量的参数.

![|inline](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307060942712.png)

C++11之后, 不仅函数可以支持可变参数, 模板也可以支持可变参数了:

```cpp
template <class ...Args>
void ShowList(Args... args) {}

/*
其中 Args是一个模板参数包，args是一个函数形参参数包
声明可变一个参数包用 Args...args
此参数包, 可以看作是一个按参数传入顺序将传入的参数存储起来的一个数据结构
*/
```

C++11之前, 模板只能设置固定数量参数; C++11之后, 模板支持可变参数.

但是, 函数拿到可变参数包之后, 并 **`不能直接通过实参 来获取 参数类型、内容`**.

语法没有支持, 类似这样的获取参数包中 参数详情的使用方法:

```cpp
template <class ...Args>
void ShowList(Args... args) {
	args[0];
    // 类似这样的方法, 以及范围for, 都无法使用. 
}
```

但是可以通过其他方法 来获取参数类型或内容:

1. 递归 展开参数包

    ```cpp
    // 递归终止函数
    template <class T>
    void ShowList(const T& t) {
        cout << typeid(t).name() << ":";
        cout << t << endl;
    }
    
    // 展开函数
    template <class T, class ...Args>
    void ShowList(T value, Args... args) {
        cout << typeid(value).name() << ":";
        cout << value <<"    ";
        ShowList(args...);
    }
    
    int main() {
        ShowList(1);
        ShowList(1, 'A');
        ShowList(1, 'A', std::string("sort"));
    
        return 0;
    }
    ```

    我们可以通过在模板可变参数之前, 添加一个普通模板参数. 那么 传入模板的 **第一个参数** 就是 **可直接使用** 的.

    我们只需要在此函数内, 递归调用此函数. 那么 就可以不断 **获得参数包内的第一个参数**. 

    直到递归到最后, 参数包内只剩一个参数时, 开始返回.

    > 这里递归结束的函数是 针对`ShowList()`实现了一个只有一个参数时的特化

    这段代码执行结果是:

    ![|inline](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307061006102.png)

2. 列表初始化 展开参数包

    ```cpp
    template <class T>
    void PrintArg(T t) {
        cout << typeid(t).name() << ":";
        cout << t << "    ";
    }
    
    //展开函数
    template <class ...Args>
    void ShowList(Args... args) {
        int arr[] = { (PrintArg(args), 0)... };
        cout << endl;
    }
    
    int main() {
        ShowList(1);
        ShowList(1, 'A');
        ShowList(1, 'A', std::string("sort"));
        
        return 0;
    }
    ```

    在这种方法中, 我们使用逗号表达式保证 `(PrintArg(args), 0)` 的值为 0.

    然后还使用了列表初始化 来初始化一个变长数组.

    `int arr[] = { (PrintArg(args), 0)... }` 

    会被展开为 

    `int arr[] = { (PrintArg(arg1), 0), (PrintArg(arg2), 0), (PrintArg(arg3), 0)... }`

    当然, 这里的逗号表达式不是必须的, 只需要将 `PrintArgs()` 设置一个整型返回值, 就可以不用逗号表达式.

    ```cpp
    template <class T>
    int PrintArg(T t) {
        cout << typeid(t).name() << ":";
        cout << t << "    ";
        
        return 0;
    }
    
    //展开函数
    template <class ...Args>
    void ShowList(Args... args) {
        int arr[] = { PrintArg(args)... };
        cout << endl;
    }
    ```

    这种方法的执行结果为:

    ![|inline](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307061057057.png)

## `emplace_back()`

`emplace_back()` 是C++11之后, 添加到STL容器中的一个 使用可变参数的元素插入接口.

我们都知道, STL容器都是模板类, 所以 `emplace_back()` 其实使用的就是模板可变参数.

![|inline](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307061101052.png)

这个接口的使用也很简单:

```cpp
int main() {
    std::vector<pair<int, std::string>> arr;
    arr.emplace_back(11, "十一");
    arr.emplace_back(20, "二十");
    arr.emplace_back(make_pair(30, "三十"));
    arr.push_back(make_pair(40, "四十"));
    arr.push_back({ 50, "五十" });
    
    for (auto e : arr) {
		cout << e.first << ":" << e.second << endl;
    }
    
    return 0;
}
```

![|inline](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307061109378.png)

从结果来看好像没有区别. 从用法来看, 好像也没有什么太大的改变, 无非就是支持了 直接使用 构建 `pair` 的参数来插入.

但是, 实际的执行上是有一些细小的差别的.

1. `arr.emplace_back(11, "十一");` 和 `arr.emplace_back(20, "二十");`

    `emplace_back()` 会根据传入的两个参数, 直接调用 `pair` 的构造函数 构造一个 `pair`, 然后存储在 `arr` 末尾

2. `arr.emplace_back(make_pair(30, "三十"));`

    先执行 `make_pair()` 创建了一个临时的 `pair` 对象. 然后通过 `emplace_back` 在 `arr` 末尾创建了这个对象的副本. 它调用了两次 `pair` 构造函数: 一次在 `make_pair`, 一次在 `emplace_back`.

3. `arr.push_back(make_pair(40, "四十"));`

    先执行 `make_pair()` 创建了一个临时的 `pair` 对象. 然后通过 `push_back` 将创建这个对象的副本, 并将这个对象的副本插入到 `arr` 的末尾. 它也调用了两次 `pair` 构造函数：一次在 `make_pair`, 一次在 `push_back`

4. `arr.push_back({ 50, "五十" });`

    首先通过 `列表初始化` 创建了一个临时的 `pair` 对象. 然后通过 `push_back` 将创建这个对象的副本, 并将这个对象的副本插入到 `arr` 的末尾. 它也调用了两次 `pair` 构造函数：一次在 `列表初始化`, 一次在 `push_back`

总的来说, 就是当容器的元素类型是自定义类型时. 可以直接使用 `emplace_back()` 传入数据. `emplace_back()` 会通过传入的数据 **`直接构造元素并插入容器的末尾`**, 不会再拷贝或者移动元素.

