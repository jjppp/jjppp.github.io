---
title: Futamura Projections
date: 2024-03-19
permalink: /posts/2024/03/Futamura-Projections/
tags:
  - Eureka Moments
---

看论文时发现的潮词

对于一个程序 $f$，它的输入是 $(a,b)$，输出是 $o$，我们写作 $f\colon A\times B\to O$

如果存在一个奇妙程序 $s$，它能把给定的程序 $f$ 和一部分输入 $a$ 结合在一起，得到一个新的程序 $f_a$，那么我们就把 $s$ 叫做 specializer，这里 $s\colon F\times A\to F$

从几何的角度看，这就是一个高维向量在 $a$ 处的投影，所以世人也叫 $s$ 为 projection



如果 $f$ 是一个解释器，$a$ 是一段代码，$b$ 是程序的输入，那么

# type I

$$
\forall b\in B, s(f,a)(b)\overset{\text{def}}= f_a(b)=f(a, b)
$$

利用 specializer $s$，我们就从解释器 $f$ 和代码 $a$ 得到了可执行文件 $s(f, a) = f_a$

# type II

$$
\forall a\in A, s(s, f)(a)\overset{\text{def}}= s_f(a)=s(f, a)=f_a
$$

根据 type I，$f_a$ 是可执行文件，所以这里 $s(s, f) = s_f$ 是一个只要给出代码就能产生可执行文件的——编译器

我们就从 specializer $s$ 和解释器 $ f$ 得到了编译器 $s_f$

# type III

$$
\forall f\in F,s(s, s)(f)\overset{\text{def}}= s_s(f)=s(s, f)\overset{\text{def}}=s_f
$$

根据 type II，$s_f$ 是编译器，所以这里的 $s_s$ 是一个只要给出解释器就能产生编译器的——元编译器！

我们就从两个 specializer $s$ 得到了 compiler generator $s_s$
