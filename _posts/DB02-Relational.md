---
title: DB02 Relational
date: 2024-01-23 21:54:30
tags: Database
---

# Def

简单来说，我们称 $R\subseteq D_1\times D_2\cdots\times D_n$ 这样一个 $n$-tuple 构成的集合 $R$ 为为 $n$-relation over domain $D_1\ldots D_n$，简称为 $n$-relation。简单地说，一个 relation 包含了所有使这个关系成立的元素（组），因此在数据库的语境下“关系”与“集合”是等价的。

直观来看，一个关系实为一张巨大的 $n$ 列表，每行为一个满足该关系的元素，每列代表了一个属性。

需要注意的是，在数据库的语境下，“集合”通常指“无序可重集合”，即两个元素的比较是 identity equality 而非 domain-induced equality。因此后面讲到的 primary key/super key 的重要性也就很直观了，因为这里（可能）需要统计重复出现的元素。

这两个视角没什么区别，感觉也没啥好说的。

一些边角要求

* 通常 domain 有一个特殊的占位值，即取 $D_\bot=D\cup\{\bot\}$ 作为真正使用的 domain（群友最爱的 `optional<T>`）
* 一般要求 $D$ 中的元素是原子的，例如 `int`；非原子的例子有 `varchar`、逻辑上的集合

这一切都看起来和命题演算那么熟悉，像啊，很像啊

## Strict structure

关系这样非常固化的结构带来了一些后果：

* 处理起来非常简单
* 用起来也非常简单
* 不够灵活多变

# Keys

数据库语境下的关系 $(R,\equiv)$ 实际上是二元组，分别由定义域和 tuple 上的等价关系组成

以网课签到为例，一个签到记录由三元组 `(学号，名称，地点)` 组成。

* 一个人可以在不同地方签到多次，但只应被记录一次
* 两个人只要学号和名称不同时相等，那么他们的签到就应当分开算

## primary

这里 `(学号，名称)` 就是一组 primary key，它唯一确定了一个 tuple 的 identity

比我聪明的读者会发现这也说明

* 签到表本质上是一个函数，即 `(学号，名称) -> 地点`
* 以及 primary key constraint 本质上是 deterministic function constraint
* 接上条，数据的本质是一堆 primary key 间的映射

## candidate

指极小的列集合 $K$ 满足 $\forall r_1,r_2\in R,\forall k\in K, r_1\text{.}k\ne r_2\text{.}k\rightarrow r_1\text{.}k\not\equiv r_2\text{.}k$

说人话：只要 $K$ 中的属性不全相同，就认为是不同的 tuple

此处 candidate 的意思是可以作为 primary key 的备胎

去掉极小限制后的 candidate 叫做 super key

## foreign/referential

考虑 tuple $t_1\in R_1,t_2\in R_2$，如果 $\text{$t_1$.$k_1$=$t_2$.$k_2$}$，那么我们称 $k_1,k_2$ 是 $R_1,R_2$ 之间的 referential key。不失一般性地，如果 $k_1$ 是 primary key，那么 $k_2$ 就称为 $R_2$ 到 $R_1$ 的 foreign key

如果用 $\sigma_k$ 表示投影到 key $k$，那么约束写出来就是 $\sigma_{k_1}(R_1)\subseteq \sigma_{k_2}(R_2)$，so easy

需要注意的是，某个 foreign key 可能也是 primary key，这二者并不矛盾

### 联谊

以大学生喜闻乐见的脱单为例，不妨假设有一个男女配对环节，表 `B=(名字，身高)`， `G=(名字，院系)` 和 `N=(名字，twitter_id)` 分别表示男女和非二元性别，并且名字都是 primary key

* 一张用来表示心动嘉宾的表 `L=(名字，名字)` 就由两个 foreign key 组成，foreign key constraint 的意义在于每个人的心动嘉宾应当出席了活动

* 一张用来表示心动身高的表 `D=(名字，身高)` 就由 foreign key 和 referential key 组成，因为身高并非 primary key。referential key constraint 的意义在于活动中确实有人长这么高

不难发现 foreign key 是 referential key 的一个特例。直观上看，foreign key 更好实现（目标明确），referential key 适用范围更广（海王）

# Relational algebra

非常的简单，书上介绍了这么几种：

* project：删去一些列
* select：filter by predicate
* product：就是你想的那个 product
* join：product + select 的语法糖
* rename/assign：term rewriting 的语法糖
