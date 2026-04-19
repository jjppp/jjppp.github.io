---
title: SA04 Abstract interpretation
date: 2024-01-02
permalink: /posts/2024/01/SA04-Abstract-interpretation/
tags:
  - Static Analysis
---

整点抽象的

下面默认 $x\le y$ 意味着 $x$ 比 $y$ 更精确，$y$ 比 $x$ 更保守。

# Galois connection

说的是这么一件事情：

如果给定了

1. 程序的语义 $f_{s}$ 和所有可能的执行状态 $\Sigma_s=\Set{\sigma_s}$
2. 分析的语义 $f_{a}$ 和所有可能的分析状态 $\Sigma_a=\Set{\sigma_{a}}$
3. 指派函数 $\alpha\colon2^{\Sigma_s}\to\Sigma_a$，将给定的执行状态集**抽象**成分析状态
4. 指派函数 $\gamma\colon\Sigma_a\to2^{\Sigma_s}$，将给定的分析状态**还原**为可能的执行状态集

并且有如下性质：

1. $\forall x,x\subseteq\gamma(\alpha(x))$：抽象后再还原可能丢精度，但仍是保守估计（不会低估）
2. $\forall y,\alpha(\gamma(y))\le y$：还原后再抽象不会损失精度

那么 $(\alpha,\gamma)$ 就称为一对 galois connection

需要注意，通常 $\alpha$ 是构造的，$\gamma$ 往往是 $\alpha$ 的伴随产物（这也意味着通常有 $\alpha\circ\gamma=\text{id}$），因此上面两个条件也在说：

1. 抽象必须保守
2. 抽象要尽可能避免丢精度

随机附赠的性质有：

* $\alpha(\bigcup A)=\bigcup_{a\in A}\alpha(a)$
* $\gamma(\bigcap A)=\bigcap_{a\in A}\gamma(a)$
* $\alpha(\bot)=\bot$，$\gamma(\top)=\top$

# Soundness

当我们说一个分析 sound 的时候，说的就是 $\forall \sigma, \alpha(f_s(\sigma))\le f_a(\alpha(\sigma))$

# Optimality

对于给定的 $\Sigma_a$，最精确的分析 $f_a$ 由 $f_a=\alpha\circ f_s\circ\gamma$ 给出

简单证明一下：

任取单调、sound $g\colon \Sigma_a\to\Sigma_a$，由 $g$ sound 立刻有 $\forall \sigma_a\in\Sigma_a, \alpha(f_s(\sigma_a))\le g(\alpha(\sigma_a))$

这说明 $\forall \sigma_s\in\Sigma_s, \alpha(f_s(\gamma(\sigma_s)))\le g(\alpha(\gamma(\sigma_s)))$。根据定义，$\alpha(\gamma(\sigma_s))\le \sigma_s$；$g$ 单调，$g(\alpha(\gamma(\sigma_s)))\le g(\sigma_s)$

串起来就是 $\alpha\circ f_s\circ\gamma\le g$，由 $g$ 的任意性可知 $\alpha\circ f_s\circ\gamma$ 最优。

这告诉我们对于给定的格抽象，理论上总是存在一个最优的抽象解释器：粗暴地把抽象状态具体化，交给具体语义的解释器执行，最后把可能的结果收集起来再抽象。

>这里有个比较重要的点是这样的：即便所有的原子操作语义都做到了最优抽象，它们的组合也可能达不到最优。
>
>办法：
>
>1. 对组合后的原子操作建模，扩大上下文。比如直接对数组（ptr offset deref => array load）和对象（ptr offset deref => load field）操作整体建模
>2. 做一些语义等价但利于分析的程序变换。比如用易于分析的 IR、做优化
