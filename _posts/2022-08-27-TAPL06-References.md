---
title: TAPL06 References
date: 2022-08-27
permalink: /posts/2022/08/TAPL06-References/
tags:
  - TAPL
---

# Intro

之前提到的 `x=term` 都是 name binding。因为所有的项都可以求值，并且都是纯的计算模型（只和 substitution 有关），所以本质上是 meta level 的语法替换。可以简单地理解为宏展开。

这里引入的是可变引用（指针、C++的引用、rust的可变引用）这样的东西，即我们给某个变量绑定的不再是一个项，而是一个储存空间，其中储存着项。同时有两种方法“取出”和“存入”项。这就使得计算模型发生了变化（回想 SICP 引入 `set!` 之后的那一大坨话）

同样是引用，不同的语言在语法设计上也存在差异，例如 C 语言的变量默认是可变的，会在使用过程中隐式地解引用；也可以设计成 `val` 和 `var` 的区别以显式地区分可变与不可变变量。

当然书上的写法还是很像 C 的。



印象里去年上课还扩充了 magic wand 之类的高端内容，当时是完全没听懂了。



# Side-effect, alias, shared state

副作用和状态很好理解，考虑如下项

```ocaml
gen start = 
	let counter = ref start in {
		inc = fun _: Unit (counter := !counter + 1),
		get = fun _: Unit (!counter)
	}
```

我们调用 `gen 0`  即得到一个 variant，每次调用 `variant.inc unit` 就会造成 `counter` 所存内容的增加，调用 `variant.get unit` 就可以查询到这个值。

alias 也很好理解，考虑

```ocaml
x = ref 0;
y = x;
x := !x + 1;
!y
```

结果就会是 `1`。 alias 使得分析变得困难，这也是为什么别名分析需要花那么大力气去对 heap 和 obj 建模。

共享状态就更好理解了，写过并发程序的话应该都懂的。



# location $L$

store 是对 memory 的抽象，即我们认为内存是一个 $L\mapsto V$ 的映射。此时对分配操作而言，其结果就是产生一个 $l\in L$，把值存进 $\mu$。而后续通过这个 location $l$ 我们就可以得到具体的值。

引出的一个问题就是 gc。在形式化讨论的时候完全可以不管 gc，如果需要对 gc 建模则要

1. 形式化定义 garbage
2. 形式化定义 gc 的过程（即 $\overset{\text{gc}}\rightarrow$ 的操作语义）

# Typing

对于新加入的 location $l$ 也需要赋予一个类型。

naive 的做法是通过访存 $\mu$ 来做 structural 的类型推导。考虑如下 store
$$
\Set{l_1: \lambda x\colon\text{Nat}. (!l_2).x,l_2: \lambda x\colon\text{Nat}. (!l_1).x}
$$
这种 store 上的循环依赖将导致 naive 的类型推导不终止

一个更加简单的方法是给每个 location 标注类型，即所有被动态分配出来的内存都应当维护这段内存中的比特串应当被解释为什么。

