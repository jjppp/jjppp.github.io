---
title: TAPL09 Recursive Types
date: 2022-08-29
permalink: /posts/2022/08/TAPL09-Recursive-Types/
tags:
  - TAPL
---

# Intro

 有种前菜只是刚刚上完的感觉。

递归数据结构是非常常见的结构，给递归数据结构赋予的类型通常会是一个递归的类型。

经典的例如

```haskell
data IntList
	= Nil
	| Cons Int IntList
```

当然这里还没有讲到参数化类型，因此这里 List 作为“容器”的概念，其内容类型的区分是通过名字完成的。

为了避免这样自引用的情况（类似于 ULC 中定义递归的方式），引入算子 $\mu$。那么上面的 haskell 类型定义就可以形式化地写成下面的样子
$$
\text{IntList=$\mu X.\left<\text{Nil}, \Set{\text{hd=Int, tl=}X}\right>$}
$$
一个分析中的例子可以是这样的：给定函数 $f$，定义算子 $\gamma$ 使得 $\gamma f$ 为方程 $f(x)=x$ 的解集。此处 $\mu$ 的作用是类似的。

类比 $\text{fix}$ combinator，这个 $\mu$ 同样可以像剥洋葱一样一层层剥开类型并且不收敛。

# Equivalence

书上讲有两种构造递归类型的方法，它们的区别在于对如下问题的回答不同：

> 递归类型本身和它剥开一层后的类型，二者关系如何？

这个“剥开”函数一般叫 $\text{fold}\colon T\mapsto T$

* equi-recursive approach 认为它们是同一个类型，在语境下可以等价替换，type checker 负责决定把具体的递归类型剥开到什么程度。
* iso-recursive approach 认为它们不同，但是是**同构的**。即 $\text{unfold}\circ\text{fold}=id_T$。同时代码需要显式地在需要剥开的地方写出 $\text{fold}$ 和 $\text{unfold}$

# Induction, Coinduction

## 定义

若 $F:2^U\mapsto 2^U$ 满足 $\forall X\subseteq Y, F(X)\subseteq F(Y)$， 则称 $F$ 是 $U$ 上的单调函数

通常把这个（朴素意义的）集合 $U$ 叫做 Universe，即我们所有可能讨论的所有元素组成的集合。

* 若 $F(X)\subseteq X$，则称 $X$ 是 $F$-closed 的
* 若 $X\subseteq F(X)$ ，则称 $X$ 是 $F$-consistent 的
* 若 $F(X)=X$，则称 $X$ 是 $F$ 的不动点（fixed point）

可以发现 $F$ 本质上是定义在格上的函数，并且这个格是由 $\left<U,\subseteq\right>$ 导出的。

通常把 $F$ 的最小不动点写成 $\mu F$，最大不动点写成 $\nu F$。

## 定理

Knaster-Tarski 定理的叙述如下：

1. 所有 $F$-closed 集之交是 $F$ 的最小不动点
2. 所有 $F$-consistent 集之并是 $F$ 的最大不动点

证一下①：

考虑 $C=\Set{S\mid F(S)\subseteq S}$，不妨记 $M=\bigcap C$，下面即证明 $F(M)=M$ 且 $\forall X\subseteq U, F(X)=X\Rightarrow M\subseteq X$

注意到 $\forall S\in C$ 都有 $M\subseteq S$，因此 $\forall S\in C$ 都有 $F(M)\subseteq F(S)$，这说明 $F(M)\subseteq \bigcap_{S\in C} F(S)$

又由 $C$ 的定义可知 $F(M)\subseteq\bigcap_{S\in C} F(S)\subseteq \bigcap C=M$，因此 $F(M)\subseteq M$；

另一侧由反证法，不妨假设存在 $x\in M$ 使得 $x\not\in F(M)$，此时记 $M'=F(M)$，那么有 $M'\subseteq M$，根据单调性有 $F(M')\subseteq F(M)=M'$，因此 $M'\in C$，这说明 $M=\bigcap C\subseteq M'=F(M)$，矛盾；故假设不成立，即 $M\subseteq F(M)$

综上就有 $M=F(M)$，这说明 $M=\bigcap C$ 是 $F$ 的一个不动点，下面证明这是最小的不动点。

由反证法设存在更小不动点 $M''\subsetneq M$，则根据不动点定义有 $M''\in C$，这说明 $M=\bigcap C\subseteq M''\subsetneq M$，矛盾；故假设不成立，不存在更小的不动点了。

②的证明是对偶的。定理的另一种叙述如下：

1. *Principle of induction* 若 $X$ 是 $F$-closed 集，那么 $\mu F\subseteq X$
2. *Principle of coinduction* 若 $X$ 是 $F$-consistent 集，那么 $X\subseteq\nu F$



# Subtyping

tnnd，卖了一个很大的关子就跑了，今天就到这吧。
