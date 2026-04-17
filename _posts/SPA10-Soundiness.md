---
title: 软件分析10 Soundiness
tags: Static Analysis
date: 2021-08-14 02:45:00
excerpt: Soundiness 横空出世
---
\newcommand\norm[1]{\left\lVert#1\right\rVert}
\newcommand\abs[1]{\left\lvert#1\right\rvert}
## Soundness


Conservative approximation: captures all program behaviors, or the analysis
result models all possible executions of the program


然鹅


### Academia


Virtually all published whole-program analyses are unsound when applied to
real programming languages.


### Industries


Virtually all realistic whole-program static analysis tools have to make
unsound choices.


## Hard-to-analyze Language Features


不同的语言存在一些难以分析的特性，如果对这些特性采取过于保守的措施，那么我们将会得到正确但无用的分析


PPT给了几个例子，发现自己除了C的几个都不会....提上日程！


#### Java


Reflection, native code, dynamic class loading


#### JavaScript


eval, document object model(DOM)


#### C/C++


Pointer arithmetic, function pointers


这就导致了许多声称sound的做法只是保证了核心语法的sound，而对部分功能的sound有所取舍(说白了就是挑结果发论文，坠入生化环材的灌水之路)。


学术圈出现的乱象，需要正义的铁拳！(大雾，于是就有了一个Manifesto宣言，提出了Soundiness的概念


## Soundiness


造词来自Truthiness


> Truthiness: a truthful or seemingly truthful quality that is claimed for
something not because of supporting facts or evidence but because of a feeling
that it is true or a desire for it to be true


也就是把原本的Soundness升华(?)了


A soundy analysis means that the analysis is mostly sound, with
well-identified unsound treatments to hard/specific language features.




区分之后的三级分类


* A sound analysis requires to capture all dynamic behaviors

* A soundy analysis aims to capture all dynamic behaviors with certain hard
language features unsoundly handled within reason

* An unsound analysis deliberately ignores certain behaviors in its design for
better efficiency, precision or accessibility.


也就是以后的论文必须要明确分析自己哪里准了，哪里还不准，以及原因。




然后讲了两个具体的例子来对reflection和native code分析


## Reflection


难点在于class、method和field的具体对象是运行时确定的，并且是由一段字符串决定的，因此对于非静态决定的字符串难以求解上面的三个东西


动态的字符串可能来自


* 终端输入

* 配置文件

* 可能含有加密解密信息

* 网络


这些因素都使得反射机制难以静态分析




最早的做法是通过结合指针分析来对静态字符串进行分析，进而推断出一些目标方法。而另一个想法就是，我们不在求解定义时寻找对应的方法，而是在使用它的时候寻找。大意就是在调用方法的时候通过参数的类型和数量来推断可能的目标方法。




还有一些complete方法，例如说跑几个测试用例来找到一些必然真的目标方法。




## Native Code


Java在需要和OS交互的时候，需要调用一些C/C++代码，这样的代码称为Native Code


Native
Code难以分析比较好理解，因为语言不同，导致需要不同的分析策略。一种方法是根据函数的语义对常用的函数进行建模。PPT给的例子就是一个`ArrCopy()`可以建模成一整段的`for`+赋值
