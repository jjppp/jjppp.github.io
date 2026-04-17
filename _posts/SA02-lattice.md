---
title: SA02 lattice
date: 2023-08-11 15:52:38
tags: Static Analysis

---

因为懒，所以具体的含义请读之前仔细认领一下，不对符号重载滥用造成的误解负责，但可以戳我...

# Def

## posets & bounds

给定集合 $S$ 与其上一个二元偏序关系 $\le$，这样的结构 $(S,\le)$ 称为偏序集（poset）

通常这样的偏序关系是带有信息（方向）的，即规定 $x\le y$ 表明 $x$ 比 $y$ 更精确（或 $y$ 是 $x$ 的估计）。

元素间的偏序关系可以推广到集合上，即对于 $X\subseteq S$，$X\le y$ 定义为 $\forall x\in X, x\le y$，此时称 $y$ 为集合 $X$ 的**一个**上界；关于下界的讨论是类似的。

上界可能有零个或多个。如果存在上界，那么可以讨论**最小**上界。将 $X$ 的最小上界记为 $\text{lub}(X)$，它应当满足 $\forall y\in S, X\le y\Rightarrow \text{lub}(X)\le y$。

## lattice

如果任取 $x,y\in S$ 都满足：

1. 存在 $z_1\in S$ 使得 $z_1=\text{lub}(\Set{x,y})$
2. 存在 $z_2\in S$ 使得 $z_2=\text{glb}(\Set{x,y})$

即 $\text{lub, glb}$ 是 $\Set{X\in 2^S\mid |X|=2}$ 上的全函数，那么称 $(S,\le)$ 是格（lattice），并规定

1. $x\cup y=\text{lub}(\Set{x,y})$
2. $x\cap y=\text{glb}(\Set{x,y})$

如果对于任意的 $X\subseteq S$ 都存在 $\text{lub}(X)$ 和 $\text{glb}(X)$，即 $\text{lub, glb}$ 是定义在 $2^S$ 上的全函数，那么称 $(S,\le)$ 是全格。

注意这里并没有构造地给出 join 和 meet，而只是表明了它们应有的性质

> 有限格必然是全格
>
> 任取 $x, y, z$，我们 claim $\text{lub}(\Set{x,y,z})=(x\cup y)\cup z$。
>
> 首先证明这是良定义的。注意到 $\text{lub}$ 是定义在集合上的，因此 $x,y,z$ 应当有轮换对称性。而 $(x\cup y)\cup z$ 是 $\Set{x,y,z}$ 的一个上界，因此也是 $\Set{y,z}$ 的一个上界，有 $y\cup z\le (x\cup y)\cup z$ 和 $x\le (x\cup y)\cup z$，这说明 $  x\cup( y \cup z)\le (x \cup y)\cup z$，类似可以证得 $x\cup(y\cup z)=(x\cup y)\cup z=(x\cup z)\cup y$，故这个取法是良定义的
>
> 接着证明这是最小上界。任取 $\Set{x,y,z}$ 的上界 $t$，根据定义有 $x\le t, y\le t,z\le t$，因此有 $(x\cup y)\cup z\le t$。由 $t$ 的任意性可知 $(x\cup y)\cup z$ 是三者最小上界。
>
> 然后就可以愉快地根据集合大小归纳了，$S$ 的有限性保证了归纳法的正确。

更进一步，$\text{glb, lub}$ 二者只需要其一就能导出另一个

> 若偏序集 $(S,\le)$ 上有全函数 $\cup$，那么必然也存在全函数 $\cap$
>
> 记 $L=\Set{y\in S\mid y\le X}$，取 $\bigcap X=\bigcup L$，即取出集合 $X$ 所有的下界，然后找到这些下界的最小上界，它就是 $X$ 的最大下界。
>
> 最大是显然的。下界：$\bigcap X$ 是 $X$ 所有下界的最小上界，这说明 $\forall z\in S, L\le z\Rightarrow \bigcap X\le z$。可以发现此处满足条件的 $z$ 包含了全部 $X$ 中的元素，这说明 $\bigcap X$ 是 $X$ 的一个下界。

如果是有限全格，那么必然存在全局最大和最小值，通常记为 $\top, \bot$。最长的形如 $\bot\le\cdots\le\top$ 的链的长度称为格的高度。

# Common lattices

鉴于这本书实际上想讲的是静态分析，所以通常只关心有限格，而通常它们又都是全格。可以开动脑筋想想介绍这些特殊格的动机。

任给集合 $S$ 都自动构成一个格 $(2^S,\subseteq)$。

定义 $flat(S)=(S\cup\Set{\bot,\top}, (\Set{\bot}\times S)\cup (S\times\Set{\top}))$，即只有 $\top, \bot$ 可以比较的格。

给定若干格 $\Set{(L_n,\le_n)}$，那么 $(\prod_{i=1}^n L_i,\le')$ 也是格，其中 $(\ldots x_i\ldots)\le'(\ldots y_i\ldots)$ 定义为 $\forall i, x_i\le_i y_i$。这个叫 product lattice

给定集合 $D,S$ 和函数集 $F= S^D$，若 $(S,\le)$ 是全格，那么 $(F, \le')$ 也是全格，此处 $f\le' g$ 定义为 $\forall x\in D, f(x)\le g(x)$。这个叫 map lattice

给定全格 $(S,\le)$，那么 $(S\cup\Set{\bot}, \le')$ 也是全格，此处 $\le'$ 的定义就是你想的那样。这个叫 lift lattice

# Properties

下面默认 $f\colon S\mapsto T$，$(S,\le)$ 和 $(T,\le')$ 都是全格

## Distributive

若 $\forall x, y \in S$，$f(x\cup y)=f(x)\cup f(y)$，那么称 $f$ 可分配。

## Extensive

任取 $x\in S$，$x\le f(x)$。

注意和单调性作区分。

## Monotone

若 $\forall x, y\in S, x\le y\Rightarrow f(x)\le f(y)$，则称 $f$ 是单调的，含义为给 $f$ 更精确的输入不会让结果更不精确。

单调的定义可以拓展到多参输入，这里只需要对单个参数单调即可。

1. > 单调函数在复合运算下封闭
   >
   > 这个是显然的。

2. > 常函数单调
   >
   > 这个也是显然的。

3. > 集合差操作 $\backslash$ 非单调
   >
   > 这个也是显然的。

   话说这个能不能叫协单调/逆单调（认真脸

4. > $\cap, \cup$ 都单调
   >
   > 固定 $x, y$，任取 $z$
   >
   > 1. 假设 $x\le z$，那么 $x\le z\le z\cup y, y\le z\cup y$ ，故 $z\cup y$ 是 $\Set{x, y}$ 的上界，由 $x\cup y$ 的定义即得 $x\cup y\le z\cup y$
   > 2. 由对称性直接得到
   >
   > $\cap$ 的讨论是类似的。

5. > 可分配函数单调
   >
   > 若 $x\le y$，则 $x\cup y=y$
   >
   > 带入定义有 $f(y)=f(x\cup y)=f(x)\cup f(y)$，而 $f(x)\le f(x)\cup f(y)=f(x\cup y)=f(y)$

6. > $f$ 单调的一个充要条件是 $\forall x,y\in S$，$f(x)\cup f(y)\le f(x\cup y)$
   >
   > $\Rightarrow$：
   >
   > 显然有 $x\le x\cup y, y\le x\cup y$，由单调性得 $f(x)\le f(x\cup y), f(y)\le f(x\cup y)$
   >
   > 又 $f(x)\cup f(y)$ 是 $\Set{f(x), f(y)}$ 的最小上界，因此 $f(x)\cup f(y)\le f(x\cup y)$
   >
   > $\Leftarrow$：
   >
   > 不妨设 $x\le y$，那么 $x\cup y = y$，带入有 $f(x)\cup f(y)\le f(x\cup y)=f(y)$
   >
   > 这说明 $f(y)$ 是 $\Set{f(x), f(y)}$ 的上界，根据定义 $f(x)\le f(y)$。

7. > 给定单调函数 $f\colon L_1\mapsto(A\mapsto L_2)$，$g\colon A\mapsto L_2$，其中 $L_1, L_2$ 都是全格，$A$ 是定义域，$A\mapsto L_2$ 是 map lattice。对于给定的 $a\in A$，证明 $h(x)=f(x)[a\mapsto g(x)]$ 单调
   >
   > 不妨假设 $x\le y$，我们需要证明 $f(x)[a\mapsto g(x)]\le f(y)[a\mapsto g(y)]$，而它的定义则是 $\forall w\in A, f(x)[a\mapsto g(x)](w)\le f(y)[a\mapsto g(y)](w)$
   >
   > 1. 如果 $a=w$，那么我们只需要证明 $g(x)\le g(y)$，而 $g$ 单调；
   > 2. 如果 $a\ne w$，那么我们只需要证明 $f(x)(w)\le f(y)(w)$，而 $f$ 单调；
   >
   > 综上，$h$ 单调。

   这个的证明像在写 coq

# Fixed point

抽象解释本质上是在构造一组抽象映射 $\sigma$ 的方程，最后我们希望能求出一个在精确意义上最优的解。

即对于一系列方程
$$
\left\{
\begin{align*}
\sigma_1&=f_1(\sigma) \\
&\cdots \\
\sigma_n&=f_n(\sigma) \\
\end{align*}
\right.
$$
其中
$$
\sigma=(\ldots \sigma_i \ldots)
$$
$\sigma_i$ 表示在 program point $i$ 处的抽象映射。把 $\Set{f_i}$ 视为一整个大函数 $F$，则上面的方程组实质上是
$$
\sigma=F(\sigma)
$$
即我们要求 $F$ 的不动点。解可能有多个，最好是精确意义上最优的解。

## Iteration

喜闻乐见的迭代，考虑 $\bigcup_{i>0} f^i(\bot)$，收敛性、正确性和解的最优性质都是比较简单的。解方程的部分还有一些和线性规划类似的把不等号改写成等号的技巧。

实际的 solver 写起来有各种各样的技巧，比如利用单调性避免重复计算/根据CFG的结构选择计算顺序/SCC缩点后按照拓扑序计算等等
