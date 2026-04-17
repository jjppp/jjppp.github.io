---
title: TAPL04 Typed Lambda
date: 2022-08-04 19:16:48
tags: TAPL
---

Simply Typed Lambda Calculus 通常记为 $\lambda_{\rightarrow}$

# Intro

在 ULC + Bool + Nat 中会出现某个项 $T$ 卡住的情况，意思是

1. $T$ 不是值，且
2. 不存在 $T'\neq T$ 使得 $T\rightarrow T'$

例如经典的 `suc True` 就没法推导



解决方案是**静态地**给项加上类型，在运行前做类型检查，然后证明*所有通过类型检查的项都终会推导成值*



一个比较有意思的点是，我们可以将每个项都视为一种类型，这时候做类型检查（类型推导）就完全等价于对项本身进行推导。并且由于 LC 是图灵完备的（例如经典的 $\Omega$），这样的类型推导可能不终止。

这说明所有**有用的**类型推导（停机的类型推导）必将做出过近似。意思是存在一个不会卡住（well-behaved）的项 $T_w$，$T_w$ 没法通过类型检查。

还可以回想静态分析做的事情（也是在静态时过近似，用各种 derivation rules 做约束求解），所以说这玩意是同属于 PL 下的细分分支。甚至对于指针分析，只需要把对象集看成类型，那么求解过程本质上就是在定 type 了。



# Definition

## Types

定义类型集合 $\scr T$ 为满足如下约束的最小集

1. $\text{Bool}\in {\scr T}$
2. $\forall A,B\in {\scr T}, A\rightarrow B\in{\scr T}$

②实际上是引入了产生复杂类型的方法，称 $\rightarrow$ 为类型构造子。例如我们可以有 $\newcommand{tBool}{\text{Bool}} (\tBool\rightarrow\tBool)\rightarrow\tBool$ 这样的类型



## Typing relation

我们把 $M: \tau$ 称为 typing judgement。又因为这实际上是 $T, {\scr T}$ 上的二元关系，因此也叫 typing relation。

对于这样的二元关系也存在推导（derivation rules），通常写成 $\dfrac{\text{premise}}{M:\tau}$ 这样的形式。

很显然并非所有的项都可以推导出一个类型（例如经典的 $\Omega$），因此做关于类型的表述时通常说 “若项 $M$ 存在类型 $\tau$” 这样的句子。



## $\lambda$-abstractions

给出了类型集合的定义后，需要考虑的就是如何给项标记类型（怎么做 structural 的类型推导）。关键的地方在于形如 $\lambda x.\; M$ 的项要如何确定类型。

> 这里插几句。由于前面提到的 $\tBool$-calculus 是没有变量的，所有的符号都是常元，因此所有的类型都可以通过 structural 的过程得出。
>
> 在这里因为引入了 $\lambda$-abstraction，所以会出现变量（回忆函数内访问参数）。如何推导函数的类型，本质上就是如何推导函数参数和返回值的类型。

1. explicitly typed: 可以手动给参数加上类型注解，然后推导返回值的类型
2. implicitly typed: 可以约束求解，此时得到的参数类型是一个可能类型的集合

①很好理解，②可以类比 haskell 里面无类型注解的函数会用 typeclass 来限制参数的类型，这是一种从 body 到参数的类型推导

为了简单，书选择了把项写成 $\lambda x:\tau. M$ 的形式



## Typing context

同时因为 $\lambda$-abstraction 的出现，项 $T$ 中可能存在自由变元。这时候的 $T$ 的类型在这些自由变元取不同类型时可能有不同的类型，因此引入typing context $\Gamma$。

$\Gamma$ 为 typing judgement 的集合，类似命题逻辑有 $\Gamma\vdash M:\tau$ 这样的记号，含义为“在假设 $\Gamma$ 成立时，$M$ 类型为 $\tau$”



## Derivation rules

这个形式和命题逻辑的推理系统 G' 具有形式上的一致性，非常漂亮的一个结论。当然也可以认为做类型推导本身就是在构造一个逻辑系统，反过来命题逻辑恰好足够表达 STLC 的类型推导。
$$
\frac{}{\Gamma,x:\tau\vdash x:\tau}
$$

$$
\dfrac{\Gamma,x:\tau\vdash M:\sigma}{\Gamma\vdash\lambda x:\tau.\; M:\tau\rightarrow\sigma}
$$

$$
\frac{\Gamma\vdash M:\tau\rightarrow \sigma\quad \Gamma\vdash N:\tau}{\Gamma\vdash M\; N:\sigma}
$$

# Meta Theorems

证明都是简单的，熟练掌握归纳法就可以了

## Uniqueness of types

若 $M:\tau$，则 $\forall \sigma\in{\scr T}$，$M:\sigma\Rightarrow \tau=\sigma$

只需要证明 derivation tree 到项存在双射即可。

## Progress

若 $\vdash M:\tau$，则

1. 要么 $M$ 为值
2. 要么 $\exists M'\neq M$ 使得 $M\rightarrow M'$

说的是我们的类型系统可以保证所有通过类型检查的项不会卡住



## Preservation

几个比较显然的引理：

1. 若 $\Gamma\vdash M:\tau$，则 $\Gamma,x:\sigma\vdash M:\tau$
2. 若 $\Gamma\vdash M:\tau$，则 $\Delta\vdash M:\tau$，其中 $\Delta$ 为 $\Gamma$ 的任意重排
3. 若 $\Gamma,x:\tau\vdash M:\sigma$ 且 $\Gamma\vdash N:\tau$，则 $\Gamma\vdash M[N/x]:\sigma$。证明只需要对 $M$ 结构归纳即可。

若 $\Gamma\vdash M:\tau$，且 $M\rightarrow M'$，则 $\Gamma\vdash M':\tau$

同样只需要对 $M$结构归纳即可。



## Erasure Property

定义类型擦除函数 $\text{erase}(\cdot)$。Erasure Property 说的是 $\text{erase}$ 函数建立了 $\lambda_\rightarrow$ 到 $\lambda$ 的同态（很显然不是双射）。或者你也可以说成是 $\text{erase}$ 函数与 $\rightarrow$ 是可交换的（虽然这里的两个箭头是各自项集上的二元关系）。

通常把这个 erasure 也叫抽象的 compilation，这时候就说的是无类型的 low-level（汇编层面）语义和原语言等价。
