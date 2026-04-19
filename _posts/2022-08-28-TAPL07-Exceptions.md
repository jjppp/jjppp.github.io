---
title: TAPL07 Exceptions
date: 2022-08-28
permalink: /posts/2022/08/TAPL07-Exceptions/
tags:
  - TAPL
---

# Intro

异常处理一直是老大难问题，大概有这么几个流派

1. 用返回值区分，Rust 的 `Result`、Haskell 的 `Either a b`、C 的 `errno` 都是这类（其实我还想说 go 的）。
2. 用异常控制流（本质上是non-local jumps），例如 Java 的 `try ... catch ... finally`、Python 类似、C++ 也能做。
3. 遇到异常直接终止程序，例如防御性的 `assert()`、`panic()` 等等

①和②往往针对可恢复的异常。书上大概讲的是②和③这种方法。



# Exceptions

## Naive raise

加入一个新的项 $\text{error}$，规定
$$
\begin{aligned}
\text{error}\; t\rightarrow \text{error}\\
v\;\text{error}\rightarrow \text{error}
\end{aligned}
$$
这表明 $\text{error}$ 具有“传染性”，当一处异常出现时，后续的求值将使得这个异常扩散到整个程序（最终 abort）。可以发现这两条规则保持了执行的顺序，即一个异常不会过早被扔出来，也不会过晚才出现（以免执行了后续有副作用的部分）

同时 $\text{error}$ 本身并不是值，这表明 $(\lambda x:\text{Top}.\;x)\;\text{error}$ 将不会执行 apply，而是直接变成 $\text{error}$。否则将引入二义性的问题。



## Naive raise Typing

有几个技巧

1. $\text{error}$ 可以具有类型 $\text{Bot}$，这样在有子类型的类型系统中就可以被**提升**为任意其它类型
2. $\text{error}$ 可以具有类型 $\forall X.X$，这样就可以被**实例化**为任意其它类型
3. 或者留着不动，在做 typecheck 的时候赋予 $\text{error}$ 我们需要的类型。但这样就违背了 unique type 的性质

同时 preservation TH 也需要变换表述，即最终总会得到一个值**或 $\text{error}$**



## Catch

就是设计这样的项 $\text{try }t_1\text{ with }t_2$，规定
$$
\begin{aligned}
\text{try error with } t_2&\rightarrow t_2 \\
\text{try $v$ with $t_2$} &\rightarrow v
\end{aligned}
$$
即如果异常了就执行后面的代码擦屁股，否则就正常返回。你也可以当成是套了一个 meta-level 的 wrapper

这玩意的 typing 要求 $t_1, t_2$ 有相同的类型（回想 $\text{if-then-else}$）。



## Raise with value

最经典的就是 C++ 里的 `e.what()`，即我们可以给异常带上一个值用于区分这是什么异常，同时在 handler 中利用这个携带值做一些事情。

引入
$$
\begin{aligned}
&\text{raise $t$}\\
&\text{try $t_1$ with $t_2$}
\end{aligned}
$$
其中 $t_2$ 是一个类型为 $T'\mapsto T$ 函数，$T,T'$ 分别为 $t_1$ 和 $\text{raise}$ 中信息的类型。

书上规定 $\text{raise $t$}$ 仍然具有任意的类型。这里一个小问题就是我们没法通过 structural 的方法静态地知道一个异常内信息的类型了（否则等价于用 `Either a` Monad 做返回值）。

当然书上给出的 argue 说的也是针对不可恢复的错误，我们是不需要关注其可能抛出的异常的（毕竟最终会支配整个程序）。但是对于可恢复的程序还是有必要的，例如 Java 就偏向于明确写出 `public foo() throws BarException` 这样的东西。

此外，用一些别的小技巧还是可以静态分析出一段代码可能会扔什么异常的。

另一个可以设计的地方在于内部信息的选择

1. 用数字，这就是 C 的做法
2. 用 `String`，那就是最简单的 `e.what()`
3. 用一个 Sum Type（或者像 Java 那样用类），这样就可以 match 不同的情况做不同的处理
