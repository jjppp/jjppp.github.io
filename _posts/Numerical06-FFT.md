---
title: 计算方法06 FFT
tags: Numerical Method
date: 2022-06-14 14:49:00
excerpt: 据助教说不考这么麻烦的计算，我就摸了
---
\newcommand\norm[1]{\left\lVert#1\right\rVert}
\newcommand\abs[1]{\left\lvert#1\right\rvert}


# 多项式的表示


最熟悉的是系数表示法，例如 $P(x)=\sum_{i=0}^n a_i x^i$


系数表示法有它的好处：例如将 $\Set{1,x,x^2\ldots}$ 视为一组基时，系数恰好就是多项式在这组基下的坐标；例如对于任给的点 $x$
可以计算多项式的值（$O(n)$ 次乘法）。但是在给出系数表示法后，对两个 $n$ 次多项式很难高效算乘积（还记得天才本科生Karatsuba吗）




还可以是点值表示法。根据lagrange插值可知，一个 $n$ 次多项式可以由 $n+1$ 个点唯一确定。因此
$P(x)\overset{\text{def} }= \Set{(x_0,y_0),(x_1,y_1)\ldots}$


此时计算 $P(x)Q(x)$ 只需要求出 $2n+1$ 个点值，然后将这些取值对应相乘即可得到 $R(x)=P(x)Q(x)$ 的点值，也就唯一确定了
$R(x)$




因此我们希望有一种方法，对于给定的系数表示多项式 $F(x),G(x)$，能够选取 $2n+1$ 个点


对这 $2n+1$ 个点做插值后得到 $F^*(x),G^*(x)$ 的点值表示，相乘最后还原得到 $F(x)G(x)$ 的系数表示。


这就是FFT




# FFT


## DFT


不妨假设多项式是 $n=2^k$ 次的


考虑方程 $x^n=1$ 的所有复数解，它们构成了复平面单位圆上等距分布的 $n$ 个点，记为 $\Set{\omega_i\mid
\omega_i^n=1}$


选择这些点出自如下考虑：取 $\Omega_n$，那么有 ${\Omega_{2n} }^2\overset{\text{def}
}=\Set{x^2\mid x\in \Omega_{2n} }=\Omega_n$。这暗示我们可以通过类似分治的算法来每次将问题的规模减小一半。并且对于
$\Omega_n$ 而言，只需要其生成元 $\omega_1=e^{\frac{2\pi}{n+1}i}$ 即可完全表示。


假设需要对 $F(x)=\sum_{i=0}^{2n} a_ix^i$ 插值，那么记 $G(x)=\sum_{i=0}^{n}
a_{2i}x^{i}$，$H(x)=\sum_{i=0}^n a_{2i+1}x^{i}$，显然有


1. $F(x)=G(x^2)+xH(x^2)$

2. 原本要对 $F(x)$ 在 $\Omega_{2n}$ 上插值，现在要对 $G(x),H(x)$ 在 $\Omega_n$ 上插值


主定理算一下就是 $O(n\log n)$ 的。


## IDFT


考虑如下矩阵形式的DFT

\begin{aligned}

\begin{bmatrix}

\omega^0 & \omega^0 & \cdots & \omega^0

\\

\omega^0 & \omega^1 & \cdots & \omega^n

\\

\vdots   & \vdots   & \ddots & \vdots

\\

\omega^0 & \omega^n & \cdots & \omega^{n^2}

\end{bmatrix}

\begin{bmatrix}

a_0 \\ a_1 \\ \vdots \\ a_n

\end{bmatrix}

=

\begin{bmatrix}

F(\omega^0) \\ F(\omega^1) \\ \vdots \\ F(\omega^n)

\end{bmatrix}

\end{aligned}

即我们的 $\Omega_n$ 是一组基，$\Set{a_n}$ 则是基下的坐标，作用一下就能得到多项式在 $\Omega_n$ 上的点值。


这个矩阵 $W$ 非常经典，算一下行列式和逆就会发现对于IDFT的操作只需要取 $\omega_1'=-\omega_1$
做DFT即可实现逆变换，最后需要除掉一个系数 $\frac{1}{n+1}$
