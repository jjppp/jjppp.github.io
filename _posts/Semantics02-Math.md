---
title: 形式语义02 Math
tags: Formal Semantics
date: 2021-09-09 21:44:00
---
\newcommand\norm[1]{\left\lVert#1\right\rVert}
\newcommand\abs[1]{\left\lvert#1\right\rvert}
## Basic Set Theory


没啥好讲的


$\bigcap S=\left\{\;x\;|\; \forall T\in S,x\in T \;\right\}$


记 $R=\bigcap\emptyset$，则 $\forall x. \forall T\in\emptyset\wedge x\in
T\rightarrow x\in R$


注意到命题的前件为假，因此 $R$ 是"a set of everything"


根据幂集公理这是不成立的，因此我们规定它无意义。




## Relations


没啥好讲的




## Functions


$f\subseteq D(f)\times R(f)$，且 $\forall x,y,y'$，$(x,y)\in f\wedge (x,y')\in
f\Rightarrow y=y'$




### Total Functions $f$ on $A\times B$


$A=D(f)$




### Partial Functions $f$ on $A\times B$


$D(f)\subseteq A$




### $\lambda$-Expression


$\lambda x\in D. f(x)$ 和 $\left\{\;(x,f(x))\;|\;x\in D\;\right\}$ 是等价的




### Function Variant


我们用 $f\left\{x\leadsto y\right\}$ 记号表示 $f\cup (x,y)$ 这个新的函数，此处的 $x\in D(f)$
不一定成立


显然 $D(f\left\{x\leadsto y\right\})=D(f)\cup\left\{\;x\;\right\}$




### Function Type


我们用 $A\rightarrow B$ 表示 $A\mapsto B$ 的所有函数，且规定 $\rightarrow$ 右结合


即 $A\rightarrow B\rightarrow C=A\rightarrow (B\rightarrow C)$ ，表明这是一个 $A\times
B\mapsto C$ 的函数，或者 $A\mapsto (B\mapsto C)$ 的函数


这里给出的 Function Type 默认是对应集合的 Total Function.




很显然 $A_1\mapsto A_2\mapsto \cdots\mapsto A_n\mapsto R$ 的表达能力要强于 $A_1\times
A_2\times\cdots\times A_n\mapsto R$，即我们可以先后给出参数


表达能力的弱化无需额外做什么，而强化则需要currying操作




### Tuples as Functions


二元组 $(x,y),\, x,y\in D$ 可以看成是 $\left\{\;0,1\;\right\}\mapsto D$ 的**一个**函数，即
$f(0)=x,f(1)=y$，$D\times D$ 就可以看成是 $\left\{\;0,1\;\right\}\mapsto D$ 的**所有**函数


形式化地写出来就是 $A\times B=\left\{\;f\;|\;D(f)=\left\{\;0,1\;\right\}\wedge f(0)\in
A\wedge f(1)\in B\;\right\}$




于是自然，$A_0\times A_1\times\cdots\times A_{n-1}$ 就可以看成是 $\left\{\;0,1,2\ldots
n-1\;\right\}\mapsto D$ 的所有函数，其中 $D=\bigcup\limits_{i=0}^{n-1} A_i$


写出来就是 $\prod\limits_{i=0}^{n-1} A_i=\left\{\;f\;|\;D(f)=[0..n-1]\wedge\forall
i\in[0..n-1].f(i)\in A_i\;\right\}$


再进一步就是 $\prod\limits_{i\in I}S_i=\left\{\;f\;|\;D(f)=I\wedge\forall i\in
I.f(i)\in S_i\;\right\}$




### Product of Functions


现在，我们令 $\theta: \alpha\mapsto \left\{\;S_\alpha\;\right\}$，即是以 $\alpha$
为指标集的一个集合族，$\theta$ 将一个下标映射到集合族里对应下标的某个集合


我们同样可以定义 $\theta$ 上的乘积(product)


即 $\sqcap\theta=\left\{\;f\;|\;D(f)=\alpha\wedge \forall
x\in\alpha.f(x)\in\theta(x)\;\right\}$，其中 $\alpha=D(\theta),\;\theta(x)=S_x$




当 $\theta$ 是常函数时，$\forall x. \theta(x)=S$，那么
$\sqcup\theta=\left\{\;f\;|\;D(f)=\alpha\wedge\forall x\in\alpha. f(x)\in
S\;\right\}=\prod\limits_{i\in\alpha}S=S^\alpha=\left\{\;f\;|\;f:\alpha\mapsto
S\;\right\}$


大概的意思是说，我们有一个定义域 $\alpha$ 到若干集合的映射 $\theta$，现在把每个 $\alpha$ 中的元素 $x$ 的像限制到
$\theta(x)$ 里的一个具体元素，就可以得到一个具体的 $D(\theta)\rightarrow \bigcup R(\theta)$
函数。如果我们取遍所有可能的像，那么就恰好遍历完了所有这样的函数。




具体到程序设计语言，就是若干个 Type 经过这样的操作，就可以得到一个 Product Type. 即我们把一个 Type
看成是一个**单点函数**的集合，那么就可以通过简单的 Product 操作复合得到新的 Type.




### Sum of Functions


对于任意集合 $A,B$，规定


$A+B=\left\{\;0\;\right\}\times A\cup\left\{\;1\;\right\}\times B$


称 $A+B$ 为 $A,B$ 的不交并。




自然就有 $\sum\limits_{i\in \alpha} S(i)=\left\{\;(i,x)\;|\;i\in\alpha\wedge x\in
S(i)\;\right\}$


同样的，如果我们把 $\theta$ 看作是 $\alpha\rightarrow
\bigcup\limits_{i\in\alpha}S(i)$，那么就有
$\Sigma\theta=\left\{\;(i,x)\;|\;i\in\alpha\wedge x\in S(i)\;\right\}$




当 $\theta$ 是常函数 $\theta(i)=S$
的时候，$\Sigma\theta=\sum\limits_{x\in\alpha}S=\left\{\;(x,y)\;|\;x\in\alpha\wedge
y\in S\;\right\}=\alpha\times S=\alpha\rightarrow S$




这个的意思是，我们可以通过若干 Type 得到 Sum Type，新的 Sum Type 中的变量只能是组成它的若干 Type 中的某一个。




### 感想


写着写着就突然悟了，注意到一个编程语言中的类型可以看成是值域 $R$ 构成的集合


而带**类型**的变量可以看成是单点函数的集合 $\left\{\;f\;|\;f:\left\{\;var\;\right\}\rightarrow
R\;\right\}$，变量的一个具体**取值**则对应一个具体的单点函数。需要说明的是，我们这里不妨规定变量两两不重名(否则可以很容易用自然数命名而不改变程序的行为)




那么上面说的 Sum 和 Product 实际上就是通过简单类型构造出复杂类型的两类方法


而我们知道，一个程序的所有状态由所有变量的取值组成，那么一个程序的特定状态就可以表达成一个具体的函数 $P:\left\{\;\;|\;x\text{ is
variable}\;\right\}$
