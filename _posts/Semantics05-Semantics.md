---
title: 形式语义05 Semantics
tags: Formal Semantics
date: 2021-11-07 17:14:00
---
\newcommand\norm[1]{\left\lVert#1\right\rVert}
\newcommand\abs[1]{\left\lvert#1\right\rvert}
## Formal Semantic


一般分三种


1. Operational Semantic，操作语义

2. Denotational Semantic，指称语义

3. Axiomatic Semantic，公理语义


主要记一下操作语义




## Operational Semantic


对命令式语言而言，一个程序的运行状态可以用 状态$\times$当前语句地址 二元组确定，那么就可以用 $\left<c,\sigma\right>$
描述一个执行状况，表示接下来要执行 $c$，目前的状态为 $\sigma$


状态一般指的是 $\text{Value}^\text{Var}$ 的函数，这里规定 $\text{Var}$ 表示变量的集合，$\text{Value}$
表示值（常量）的集合。注意这里的集合内都是符号，与语义无关。例如我们说$23456$是一个值，但它的具体含义是什么在这里并不关心。具体给值的符号赋予含义的操作，一般由另外的函数进行，例如说
$\text{Meaning}:\text{Value}\mapsto \mathbb N$ 就是一个给所有Integer
Tokens赋予具体整数含义的映射


操作语义一般分为大步语义（$\Downarrow$Big-Step）和小步语义（$\rightarrow$Small-Step），分别用于不同目的的研究。一般操作语义要依靠语法结构进行，即它是Syntax
Directed的。因此有时候也叫做SOS（Structural Operational Semantics）


通常讲语义会基于一个具体的语言给一些例子，建议直接找PLDI和POPL的形式语义论文看




### Evaluation


\begin{aligned}

\frac{\left<x,\sigma\right>\rightarrow\left<{\rm{n_1}
},\sigma'\right>}{\left<x+y,\sigma\right>\rightarrow\left<{\rm{n_1}
}+y,\sigma'\right>}

\\

\\

\frac{\left<y,\sigma'\right>\rightarrow \left<{\rm{n_2}
},\sigma''\right>}{\left<{\rm{n_1} }+y,\sigma'\right>\rightarrow\left<{\rm
n_1+n_2},\sigma''\right>}

\\

\\

\frac{x\in\text{Var}
}{\left<x,\sigma\right>\rightarrow\left<\sigma(x),\sigma\right>}

\\

\\

\frac{\text{Meaning}({\rm n_1})+\text{Meaning}({\rm n_2})=\text{Meaning}({\rm
n_3})}{\left<{\rm n_1+n_2},\sigma\right>\rightarrow\left<{\rm
n_3},\sigma\right>}

\end{aligned}


注意这里$\text{Meaning}$映射后的加法实际上是正整数的加法，而二元组中的$+$仅作为符号，具体含义由$\text{Meaning}$决定


以求值为例，来展示一下小步语义的特点




给出的求值语义有一些比较好的性质：


1. 可以看到上面给出的求值语义是确定的(Deterministic)，这**并不是**所有语言必备的

2. 并且我们对代码、状态二元组的推导只能通过每次寻找当前语句的结构中，最左的可推导的句子，这**也不是**所有语言必备的；


到这里就可以发现一些情况了。所谓的形式语义无非是刻画某些东西性质的一类工具，具体只能作为解决问题的帮手，而本身只起到一种规范的作用，也就是Reference
Manual了。并且这种语义是建立在数学符号上的，**可以**与具体的实现无关（当然也可以有关，但这并不是我们希望的），可以非常容易地进行推理




对于上面给出的例子，可以用写表达式求值辅助：对于给定的表达式**二叉树**（why
二叉？），我们规定必须先算左子树，再算右子树。如果遇到了变量，就替换为它在环境中的值；如果遇到了常量+常量，就把token转化为整数，计算之后再变回token。


到这里就又可以发现一些事情。操作语义几乎就是在模拟一个具体的算法、一个抽象计算机上编程语言的执行过程、一个解释执行的解释器。反过来，有了操作语义，我们也可以以此指导编译器、解释器的实现，规范具体细节。


这有什么用？用JYY的话说，我们就可以用一个**甚至还没有被实现的**语言来编写程序、模拟执行，并且预测这些程序的结果、证明它的细节了。这是多酷的一件事情！




### While


\begin{aligned}

\frac{[\![exp]\!]\sigma=\textbf{true} }{\left<{\textbf{while}
}\;exp\;\textbf{do}\;stmt,\sigma\right>\rightarrow\left<stmt\textbf;\;\textbf{while}\;exp\;\textbf{do}\;stmt,\sigma\right>}

\\

\\

\frac{[\![exp]\!]\sigma=\textbf{false} }{\left<{\textbf{while}
}\;exp\;\textbf{do}\;stmt,\sigma\right>\rightarrow\left<\textbf{skip},\sigma\right>}

\\

\\

\frac{\left<stmt_1,\sigma\right>\rightarrow
\left<stmt_1',\sigma'\right>}{\left<stmt_1\textbf;\;stmt_2,\sigma\right>\rightarrow\left<stmt_1'\textbf;\;stmt_2,\sigma'\right>}

\\

\\

\frac{}{\left<\textbf{skip;}\;stmt,\sigma\right>\rightarrow\left<stmt,\sigma\right>}

\\

\\

\frac{}{\left<\textbf{skip},\sigma\right>\rightarrow\left<\textbf{skip},\sigma\right>}

\end{aligned}


这里主要想讲 $[\![exp]\!]\sigma$ 这个符号的用法。这里表示在环境 $\sigma$
下，表达式求出的值是什么。其具体含义与操作语义的求值相同，但是注意到操作语义在执行的过程中会丢失表达式的结构信息（我们在一步步化简），因此通常需要把求值和化简分开，这里表现为用单独的函数表示求值操作，以此简化$\textbf{while}\;\ldots\;\textbf{do}$
的操作语义规则，否则我们得这么写：

\begin{aligned}

\frac{}{\left<{\textbf{while}
}\;exp\;\textbf{do}\;stmt,\sigma\right>\rightarrow\left<\textbf{if}\;exp\;\textbf{then}\;(stmt\textbf;\;\textbf{while}\;exp\;\textbf{do}\;stmt)\;\textbf{else}\;\textbf{skip},\sigma\right>}

\end{aligned}



### Evaluation Context


人总喜欢偷懒，观察到上面的操作语义其实在做很重复的事情：


1. 我们写一堆加法的rules，然后获得了加法操作

2. 再写一堆乘法的rules

3. 再写一堆减法的rules

4. 再写一堆二元运算的rules....


可以发现，既然这些都是二元运算，那么能不能给所有的二元运算定义一次求值顺序（e.g.
最左推导、确定性blabla），然后只需要根据不同情况带入具体算符就可以了？


这就是所谓的Evaluation Context了


仍然以Evaluation为例，可以规定

\begin{aligned}

{\cal E}:=[\;]{\,\Big |\,}{\cal E}+exp{\,\Big |\,}{\rm n}+{\cal E}{\,\Big
|\,}{\cal E}-exp{\,\Big |\,}{\rm n}-{\cal E}

\end{aligned}

这里我们称 $\cal E$ 为Evaluation Context，对于带入其中的项 $r$ 称为Redex


那么我们只需要如下几条rules，就足够表达出带有加减法的表达式求值语法：

\begin{aligned}

\frac{\text{Meaning}({\rm n_1})+\text{Meaning}({\rm n_2})=\text{Meaning}({\rm
n_3})}{\left<{\rm n_1+n_2},\sigma\right>\rightarrow\left<{\rm
n_3},\sigma\right>}

\\

\\

\frac{\text{Meaning}({\rm n_1})-\text{Meaning}({\rm n_2})=\text{Meaning}({\rm
n_3})}{\left<{\rm n_1-n_2},\sigma\right>\rightarrow\left<{\rm
n_3},\sigma\right>}

\\

\\

\frac{\left<s,\sigma\right>\rightarrow \left<t,\sigma\right>}{\left<{\cal
E}[s],\sigma\right>\rightarrow\left<{\cal E}[t],\sigma'\right>}

\end{aligned}
