---
title: Algebra03 同态与同构
date: 2022-07-09 00:26:16
tags: Algebra
---
\newcommand\norm[1]{\left\lVert#1\right\rVert}
\newcommand\abs[1]{\left\lvert#1\right\rvert}

群的同构意味着二者本质上没有区别, 同态则可以导出某些同构.

下面默认 $G_1, G_2$ 是两个群.

# 同态

若存在映射 $f\colon G_1\mapsto G_2$, 且 $\forall a,b\in G_1, f(ab)=f(a)f(b)$, 那么称 $f$ 是 $G_1$ 到 $G_2$ 的同态.

若 $f$ 为单射, 则 $f$ 为 $G_1$ 到 $G_2$ 的单同态.

若 $f$ 为满射, 则 $f$ 为 $G_1$ 到 $G_2$ 的满同态.

下面默认 $f$ 是 $G_1\mapsto G_2$ 的同态.

## 定理1

若有同态 $f\colon G_1\mapsto G_2, g\colon G_2\mapsto G_3$, 那么有同态 $g\circ f\colon G_1\mapsto G_3$ 

证明是简单的.

## 定理2

$f(e_1)=e_2$, 其中 $e_1, e_2$ 分别为 $G_1, G_2$ 的幺元.

注意到 $f(e_1)f(e_1)=f(e_1e_1)e_2=f(e_1)e_2$, 由消去律 $f(e_1)=e_2$.

## 定理3

$\text{Im}f\leqslant G_2$.

1. 结合律是显然的
2. $f(e_1)$ 是幺元
3. ${f(a)}^{-1}f(b)=f({a}^{-1}b)\in\text{Im}f$

因此 $\text{Im}f$ 成群. 又因为 $\text{Im}f\subseteq G_2$, 所以 $\text{Im}f\leqslant G_2$.

这说明 $f$ 即使不是满同态, 也可以选取 $G_2'=\text{Im}f$ 来使得 $f$ 为 $G_1\mapsto G_2'$ 的满同态.

# 核

定义 $\ker f=\Set{a\in G_1\mid f(a)=e_2}$, 称为 $f$ 的核, 也叫零空间.

## 定理4

$\ker f\lhd G_1$.

首先 $e_1\in \ker f$, 且 $\forall a,b\in G_1$, 若 $f(a),f(b)\in \ker f$, 则 $f({a}^{-1}b)=f({a}^{-1})f(b)=e_2$, 因此 $\ker f\leqslant G_1$.

接着 $\forall g\in G_1, h\in \ker f$, 有 $f({g}^{-1}hg)=f({g}^{-1})f(h)f(g)=f({g}^{-1}g)=e_2$.

## 定理5

$f$ 是单同态, 当且仅当 $\ker f=\set{e_1}$.

若 $\ker f=\set{e_1}$, 由反证法, 设 $\exists a,b\in G_1 \text{ s.t. } f(a)=f(b)=c\in G_2$

那么有 $f({a}^{-1}b)={f(a)}^{-1}f(b)={c}^{-1}c=e_2$, 而 ${a}^{-1}b\neq e_1$, 这与 $\ker f=\set{e_1}$ 矛盾. 故假设不成立.

另一侧是显然的.

# 同构

若 $f\colon G_1\mapsto G_2$ 既是满同态, 又是单同态, 那么 $f$ 就是一个同构.

## 群同态第一定理

定理的顺序有很多说法

若 $f\colon G_1\mapsto G_2$ 是满同态, $\ker f\lhd G_1$, 那么 $\sigma\colon G_1/\ker f\mapsto G_2$ 是同构, 其中 $\sigma(a\ker f)=f(a)$.

有些地方会表述成 $\sigma\colon G/\ker f\mapsto \text{Im }f$, 看起来更漂亮一些.

首先需要证明 $\sigma$ 是良定义的映射, 即 $\forall a,\forall b\in [a], \sigma(a\ker f)=\sigma(b\ker f)$. 注意到 ${a}^{-1}b\in\ker f$, 因此 $f({a}^{-1}b)={f(a)}^{-1}f(b)=e_2$, 立即有 $f(a)=f(b)$.

$\sigma$ 显然是满同态; 

$\forall a,b\in G$, $f(a)=f(b)\Rightarrow {a}^{-1}b\in \ker f\Rightarrow a\ker f=b\ker f$. 说明 $\sigma$ 是单射. 故 $\sigma$ 是同构.

可以回忆一下高代里讲过 $\dim V = \dim \ker V + \dim \text{Im }V$. 又因为 $\dim (V/W)=\dim V - \dim W$, 结合有限维线性空间同构当且仅当维数相同, 其实就是 $V/\ker V\mapsto \text{Im }V$ 同构.

## 对应定理

设 $f$ 是群 $G$ 到 $\text{Im}f$ 的满同态，定义 $\sigma\colon K\mapsto \Set{f(k)\mid k\in K}$，则 
1. $\sigma$ 为 $\Set{H\leqslant G\mid \ker f\subseteq H}\to \Set{H\leqslant\text{Im}f}$ 的双射
2. $\sigma$ 将正规子群映射为正规子群
3. 若 $H\lhd G$ 且 $\ker f\subseteq H$，那么 $G/H\cong \text{Im} f/\sigma(H)$

这个定理讲的是通过群之间的满同态 $f$，我们可以诱导出使得子群间一一对应的双射 $\sigma_f$

为了追求双射的性质，这里对 $G$ 的子群也就是 $\sigma_f$ 的定义域作出了限制：只考虑包含 $\ker f$ 的那些子群

对应定理的一个特殊情形就是当 $f$ 为 $G$ 到其商群 $G/N$ 的自然同态时，$\ker f=N$，$\text{Im} f=G/N$，$\sigma(H)=H/N$，写出来就是 $G/H\cong (G/N)(H/N)$，当 $H=N$ 时即为群同态第一定理
