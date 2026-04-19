---
title: Automata03 CFL
date: 2022-11-09
permalink: /posts/2022/11/Automata03-CFL/
tags:
  - Automata
---

# Intro

上下文无关语言（Context Free Language）是一类比正则语言更强的语言。描述上下文无关语言的文法叫上下文无关文法（CFG）。

# CFG

上下文无关文法定义为四元组 $G=(T,V,S,P)$，其中

* $T$ 是符号集，也叫Terminals
* $V$ 是变量集
* $S$ 是初始变量
* $P$ 是产生式的集合（Set of Productions）

定义 $T^*$ 是串，$(T\cup V)^*$ 是句子（sentential form）

上下文无关文法的大概想法是这样的：我们总是可以从初始变量开始，通过不断地把一个句子中的符号替换成另一个句子，最终得到一个串。这里“替换”指的就是关于**产生式**的应用，并且“上下文无关”的名字来源于“我们可以无脑替换”这一操作策略。

一个比较无聊的帮助理解的例子是这样的：可以把变量看成 STLC 里的变量，然后在最前面加上 quantifier，这样所谓的应用产生式就是 function application，串就是 value，产生式就是 function。

若 $S$ 能经过若干步替换得到句子 $\alpha$，我们就写作 $S\Rightarrow^* \alpha$，称 $S$ 推导出（derives）$\alpha$。通常不要求 $S$ 一定是变量。

很显然 CFG $G=(T,V,S,P)$ 定义的语言就是 $\Set{s\mid S\Rightarrow^* s}$

## Derivation

这部分比较绕。

注意到在定义“推导”的时候我们并没有规定替换符号的顺序，因此给定语法 $G$ 时，对于某个串 $s$ 可能存在多个推导序列。一个平凡的例子是 $S\rightarrow AA, A\rightarrow 0$

1. $S\Rightarrow AA\Rightarrow 0A\Rightarrow 00$
2. $S\Rightarrow AA\Rightarrow A0\Rightarrow 00$

当然我们更关注的其实是推导作为树的结构，而不是线性序的形状，因此可以通过限定变量替换的顺序来得到推导树与推导序列的**双射**。常用的就是最左和最右推导。

关于双射的证明可以用简单的归纳得到...

## Ambiguity

消除了线性序上的歧义还不够，对于给定的 $G$ 和串 $s$，如果存在 $s$ 的两个不同推导树，那么就称 $G$ 是有歧义的（ambiguous）。由推导树与最左推导的双射可知，$G$ 有歧义 iff 存在 $s$ 有两个不同的最左推导。

注意到这里的歧义是语法的性质，如果是编译课就会讲怎么设计文法消除歧义。但实际上并非所有的上下文无关语言都能消除歧义，一个例子是 $L=\Set{0^i1^j2^k\mid i=j\vee j=k}$：

1. 先推出 $i=j$，再推 $j=k$
2. 先推出 $j=k$，再推 $i=j$

## Normal Form

讲的是对于给定的语法 $G$，怎么化简得到等价的 $G'$。

这里需要特殊处理一下 $\epsilon$。若 $\epsilon\in L(G)$，那么我们实际上可以构造 $G'$ 使得 $L(G')=L(G)-\Set{\epsilon}$，最后单独考虑 $\epsilon$ （通过单独加产生式）。之所以特殊处理是为了形式更简单更好看（$\epsilon$ 在连接操作下是幂等的，可以任意出现而不改变语言，神出鬼没）。

### $\epsilon$-Production

若 $V\Rightarrow \epsilon$，那么就称这个产生式是 $\epsilon$-Production，这一步的目的是去掉长成这样的产生式。

#### Identification

归纳地定义变量上的谓词 $\text{Nullable}$ 如下：

1. 若 $V\to \epsilon$，则 $\text{Nullable}(V)$
2. 若 $V\to \alpha$，且对 $\alpha$ 中的任意变量 $X$ 都有 $\text{Nullable}(X)$，则 $\text{Nullable}(V)$

容易看出 $\text{Nullable}(V)$ 的含义是 $V\Rightarrow^* \epsilon$。

#### Removal

对于一条产生式 $V\to \alpha$，我们可以根据 $\text{Nullable}()$ 的结果将这条产生式变成若干产生式。

具体而言是这样的，假设 $A\in\text{Nullable}$，那么我们必然可以引入中间变量 $A'$ 写成如下形式：

1. $A\to \epsilon$
2. $A\to A'$，且 $A'\not\in \text{Nullable}$

当 $A$ 出现在产生式的右侧时，我们不能直接删去1，考虑这个例子：

$S\to AB$，$A\to \epsilon\mid a$，$B\to b$，这个语法规定的语言是 $\Set{ab,b}$。假设我们删掉了 $A\to\epsilon$，那么语言就会变成 $\Set{ab}$，这样引入了语义变化。

注意到表达“空”有两种办法：

1. 变量推导出 $\epsilon$
2. 从产生式里删掉这个变量

因此可以先构造 $S\to A'B\mid B$ 来语法化表示 $\epsilon$，此处的 $A'$ 定义为 $A'\to a$，这样就维持了语义等价。

推广到产生式右侧有 $k$ 个 $\text{Nullable}$ 变量，那么就是新构造 $2^k$ 个产生式，然后分别递归消消乐就好了。

### Unit Production

Unit Production 说的是形如 $A\to B, B\to C$ 这样的产生式。

如果 $A\Rightarrow ^* B$，且 $B\to \alpha$，$\alpha$ 中不止一个变量，那么就引入新产生式 $A\to \alpha$。

反复添加直至收敛，然后就可以删去那些 Unit Production 了。

这个有点 copy propagation 的感觉，或者可以认为是图的 contraction。

### Divergent

如果一个符号 $A$ 满足 $L(A)=\varnothing$，那么以 $A$ 为起点的推导将永不终止。这样的符号也可以直接删掉。

#### Identification

构造谓词 $\text{Stoppable}$ 如下：

1. 若 $V\to s$，那么 $\text{Stoppable}(V)$
2. 若 $V\to\alpha$，且 $\alpha$ 中的一切变量 $X$ 都有 $\text{Stoppable}(X)$，那么 $\text{Stoppable}(V)$

#### Removal

对于 $V\not\in \text{Stoppable}$，删去 $V$ 所处的产生式（左侧和右侧都算出现）

### Unreachable

若 $S\not\Rightarrow^* V$，那么 $V$ 就是 unreachable symbol，它和它的产生式都可以直接删掉。

这个就是 deadcode 了。

## CNF

大名鼎鼎的 Chomsky Normal Form

大概意思是在做完上面那些 clean up 之后，我们的语法中的产生式右侧长度至少为 $2$（没有 Unit Production 和 $\epsilon$-Production）

那么可以人为地引入变量把语法树变成二叉的，并且叶子全是 $V\to c$，$c\in\Sigma$ 这样的形式。

之所以这么做是为了证明上的方便，比如 CNF 的形式可以给树高估一个界。
