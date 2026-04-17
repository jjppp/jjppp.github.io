---
title: 密码学06 HASH
tags: Cryptography
date: 2021-12-21 15:06:00
---
\newcommand\norm[1]{\left\lVert#1\right\rVert}
\newcommand\abs[1]{\left\lvert#1\right\rvert}

## Cryptographic Hash Function


所谓Hash，就是一个实现“压缩”比特序列的函数（有时也叫摘要）。与普通的散列和哈希不同，密码学上的hash要求要更严格一些：

1. 要是抗碰撞(collision resistant)的

2. 最好还有PRF的性质


类似PRF，Hash也是一个keyed function，只不过更多的时候 $k$ 是由某种方法选出来的固化公开常量，而非随机选出来的。


## 定义

Hash $\Pi=(\text{Gen,H})$ 是一对PPT算法，满足：

1. $k=\text{Gen}(n)$ 是一个**公开**的参数

2. $H(k,x)=H^k(x)\in\left\{0,1\right\}^{l(n)}$，其中 $l(n)$ 是expansion factor


如果有 $l(n)<n$，那么我们就称 $H$ 是一个压缩映射(compression function)


### Collision resistant


首先要定义一个game $\text{Hash-coll}_{\cal A,\Pi}(n)$，这个game很直白，就直接说了：

我们公开Hash的参数 $s$，然后让Adversary输出一对比特串 $(x_1,x_2)$。若有
$H^s(x_1)=H^s(x_2)$，我们就说Adversary找到了一个碰撞(collision)


称一个Hash scheme是c.r.的，当且仅当有 $Pr\left[\text{Hash-coll}_{\cal
A,\Pi}(n)=1\right]\leqslant\text{negl}(n)$ 对任意的PPT Adversary成立


此外还有一些更弱的对Hash的要求


### Second preimage resistant



如果给定了参数 $s$ 和一个比特串 $x$，有任意的PPT Adversary都不能找到 $x'\neq x$ 使得
$H^s(x)=H^s(x')$，就称这个Hash scheme是2nd p.r.的


注意到如果Adversary没法对Hash找到任意一对碰撞（满足c.r.），那么它一定无法对一个给定的比特串找到碰撞（满足2nd p.r.），因此有
c.r. $\Rightarrow$ 2nd p.r.


### Preimage resistant


如果给定了参数 $s$ 和一个比特串 $y$，有任意的PPT Adversary都不能找到 $x$ 使得 $H^s(x)=y$，就称这个Hash
scheme是p.r.的


注意到如果Adversary可以对给定的 $y$ 找到原象 $x$（not p.r.），那么它就可以通过找出 $y$
的所有原象，从中挑一对出来就能找到一对碰撞了（not c.r.）。这是因为在Hash是压缩映射的时候，至少有一半的元素有两个原象


## Merkel-Damgard Transform


其实很套路了。假如我们有一个定长的c.r.压缩映射，要怎么才能做到构造出一个任意长的压缩映射？


不失一般性，假设我们已有的小映射是 $\left\{0,1\right\}^{a+b}\mapsto\left\{0,1\right\}^{a}$



有一个很直观的图，大概是长这样的

**插入图片**


大概分成三步走：

1. 先做补全（padding），得到长度为 $b$ 的整数倍的一个比特串

2. 按照每 $b$ 位切成一块，每一块记作 $B_i$，最后额外加入一块 $L$，表示**原始比特串**的长度

3. 确定一个 $z_0=IV$，有 $z_i=H(z_{i-1}|| B_i)$，最终的那一块 $z_{last}$ 就是整个Hash的输出


c.r.的证明可以通过反证法和归纳来得到，同样是如果能构造出大映射的碰撞，那么一定是构造出了一个小映射的碰撞，因此就矛盾了。


## 和MAC的应用


回想前一篇MAC的内容，我们没有一个很好的办法能构造任意长度的MAC。但是现在有了任意长度到固定长度的c.r.压缩映射，我们就可以用这个c.r.
Hash和secure fixed-length MAC一起来构造一个任意长度的MAC了


只需要规定 $\widehat {\text{Mac} }_k(x)=\text{Mac}_k(H^s(x))$ 即可，$\text{Vrfy}$
的部分只需要用canonical方法就好了


安全性的证明可以通过反证法说明：我们下面要证明这是一个secure arbitrary-length MAC

假设 $\cal A$ 构造出了一个新的tag $(m',t')$，那么有如下可能：

1. 存在一个 $m''$，使得 $\cal A$ 查询了 $m''$，并且有 $H^s(m')=H^s(m'')$。显然这与c.r.矛盾

2. 所有 $\cal A$ 查询过的消息在Hash之后都不与 $m'$ 构成一对碰撞，此时意味着 $\cal A$
构造出了一个全新的tag，这又与secure-MAC矛盾了


## Hash攻击

这部分暂且咕咕咕
