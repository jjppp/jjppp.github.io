---
title: Algebra02 子群和商群
date: 2022-07-06 18:06:58
tags: Algebra
---

\newcommand\norm[1]{\left\lVert#1\right\rVert}
\newcommand\abs[1]{\left\lvert#1\right\rvert}

主要关注怎么由已有群得到新的群. 如果上过正经高代应该会对这一套流程比较熟悉

# 子群

$ G$ 为子群, 非空子集 $H\subseteq G$ 是 $G$ 的子群当且仅当 $H$ 在 $G$ 中同一运算下为群.

必然有 $e\in H$, 且 $e_H=e_G$. 由反证法设 $e_H\neq e_G$, 则 $e_H e_G = e_H = e_G$, 此处的运算均在 $G$ 中进行.

# 子群的等价定义

## 定义1

$H\subseteq G$ 为子群, 当且仅当

1. $\forall a, b\in H, ab\in G$
2. $\forall a\in H, a^{-1}\in G$

容易验证此定义与 $H$ 为群等价.

## 定义2

$H\subseteq G$ 为子群, 当且仅当 $\forall a, b\in H, ab^{-1}\in H$

注意到 $\forall a\in H, aa^{-1}=e\in H$, 且 $\forall a\in H, ea^{-1}=a^{-1}\in H$. 易得定义2与定义1等价.

## 子群之交

子群之交仍为子群.

若 $H_1,H_2\leqslant G$ 且 $H_1\neq H_2$, 那么 $H_1\cap H_2\leqslant G$

很显然 $H_1\cap H_2$ 非空 (至少有 $e\in H_1, e\in H_2$)

$\forall a,b\in H_1\cap H_2$, 那么有 $a,b\in H_1$ 且 $a,b\in H_2$.
这说明 $ab^{-1}\in H_1$ 且 $ab^{-1}\in H_2$, 即 $ab^{-1}\in H_1\cap H_2$. 由定义2可知 $H_1\cap H_2$ 为群.

## 子群之并

子群之并不一定仍为子群.

取 $H_1,H_2\leqslant G$, 且 $H_1\neq H_2$ 没有包含关系.

由反证法, 设 $H_1\cup H_2$ 仍为子群, 则取 $a\in H_1\backslash H_2, b\in H_2\backslash H_1$, 观察 $c=ab$

1. $c\in H_1$, 此时 $a, c\in H_1$, 由定义2 $ca^{-1}=b\in H_1$ 矛盾;
2. $c\in H_2\backslash H_1$, 此时 $b, c\in H_2$, 由定义2 $cb^{-1}=a\in H_2$ 矛盾;

故假设不成立, $H_1\cup H_2$ 不是子群.

当然在 $H_1\leqslant H_2\leqslant G$ 的情况下, $H_1\cup H_2=H_2\leqslant G$ 是一个子群.

## 有限子群

有限群 $G$ 的子集 $H$ 若对运算封闭, 那么 $H\leqslant G$

考虑任取 $a\in H$, $\Set{a^1， a^2\ldots a^k}$, 由 $G$ 的有限性必然存在 $k'\in \mathbb{N}$ 使得 $a^{k'}=e$. 由这个序列容易得到 ${a}^{-1}=a^{k'-1}$.

# 商群

对于群 $G$ 和子群 $H\leqslant G$, 可以定义 $G$ 上的二元关系 $R_H$ 为

$$
\forall a,b\in G, aR_Hb\iff a^{-1}b\in H
$$

易验证 $R_H$ 是 $G$ 上的等价关系, 由此可导出一个 $G$ 的划分 $G/R_H$, 也可记作 $G/H$.

这个等价关系看起来也许有些奇怪, 可以考虑一个线性空间中的直观例子: 在三维欧式空间 $V$ 中, 取 $z$ 轴这个子空间 $Z$. 

$\forall \alpha,\beta\in V$, 定义二元关系 $\alpha R\beta\iff \alpha-\beta\in Z$. 可以发现, 这样定义的二元关系"连接"了 $z$ 轴垂直穿过的所有水平平面上的点, 此时作陪集划分 $V/X$ 就是把三维空间"压成"了二维平面, 新平面中的每个点表示原本的一条垂直线.

## $G$ 上关于 $H$ 的陪集划分

给定 $H\leqslant G$, 关于元素 $a\in G$ 的左陪集定义为 $aH=\Set{ah\mid h\in H}$

有 $aH=[a]_R$, 即元素 $a$ 的左陪集恰好等于其在关系 $R_H$ 下的等价类. 即 $\forall b\in G, aR_H b\iff b\in aH$. 证明是简单的.

通常把 $G/H$ 称为 $G$ 关于 $H$ 的左陪集划分, 也叫 $G$ 关于 $H$ 的左商集空间.

### 指数和 Lagrange 定理

$G$ 为有限群.

定义 $[G:H] = \abs{G/H}$ 为 $H$ 在 $G$ 中的指数. 由左消去律可知 $\forall a\in G, \abs{aH}=\abs{H}$, 且 $G/H=\Set{aH\mid a\in G}$ 为一个划分, 因此 $\abs{H} [G:H] = \abs{G}$

若有 $H\leqslant K\leqslant G$ 为群, 那么有

$$
[G:K]=[G:H] [H:K]
$$

连续两次 Lagrange 定理即得.

## 同余关系

$G$ 中运算导出到 $G/H$ 中, 要求 $R$ 是同余关系, 即

$$
\left.
\begin{aligned}
a^{-1}b\in H
\\
c^{-1}d\in H
\end{aligned}
\right\}
\Rightarrow 
{c}^{-1}{a}^{-1}bd\in H
$$

为了形式上的对称, 可以写成

$$
{c}^{-1}{a}^{-1}b(c {c}^{-1})d={c}^{-1}({a}^{-1}b)c ({c}^{-1}d)\in H
$$

注意到 ${c}^{-1}d\in H$, 因此上式成立只需要 ${c}^{-1}({a}^{-1}b)c\in H$. 又由 $a,b$ 的任意性, 只需要 ${c}^{-1}hc\in H$ 对任意的 $h\in H$ 成立. 这是正规子群的来源.

## 正规子群

$H\leqslant G$, 称 $H$ 为 $G$ 的正规子群, 当且仅当 $\forall g\in G, \forall h\in H, {g}^{-1}hg\in H$, 记为 $H\lhd G$


### 左右陪集定义

若 $H\leqslant G$ 且 $\forall a\in G$ 都有$aH = Ha$ 则 $H\lhd G$

注意到 $aH=Ha$ 可得 $\forall h\in H, ah {a}^{-1}\in H$, 且 $H\lhd G $ 可得 $\forall h\in H, \exists h'\in H \text{ s.t. } ah {a}^{-1} = h'$, 此时 $ah\mapsto h'a$ 即为 $aH\mapsto Ha$ 的双射.

### 陪集运算定义

若 $H\leqslant G$ 且 $\forall a,b\in G, aH bH=\Set{ah_1bh_2\mid \forall h_1,h_2\in H}=abH$, 那么有 $H\lhd G$.

若 $H\lhd G$, 由正规子群的左右陪集定义, $ah_1bh_2=a(h_1b)h_2$, 而 $h_1b\in Hb=bH$, 因此存在 $h_1'$ 使得 $ah_1bh_2=abh_1'h_2\in abH$

若 $aHbH=abH$, 则取 $b={a}^{-1}$ 就有 $aH {a}^{-1} H=H$. 这说明 $\forall h_1,h_2\in H$, $ah_1 {a}^{-1} h_2\in H$, 即 $ah_1 {a}^{-1}\in H$.

非常奇妙, 我们怀着凑出一个同余关系的目的, 找到了一类特殊的子群, 并且发现在这类子群下的左右陪集划分竟然相等, 对称性在这里统一了.

## 同余关系和正规子群

给定子群 $H\leqslant G$, 我们有

$$
\text{$R_H$ 是 $G$ 上的同余关系}\iff H\lhd G.
$$

"$\Rightarrow$":

由同余关系的定义, $\forall a,b,c,d\in G, a R_H b, c R_H d\Rightarrow ac R_H bd$

即给定 ${a}^{-1}b\in H, {c}^{-1}d\in H$ 必然有 ${\left({ac}\right)}^{-1}bd\in H$, 求证 $H\lhd G$.

任取 $g\in G, h\in H$, 显然有 $ghR_H g$ 且 ${g}^{-1}R_H e$. 由同余关系, $gh {g}^{-1} R_H g$, 即 $gh {g}^{-1}\in H$.

"$\Leftarrow$":

由正规子群的左右陪集定义, $aH=Ha$

注意到 ${a}^{-1}b\in H$, 因此 ${\left({ac}\right)}^{-1}bd={c}^{-1}{a}^{-1}bd\in {c}^{-1}Hd={c}^{-1}dH=H$.

