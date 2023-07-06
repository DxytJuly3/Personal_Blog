---
layout: '../../layouts/MarkdownPost.astro'
title: '[C++] C++11新特性分析介绍(2)'
pubDate: 2023-07-06
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

![|wide](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307061930908.png)

---

# C++11

上一篇介绍C++11常用的新特性只介绍了一部分. 本篇文章继续分析介绍.

## **`lambda` 表达式**

C++11之前, 我们使用 `std::sort()` 接口对自定义类型进行排序 或者 设置降序排列, 需要使用 **仿函数**, 就像这样:

```cpp
#include <iostream>
#include <iterator>
#include <vector>
#include <string>
#include <algorithm>
#include <functional>

using std::cout;
using std::endl;
using std::greater;
using std::string;
using std::vector;

struct Goods {
    string _name;  // 名字
    double _price; // 价格
    int _evaluate; // 评价
    Goods(const char* str, double price, int evaluate)
        : _name(str)
        , _price(price)
        , _evaluate(evaluate)
    {}
};

// 价格升序仿函数
struct ComparePriceLess {
    bool operator()(const Goods& gl, const Goods& gr) {
        return gl._price < gr._price;
    }
};

// 价格降序仿函数
struct ComparePriceGreater {
    bool operator()(const Goods& gl, const Goods& gr) {
        return gl._price > gr._price;
    }
};

int main() {
    int array[] = {4, 1, 8, 5, 3, 7, 0, 9, 2, 6};

    // 默认按照小于比较，排出来结果是升序
    std::sort(array, array + sizeof(array) / sizeof(array[0]));
    cout << "升序" << endl;
    for (auto e : array) {
        cout << e << " ";
    }
    cout << endl << endl;

    // 如果需要降序，需要改变元素的比较规则
    std::sort(array, array + sizeof(array) / sizeof(array[0]), greater<int>());
    cout << "降序" << endl;
    for (auto e : array) {
        cout << e << " ";
    }
    cout << endl << endl;

    vector<Goods> v = {
        {"苹果", 2.1, 5},
        {"香蕉", 3.1, 4}, 
        {"橙子", 2.2, 3}, 
        {"菠萝", 1.5, 4}
    };

    sort(v.begin(), v.end(), ComparePriceLess());
    cout << "价格升序" << endl;
    cout << "物品   价格   评价" << endl;
    for (auto e : v) {
        cout << e._name << "   " << e._price << "    " << e._evaluate << endl;
    }
    cout << endl;

    sort(v.begin(), v.end(), ComparePriceGreater());
    cout << "价格降序" << endl;
    cout << "物品   价格   评价" << endl;
    for (auto e : v) {
        cout << e._name << "   " << e._price << "    " << e._evaluate << endl;
    }
    cout << endl;

    return 0;
}
```

`std::sort()` 针对标准类型, 默认是按照降序排序的.

在上述代码中, 我们通过 标准中的仿函数 `std::greater` 可以让 `std::sort` 对数组升序排序.

而如果我们想要实现 对自定义类型进行排序, 就需要根据排序需求 定义相应的仿函数传给 `std::sort()` 然后才可以正确的排序.

上面代码的执行结果是:

![|inlinecat](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307061953432.png)

但是, 如果每次想要对不同自定义类型实现按需求排序, 那就 每次都需要定义一个满足需求的仿函数. 这样好像太麻烦了. 

所以, C++11 引入了 `lambda` 表达式

什么是 `lambda` 表达式呢?

先来看一下效果:

```cpp
// 其他部分还是 上面代码中的部分
void printGoods(vector<Goods>& v) {
    cout << "物品   价格   评价" << endl;
    for (auto e : v) {
        cout << e._name << "   " << e._price << "    " << e._evaluate << endl;
    }
    cout << endl;
}

int main() {
    vector<Goods> v = {
        {"苹果", 2.1, 5},
        {"香蕉", 3.1, 4}, 
        {"橙子", 2.2, 3}, 
        {"菠萝", 1.5, 4}
    };
    
    sort(v.begin(), v.end(), [](const Goods& g1, const Goods& g2){
        return g1._price < g2._price; });
    cout << "价格升序" << endl;
    printGoods(v);
    
    sort(v.begin(), v.end(), [](const Goods& g1, const Goods& g2){
        return g1._price > g2._price; });
    cout << "价格降序" << endl;
    printGoods(v);
    
    sort(v.begin(), v.end(), [](const Goods& g1, const Goods& g2){
        return g1._evaluate < g2._evaluate; });
    cout << "评价升序" << endl;
    printGoods(v);
    
    sort(v.begin(), v.end(), [](const Goods& g1, const Goods& g2){
        return g1._evaluate > g2._evaluate; });
    cout << "评价降序" << endl;
    printGoods(v);

    return 0;
}
```

这段代码的执行结果是:

![|inline](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307062006728.png)

我们没有实现仿函数, 但是依旧实现了按需求排序. 这就是因为使用了 `lambda` 表达式

上面的例子中, `std::sort()` 的第三个参数:

![|inline](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307062010176.png)

这些都是 `lambda` 表达式. 不过 并不完整, `lambda` 表达式的完整格式为:

```cpp
[captureList] (parameters) mutable -> returnType { statement }
[]()mutable->xxx{ }
```

这些都表示什么意思呢?

1. **`[capture-list]`: `捕捉列表`**

    该列表总是出现在 `lambda` 表达式的开始位置, 编译器根据`[]`来判断接下来的代码是否为 `lambda` 表达式, 捕捉列表能够捕捉上下文中的变量供 `lambda` 表达式使用

2. **`(parameters)`: `参数列表`**

    与 普通函数的参数列表 一致. 如果不需要参数传递, 则可以连同`()`一起省略

3. **`mutable`**

    默认情况下，`lambda` 函数总是一个 `const` 函数, 即 它的捕捉的 **值对象** 可以被看作默认被 `const` 修饰. `mutable` 可以取消其常量性. 使用该修饰符时, **`参数列表不可省略(即使参数为空)`**

4. **`-->returnType`: `返回值类型`**

    用 **追踪返回类型形式** 声明函数的返回值类型, 没有返回值时此部分可省略. 返回值类型明确情况下. 也可省略. 由编译器对返回类型进行推导

    > 什么是返回值类型明确的情况下?
    >
    > 看一看上面的例子, 结果一定是一个 `bool` 所以可以省略返回值声明
    >
    > 类似的情况就可以省略

5. **`{ statement }`: `函数体`**

    与普通函数的编写方式相同. 但是 在此作用域内, **只能使用传入的参数 或 捕捉到的父级作用域对象**.

> `参数列表`、`mutable`、`返回值类型` 在特定情况下都可以省略

从 `lambda` 表达式的组成来看, 与函数非常的相似. 而实际上, `lambda` 表达式就是一个 `匿名函数对象`, 可以被称为 `匿名函数`, 因为它没有函数名. 

且, `lambda` 表达式 除了可以直接匿名使用之外, 还可以这样:

```cpp
int main() {
    vector<Goods> v = {
        {"苹果", 2.1, 5},
        {"香蕉", 3.1, 4}, 
        {"橙子", 2.2, 3}, 
        {"菠萝", 1.5, 4}
    };
    
    auto priceUp = [](const Goods& g1, const Goods& g2) {
        return g1._price < g2._price; };
    sort(v.begin(), v.end(), priceUp);
    cout << "价格升序" << endl;
    printGoods(v);
    
    return 0;
}
```

通过 `auto` 来对 `lambda` 表达式 定义一个变量. 这样 相应的 `lambda` 表达式 就可以通过此变量被调用.

认识了 `lambda` 表达式之后, 有关于 `lambda` 表达式的使用还有一些关于 `捕捉列表` 细节需要介绍.

我们使用 `lambda` 表达式, 尝试实现交换对象的值:

```cpp
int main() {
    int a = 10, b = 20;
    cout << a << "  " << b << endl;
 
    auto lamSwap1 = [](int x, int y){
        int tmp = x;
        x = y;
        y = tmp;
    };
    auto lamSwap2 = [](int& x, int& y){
        int tmp = x;
        x = y;
        y = tmp;
    };
    lamSwap1(a, b);
    cout << "lamSwap1:: "<< a << "  " << b << endl;
    
    lamSwap2(a, b);
    cout << "lamSwap2:: "<< a << "  " << b << endl;
    
    return 0;
}
```

上面两个 `lambda` 表达式, 哪一个可以成功让 `a` 和 `b` 交换值?

一定是 `lamSwap2`, 因为 传入函数内的参数 如果是临时变量改变 不会影响到原数据. 而 `lamSwap2` 使用的是 左值引用.

![|inline](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307062249430.png)

直接传参可以实现交换数据.

不过我们还可以是用另外一种写法. 使用 **`捕捉列表`**

**`捕捉列表`** 描述了上下文中哪些数据可以在 `lambda`表达式作用域内 使用, 以及使用的方式传值还是传引用.

1. `[var]`：表示值传递方式捕捉变量`var`
2. `[=]`：表示值传递方式捕获所有父作用域中的变量(包括`this`)
3. `[&var]`：表示引用传递捕捉变量`var`
4. `[&]`：表示引用传递捕捉所有父作用域中的变量(包括`this`)
5. `[this]`：表示值传递方式捕捉当前的`this`指针

具体的使用如下:

```cpp
int main() {
    int a = 10, b = 20;
    cout << a << "  " << b << endl;
 
    auto lamSwap1 = [a, b](){
        int tmp = a;
        a = b;
        b = tmp;
    };
    auto lamSwap2 = [&a, &b](){
        int tmp = a;
        a = b;
        b = tmp;
    };
    lamSwap1();
    cout << "lamSwap1:: "<< a << "  " << b << endl;
    
    lamSwap2();
    cout << "lamSwap2:: "<< a << "  " << b << endl;
    
    return 0;
}
```

此次的两个 `lambda` 表达式, 哪一个可以成功让 `a` 和 `b` 交换值?

`lamSwap2` 肯定可以, 不过 `lamSwap1` 可以吗?

![inline](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307062256233.png)

很可惜, 这段代码编译无法通过, 并且 问题出在了 某个 `lambda` 表达式上.

是 `lamSwap1`. 我们在介绍 `lambda` 表达式的结构时 介绍过一个内容:

> 默认情况下，`lambda` 函数总是一个 `const` 函数, 即 它的捕捉的 **值对象** 可以被看作默认被 `const` 修饰. `mutable` 可以取消其常量性. 使用该修饰符时, **`参数列表不可省略(即使参数为空)`**

在 `lamSwap1` 中, 我们捕捉的 `[a, b]` 就是 **值对象**. 所以, 此时 `lamSwap1` 内的 `a` 和 `b` 都是被 `const` 修饰的值.

所以是无法修改的. 要想在内部修改, 就要使用 `mutable` :

```cpp
auto lamSwap1 = [a, b]()mutable{
    int tmp = a;
    a = b;
    b = tmp;
};
```

修改之后:

![|inline](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307062303621.png)

可以看到, 可以编译通过. 但是 执行之后, 值并没有被交换. 因为, `lamSwap1` 是 `传值捕捉`.

所以, 其实 `mutable` 这个关键词, 显得很没有用. 

传值捕捉 默认`const`修饰, 想要修改需要使用`mutable`. 但是使用之后, 又因为传值捕捉, 导致无法修改实际内容. 所以 `mutable` 好像没有什么太大作用.

不过, 现在我们知道了 `[]捕捉列表` 的用法. 可以捕捉 父级作用域的对象, 让其可以在 `lambda` 表达式内部使用.

并且, **`使用 [&var] 就是传引用捕捉, [var] 就是传值捕捉`**.

但是, 如果想要使用父级作用域的所有对象, 如果还需要一一显式捕捉的话, 就太繁琐了

所以, C++11还设计了, **`使用 [&] 就是传引用捕捉所有父级作用域内的对象, 使用 [=] 就是传值捕捉所有父级作用域内的对象`**

并且, C++11还设计了 混合使用. 即, 可以实现类似下面这样的捕捉:

```cpp
[&, a] 			// a对象传值捕捉, 其他父级作用域内的所有对象 传引用捕捉
[=, &a]			// a对象传引用捕捉, 其他父级作用域的所有对象 传值捕捉
```

所以, `lambda` 表达式的使用, 实际上是非常灵活的.

但是, `lambda` 表达式 **`禁止重复捕捉`**:

```cpp
[&, &a]
[&a, &a]
[=, a]
[a, a]
```

像类似上面的这些操作, 在某些编译器上 是会报错的:

![|inline](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307062320544.png)

> GCC不会报错, 只会报出警告:
>
> ![|wide](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307062321998.png)

还有一点就是, `lambda` 表达式 **`只能捕捉父级作用域中的对象`**. 即 包含此 `lambda` 表达式的作用域

---

关于 `lambda` 表达式 还有一个非常重要的内容是 关于它的类型. 

我们可以通过 `auto` 接收一个 `lambda` 表达式. 但是他究竟是什么呢?

`lambda` 表达式能不能相互赋值呢?

答案是, **不能**

即使是像这样的 看起来几乎一样的 `lambda` 表达式, 也不能相互赋值:

```cpp
int main() {
    auto f1 = []{cout << "hello world" << endl; };
    auto f2 = []{cout << "hello world" << endl; };
    f1 = f2;

    return 0;
}
```

![|inline](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307062328843.png)

但是, 可以通过 **拷贝构造的形式**, 创建一个 `lambda` 表达式的副本.

也可以, 将 `lambda` 表达式赋值给一个 **参数和返回值类型都相同的函数指针**

```cpp
void (*pF)();
int main() {
    auto f1 = []{cout << "hello world" << endl; };
    
    auto f2(f1);
    f2();
    
    pF = f1;
    pF();

    return 0;
}
```

![|inline](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307062332599.png)

这就是 `lambda` 表达式.

这是为什么呢?

为什么会出现 这一部分现象呢? 其实是由于 `lambda` 的底层实现 是 **`仿函数`**.

对, 就是可以像函数一样使用的对象, 就是在类中重载了 `operator()运算符` 的类对象

这一点, 可以在 VS中通过反汇编代码看出来:

![|inline](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307062341565.png)

## 包装器

C++11 增添了 `lambda` 表达式之后. 

C++中 可调用类型的对象就变成了三种: `函数指针`, `仿函数`, `lambda`

可以通过这三种对象 调用函数.

似乎有点太多了. 
