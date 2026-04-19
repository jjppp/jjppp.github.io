---
title: Tour de SAT
date: 2023-12-30
permalink: /posts/2023/12/Tour-de-SAT/
tags:
  - SAT
---

最近对 SAT 求解器很感兴趣，来个三天带我入门

# Basics

why SAT

* 通用，意味着解决了它就能解决很多问题
* 重要，例如做硬件验证大量用到 SAT 建模问题
* 高情商：有理论上的意义

why PL

* 简单，存在判定算法
* 和一阶逻辑比起来虽然表现力不够，但足够解决一些实际问题
* 与有限结构的一阶逻辑等价

## Quantified Propositional logic

除去大家都懂的部分，额外多了

* $\gamma\mid R$
* $\theta\mid\neg R$
* $\forall P,\alpha$
* $\exists Q,\beta$

它们分别被定义为

1. $\gamma\mid R=\gamma[R\mapsto \bold{true}]$
2. $\theta\mid \neg R=\theta[R\mapsto \bold{false}]$
3. $\forall P,\alpha=(\alpha\mid P)\wedge (\alpha\mid\neg P)$
4. $\exists Q,\beta=(\beta\mid P)\vee(\beta\mid \neg P)$

这方面一些关于知识表示/AI 的术语非常高大上

## CNF

$$
\bigwedge_{i}\left(\bigvee_{j} C_{j}\right)
$$

一些术语：

* atom/literal：$P$ or $\neg P$
* clause：$\vee$ of atoms
* term：$\wedge$ of terms
* $\alpha$ satisfiable：存在一组解 $\sigma$ 使得 $\sigma\models\alpha$

## Operations

### Normalization

说人话：把任给的公式“编译”到特定形式，一般默认是 CNF。这就是逻辑人的 IR。

CNF 是 universal 的，这意味着它有足够的表现力

可以通过一些基本操作得到 CNF

* $\neg$ 分配
* $\vee$ 分配
* $\neg\neg$ 消去

也可以通过引入新变量构造 CNF

why CNF：

* 简单
* 相较而言 DNF 下的 SAT 问题是平凡的

### Decomposition

对于公式 $\alpha$，可以提取变量得到 $(P\wedge(\alpha\mid P))\vee(\neg P\wedge(\alpha\mid\neg P))$

相当于把一个公式拆成并行的两份分别运算，最后用一个 MUX 挑出真正的结果

### Resolution

对于形如 $(P\vee\alpha)\wedge(\neg P\vee\beta)$ 的公式，可以推导得到 $(\alpha\vee\beta)$

这样的操作叫 resolution between $(P\vee \alpha)$ and $(\neg P\vee\beta)$ over $P$，得到的结果 $(\alpha\vee\beta)$ 叫 $P$-resolvent

#### Completeness

就是你想的那个 completeness。

Resolution 并非 complete，考虑 $\alpha=A$，它并不能 resolve 得到 $\neg\neg A$

但 resolution is refutation-complete，意味着如果 $\alpha$ 不可满足，那么对 $\alpha$ 做 resolution 将得到形如 $P\wedge\neg P$ 的结果

这说明可以利用 resolution 判定公式是否不可满足

### Unit resolution

也叫 Boolean propagation。对于只含一个变量的 clause $P$，为了保证可满足需要赋予 $\sigma(P)=\textbf{true}$。这里的 $P$ 就叫 unit clause。

有点 `simpl.` 的感觉

## Strategies

现在问题变成了：

* 可以挑一个变量对 CNF 做 resolution
* 可以挑一个变量猜一个值，猜错了回溯

由于所有的 solver 本质上都是爆搜+回溯，那么搜索策略就变得比较有讲究了：

* 挑哪个变量做 resolution？
* 猜什么值？

### Linear resolution

resolve $\alpha,\beta$ iff

1. $\alpha,\beta$ are roots, or
2. $\alpha$ is the ancestor of $\beta$

### Directed resolution(DP)

大名鼎鼎，而且很古老

指的是按照一个顺序 resolve 变量，每次只 resolve 那些没有被动过的 clause（不走回头路）

更具体地，给定变量集 $V$ 上的顺序 $\preceq$，定义 $c(P)=\Set{\text{$\alpha$ is a clause}\mid \forall R\in var(\alpha), P\preceq R}$，即所有出现了 $P$ 但没有出现比 $P$ 更小的变量的 clause。

算法是这样的：

1. 初始化 $c$
2. 按顺序枚举变量 $P$
3. 两两 resolve $c(P)$ 中的 clause
4. 将结果更新到对应的 $c$ 中

### DPLL

名气更大

其实就是爆搜。枚举每个变量的取值后将得到一棵 dfs 树

每次搜完都利用 unit resolution 做一步化简，通过花费一点线性代价来换取搜索空间变小

对决策树做子树合并就得到了大名鼎鼎的 BDD

按照 DP 逆序操作带入也能还原出这棵决策树
