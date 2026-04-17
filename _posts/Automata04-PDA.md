---
title: Automata04 PDA
date: 2022-11-09 16:53:04
tags: Automata
---

# Intro

一般说的 PDA 本质上是带栈的 $\epsilon$-NFA。PDA 这块可以看到非确定性引入的能力差别，这在正则语言、递归可枚举语言里是没有的。

# Def

形式化的 PDA 定义为七元组 $(Q,\Sigma,\Gamma,\delta,q_0,Z_0,F)$，其中 $\Gamma$ 表示栈上的符号集，$Z_0\in\Gamma$ 表示栈上的初始符号。

转移函数 $\delta$ 定义为 $\delta(q,c,Z)=\Set{(p,\alpha)\mid p\in Q, \alpha\in\Gamma^*}$，表示在状态 $q$ 下看到符号 $c$ 和栈顶为 $Z$ 时，可以从集合中随便挑一个元组 $(p,\alpha)$，然后走到状态 $p$，弹出栈顶 $Z$，压入一串新的栈符号 $\alpha$。

## ID

要描述一个运行中的 PDA 状态只需要一个三元组 $(q,w,\alpha)$，其中 $q\in Q,w\in\Sigma^*,\alpha\in \Gamma^*$。

若 $w=cx$，$\alpha=Z\beta$，且 $(p,\gamma)\in\delta(q,c,Z)$，那么当前状态可以转移到 $(p,x,\gamma\beta)$，记成 $(q,w,\alpha)\vdash(p,x,\gamma\beta)$。由 $\vdash$ 归纳得到 $\vdash^*$ 是简单的。



有定理：若 $(q,w,\alpha)\vdash^* (p,y,\beta)$，那么对一切 $x\in\Sigma^*,\gamma\in \Gamma^*$ 都有 $(q,wx,\alpha\gamma)\vdash^*(p,yx,\beta\gamma)$，反过来不成立。

直观地讲，这是因为栈单边有限，前一个转移序列的成立说明用不到后续的符号，因此可以任意叠。反过来不成立也是因为去掉底部垫着的符号可能会对空栈做 pop 操作。

## $L(P)$

对于给定的 PDA $P$，一种定义它接收的语言的方法为 $L(P)=\Set{w\in \Sigma^*\mid (q_0,w,Z_0)\vdash^* (q,\epsilon,\alpha), q\in F}$

即如果消耗完了整个串之后进入了接收状态，那么就认为这个串被接收，很符合命名。

## $N(P)$

另一种定义为 $N(P)=\Set{w\in\Sigma^*\mid (q_0,w,Z_0)\vdash^*(q,\epsilon,\epsilon)}$

即一旦栈空了，那么就认为这个串被接收。

## $L(P),N(P)$ Equivalence

任给一个 $P$，我们必然可以构造一个 $P'$ 使得 $L(P)=N(P')$

只需要在 $P$ 的接收状态之后通过自环不断弹栈直到空即可。

但这样可能会引入问题：在 $L(P)$ 中可能经过某些非接收状态，并且栈恰好是空的。如果我们什么都不做的话就会引入原本无法接收的串。

一个技巧是 $P'$ 引入一个新的栈符号 $Z'$ 放在栈底，然后再模拟 $P$，这样由于 $P$ 没有针对 $Z'$ 的转移，因此无论如何都不会空栈；并且最后的清理状态一视同仁地弹栈，也能保证接收。



任给一个 $P$，我们必然可以构造一个 $P''$ 使得 $N(P)=L(P'')$

同样引入 $Z'$，如果遇到 $Z'$ 在栈顶就直接进入接收状态。即令 $\forall q\in Q,(F,Z')\in\delta(q,\epsilon,Z')$

## DPDA

一般的 PDA 说的是 NPDA

DPDA 即必须有 $\forall c\in\Sigma,\lvert\delta(q,c,Z)\rvert=1$，且 $\forall c\in\Sigma,\lvert\delta(q,c,Z)\rvert + \lvert \delta(q,\epsilon,Z)\rvert <2$ 成立

确定性的引入带来了能力上的损失，考虑语言 $\Set{ww^R\mid w\in\Sigma^*}$。由于确定性可知对于串 $0^n110^n$，DPDA 必然会在某个位置把串划分为前后两部分，不妨假设这个 DPDA 正确地划分出了 $0^n1|10^n$。考虑新的输入 $0^n11110^n$，这个新的输入和之前的输入有相同的前缀，因此必然有相同的分隔位置，但是是错的。

## Containment

$$
\text{RE $\subseteq$ DPDA(L(P)) $\subseteq$ NPDA} \\
\text{RE $\not\subseteq$ DPDA(N(P)) $\subseteq$ NPDA}
$$

第一行是比较显然的。

第二行的不满足可以考虑 $\Set{0}^*$ 这个语言，它是正则的，但不存在 $P$ 使得 $\text{DPDA(N(P))}=\Set{0}^*$

## Ambiguity

若存在 DPDA $P$ 使得 $L=L(P)$，那么 $L$ 必然存在非歧义的语法，反过来则不一定成立（考虑 $\Set{ww^R\mid w\in \Sigma^*}$）

由 DPDA 可知任意串存在唯一确定的 ID 推导序列，这唯一对应了一个文法中的推导序列。

# CFG Equivalence

任给一个 CFG $G$，都能构造出一个 PDA $P$，使得 $L(G)=L(P)$；反过来也成立。

对于给定的 $G$，构造只有一个状态的 $P$，初始栈上放着变量 $S$，每次从栈顶弹出符号 $V$

1. 存在产生式 $V\to w\alpha$，其中 $w$ 是终止符号的序列，那么就消耗掉 $w$，压入 $\alpha$
2. 存在产生式 $V\to \beta$，其中 $\beta$ 的开头是非终止符，那么就压入 $\beta$

运用人类智慧可以发现这个东西本质上是在做非确定性的 DFS

---

对于给定的 PDA $P$，构造文法 $G$，其中变量都是 $[pXq]$ 的形式，我们希望有 $L([pXq])=\Set{w\mid (p,w,X)\vdash^*(q,\epsilon,\epsilon)}$ 成立

接下来对状态 $p$ 处的转移分类讨论：

1. $(q,\epsilon)\in \delta(p,c,X)$，说明这个转移只弹出 $X$。因此引入产生式 $[pXq]\to c$
2. $(r,Y)\in\delta(p,c,X)$，说明这个转移会先读一个 $c$，再走到 $r$，然后把栈顶的 $X$ 换成 $Y$。因此引入产生式 $[pXq]\to c[rYq]$
3. $(r,YX)\in\delta(p,c,X)$，即压入一个 $Y$。我们枚举弹出当前 $X$ 之后的状态 $s$，引入产生式 $[pXq]\to c[rYs][sZq]$

有这几个就足够了。最后还需要引入一个全新的初始变量 $S$，然后规定 $\forall p, S\to [q_0Z_0p]$

