---
title: TAPL05 Extensions
date: 2022-08-09 11:56:13
tags: TAPL
---



# Intro

这一章主要是给 STLC 加上各种常用的 construct，也就是在做语法糖化的过程。比较轻松



# Derived Form

考虑语言 $\lambda$ 及其拓展 $\lambda^E$。若存在一个语法上的翻译函数 $t\colon \lambda^E\mapsto \lambda$ 使得
$$
\forall m,m'\in \lambda^E
\\
\begin{aligned}
& m\rightarrow m'\iff t(m)\rightarrow^E t(m')
\\
& \Gamma\vdash m\colon \tau \iff \Gamma \vdash^E t(m)\colon\tau
\end{aligned}
$$
即如果某个语法和语义上的扩展，可以用纯语法替换的方式等价得到，那么就称这个扩展是“语法糖”(syntatic sugar)。此处等价特指 reduction relation 和 typing relation 保持。

当然这里语法糖的要求是单步推导等价，这个条件还是比较强的。



这里的一个好处是可以在不修改语言 core 的情况下添加各种 feature。一个类似的例子是微内核+形式化验证的套路，把用户态组件看成是语法糖就行了。



# Base Types

一般的语言会提供若干基础类型用于表示一些基本的值，在此之上可以用一些类型构造子构建更复杂的类型以表示复杂的数据。

经典的比如 `int, float, char` 之类。

通常不关心基础类型内部的操作，因为此处主要讲的是类型系统的构建，因此可以把这些基础类型视为“原子的”，即它们只能向上构建新类型。



# `Unit` Type

也是常说的 trivial type。这个类型所含值的集合为单元集(singleton set)。

扩展是通过加入一个特殊的值 `unit`，并规定其类型为 `Unit` 来实现的。



# `Bot` Type

为了方便，通常规定 `Bot` 是所有类型的子类型，用于表示某些异常项（例如 exception），同时保证带有 error handling 的项可以通过类型检查。

这个类型所含的值的集合为空集：

由反证法，设存在 $\Gamma\vdash v\colon \text{Bot}$，那么根据子类型同时有 $\Gamma\vdash v\colon \text{Top$\rightarrow$Top}$ 和 $\Gamma\vdash v\colon \{\}$。而 $v$ 不可能既是函数又是 variant，因此矛盾。



# Subtyping

称一个类型 $A$ 是 $B$ 的子类型，当且仅当类型为 $A$ 的项可以无痛替换掉类型为 $B$ 的项，程序仍然可以正常执行。记作 $A<:B$

一个例子是 $A=\{x:\text{Nat},y:\text{Bool}\}$，它是 $B=\{x:\text{Nat}\}$ 的子类型，因为所有用到类型 $B$ 的项至多访问 field $x$。可以发现子类型的定义是完全根据喜好来的，通常可以根据需求规定不同的子类型关系。

例如我们的 variant type，就可以通过规定所有 field 可以交换、加上若干 field 是子类型、field 的类型是子类型仍是子类型 这三条来得到一些比较方便的性质。

可以发现 `Unit` 天然的就是 `Top`，并且我们很难通过构造的方法给出一个 `Bot` 类型，这里的 `Bot` 有点 limit order $\omega$ 的意味在里面了。



# Sequential

即 `s1; s2` 这样的写法，用于保证求值顺序。书上提到的 desugar 成 ($\lambda x\colon \text{Unit}.\; s_2)\; s_1$ 能够等价的前提是推导是 Call-by-value 的



# Wildcard

即 $\lambda \_.\; m$ 这样的写法，表示我们并不会真的用到这个 binding。一个具体的例子就是 `s1; s2` 的 derived form

desugar 之后就是选取 $FV(m)$ 中的 fresh 做 binding。



# Ascription

也就是说的程序员自己写的类型注解，理解成 term-type assertion 也没啥问题

ascription 本身也可以作为 derived form 添加进去，即 $(\lambda x\colon \tau.\; x)\; t\equiv t\text{ as } \tau$

具体定义去看书



# `let` Binding

即可以写出这样的代码(以 haskell 为例)

```haskell
dist:: Float -> Float -> Float
dist x y = 
	let sqrX = x * x
		sqrY = y * y
		in sqrt $ sqrX + sqrY
```

这同样是一个 derived form，但是区别在于在 derive 的时候需要有一个 type checker 来获得 binding 的类型。

因为涉及到 typing relation 的修改，所以这个语法糖和前面的比起来要更不 trivial 一点。但是执行过程是没有区别的。

具体定义去看书



# Record

也叫 Product Type，一个例子就是 C 里面的 `struct`

field 是数字时就退化为 tuple，长度为 2 则退化成 pair。

书上还讨论了顺序的问题，这里目前定义出来的都是要求顺序一致的。后续解决顺序可以用 subtyping 来做，咕咕咕咕。



# Pattern matching

这里的 pattern matching 仅能用作 `getter`

具体而言是通过定义语法上的 pattern，然后把 pattern matching 的行为定义为若干单变量替换的复合

具体去看书。



# Variant

也叫 Sum Type，一个例子就是 C 里面的 `union`

所有构造子都只吃掉一个 $\text{Unit}$ 的时候退化成 Enum。只有两种的时候退化成 Either，其中一个是 $\text{Unit}$ 的时候退化成 Optional

因为类型中的值是有限的，所以就可以 `case value of` 来进行操作的分派了。

一个比较有意思的地方在于 Either 的出现会破坏类型的唯一性，即我们很难静态地确定 $\text{Left 42}$ 到底是什么类型。最简单的解决方法就是要求程序猿显式加上注解。



# Dynamic

即给项加上类型的标签，使得类型在动态运行时仍然附着在项上。同时提供一些动态对于类型的 `case` construct 来支持操作的分派。



# Recursion

由于 STLC 中没法给 Y-Combinator 定类型，因此需要把这个东西提升到元语言的层级，作为一个 primitive。

好玩的地方在于 `fix` 并没有规定只能求一个函数的不动点，因此可以把很多函数用 Record 打包，实现类似常系数线性齐幂次递推的互相调用的递归函数。树上给的例子是关于 isEven 和 isOdd 的。



# List

重要的地方在于给出了一个不同于 $\rightarrow$ 的类型构造子 $\text{List}$，剩下的就没啥了。



# Exception

