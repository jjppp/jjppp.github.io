---
title: Cpp Lambda Quirks
date: 2023-08-23
permalink: /posts/2023/08/Cpp-Lambda-Captures/
tags:
  - Cpp
---

C++11 引入了匿名函数。类似 Java，C++ 中的匿名函数也是通过构造匿名类来实现的，不同之处在于 C++ 允许重载 `operator()`，因此可以实现看上去和函数一样的调用语法。本文主要关注 lambda 函数中一些反直觉行为，以及这些反直觉背后的直觉。

文章里关于手册的转述可能不够严谨，部分名词不知道怎么翻译，也可能存在时效问题，建议优先读一手资料。

# 例1

先来看一段代码

```c++
int g = 0;
void ex1() {
    auto l1 = []() { return g + 1; }
    auto l2 = [=]() { return g + 1; }
    auto l3 = [g=g]() { return g + 1; }
    g = 20;
    std::cout << l1() << " " // 21
              << l2() << " " // 21
              << l3() << " " // 11
              << std::endl;
}
```

乍一看这三个匿名函数没有区别，但注释里的输出表明它们是有细微的区别的。

查 cppreference 之后可以知道：

1. 对应 `l1()`：匿名函数体内部可以直接引用全局变量而无需捕获
2. 对应 `l2()`：对于默认值捕获，捕获对象为包裹匿名函数的 enclosing scope 中的 automatic variables，故代码中的 `g` 并没有进入捕获列表，此时等价于 `l1()`
3. 对应 `l3()`：显式地捕获了全局变量 `g` 的值，内部对 `g` 的引用并非全局

这么做背后的原因有几个猜测：

1. 全局变量在各个函数间都能访问，无需默认捕获，但需要的时候可以显式指明。此为“不需要则默认不做”
2. 匿名函数的函数体实质上是一个单独的函数 `operator()`，在 callee 内无法引用 caller 的局部变量，但可以将这些局部变量作为函数参数传入，恰好对应按值/按引用捕获。此为“必须做的不做不行”

引用捕获则要麻烦得多，需要能证明生命周期的嵌套。更多时候更无脑的办法是用 `shared_ptr` 来辅助管理

# 例2

```c++
auto factory1(int x) {
    return [=] {
        static int y = 0;
        return (++ y) + x;
    };
}

void ex2_1() {
    auto l1 = factory1(114);
    auto l2 = factory1(514);
    std::cout << l1() << " " // 114
              << l2() << " " // 515
              << std::endl;
}
```

这个例子表明 `static` 关键字的含义仍然没有发生变化。由于匿名函数的实现是匿名类，因此 `static int y` 仍然是相对于 `operator()` 这个函数而言的静态变量，`y` 是在多个匿名函数之间共享的

一个更能佐证的例子是这样的：

```c++
auto factory2(int x) {
    return [=] <typename T> (T z) {
        static T y = 0;
        return (++ y) + x;
    };
}

void ex2_2() {
    auto l1 = factory2(114);
    auto l2 = factory2(514);
    std::cout << l1(0)  << " " // 114
              << l2(0.) << " " // 514
              << std::endl;
}
```

这里的两个调用会产生两个匿名函数，它们对应于两个不同的实例化后的匿名类，因此会分别有 `<int>::operator()` 和 `<double>::operator()` 两个函数体，自然也就有两份静态变量

# 例3

如果希望向 SICP 里那样用闭包保存状态该怎么做？

```c++
void ex4() {
    auto l1 = [state=INITIAL_STATE] (int input) mutable {
        switch (input) {
            // transitions ...
        }
    };
}
```

1. 匿名函数的 `operator()` 默认是 `const` 的，即内部不能更改捕获列表中的内容，如果希望有状态的跳转需要标记 `mutable`
2. 值捕获本质上是用当前作用域中的表达式初始化匿名类的成员，此处相当于声明储存状态的成员
3. 不能用 `static`，原因你懂的
