---
title: Automata06 Complexity
date: 2022-12-08
permalink: /posts/2022/12/Automata06-Complexity/
tags:
  - Automata
---

# Intro

TM 是比较基本的计算模型，后续对“计算”本身的讨论也都会基于 TM 来做。

对于给定的问题，我们会粗略地考虑“计算难度”上的分类：

1. 这个问题可以判定吗？
2. 如果是可判定的，它花费的**计算代价**（Cost）是可接受的（Tractable Problems）吗？

一般的套路就是各种各样的规约，再加上各种各样已知的问题，在密码学的时候就应该见得很多了（虽然我学得很烂..）

# Decidability

从最大的讲起

## $\Sigma^*$

注意到 TM 本身可以被编码（七元组中的元素都是有限的）为串，因此 $\lvert{\Set{M\mid M\text{ is Turing Machine}}}\rvert=\aleph_0$ 

而 $\lvert\Sigma^*\rvert=\aleph_0$，因此所有**语言**组成的集合 $\lvert 2^{\Sigma^*}\rvert >\aleph_0$ 本身是不可数的，这说明不存在所有 TM 到所有语言的双射。

即必然存在某个语言没法用 TM 描述。

## RE/Partially Decidable

PD 的定义和 RE 是等价的，即不能保证停机的 TM $M$ 定义出的语言恰好就是半可判定的。经典的半可判定算法有一阶谓词逻辑中判定 skolem 范式是否永真的那个算法。

## co-RE

前面说过 RE 在补操作下不封闭，那么某个 RE 的补会是什么呢？

定义 $L$ 是 co-RE 当且仅当 $\bar L$ 是 RE。容易发现 $\text{RE}\subsetneq \text{co-RE}$，且 $\text{Decidable}=\text{co-RE}\cap\text{RE}$

## Decidable

称必然停机的 TM $M$ 为算法（Algorithm），$L(M)$ 为递归语言或可判定

# Complexity

对于可判定的问题才有讨论计算代价的必要，因此下面提到的 TM 默认都是算法（会停机），默认讨论的是时间复杂度。

渐进意义的复杂度已经讲烂了，这里只需要额外规定

1. TM 上的代价可以用磁头操作的次数表示（不动也算一次）
2. TM 的输入规模用输入串的长度表示

同样是从最大的讲起

## EXP

指需要代价为 $O(2^{\text{poly}(n)})$ 的 DTM。目前并不清楚是否有 $\text{NP $\subsetneq$ EXP}$。

## PSPACE

指需要**空间**代价为 $O(\text{poly}(n))$ 的 DTM。

## NP-hard

指难度不弱于 NP 的问题。即如果问题 A 满足 $\forall B\in NP$ 都有 $B\le_P A$，那么 A 就是 NP-hard 的。

这个定义可以拓展到 X-hard。即 $A\in \text{X-hard}\iff \forall B\in \text{X},B\le_P A$

## NP-complete

指 NP 中最难的那一批问题。定义为 $\text{NP-complete}= \text{NP} \cap\text{NP-hard}$

这个定义可以拓展到 X-complete。

### 例子

1. HAMILTON
2. 3-SAT
3. k-CLIQUE

## NP

指需要代价为 $O(\text{poly}(n))$ 的 NTM。关于对 NTM 的时间复杂度的定义可以[看这里](https://cs.stackexchange.com/questions/146318/how-is-the-time-complexity-of-a-non-deterministic-turing-machine-defined)。

另一个等价的定义是这样的：

对于一个 NP 问题（语言） $L$，我们总可以构造一个二元关系 $R$ 使得 $L$ 的一个等价定义为 $L=\Set{w\mid \text{$\exists s$, s.t. $(w,s)\in R$}}$，并且 $R$ 作为语言是个 P 问题。

这里的 $\exists s$ 是比较灵活的，并不需要严格是“问题本身的答案”。

一个经典的套路是用 DTM 模拟 NTM，此时状态数是指数增长的 $\sum_{i}k^i$，其中 $k$ 为 NTM 每一步的可选后继状态的最大值。

## P

需要代价为 $O(\text{poly}(n))$ 的 DTM。

# 规约

已知问题 A 的可判定性/复杂性，用来证明问题 B 的可判定性/复杂性

## Decidability

给几个不可判定的例子

1. HALT
2. $A_{TM}=\Set{\braket{M,w}\mid \text{$M$ accepts $w$}}$
3. $E_{TM}=\Set{\braket{M}\mid L(M)=\varnothing}$

HALT 的证明最经典，剩下两个都可以规约到 HALT。

实际上根据 Rice's Theorem 可知，TM 上的非平凡性质都是不可判定的。

## Complexity

涉及到复杂度的规约需要强调规约额外引入的复杂度。

如果问题 A 能被多项式规约到 B，且 B 是 P，那么 A 也是 P。

用符号描述就是 $A\le_P B$ 且 $B\in P$，那么 $A\in P$。

### 例子

1. k-SHORT-PATH 是 P 的
2. k-LONG-PATH 是 NP-hard 的，可以把 HAMILTON 规约到它

# Rice's Theorem

TM 上的非平凡性质不可判定。称性质 $P$ 是非平凡的，当且仅当 $\Set{M\mid P(M)}$ 和 $\Set{M\mid \neg P(M)}$ 都非空。

由反证法，设某个非平凡性质 $P$ 存在判定算法 $D$，我们以此来构造一个用来判定 HALT 的算法 $H$。

由于 $P$ 非平凡，因此必然存在 $M$ 满足 $P(M)$。不妨假设 $\varnothing\not\in P$，否则取 $\bar P$ 即可。

我们的 $H$ 设计如下：

1. 读入 $\braket{TM,w}$
2. 构造一个算法 $S$ 如下：
   1. 读入 $x$
   2. 模拟把 $w$ 喂给 $TM$
   3. **执行完2之后**，模拟把 $x$ 喂给 $M$，然后输出结果
3. 然后观察 $D(S)$
   1. 如果 $D(S)$ 接受，则说明 $S$ 最终等价与一个 $M$，意味着 $TM$ 停机了
   2. 如果 $D(S)$ 拒绝，则说明 $TM$ 没停机，$L(S)=\varnothing$

写成代码大概是这样

```go
func H(TM, w) {
    func S(x) {
        TM(w);
        return M(x);
    }
    if (D(S)) {
        return true;
    } else {
        return false;
    }
}
```

即我们利用了一个阻塞的计算过程使得 $S$ 是否拥有性质 $P$ 取决于 $\braket{TM,w}$ 是否停机，这样就可以利用 $P$ 的算法来判定停机问题，从而导出一个矛盾。