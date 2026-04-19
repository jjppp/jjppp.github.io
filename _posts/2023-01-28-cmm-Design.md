---
title: 'cmm Compiler Design'
date: 2023-01-28
permalink: /posts/2023/01/cmm-Design/
tags:
  - Compiler
---

# Intro

cmm 是用 C99 编写的编程语言 cmm 的编译器

本文的主要目的是记录 cmm 的细节和取舍，以记录分析和优化相关的内容、IR 设计的知识和从 LLVM/JDK 那里抄来的东西（

# Parsing

主要是基于 Flex/Bison 这套工具做的，然后顺便设计了一下 AST 的结构

## Flex/Bison

讲几个比较有用的小技巧（全都是从手册里抄来的）

1. 可以给符号定义 `%destructor`，从而实现一些资源的管理。例如 Bison 的错误处理会不断从栈上弹出符号，这些符号相关资源的释放则会调用这个自定义的析构函数。
2. 可以设置自定义报错 `%define parse.error custom`，然后在里面调用一些 Bison 提供的函数获取一些解析。例如做成 “expected symbol XXX, got XXX”

## AST

用了一个 X-macro+OO 的操作来写得比较爽一点，具体可以看 repo。

有了这俩之后就能做 visitor 了。具体而言有这么几个：

1. `ast_alloc`，节点的构造函数
2. `ast_free`，节点的析构函数
3. `ast_check`，对节点做语义分析

本来还想做一些 AST 上的常量折叠/表达式旋转的，但是摸了。

当然这玩意的本质还是一个大的 switch-case，或者按照 tls 的说法是“二维dispatch”

## Polymorphism

实验手册给出的实现多态的方案是这样的：

```c
struct base {
    type_t type;
    union {
        struct derived1 field1;
        struct derived2 field2;
        /* other subclasses */
    };
};
```

这样的好处是比较简单，而且很有 C 的风味。

与手册中推荐的 `union` 方式不同，我这里选取了"继承"的路子。考虑如下代码：

```c
struct base {
    type_t type;
    /* base fields */
} ;
struct derived1 {
    struct base super;
    /* other fields */
};
struct derived2 {
    struct base super;
    /* other fields */
};
```

通过排布内存布局和指针类型的强转就可以实现一部分继承和子类型多态的特性。

这样的写法可以让我们忽略 AST 节点的具体类型，但仍然保留遍历 AST 的能力（回忆内核里的侵入式链表）；

同时不需要滥用 `void *`，方便内存管理；在 cast 之后还可以获得编辑器的补全，无需考虑 `union` 带来的误用。

# Typechecking

typechecking 可以用一个 visitor 实现。需要注意的细节会比较多。

## symtab

符号表需要支持

1. 嵌套作用域
2. 插入符号（长度有限的字符串），得到一个 `entry` 或 `NULL`（表示重复插入），`entry` 可以写入信息
3. 查询符号，得到一个 `entry` 或 `NULL`（表示不存在）

实际上类似的结构在 IR 上的分析也会用到，因此抽出来一个 `hashtab` 用于查询符号，插入则通过简单的写入 field 来实现。`symtab` 只是简单包装了一层 `hashtab`，这样底层的哈希函数和查找逻辑后面都可以改动（咕咕咕）。

## type

类型相关的设计非常糟心。理想的类型应当是 ADT，但实际上做起来很麻烦。

1. 数组类型（`TYPE_ARRAY`）可以做成

   1. ```rust
      enum Type {
          TypeInt,
          TypeFlt,
          TypeArr(u32, Box<Type>)
      }
      ```

      这样本质上是将数组视为类型构造子，有一层一层剥开的感觉，做 partial addressing 或者是 type equivalence checking 会很简单，但是生成 IR 的时候算下标会比较麻烦。

   2. 或者直接压平，做成

      ```rust
      enum Type {
          TypeInt,
          TypeFlt,
          TypeArr(u32, Vec<Type>)
      }
      ```

   当然最佳实践应当是做成多级的 IR，这样就可以二者兼顾了。但我比较懒所以直接压平了。

2. cmm 里可以定义结构体，考虑下面这么一段代码：

   ```c
   struct A {
       int a, b, c;
   } d;
   ```

   这一段看上去只是 VarDecl，但它实际上还构造了一个类型 `struct A`。这也解释了有没有 `d` 不影响语法正确性。

   这里我引入了一个 TypeDecl 的节点用来处理这种情况，那么其余所有地方都实质性地变成了对 `struct A` 的引用。

3. 最怪的地方：cmm 并不支持浮点数和整数的任意转换。当然到了后面才会发现这么做是因为 cmm 里的浮点数都是酱油。

## err propagation

不妨假设类型检查出错了，例如考虑如下代码：

```c
int foo(int x) { return x; }
float foo(int arr[114]) {
    return foo(arr);     // ?
}
int main() {
    float f = foo(1, 2); // ?
    float f = bar(1);    // ?
}
```

1. `foo` 重复定义，那么第二个 `foo` 里的错误要不要报？第三行的 `foo` 是哪个函数的引用？

2. `foo` 的参数不对。这里 `foo` 的返回值仍然能保证是 `int` 吗？

3. `bar` 不存在，这里返回值的类型应该是什么？应不应该报赋值两侧类型不匹配？

   （思考：C语言又会是什么情况？）

# IR

咕咕咕

# Analysis

目前只实现了一个数据流分析，并且还没有抽出一个比较通用的数据流分析框架，因此下一步就是实现框架，通过“注册-管理”的模式来跑各种分析。

## Framework

这个框架可以说是改得最痛苦的部分了。实现了这么几个优化：

### Order

事实上这个是最大的大头。这里抄一段讲义原话：

> 格理论告诉我们，in和out集合的运算顺序不影响数据流方程解的收敛性，但会影响解的收敛速度。对于上述数据流方程而言，按照i从大到小的顺序来计算in和out往往要比按照i从小到大的顺序进行计算要快得多。

### Merge

对于 AVL 实现的 set 的 merge，可以通过两次中序遍历+线性合并+中序遍历建树来实现 $O(n)$ 合并。

打了一些 log 之后发现跑 `14Lab3Impossible.0.cmm` 的时候 set 的大小普遍在 2K 左右，结合 perf 可以发现这样做确实有不错的性能提升

### Monotonicity

注意到经典的数据流分析实质上在解一堆数据流约束，并且每个图上的节点对应的抽象数据都存在单调性。也就是大概长这样：

```python
def process(BLOCK):
    for PRED in BLOCK.pred():
        IN[BLOCK].merge(OUT[PRED])			# A <= MERGE(A,B) && B <= MERGE(A, B)
    OUT[BLOCK] = BLOCK.transfer(IN[BLOCK])
```

因此可以这么优化：如果某个节点在 `merge` 之后不变，那么无需触发后续的计算。

这要求 `merge` 必须是单调的、能发现单调性的变化。

当然 `transfer` 却并不一定是单调的。考虑如下情况：

```
a = NAC
a = 0
a = NAC
```

这个基本块内的 `a` 的抽象值显然不单调。这种情况可以通过

1. 改写成 SSA 形式（思考：为什么）
2. 以语句为单位而不是基本块为单位求解

方案1是比较好的解法，但是写起来繁琐（实验要求的 IR 并不支持 $\phi$ 操作）；方案2是比较简单的解法，但是存在效率上的问题。

由于这里的数据流分析框架并没有对 IR 的形式有任何要求，我也就没实现这个优化。

### Live

问了tls之后得知可以从 fact 中删去 dead 的那些，这样就可以节省集合操作了。

看了一下效果确实很不错。

## Live Variables

简单的反向 may analysis。即判断一个赋值的 lhs 在未来是否仍然有用。

# Optimization

调优化的过程可以说是非常坑了，大概花去了一周的时间才勉强克服各种 bug 实现了两个半正确性有一定保障的优化。

## Local Value Numbering

发现这玩意本质上就是 Available Expressions，但是不需要对两个 valtab 进行 merge 操作，所以就更简单。

定义：

1. $\text{cvar}\colon \text{Val}\mapsto 2^\text{Var}$，即 $\text{cvar(v)}$ 表示持有值 $\text v$ 的所有变量
2. $\text{holding} = \text{cvar}^{-1}$，即 $\text{holding(v)}$ 表示变量 $\text v$ 持有的值
3. $\text{val}\colon \text{Expr}\mapsto \text{Val}$，即给表达式标号

对 $\text{val(e)}$ 递归定义如下：

1. 若 $\text{e}$ 是 `OPRD_LIT`，那么 $\text{val(e) = e}$
2. 若 $\text{e}$ 是 `OPRD_VAR`
   1. 如果 $\text{e}$ 在当前块内、当前指令之前被定义过，那么 $\text{val(e) = holding(e)}$
   2. 如果没有，那么 $\text{e}$ 本身是 value （safe-approximation），$\text{val(e) = encode(e)}$
3. 若 $\text{e}$ 形如 $\text{v$_1$ OP v$_2$}$，那么 $\text{val(e) = encode(val(v$_1$), OP, val(v$_2$))}$

那么假设现在遇到了表达式 `tar = lhs OP rhs`，那么先根据递归定义求出 `val(lhs), val(rhs), val(lhs OP rhs)`。

观察 $\text{cvar(n)}$：

1. 若非空则当前指令可以写成 `tar = cvar(n)`。
2. 否则我们分别观察 `lhs` 和 `rhs`，用一些启发式的规则来重写（以 `lhs` 为例）：
   1. 如果 $\text{cvar(val(lhs))}$ 中存在**非临时变量**，那么把 `lhs` 重写成该变量
   2. 如果全是临时变量，那么挑一个最早的（编号最小的）

这么做有如下考虑：

1. 减少重复计算
2. copy-propagation，即尽可能少的使用临时变量，尽可能让临时变量 dead，尽可能让晚出现的临时变量 dead

## Constant Propagation

CP 说起来简单，但是实现也并非那么容易的。

本质上的原理是设计了一个全格用于表示抽象值，每条运算指令都会在一个抽象的 Configuration 下抽象执行，比较坑的地方有这么几个：

1. 抽象值的运算法则。比如说除0怎么处理、运算溢出怎么处理（特别还是用 C 写）、怎么利用代数恒等式提精度
2. 数据流框架怎么加速。因为传递的数据实际上是抽象 Configuration，这样一个比较重的 map 对象的拷贝和修改都是很耗时的，就要挤各种效率

## Dead Code Elimination

### Unreachable

非常简单的分析，但是存在比较大的问题：我的 cfg 并不支持很优雅地删除节点及其附带的边，所以暂时没有打开这个优化。

### Dead Assignment

利用 live 的分析结果，删去无用的指令。
