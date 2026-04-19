---
title: TAPL03 Untyped Lambda
date: 2022-07-23
permalink: /posts/2022/07/TAPL03-Untyped-Lambda/
tags:
  - TAPL
---

之前写形式语义已经来过一次了，这里就跳过一点写过的

# Untyped Lambda Calculus

如果说理解C语言的最好办法就是看看编译后得到的指令序列的执行过程，那么理解FP的最好办法就是看看最底层的 $\lambda$-calculus 的执行过程。并且此处是 Untyped $\lambda$-calculus，与汇编有奇妙的对应关系。

$\lambda$-calculus 本身为理解语言(desugar)提供了工具，并且可以很容易地添加各种 construct 来得到更高级的语言，这与利用汇编来理解C语言和为 runtime 增加特性以支持上层语言也是类似的。

## Syntax

$$
T::= x\mid \lambda x.\; T \mid T_1\;T_2
$$

分别叫 Variable Abstraction Application

### Value

定义 ULC 中的值为形如 $\lambda x.\;T$ 的项，其中 $T$ 是任意项。可以发现这样意味着函数和值是等价的。

### $\alpha$

即 renaming。若项 $T$ 经过 binder 换名后得到 $T'$，那么就称 $T,T' \;\alpha$-equivalent，记作 $T=_\alpha T'$，这是 $T$ 上的一个等价关系(同余关系)

### $\beta$

定义形如 $(\lambda x.\; T)\;T'$ 的项为 redex(Reducible Expression)，含义为可以进行一步执行的语句。这样一次 apply 操作也叫做 $\beta$-reduction。

基于 $\beta$-reduction 可以定义二元关系 $\rightarrow_\beta$、$\twoheadrightarrow_\beta$、$=_\beta$，分别表示一步规约、任意多步规约、在规约意义下的等价关系

## Semantics

这里默认大家都会项的替换了(回去看数理逻辑/形式语义课件)，关键在于给 redex 定序

### full $\beta$-reduction

$$
\begin{aligned}
& \frac{T\rightarrow T'}{\lambda x.\; T\rightarrow \lambda x.\; T'} \\\;\\
& \frac{T_1\rightarrow T_1'}{T_1\;T_2\rightarrow T_1'\; T_2} \\\;\\
& \frac{T_2\rightarrow T_2'}{T_1\;T_2\rightarrow T_1\; T_2'} \\\;\\
& \overline{(\lambda x.\; T)\;T'\rightarrow T[T'/x]}
\end{aligned}
$$

这四条规则是平级的，意思是每次随便挑一个 redex 做 $\beta$-reduction

### normal order strategy

$$
\begin{aligned}
& \overline{(\lambda x.\; T)\;T'\rightarrow T[T'/x]} \\\;\\
& \frac{T\rightarrow T'}{\lambda x.\; T\rightarrow \lambda x.\; T'} \\\;\\
& \frac{T_1\rightarrow T_1'}{T_1\;T_2\rightarrow T_1'\; T_2}
\end{aligned}
$$

这三条规则是从上到下的顺序依次执行的。normal order strategy 说的就是每次找到最左、最外侧的 redex 执行一步。

### call by name strategy

$$
\begin{aligned}
& \overline{(\lambda x.\; T)\;T'\rightarrow T[T'/x]} \\\;\\
& \frac{T_1\rightarrow T_1'}{T_1\;T_2\rightarrow T_1'\; T_2}
\end{aligned}
$$

意思是不能对函数体预先执行，并且在计算函数的时候将实参表达式作为整体替换掉函数体中的形参(回忆 $\lambda$-calculus 中项的替换需要一些小操作满足替换后语义不变)，这就是 call-by-name 的含义。

haskell 使用的是 call-by-name 的一个变体 call-by-need，含义就是把相同的多个表达式捏成一个(指向同一个表达式实体)，参数仍然整体代换，并只在需要时求值。这样得到的其实是一个 Abstract Syntax Graph。

### call by value strategy

$$
\begin{aligned}
& \overline{(\lambda x.\; T)\;(\lambda y.\; T')\rightarrow T[\lambda y.\; T'/x]} \\\;\\
& \frac{T'\rightarrow T''}{(\lambda x.\; T)\;T'\rightarrow (\lambda x.\; T)\;T''} \\\;\\
& \frac{T_1\rightarrow T_1'}{T_1\;T_2\rightarrow T_1'\; T_2}
\end{aligned}
$$

含义是所有的函数体不能预先执行、实参必须化简为值后才能做形参代换。

一个比较有意思的例子是这样的，考虑经典的

$$
\begin{aligned}
\Omega&=(\lambda x.\; x\; x)\;(\lambda x.\; x\; x)
\\
\text{id}&=\lambda x.\; x
\\
\text{false}&=\lambda t.\;\lambda f.\; f
\end{aligned}
$$

对于如下程序

$$
\text{false}\;\Omega\; id
$$

CBV 将不会终止，CBN 和 normal order 将会得到 $\text{id}$，full $\beta$-reduction 的结果则是不确定的(很显然它至少包含了其余三种的结果)

## ADT

之前也记了很多用 $\lambda$-calculus 来编码数据的操作。当我们在讨论 encode sth. in ULC 的时候，我们**实际上是在建立某个数学模型到代码的同构**。

常数据、数据构造子、数据上的操作函数共同构成了抽象数据类型的“公理”，即定义了这个类型中元素的集合。

以 list[A] 为例。我们不妨假设 $A$ 可以被 ULC encode，此时记 $L$ 为集合 $A$ 上所有 list 构成的集合，那么有

$$
\begin{aligned}
& \overline{\text{NIL}\in L} \\\;\\
& \frac{l\in L,a\in A}{\text{cons}(a,l)\in L}\\\;\\
& \frac{l\in L,a\in A}{\text{head}(\text{cons}(a,l))=a}\\\;\\
& \frac{l\in L,a\in A}{\text{tail}(\text{cons}(a,l))=l}
\end{aligned}
$$

此处 $\text{NIL}$ 为常元，$\text{cons}\colon A\times L\mapsto L$ 是数据构造子，$\text{head}\colon L\mapsto A$ 和 $\text{tail}\colon L\mapsto L$ 是数据上的操作函数。这就是一个简单的代数(公理)系统。

encoding 的过程就是构造所有 ULC 的项 $T$ 到所有链表 $L$ 的双射 $\pi\colon L\mapsto T$，以及对应的数据构造子、操作函数，使得这些映射对于这些操作而言是同构的。

从工程的角度看，这就是给出了一个 formal specification，然后我们用 formal 的方法证明了这个encoding(实现)是完全符合 spec 的。就这么简单。

## Recursion

大名鼎鼎的 Y-combinator，给出 call-by-name 时的一种定义

$$
Y = \lambda F.\; (\lambda x.\; F\; (x\; x))\; (\lambda x.\; F\; (x\; x))
$$

只需要注意到

$$
YF = F\; ((\lambda x.\; F\; (x\; x))\; (\lambda x.\; F\; (x\; x)))=F(YF)
$$

在 call-by-value 时，求值顺序会强迫 $Y$ 不断执行，意味着没法终止。这时需要一个小技巧把 $x\; x$ 这样的项在不改变语义的情况下变成值

$$
Y'=\lambda F.\; (\lambda x.\; F\; (\lambda y.\; x\;x\;y))\; (\lambda x.\; F\; (\lambda y.\; x\;x\;y))
$$

非常简单，用C语言类比也就是把一条可以执行化简的表达式 `a+b+c;` 变成了一个函数 `int func() { return a+b+c; }` 使得执行不能深入函数体内(hso)。

# De Bruijn Form

Q: 这个名字怎么念？

$=_\alpha$ 是等价关系，因此可以做 $T/=_\alpha$ 得到所有项在换名等价意义下的代表元。即我们希望研究一种等价意义下最简单的形式。

实际上 De Bruijn 敏锐意识到自然数就可以作为变量名，并且自然数天生可以表征一些 AST 上的结构信息，使得我们的解释器可以写起来更方便。

## intuition

考虑 ULC 这个树形结构，一个约束变量的 binder 必然是其出现节点的祖先。De Bruijn Form 考虑的就是用自然数来表征每个约束变量 binder 的位置，即从当前节点要往上爬多少层 Abstraction 才能走到 binder(因为只有 Abstraction 会引入新的 binder)。不妨记这个为 binder depth。

## naming context

若 $FV(T)=\varnothing$，则称 $T$ 为封闭项(closed term)或组合子(combinator)。

还可以定义 $\Lambda^k=\Set{t\in T\mid \lvert{FV(t)}\rvert=k}$，那么 combinator 就是 $\Lambda^0$

$FV(T)$ 中的变量是找不到 binder 的，因此引入 naming context 的概念来给自由变量规定 binder，定义 context 为 $\Gamma\colon V\mapsto \mathbb N$ 用以表示规约的环境。

## shifting

注意到替换 $T[U/x]$ 本质上是两棵 AST 的嫁接，$U$ 中的自由变量的 binder 应当整体增加 $x$ 在 $T$ 中的 binder depth。

结构归纳地定义函数 $\uparrow_c^d(t)$ 如下

$$
\begin{aligned}
\uparrow_c^d(k)&=\left\{\begin{aligned}k+d&,k\ge c\\k&,k<c\end{aligned}\right. \\
\uparrow_c^d(\lambda x.\; t)&=\lambda x. \uparrow_{c+1}^d(t) \\
\uparrow_c^d(m\; n)&=\uparrow_c^d(m)\;\uparrow_c^d(n)
\end{aligned}
$$

这个操作就叫 shifting。

## substitution

$$
\begin{aligned}
(\lambda .\; t)[t'/x]&=\lambda .\; (t[\uparrow^1(t')/(x+1)])
\end{aligned}
$$

## E-APP

$$
(\lambda .\; m)\;n\rightarrow \uparrow^{-1}(m[\uparrow^1(n)/0])
$$