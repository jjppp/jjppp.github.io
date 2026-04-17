---
title: Automata01 FSM
date: 2022-10-04 21:13:07
tags: Automata
---

# Intro

FSM(Finite State Machine/Finite Automata) 是一种 formal system，当然用处有很多了。

# Basics

## String

取定一个字符集 $\Sigma$，其中元素称为字符。

定义串是字符的有限序列，也可以归纳定义如下：

1. $\epsilon$ 是特殊的串，称为空串
2. 若 $\alpha$ 为串，且 $x\in \Sigma$，则 $x\alpha$ 也为串。此处表示为字符与串的顺序拼接。

## Language

语言即串的集合，$\varnothing$ 和 $\Sigma^*$ 是两个平凡的语言

注意区分 $\Set{\epsilon}$ 和 $\varnothing$

# DFA

## Def

定义为五元组
$$
FSM\overset{\text{def}}= (S, \Sigma, \delta, s_0, F)
$$
分别表示状态集、字符集、转移函数、初始状态、接受状态集。

可以通过简单的归纳由 $\delta:S\to \Sigma\to S$ 构造 $\hat\delta:S\to \Sigma^*\to S$

定义自动机接受串 $\alpha$ 当且仅当 $\hat\delta(s_0,\alpha)\in F$

对于自动机 $A$，记它所能接受的串的集合为 $L(A)$，称为自动机 $A$ 表示的语言。

若某个语言 $L$ 可被某个 DFA $A$ 接受，则称 $L$ 为**正则语言**

## $\Set{0^n1^n\mid n\in\mathbb N^+}$

下面证明这个不是正则语言（好像证过很多次了）

由反证法设这是正则语言，则存在 DFA $A$ 使得 $L(A)=\Set{0^n1^n\mid n\in\mathbb N^+}$

不妨设 $A$ 状态数为 $m$，则必然存在 $0^m1^m\in L(A)$。考虑 $A$ 正在识别 $0^m1^m$ 的前缀 $0^m$，由鸽笼原理可知必然存在状态 $s$ 经过了两次，即 $A$ 中存在一个不为空的圈，且圈上只有 $0$。这说明我们可以走出一个 $0^{m+k}1^m$ 的串后到达接受状态，这与 $L(A)=\Set{0^n1^n\mid n\in\mathbb N^+}$ 矛盾。

故假设不成立。

# NFA

## Def

和 DFA 的区别仅在于转移函数
$$
\delta:S\to \Sigma\to 2^S
$$
其含义为：从状态 $s$ 出发，经过一条字符边后可能到达哪些状态，NFA 的转移不一定是唯一的。

不难发现 $\delta$ 本身是一个 monad，此时的 $\hat\delta=\text{return s >>= }\delta$

定义 NFA 接受串 $\alpha$ 当且仅当 $\hat\delta(s_0,\alpha)\cap F\neq\varnothing$

## Subset Construction

给定 NFA $A=(S,\Sigma,\delta,s_0,F)$，我们可以构造 DFA $A'=(2^S,\Sigma,\delta',\set{s_0},\Set{s\in 2^S\mid s\cap F\neq \varnothing})$

关键的 $\delta'$ 可以用 monad bind 定义...

## Equivalence

我们说 DFA 和 NFA 的表达能力是等价的，意思是：

对于任意的 NFA $A$，都存在 DFA $A'$ 使得 $L(A)=L(A')$，反之亦然。

这也说明所有的 NFA 能表达的语言类与所有的 DFA 能表达的语言类相等。

"$\Rightarrow$"：

考虑 $A$ 的子集构造 $A'$，即证明对任意的串 $\alpha$，有 $\delta(s_0,\alpha)=\delta'(\set{s_0},\alpha)$。简单地对 $|\alpha|$ 归纳即可。

"$\Leftarrow$"：

很显然 DFA 是一种特殊的 NFA。

# $\epsilon$-NFA

## Def

符号集加上 $\epsilon$

引入 closure 操作 $C:S\to 2^S$，含义为从某个状态出发，只走 $\epsilon$-边能到达的状态集合。

这一定义可以通过 monad bind 简单拓展为 $C:2^S\to 2^S$，称为状态集的 $\epsilon$-closure。这个操作显然是幂等的。

不妨记不含 $\epsilon$-边的转移函数为 $\delta$，则加入 $\epsilon$-边后的新转移函数可以简单定义为 $\delta'=C\circ\delta\circ C$，意思是每走一步前后都做一次闭包。

## Equivalence

NFA 显然是一种特殊的 $\epsilon$-NFA。

注意到上面的 $\delta-\delta'$ 提供了一种消去 $\epsilon$-边的方法，这样就能由 $\epsilon$-NFA 得到等价的 NFA 了。

# Minimization

## $\simeq$

定义 DFA 上状态等价 $\simeq$，当且仅当以它们为起点能接受的串的集合相等。由此可以立即得到接受状态和非接受状态不等价（考虑 $\epsilon$）。很显然这是一个二元等价关系。

并且有引理：若状态 $A_1,B_1$ 不等价，且 $\exists c\in\Sigma$ 使得 $\delta(A_2,c)=A_1,\delta(B_2,c)=B_1$，那么 $A_2,B_2$ 不等价。

引理的证明十分简单，由 $A_1,B_1$ 的不等价性可知存在串 $s$ 使得 $A_1,B_1$ 中恰好一个接受 $s$，此时考虑 $cs$ 即为区分出 $A_2,B_2$ 的串。

引理的逆否形式也很有用，即等价状态的对应后继状态也会是等价的。



由此得到一个构造算法：首先区分出所有的终止状态和非终止状态，然后进行如下操作：

1. 枚举每对暂时无法区分的状态对，看看它们的后继状态是否可区分
   1. 可区分，则它们也可以区分
   2. 不可区分，则它们暂时不可区分
2. 更新分类，重复1操作直至收敛。

此时得到一个状态集的划分，由此构造新的 DFA 即为最小化的 DFA。

## Proof

记最小化算法得到的 DFA 为 $\bar A$，由反证法设存在更小 DFA $A$ 与之等价

1. 必然有 $\bar A$ 的初始状态与 $A$ 的初始状态等价（这里是跨越 DFA 的等价）
2. 任意等价的状态对，它们的后继状态也等价

因此必然存在 $\bar A$ 中两个状态它们与同一个 $A$ 中状态等价，由传递性可知它们彼此等价，这与 $\bar A$ 的定义矛盾。
