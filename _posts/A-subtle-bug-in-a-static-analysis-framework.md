---
title: A subtle bug in a static analysis framework
date: 2023-12-26 16:26:13
tags: Eureka Moments
---

编译器的bug已经让人痛苦了，作为抽象的编译器——静态分析器里的bug只会更微妙、更隐蔽。

# Intro

最近在关心基于抽象解释的单调分析框架和基于这类框架的分析算法设计，于是很自然地参考了一些著名的开源静态分析框架算法，在这期间我发现了一个这样的问题：

> 某个常量传播 evaluator 简化后的设计如下：
>
> ```coq
> def evalBinary(lhs, rhs, op):
> 	match (lhs, op, rhs) with
> 	| _, DIV, (Const 0) => UNDEF
> 	| (Const c1), _, (Const c2) => op.apply(c1, c2)
> 	| (Const 0), MUL, _ => (Const 0)
> 	| _, MUL, (Const 0) => (Const 0)
> 	| NAC, _, NAC => NAC
> 	| _, _, _ => UNDEF
> 	end.
> ```
>
> 可以发现这个函数不单调

这里于某次 commit 中引入了一个基于代数恒等式的简单优化，即 $\forall x, 0\times x=0$。简单思索之后可以发现这是不单调的！只需要考虑输入 $(NAC, UNDEF), (NAC, 0)$ 即可。

一个自然的疑问是：我们能利用上这个 bug 来做点有趣的事情吗？

# A quick dive into monotonicity

首先需要知道为什么我能 claim 这是一个 bug

基于格抽象的静态分析框架通过抽象程序的执行状态、定义抽象状态上的状态转换、迭代执行抽象操作直至收敛 这三步完成一个抽象解释执行程序的艰巨任务。

而收敛（算法终止）的关键则在于对程序状态的抽象上——我们需要保证抽象状态的转换“不走回头路”，也就是不会兜圈子。

这种性质在数学上的刻画就叫单调性。如果失去了单调性，分析算法可能就不能结束输出结果，而是进入一个死循环。

# Yes, but how?

再回头看我们究竟发现什么破坏了单调性：当某个变量的抽象值从 UNDEF 变为 0 时，它与 NAC 的乘积会从 NAC 变为 0。

想要构造出死循环，意味着我们需要构造至少两个彼此依赖的变量 $\Set{x_1,x_2}$ 使得：

1. $x_1$ UNDEF->0，$x_{2}$ NAC->0
2. $x_{2}$ NAC->0，$x_{1}$ 0->UNDEF

一个个看怎么办：

1. 当然是用 `x2 = x1 * NAC`
2. 看起来有点麻烦

# 柳暗花明又一村

在找到反例之前我也以为这样的例子是不存在的，直到我的朋友告诉了我一个分析器中常见的优化小技巧：edge transfer

考虑如下代码：

```c
if (x == 1) {
	return x;
} else {
	return 1;
}
```

if 的判断条件是带有额外信息的，这意味着在 true branch 内我们都可以安全地假设 `x` 的值要么是 `UNDEF`，要么是 `1`，而不可能是其它值。

形式化地讲，就是 $[\![x']\!]=[\![x]\!]\sqcap[\![1]\!]$，一个格上的 meet 操作。

利用这个 trick，我们就可以构造出程序使得某个变量的值发生 0 -> UNDEF 的变化——又一处不单调！

一个例子是这样的：

```C
if (x == 1) {
	y = x - 1;
} else {
	y = y / 0;
}
```

这里额外用到了一个 `x / 0 = UNDEF` 的 trick

当 `x` 在 0 和 NAC 之间震荡时，根据 meet 会有 `x' - 1` 在 UNDEF 和 0 之间震荡，这恰好就是我们想要的2

# Putting all together

最终人肉综合出来的程序大概长这样

```c
x = 1 / 0, y = NAC;
while (true) {
    z = x * y;
    if (z == 1) {
        x = z - 1;
    } else {
        x = 1 / 0;
    }
}
```

可以跑一跑得到这样的 trace：

```C
[x: UNDEF, y: NAC, z: UNDEF]@2
// 第一轮
[x: UNDEF, y: NAC, z: NAC]	@3
[x: UNDEF, y: NAC, z: 1]	@4
[x: 0, y: NAC, z: 1]		@5
[x: UNDEF, y: NAC, z: NAC]	@7
[x: 0, y: NAC, z: NAC]		@8

// 第二轮
[x: 0, y: NAC, z: 0]		@3
[x: 0, y: NAC, z: UNDEF]	@4
[x: UNDEF, y: NAC, z: UNDEF]@5
[x: UNDEF, y: NAC, z: NAC]	@7
[x: UNDEF, y: NAC, z: NAC]	@8
```

对比就能发现循环两次之后的抽象状态完全相等，但每次迭代之后每个基本块对应的状态都会发生变化，因此不会停机

...吗？

# Analysis Framework: A bottom-up perspective

在进行了详尽的测试之后，存在这个 bug 的框架并没有如预想般死在我面前，而是成功停机了。

读了代码之后有如下观察：

1. 框架通过一个 map 给每个基本块的入口和出口都分配了一个 `AbsEnv`
2. 每次合并前驱计算转换时，通过枚举计算结果（`Var`-`Value` mapping）来更新出口 `AbsEnv`，此为 `update` 方法
3. `AbsEnv` 中通过 absent key 来表示 UNDEF

这意味着：

1. 当出现了 0->UNDEF 的突变时，`AbsEnv` 会直接删掉这个元素
2. 枚举更新的方式则会直接忽视所有 UNDEF，进而保证了与 UNDEF 相关的非单调性不会影响终止

所以很遗憾，这是一个被框架设计兜底了的 bug

# 想法

* 抽象解释的理论很简洁，但实践很复杂
* 静态分析器也可能会有微妙的 bug
* 并非所有 bug 都会显现，有时是框架擦了屁股
* 我们能像测试编译器一样测试静态分析器吗？
