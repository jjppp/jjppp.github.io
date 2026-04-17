---
title: 形式语义01 Intro
tags: Formal Semantics
date: 2021-09-09 21:44:00
---
\newcommand\norm[1]{\left\lVert#1\right\rVert}
\newcommand\abs[1]{\left\lvert#1\right\rvert}
感觉第一节课都差不多


推荐了*Software Foundations*，搜了一圈发现非常劝退....但还是磨磨蹭蹭看完了Lists

书上说不建议贴答案和题解，那就不贴了吧（


developing general abstractions, or building blocks, for solving problems, or
classes of problems. Also considers software behavior in a rigorous and
general way, to prove that programs enjoy properties




当我们提及程序的正确性时，我们在谈论什么？


* How to describe meanings of programs? 含义

* How to describe properties of programs? 性质

* How to reason about programs? 推理

* How to tell if two programs have the same behaviors or not?

* How to design a new language?

* How to build Bug-Free Software?




现在的软件也越来越复杂


* Multi-core concurrency

* Embedded software, Limited resources

* Distributed and cloud computing, Network environment

* Ubiquitous computing and IoT

* Quantum Computing




测试的局限性


* 简单，容易自动化、流程化

* 无法保证一定没有bug

* 对并发(多核/网络)程序作用有限，发现的bug难以重现

* Testing shows the presence, not the absence of bugs. Dijkstra




对于Crash-Proof工作，如何证明？


* How to prove mathematically?

* How to define "crash"?

* How to prove a system is "crash-free"?




为什么要上这门课？


* 软件可靠性是目前严重的问题

* 是别人没有的优势

* 可以提升代码能力

* 更好地理解和比较编程语言

* 这门课很有挑战




课程内容


* 对现有语言的研究

* 讲一些定义程序行为的方法

* 讲一些证明程序性质的方法
* 如何定义性质
* 如何证明



Coq


* 可以自动化证明命题

* 可以自动化验证证明

* 我们不再需要信任证明，只需要信任证明工具，然后检验一个证明


也就是说，一个程序可以自带一个安全性proof，而运行环境自带的证明器用以验证，以此来保证程序是正确的
