---
title: Automata02 RE
date: 2022-11-09 14:56:43
tags: Automata
---

# Intro

正则表达式（Regular Expression）是描述正则语言的另一种方法，有时候用起来会比 DFA 更方便。 

# Definition

给出归纳定义：

1. 单个字符 $a\in\Sigma$ 是正则表达式
2. $\epsilon$ 是正则表达式
3. $\varnothing$ 是正则表达式
4. 如果 $R_1,R_2$ 是正则表达式，那么 $R_1+R_2$ 也是正则表达式
5. 如果 $R_1,R_2$ 是正则表达式，那么 $R_1R_2$ 也是正则表达式
6. 如果 $R$ 是正则表达式，那么 $R^*$ 也是正则表达式
7. 正则表达式仅限于此

同时给出语义函数

1. $L(a)=\set{a}$
2. $L(\epsilon)=\set{\epsilon}$
3. $L(\varnothing)=\varnothing$
4. $L(R_1+R_2)=L(R_1)\cup L(R_2)$
5. $L(R_1R_2)=L(R_1)L(R_2)$
6. $L(R^*)=L(R)^*$

我们称一个语言 $L$ 是**正则语言**（Regular Language），当且仅当存在一个正则表达式 $R$ 使得 $L(R)=L$。

# Equivalence

任给一个正则表达式 $R$，都能构造一个 $\epsilon$-NFA $N$ 使得 $L(R)=L(N)$

这个构造具体可以看之前写过的 RE-compiler



任给一个 DFA $D$，都能构造一个 $R$ 使得 $L(D)=L(R)$

大概的想法是这样的：对于任意一个 DFA $D$，我们总可以挑出一个状态 $q\in Q_D$，使得在进行一次状态削减操作后得到新的 DFA $D'$，且 $Q_{D'}=Q_D-\set{q}$。这样在至多 $\lvert{Q}\rvert$ 次操作后就能得到状态数为 $2$、仅含有一条边的 DFA。这时唯一的边上的标号即为要求的正则表达式。

PPT 上 k-PATH 的说法本质上是按照编号递增的顺序去缩减状态。

# Properties

## Membership

即给定串 $w$ 和正则语言 $L$，判定是否有 $w\in L$

只需要找到 $L$ 的 DFA $D$，然后跑一下 $w$ 即可

## Emptiness

给定正则语言 $L$，判定是否有 $L=\varnothing$

只需要取 DFA $D$，然后从 $D$ 的初始状态出发广搜，看看能不能找到接收状态即可

## Infiniteness

给定正则语言 $L$，判定是否有 $\lvert L\rvert=\infty$

只需要取最小 DFA $D$，去掉不可达状态，判断 $D$ 中是否存在以初始状态为起点的圈即可。

## Equivalence

给定两个正则语言 $L_1,L_2$，判定是否有 $L_1=L_2$

取两个 DFA $D_1,D_2$， 只需要取乘积自动机 $D_1\times D_2$，然后令 $F'=\Set{(q_1,q_2)\mid\text{$q_1$ 和 $q_2$ 恰有一个是接受状态}}$

可以发现 $w$ 被 $D_1\times D_2$ 接收当且仅当 $w$ 恰好被 $D_1$ 或 $D_2$ 中的一个接收。因此 $D_1\times D_2$ 的 emptiness 就能说明 $D_1,D_2$ 的等价性。

类似的还可以做包含关系等等集合操作。

## Pumping Lemma

若 $L$ 是正则语言，则存在泵常数 $n$ 使得一切长度大于等于 $n$ 的串 $s$ 都能写成 $xyz$ 的形式，满足

1. $\lvert y\rvert >0$
2. $\lvert xy\rvert \le n$
3. $\forall i\in\mathbb N, xy^iz\in L$

证明是比较简单的，只需要取 $n=\lvert Q_\text{DFA}\rvert$ 就有串 $s$ 在 DFA 上跑的时候必然经过了重复的两个状态。这个圈可以不断重复（或删去），最终仍然能在 DFA 上走到终点。

## Closure

对正则语言做一些操作之后仍然会得到正则语言

### Union, Intersection, Difference, Complement

用乘积自动机。Complement 本质上是 $\bar L=\Sigma^*-L$，而 $\Sigma^*$ 是正则的

一个比较好用的性质是 $L\cap R$ 仍然是正则的，我们可以构造一个特殊的 $R$ 来“规范”字符出现的顺序，并利用反证法证明 $L$ 不是正则的。例如 $\Set{s\mid s\in\set{0,1}^*, \lvert s\rvert_0=\lvert s\rvert_1}$

### Concat, Kleene Closure

这两个都是正则表达式里的操作，只需要取出语言对应的 RE 然后操作一下就好了

### Reversal

定义函数 $\text{rev::RE$\to$RE}$

1. $\text{rev($a$)=}a$
2. $\text{rev($e_1+e_2$)=}\text{rev($e_1$)}+\text{rev($e_2$)}$
3. $\text{rev($e_1e_2$)=}\text{rev($e_2$)}\text{rev($e_1$)}$
4. $\text{rev($e^*$)=}\text{rev($e$)}^*$

### Homomorphism

同态说的是给定 $\Sigma_1$ 上的语言 $L_1$ 和函数 $f\colon\Sigma_1\to {\Sigma_2}^*$，定义 $\text{map}(f,s)=\prod_{i=1}^{\lvert s\rvert} f(s_i)$，那么可以构造新语言 $L_2=\Set{\text{map}(f,s)\mid s\in L_1}$。称 $L_2$ 是 $L_1$ 关于 $f$ 的同态，记为 $L_2=f(L_1)$

只需要取 $L_1$ 的正则表达式，然后把所有符号映射一下就得到 $L_2$ 的正则表达式了。

### Inverse Homomorphism

说的是给定 $L_2$ 和 $f$，求 $L_1=\Set{s\mid \text{map}(f,s)\in L_2}$，记作 $L_1=f^{-1}(L_2)$

这里并不要求 $f$ 是可逆的

取 $L_2$ 的 DFA $D_2$，并以此构造 $D_1$：$\delta_1(x,c)=\delta_2(x,f(c))$，其余一模一样
