---
title: Memory Models
date: 2022-09-07 15:23:04
tags: Concurrency
---

# Intro

看了很多资料发现基本都是同一批人在写，[这篇](https://research.swtch.com/mm) 是我感觉讲得比较清楚同时也好懂的。

同时我发现很少有人用形式化的方法描述 model，反而都是用各种 case 的执行结果来区分不同的 model，感觉有点捉急。

Memory Model 说的是内存具体的行为。通常并发程序的执行模型是多个执行流+唯一的共享内存，Memory Model 则决定了多个并发执行流的交互结果（因为它们共同读写这块内存）

之所以要讨论这个，是因为

1. 真正的硬件提供的机制很复杂，不一定和直觉一致
2. 这些硬件机制的描述通常不准确，需要正确的建模

在开始之前要讲一下：

1. 方便起见，不妨假设每条指令都是原子的。
2. 简单来说，并发的本质是存在多个并发的执行流，使得多个并发执行的代码片段 $\Set{P_i}$（指令序列）最终以某种顺序被合并成了一个序列 $L_M$（操作发生的先后顺序），并且按照这个序列执行了。其中 $\to_P,\to_M$ 分别是 $P,L_M$ 上的全序关系（事件发生的顺序）。
3. Memory Model 更具体而言就是若干对 $L_M$ 中与内存相关的指令的限制（纯计算指令是无状态/线程局部的）
4. 方便起见，可以假设所有对于内存的操作都是立即见效的，那么 OoO（乱序执行）、batching（写缓冲）之类的操作就可以简单的理解为指令顺序的调换，同时也可以涵盖指令重排的行为。

# Sequential Consistency

这个是大名鼎鼎的 Leslie Lamport 提出来的。比较像分布式里面说的 Linearizability。

SC 的性质保证了 $\forall C_i,C_j\in P,C_i\to_P C_j\Rightarrow C_i\to_M C_j$。即最终的执行序列中，所有来自同一线程的指令保持了它们所在指令序列（代码片段）的相对顺序。这说明线程内部严格按照顺序执行指令，唯一的不确定性仅来自于线程间的交错。

规定 $read(x)=\max \Set{C\in L_M\mid C\in Store\wedge C\to_M read(x)}$，即某次读取的结果只与最后一次写入有关。

没有指令的重排意味着所有对内存的修改都是立刻全局可见的。

例如 jyy 在 os 课上展示的 model-checker 就是基于 SC 的 memory model 的。

# Total Store Order

在硬件上给每个核心一个写缓存队列（write buffer queue），所有的写入会

1. 首先被放入队列中
2. 在一段时间后核心会冲刷队列，把所有的修改提交到共享内存

所有的读都会

1. 先查询本地的缓存，命中即返回
2. 再查询内存

看起来是非常合理的优化，但是这样会导致一致性的问题，例如线程 A 对缓存的写入，线程 B 是看不到的。这就使得这样的情况可能发生：

1. 初始时 `x=0`
2. 线程 A 在缓存中写入了变量 `x=1`
3. 线程 B 读取 `x=0`

此处虽然执行序列为 `writeA(x, 1), readB(x)`，但是 B 可能读到 `x=0` 或 `x=1`（取决于队列是否被冲刷了）

对应到我们的假设（对内存的操作是立见的），那么此处的效果延迟则可以看成是“store 指令可能会被推迟”，即 $P$ 中的一对 store-load 指令 $C_s,C_l$，$C_s\to_P C_l$ 并不能保证 $C_s\to_M C_l$。

规定 $read(x)=\max \Set{C\in L_M\mid C\in Store\wedge C\to_M read(x) }\cup\Set{C\in L_M\mid C\in Store\wedge C\to_P read(x)}$，即某次读取的结果只与最后一次**全局**写入或最近一次**局部**写入有关，取决于它们的相对顺序。

此外，TSO 也保证了 所有线程见到的内存写入历史都相同。这是因为线程对内存的读取是直接的。一个经典的例子是 IRIW

> *Litmus Test: Independent Reads of Independent Writes (IRIW)*
> Can this program see `r1 = 1`, `r2 = 0`, `r3 = 1`, `r4 = 0`?
> (Can Threads 3 and 4 see `x` and `y` change in different orders?)

```
// Thread 1    // Thread 2    // Thread 3    // Thread 4
x = 1          y = 1          r1 = x         r3 = y
                              r2 = y         r4 = x
```

> On sequentially consistent hardware: no.
> On x86 (or other TSO): no.

但是如果线程1、3 线程2、4 各自共享写入缓存队列的话，考虑执行顺序如下：

```
write(x,1)
r1=x=1
r2=y=0
write(y,1)
r3=y=1
r4=x=0 // in memory
flush(1,3)
flush(2,4)
```

就会出现虽然存在一个唯一的全局写入顺序，但是两个线程各自观察到了不同的写入顺序。**它们眼中的历史出现了偏差**，这句话挺有意思的。

这件事情非常像 git 的协作模式（不妨假设只允许 rebase）。每个人都有自己本地的 git 历史，同时有一个共享的remote branch。每个人的修改都是在本地进行的，因此他人不能立刻看到我的代码提交，而只有我 push 之后我的工作才会反映给其他人。并且所有人都能见到唯一的共享历史记录。
