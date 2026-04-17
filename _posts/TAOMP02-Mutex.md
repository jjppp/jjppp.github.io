---
title: TAOMP02 Mutex
date: 2022-09-02 17:03:21
tags: TAOMP
---

# 形式化

## 定义

* **事件**是瞬间的、原子的，通常用 $read, write$ 这样的名字表示一个事件，记事件的集合为 $E$
* 定义 $\to$ 为 $E$ 上的全序关系，其含义为两事件发生的顺序。因为事件是瞬时的，因此不存在两个事件同时发生。
* **过程**或**时间段**是二元组 $(s,t)\in E\times E$ 满足 $s\rightarrow t$，其含义为两个时刻中间的时间
* 定义 $\rightarrow$ 同时为 $E\times E$ 上的偏序关系，其含义为两个过程发生的顺序。很显然存在某些过程有重叠
* 若过程 $A,B$ 重叠，则它们是**并发**的
* 某些事件存在副作用。其含义为，若 $P(A)$ 成立，且 $A\to B$，则 $P(B)$ 仍然成立，其中 $P$ 为关于事件的谓词。
* 副作用可以被**覆盖**，因此某个性质是否成立需要类似于“最后一次xx”的前提

# 锁

锁是一类对象，通常提供两种方法

* `lock()` 用于获取锁
* `unlock()` 用于释放锁

规定一把锁至多被一个线程获取。若一把锁被某个线程持有，则称锁是忙的（busy）。

## Critical Sections

> Critical Section: A block of code that can be executed by only one thread at a time

这样的性质叫 mutual exclusion，通常用锁来实现。

形式化地说就是把进入和退出 critical section 视为事件 $i,o$，那么对于线程 $\set{0,1}$ 而言，要么 $(i_0,o_0)\to(i_1,o_1)$，要么 $(i_1,o_1)\to(i_0,o_0)$

可以把 critical section 与一把特定的锁关联起来，并要求所有执行这段代码的线程都必须持有这把锁，这样就保证了 mutual exclusion 的性质。同样也可以把某个共享数据与一把锁关联，这样就起到了串行化对共享数据的访问和修改。

## Property of interests

在锁的语境下，前面提到的各种性质就可以叙述为

* mutual exclusion：任意时刻只有至多一个线程持有锁
* deadlock-free：若某个线程尝试获取/释放锁，则最终将有至少一个线程得到/释放了锁。如果某个线程在 `lock()` 中卡住，那么其余线程必然能无数次进入 critical section。意思是可以有局部的卡住，但是相应的必然有**某些**线程能执行。
* starvation-free：所有获取/释放锁的尝试最终都会成功。意思是对于每个线程而言都不会卡住。

其中 deadlock-free 指的是全局的 progress，而 starvation-free 则表明了每个线程的 progress。因此 starvation-free 必然是 deadlock-free 的。

当然还有更好的保证等待时间的性质，这就比 starvation-free 要更强了。

### Fairness

fairness 讲的是锁算法能够保证先到先得。为了形式化说明“先到”，每个锁算法应当被划分成两部分：

1. doorway section，使得存在 $n\in\mathbb N$ 保证了这部分必然在 $n$ 步之内完成。
2. waiting section，这就是忙等的部分

我们说某个锁算法满足公平性，当且仅当对于任意两个线程 $A,B$ 都有
$$
D_A\to D_B\Rightarrow CS_A\to CS_B
$$
直观含义就是，率先完成 doorway section 的线程会更早进入 critical section。注意此处都是时间段的比较。

这里引入的一个性质就叫 first-come-first-serve，如果一个锁既是 deadlock-free 又是 fcfs 的，那么它一定是 starvation-free 的。

## 锁1

伪代码是这样的

```C
typedef struct {
    bool flag[2];
} lock_t;
void lock(lock_t *lck) {
    lck->flag[current_thread_id()] = true;
    while (lck->flag[another_thread_id()])
        ;
}
```

可以发现这样是保证了 mutual exclusion 的（简单反证一下），但是没有 liveness 的保证。例如两边同时设置了 `flag` 之后就会卡住。

## 锁2

伪代码是这样的

```c
typedef struct {
    unsigned victim;
} lock_t;
void lock(lock_t *lck) {
    lck->victim = another_thread_id();
    while (lck->victim != current_thread_id())
        ;
}
```

可以发现这样是保证了 mutual exclusion 的（因为 `victim` 只能取其一），但是也没有 liveness 保证。当只有一个线程活跃的时候它将永远等待下去。

比较巧妙的点在于往 `victim` 中写入对方的 thread id，这是为了实现“先到先得”的效果。

## Peterson's algorithm

神秘的地方在于，上述两种锁

1. 都满足 mutual exclusion
2. 各自在 存在争抢 和 不存在争抢 两种情况下表现良好

所以把它们结合起来就得到了 Peterson's Algorithm，伪代码是这样的

```c
typedef struct {
    unsigned victim;
    bool flag[2];
} lock_t;
void lock(lock_t *lck) {
    lck->flag[current_thread_id()] = true;
    lck->victim = another_thread_id();
    while ( lck->flag[another_thread_id()] == true
         && lck->victim != current_thread_id())
        ;
}
```

证明可以直接讨论，此时算法退化成其中一种，就证完了。

同时 Peterson's Algorithm 是满足 starvation-free 的，证明只需要注意到 `victim` 的设置在各自线程是不可逆的就行了。

## Filter lock

也就是推广后的 Peterson's Algorithm。大概的想法是

* 设定长度为 $n-1$ 的队列
* 每个想要获取锁的线程都从队尾开始排队
  * 如果多个线程争抢队尾，则规定最后一个进入队尾的线程**滞留**在队尾，其余线程越过队尾争抢倒数第二个位置
  * 如果只有一个线程想要进入队尾，则可以立刻进入
* 所有被滞留的线程将等待直至下一个位置为空，然后争抢下一个位置。

伪代码长这样

```c
typedef struct {
    unsigned victim[N];
    unsigned level[N];
} lock_t;
void lock(lock_t *lck) {
    int curr = current_thread_id();
    for (int i = 1; i < N; ++ i) {
        lck->level[curr] = i;
        lck->victim[i] = curr;
        while (∃thread≠curr, lck->level[thread]≥i && lck->victim[i]==curr)
            ;
    }
}
```

可以发现这里的 `level[t]` 表示了线程 `t` 在队列中希望争抢的位置（`1` 是队尾），而 `victim[i]` 起到了区分争抢位置 `i` 的最后一个线程的作用

mutual exclusion 的证明可以通过归纳 level 简单做到，starvation-free 的证明则需要反向归纳一下（所有进入 $n-i$ 层的线程最终都将返回，对 $i$ 归纳）

## Bakery

每个线程都会被分配一个唯一的编号，能进入 CS 当且仅当编号在它前面的线程都离开了 CS，其中分配编号的部分就是 doorway section。

有一些比较有意思的性质：

* 编号是全局严格递增的
* 对单个线程而言，编号也是严格递增的
* 满足 FCFS

# Deadlock

## 狭义的 deadlock

虽然 Peterson's Algorithm 本身是 deadlock-free 的，但是涉及到多个锁的时候，整个系统仍然会卡住。例如经典的 lock-ordering 问题。有的时候 deadlock 指的是这样的情形。

## livelock

书上讲得很模糊，大概意思是要满足

1. 多个线程整体是卡住的
2. 它们各自在阻碍其它线程前进

livelock 通常是可以通过特殊的调度来避免的，例如 OSTEP 里面提过的经典的 try-release-retry 这样的模式带来的 livelock。
