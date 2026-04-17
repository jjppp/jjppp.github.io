---
title: Concurrency04 Promising
date: 2022-11-07 19:35:31
tags: Concurrency
---

# Intro

C11 model 的最大问题有这么几个

1. OOTA，这说明 C11 model 太弱了
2. UB，这让一些 type-safe 的语言没法用这个 model，因为 UB 本质上和 type safety 冲突。

所以几个大家都很熟悉的人提出了 promising semantics，claim 他们解决了 OOTA 的问题，同时还是 operational 的，这样就不会有 UB。

当然这一段我不是太感冒，就当是夹带私活了。

# Idea

promising 的叫法来自于 model 中的一个特殊操作，make promises。

这个操作的意义在于，一个线程可以在真正操作内存之前，向其它线程claim它将会完成一些内存的修改（写入）。

在这个 model 中，内存 $M$ 被视为一个消息的集合（set of messages），每个消息是三元组 $\left<x=v \text{ @t}\right>$，表示在 timestamp $t$ 时刻，$x$ 将被写入值 $v$。每个线程对内存的视角（view）可以不同，这里视角定义为函数 $\text{loc}\mapsto M\vert_{\text{loc}}$，论文要求每个线程的 view 关于 timestamp 必须是单调的，同时在读写内存时有一些 view 的修改的规定。

为了解决 OOTA，论文还作出了 well-behaved 这样的要求：即任意时刻，每个线程都需要保证它确实能够完成它的 promise。当然这里的描述很模糊，具体可以去看论文。当然我觉得这样有些耍赖的感觉，毕竟每一步都要保证所有的结果都是可达的，自然就不存在 OOTA 了。

感觉之所以会讲这个主要是因为 PLAX 自己也在 follow 做相关的东西。但是在不涉及真的形式化和真的验证的时候，好像很难说你真的懂了这个，以及这个东西真的有用处。看看论文就差不多了吧。
