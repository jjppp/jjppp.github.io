---
title: Concurrency01 HMM
date: 2022-09-25
permalink: /posts/2022/09/Concurrency01-HMM/
tags:
  - Concurrency
---

# Intro

弱内存模型的出现，本质上是因为单线程的程序优化(编译优化/执行优化)中应用了许多的技术，这些技术在进入多线程编程时代之后使得一些符合直觉的假设不再成立，即内存模型变"弱"了。

为了 cover 这些已经成为历史包袱的优化，同时给出这些"不合常理"行为的边界，Memory Model 就出现了。即我们希望

1. Memory Model 不能太强，这是因为有许多现存的实现、优化 break 了强内存模型的假设，此乃现状；
2. Memory Model 不能太弱，否则这意味着完全的不可控。

# HMM

规定事件为原子的，通常只关注以下几类事件：

1. 读内存
2. 写内存
3. 同步原语的操作，例如 `lck_acq()`, `lck_rel()`

根据各个线程的代码可以得到一个事件上的偏序 program order

## Happens-before Order

我们可以构造一个全局事件上的偏序关系 happens-before order：

1. 若两个事件有 program order，那它们有 happens-before order
2. 若两个事件是关于同一个同步对象的操作，那么它们有 happens-before order。例如对锁的 `release()` 必然在 `acquire()` 之后发生。

如果两个事件之间不存在 happens-before order，那么说它们是并发的。

HMM 说的是，一个读事件可以读到

1. 在它之前发生的最后一个同地址的写入
2. 与它并发的同地址的写入

在使用 HMM 的时候，通常会假设一个 outcome，然后写出 happens-before order，最后判断是否满足 HMM 的要求

## Out of Thin Air

HMM 只是一个模型，OOTA 这样的例子就能说明这一点。

OOTA 的意思是 HMM 这样先给出一个 outcome 后判断是否可行的模型，会存在一些反直觉的 outcome 符合。例如

```
thread1 {
	r1 = x;
	y = r1;
}
thread2 {
	r2 = y;
	x = r2;
}
```

它的一个 outcome `r1=42, r2=42` 是符合 HMM 的。但可以发现整个程序都没有出现过可能产生 `42` 的常量，并且这个 `42` 可以是任意的值，所以说这是凭空产生的(Out of Thin Air)

也有一个解释是这样的：CPU 在投机执行的时候会猜一个 `r1 = x` 的结果，如果它恰好猜了 `r1 = 42` 那么就会出现这个局面。