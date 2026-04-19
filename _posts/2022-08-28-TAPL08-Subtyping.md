---
title: TAPL08 Subtyping
date: 2022-08-28
permalink: /posts/2022/08/TAPL08-Subtyping/
tags:
  - TAPL
---

# Intro

Subtyping（或者说 Subtype Polymorphism）给出了一种类型间的转换关系（一种 preorder），使得程序猿可以写出很通用的代码。典型的子类型出现在各种 OO 语言里，例如 Java 的子类。

同时需要指出的是，子类型和继承并非必须捆绑在一起

* 子类型说的是具有一种类型的项可以无痛替换掉另一种类型的项
* 继承强调的是代码复用

Java 把这俩放在一起了，即创造子类型的同时天然地复用了父类型的代码。

# Subtyping

称 $A$ 是 $B$ 的子类型，记为 $A<: B$。一个简单的基于朴素集合论的解释就是：类型为 $A$ 的项构成的集合是类型为 $B$ 的项构成的集合的子集。

这说明，若 $\Gamma\vdash a:A$，那么 $\Gamma\vdash a:B$。即
$$
\frac{\Gamma \vdash a:A,\qquad A<:B}{\Gamma\vdash a:B}
$$

## Subtype relation

子类型关系是类型上的一个二元关系，由若干推导规则给出。

### Baiscs

通常子类型关系是任意的（对于不同的语言而言可以不同），因此存在可以设计的地方。但通常都包括下面两条（即子类型关系至少应该是一个 preorder）
$$
\begin{aligned}
\frac{}{A<:A}
\end{aligned}
$$

$$
\begin{aligned}
\frac{A<:B\qquad B<:C}{A<:C}
\end{aligned}
$$

### Records

书上花了很长的篇幅讲两个 record 什么情况下是子类型，其实就是看 field 的包含情况就可以了。

### Functions

这个就比较好玩
$$
\frac{S_1:> S_2\qquad T_1<:T_2}{S_1\rightarrow T_1<:S_2\rightarrow T_2}
$$

* 函数的子类型与函数参数的子类型关系是**反的**，这个叫逆变（contravariant）
* 函数的子类型与函数返回值的子类型关系是**相同的**，这个叫协变（covariant）

### Special types

定义 $\text{Bot, Top}$ 为两个特殊类型，其中 $\forall S. S<:\text{Top}$，$\text{Bot}$ 类似。

### Variant

对于 Sum Type 而言，子类型多态的引入可以省去一个 ascription，因为类型可以自动提升。

### List

$$
\frac{A<:B}{\text{List $A<:$ List $B$}}
$$

### References

$$
\frac{A<:B\qquad B<:A}{\text{ref $A<:$ ref $B$}}
$$

之所以有这个要求，是因为一个引用既可以存入、也可以取出。

我们只能往 $\text{ref $B$}$ 中存入 $B$ 的子类型，也只能取出 $B$ 的子类型，因此 $\text{ref $A$}$ 能替代 $\text{ref }B$ 的场合要求 $A,B$ 互为子类型。

也可以把读和写分离，做成类似管道的东西，那么这时候就可以在两侧分别赋予不同的类型了（逆变/协变）。

### Array

数组本质上也是一种引用（引用是长度为 1 的数组），因此很容易写出
$$
\frac{A<:B\qquad B<:A}{\text{Array $A<:$ Array $B$}}
$$
书上也提到了 Java 的设计缺陷，看一乐

## Casting

一般分成 upCast 和 downCast

* upCast 就是之前提到的 ascription，即通过 subsumption 提升为父类型，这个没什么问题

* downCast 就是 Java 里一般会遇到的 cast，即把某个父类型的项强转成某个子类型。最经典的就是 Java 在还没有泛型的时候需要这样实现 List：

  ```java
  ListOfObjects list = new ListOfObjects;
  Dog dog = new Dog();
  list.add(dog);
  Dog objDog = (Dog) list.get(0);
  ```

  虽然即使有了本质上也还是这么实现的...

  或者 C 里面的 `void *`

  当然，不正确的 downCast 会丧失 safety，因此还需要做动态的检查。也有些小技巧可以通过静态的指针分析来得到一些 sound 解，使得我们可以省去某些动态的检查。

# Coercion Semantics

这里花了一点时间才看明白在讲什么

上面讲的其实是 high-level 的 subtyping 语义，即我们真正做的实际上是**把父子类型之并视为父类型**。在操作这些值的过程中它们的内部表示仍然是各自具体类型的，这就可能涉及到装箱和拆箱的代价。这就是所谓的 subset semantics

而 Coercion Semantics 给了另一种方法，即我们可以把 high-level 的语言翻译到 low-level 且没有 subtyping 的语言。在这个过程中，我们通过添加若干 low-level 的 cast 函数来**将子类型改写成父类型**，这样在类型提升后，我们项的内部表示也得到了改变。这就是 coercion semantics

> 举个例子：
>
> struct A `{x: Nat}`
>
> struct B `{y: Bool -> Bool, x: Nat}`
>
> 很显然 $B<:A$。不妨记 `f = fun x: Nat (x + 1)`，立即有 $f:\text{Nat$\rightarrow$Nat}$
>
> #### subset semantics
>
> ```
> 	(fun arg: {x: Nat} (f arg.x)) {y = id, x = 0}
> ->	f ({y = id, x = 0}.x)
> ->	f 0
> ->	1
> ```
>
> #### coercion semantics
>
> ```
> 	(fun arg: {x: Nat} (f arg.x)) ((fun from: {y: Bool -> Bool, x: Nat} {x = from.x}){y = id, x = 0})
> ->	(fun arg: {x: Nat} (f arg.x)) {x = {y = id, x = 0}.x}
> ->	(fun arg: {x: Nat} (f arg.x)) {x = 0}
> ->	f ({x = 0}.x)
> ->  f 0
> ->  1
> ```
>
> 可以发现 subset semantics 要求我们能屏蔽一些底层细节，用统一的方法对不同类型（但属于同一个父类型）的项进行统一的操作。
>
> 而 coercion semantics 则不要求上面这一点，不同实际类型可以用不同的方法来操作，子类型的提升是通过显式的语义函数完成的。

通常这个生成 cast function 的过程是自动化的，并且是基于 typing derivation tree 来的。你可以把它叫做 compilation，没什么问题。

当然具体的规则自己去看书，只需要知道得分别对 类型、子类型提升、项 三个东西分别翻译就行了。

## Coherence

Coercion Semantics 的代码生成是基于 typing derivation tree 的，因此如果某种类型的 derivation 不唯一，那么代码生成就可能有歧义。经典的例如  $\text{Bool<:Int<:Float}$，那么在 $\text{toSring}(True)$ 时就会出现 `"1"` 和 `"1.000"` 的区别

称一个翻译 coherent 当且仅当对于同一个 typing judgement $J$，其任意 derivation tree 产生的翻译程序语义等价（原文用的 behaviourally equivalent）。
