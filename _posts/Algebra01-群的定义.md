---
title: Algebra01 群的定义
date: 2022-07-06 12:27:46
tags: Algebra
---

\newcommand\norm[1]{\left\lVert#1\right\rVert}
\newcommand\abs[1]{\left\lvert#1\right\rvert}

觉得要好好学一学这个东西了

# 代数系统

## 定义

称 $\left<G,\circ\right>$ 为代数系统, 当且仅当 $G$ 为非空集合, $\circ\colon G\times G\mapsto G$ 为 $G$ 上封闭二元运算.



## 等价关系 划分

给出 $G$ 上的等价关系 $R\subseteq G\times G$, 可得 $G$ 的划分 $P=\Set{G_i}$. 反之给出划分亦可得由划分定义的等价关系.

记 $P=G/R$ 为 $G$ 关于 $R$ 的商集, 映射 $\pi\colon g\mapsto [g]$ 称为自然映射. 其中 $[g]=\Set{x\mid xRg}$

可构造新代数系统 $\left<G/R, \circ'\right>$. 其中 $\circ'$ 定义为 $[a]\circ' [b]=[a\circ b]$.

为了保证 $\circ'$ 是良定义的 ($\forall c\in [a], c\neq a$ 都有 $[c\circ b]= [a\circ b]$), 应满足 $\forall a_1,a_2,b_1,b_2$, $a_1Rb_1\wedge a_2 R b_2\Rightarrow (a_1\circ a_2)R(b_1\circ b_2)$. 这个特殊的关系又叫**同余关系**.



# 半群

$\left<G,\circ\right>$ 是半群当且仅当 $G$ 是代数系统, 且 $\circ$ 有结合律
$$
\forall a,b,c\in G, (ab)c=a(bc)
$$


# 幺半群

半群 $\left<G,\circ\right>$ 是幺半群当且仅当存在幺元
$$
\exists e\in G \text{ s.t. } \forall a\in G, ae=ea=a
$$

## 左右幺元

左幺元定义为
$$
\exists e_L\in G \text{ s.t. } \forall a\in G, e_La=a
$$
若左右幺元皆存在, 那么它们必然相等:
$$
e_L = e_Le_R = e_R
$$

## 幺元

$e\in G$ 为幺元当且仅当 $e$ 既是左幺元, 又是右幺元.

若幺元存在, 那么其必然唯一:
$$
e_1 = e_1 e_2= e_2
$$
直接推论: 若幺半群有多余一个左幺元, 那么其必然没有右幺元 (否则假设存在, 这个右幺元必然与某个左幺元相等. 又因为幺元唯一, 与有至少 $2$ 个左幺元矛盾)



# 群

称幺半群 $\left<G,\circ\right>$ 为群当且仅当 $\forall a\in G, a^{-1}\in G$



## 左右逆元

定义元素 $a\in G$ 的左逆元 $a_1$ 为
$$
\exists a_1\in G \text{ s.t. } a_1a=e
$$
若左右逆元都存在, 那么它们相等且为 $a$ 的 逆元
$$
a_2 = (a_1 a) a_2 = a_1 a a_2 = a_1(a a_2) = a_1
$$


## 逆元

$a\in G$ 的逆元定义为
$$
\exists a^{-1} \text{ s.t. } a^{-1}a=aa^{-1}=e
$$
类似地, 若 $G$ 中 $a$ 有至少 $2$ 个左逆, 那么其必然没有右逆.



## 消去律

右消去律定义为
$$
\forall a,b,c\in G, \; ac=bc\Rightarrow a=b
$$
由逆元的存在性可知群满足消去律



## 群方程

如下形式的方程称为群方程, $x$ 是未知元
$$
\begin{aligned}
ax=b
\\
xc=d
\end{aligned}
$$
由逆元存在性和唯一性可知群方程解存在且唯一.



# 群的等价定义

## 单侧定义

称半群 $G$ 为群, 当且仅当

1. $\exists e_L\in G$ 为左幺元
2. $\forall a\in G, \exists a^{-1}\text{ s.t. } a^{-1}a=e_L$. 即任意元素存在左幺元

称为群的单侧定义, 等价性的证明需要一些技巧. 任取 $a\in G$, $a'$ 是 $a$ 的左逆元, $a''$ 是 $a'$ 的左逆元
$$
e_L=a''a'=a''e_La'=a''(a'a)a'=(a''a')aa'=e_Laa'=aa'=e_L
$$
说明 $a'$ 亦是 $a$ 的右逆元. 这个证明的关键在于添一个幺元, 且左幺元不在最右侧时效果等同于幺元.
$$
a=ae_L=a(a'a)=(aa')a=e_La
$$
说明 $e_L$ 亦是右幺元. 

证明的另一个方向是显然的, 因此单侧定义与群的一般定义等价.



## 群方程有解定义

半群 $G$ 中任意群方程有解, 则 $G$ 为群.

任取 $a\in G$, 记方程 $ax=a$ 的一个解为 $e$, 下面证明 $e$ 为 $G$ 中一个右幺元.

任取 $b\in G$, 有 $aeb=(ae)b=ab$

此时构造群方程 $xa=b$, 记一个解为 $f$, 那么有 $fa=b$

因此 $be=fae=f(ae)=fa=b$, 由 $b$ 的任意性即得 $e$ 为右幺元.



任取 $a\in G$, 方程 $ax=e$ 的解即为 $a$ 的右逆元. 由群的单侧定义可知 $G$ 为群.



## 消去律定义

**有限**半群 $G$ 中满足左右消去律, 则 $G$ 为群

### 证明1

任取 $a\in G$, 定义 $Ga=\Set{xa\mid x\in G}$, 由右消去律可知 $|G|=|Ga|$, 即 $G$ 与 $Ga$ 存在双射.

故存在 $e\in G$ 使得 $ea=a$. 下证 $e$ 是左幺元.

任取 $c\in G$, 由左消去律可知 $|aG|=|G|$, 即存在 $d\in G$ 使得 $ad=c$, 因此 $ec=ead=ad=c$. 由 $c$ 的任意性, $e$ 是左幺元.

再证明左逆元的存在性.

类似存在 $a'\in G$ 使得 $a'a=e$. 由 $a$ 的任意性可知 $G$ 存在左逆元. 由群的单侧定义可知 $G$ 为群.



### 证明2

$G$ 有限, 因此可写成 $G=\Set{a_1,a_2\ldots a_n}$, 群方程有 $2n^2$ 个, 形如 $a_ix=a_j$, $xa_i=a_j$

由右消去律, $|G|=|Ga_i|$, 因此必然存在唯一的 $a_x$ 使得 $a_x a_i=a_j$, 此时 $a_x$ 即为方程 $xa_i=a_j$ 的解.

同理可得方程 $a_i x=a_j$ 存在唯一解

故 $G$ 中任意群方程存在解, 由群方程有解定义可知 $G$ 为群.



# 阶

元素 $a\in G$ 的阶定义为最小的自然数 $n\in\mathbb N$ 使得 $a^n=e$, 记为 $o(a)$ 或 $\abs{a}$

$\abs{a}=1$ 当且仅当 $a=e$, 并且有 $\abs{a}=\abs{a^{-1}}$



## 定理1

$a^k=e$ 当且仅当 $o(a)\mid k$.

由反证法, 设根据带余除法得 $k=q\cdot o(a)+r$, $r\neq 0, r<o(a)$, 那么有
$$
a^k=a^{q\cdot o(a)+r}=a^r=e
$$
这与 $o(a)$ 的定义矛盾, 故 $r=0$.



## 定理2

记 $(a,b)$ 为 $a, b$ 最大公约数
$$
o(a^k)=\frac{o(a)}{(k,o(a))}
$$
先证明 $o(a^k)\leq \dfrac{o(a)}{(k,o(a))}$ :

注意到
$$
\left(a^k\right)^{\frac{o(a)}{(k,o(a))}}=\left(a^{o(a)}\right)^{\frac{k}{(k,o(a))}}=e
$$
根据定理1立得 $o(a^k)\le \dfrac{o(a)}{(k,o(a))}$

再证明 $o(a^k)\geq \dfrac{o(a)}{(k,o(a))}$ :

注意到
$$
\left(a^k\right)^{o(a^k)}=a^{k\cdot o(a^k)}=e
$$
根据定理1立得 $o(a)\mid k\cdot o(a^k)$. 又因为 $k\cdot o(a^k)$ 同时是 $k$ 的倍数, 所以 $k\cdot o(a^k)$ 是 $o(a), k$ 的公倍数.

而 $\dfrac{k\cdot o(a)}{(k, o(a))}$ 是 $o(a), k$ 的**最小**公倍数, 因此 $k\cdot o(a^k)\geq \dfrac{k\cdot o(a)}{(k, o(a))}$ , 化简即得证.



因此 $o(a^k)=\dfrac{k}{(k, o(a))}$



## 定理3

若 $a,b\in G$ 可交换 (即 $ab=ba$) 且 $o(a),o(b)\in \mathbb N$, 那么 
$$
\dfrac{nm}{d^2}\mid o(ab)
\\
o(ab)\mid\dfrac{nm}{d}
$$


方便起见, 记 $m=o(a), n=o(b), d=(m, n)$

首先证明 $o(ab)\mid \dfrac{mn}{d}$ :
$$
\left(ab\right)^{\frac{mn}{d}}=\left(a^m\right)^{\frac{n}{d}}\left(b^n\right)^{\frac{m}{d}}=e
$$
再证明 $\dfrac{mn}{d^2}\mid o(ab)$. 方便起见可以分别证明 $m\mid n\cdot o(ab)$, $n\mid m\cdot o(ab)$.
$$
\begin{aligned}
\left(ab\right)^{n\cdot o(ab)}&=\left[\left(ab\right)^{o(ab)}\right]^{n}=e
\\
\left(ab\right)^{n\cdot o(ab)}&=a^{n\cdot o(ab)}\left(b^n\right)^{o(ab)}=a^{n\cdot o(ab)}=e
\end{aligned}
$$
这说明 $m\mid n\cdot o(ab)$, 即 $\frac{m}{d}\mid \frac{n}{d}\cdot o(ab)$. 注意到 $\frac{m}{d},\frac{n}{d}$ 互质, 因此 $\frac{m}{d}\mid o(ab)$

同理可得 $\frac{n}{d}\mid o(ab)$, 又因为互质, 所以 $\dfrac{nm}{d^2}\mid o(ab)$.



特殊地, 若 $(o(a),o(b))=1$, 则有 $o(ab)=o(a)o(b)$

这里一个反例是 $o(a)=o(a^{-1})=n$, 但是 $o(aa^{-1})=o(e)=1\neq lcm(o(a),o(a^{-1}))=n$