---
title: JVM exception handlers
date: 2025-04-05
permalink: /posts/2025/04/JVM-exception-handlers/
tags:
  - JVM
---

又涨姿势了

最近在研究一个 Java 静态分析框架的异常分析算法。异常分析大概可以分成两个部分：方法内的异常分析，以及跨方法的异常分析。

## 方法内异常分析

简单来说就是基于（bytecode）语法的语法分析。Java 程序中的异常是这么一回事：

```java
try {
    // ...
    throw new ExceptionA(...);
} catch (ExceptionB e) {
    // ...
}
```

在 `try` block 中出现的异常会由内向外寻找，直到找到第一个能够匹配上抛出异常的类型的 `catch` block，然后进行控制流的跳转。

不难发现这样的结构是可以嵌套的，所有的 `try` block 会构成一棵树。

观察这个分析框架里的算法，它是这么写的（有简化）：

```python
for eh in getAllExceptionsHandlers():
    for stmt in getAllStmts():
        if stmt in eh.tryBlock:
            catchers[stmt].append(eh);

for stmt in getAllStmts():
    exceptions = getExceptionsThrownAt(stmt)
	for eh in catchers.get(stmt):
        # Filter out those exceptions that can be handled by handler eh
        exceptions = filter(lambda x: not eh.catch.catches(x), exceptions)
    if not exceptions.empty():
        # These exceptions will be thrown out of the method
```

看着没什么问题，这里的 `catchers.get(stmt)` 实际上是一个有序的列表，这意味着此处对于 `catch` block 的搜索是按照某种固定的顺序进行的。

这个顺序从哪里来？答案在[这里](https://docs.oracle.com/javase/specs/jvms/se12/html/jvms-3.html#jvms-3.12)。和前面铺垫的不同，JVM 实际上并没有嵌套的异常处理结构，因此 JVM 中对于异常的处理并不是在做某种树形结构上的搜索。相反，JVMS 强硬地规定了编译后的文件应当包含一个有序的 exception handler 列表，这个表中的顺序就代表了**所有**异常发生时，查找 `catch` block 的搜索顺序。用树形结构的视角看，实际上就是选取了一个固定的树上遍历顺序（一个比较直接的顺序是后序遍历）。

那么这是否就说明“管辖”同一语句的两个 exception handler 作为 `catch` block 的区间必然存在包含关系呢？答案也是否定的。`dacapo2006` benchmark 中的 `eclipse-deps.jar` 就存在这样的情况。我观察后发现这些不包含的情况（存在 handler `eh1` 和 `eh2`，使得既不 `eh1 in eh2`，也不 `eh2 in eh1`）通常是存在一个原本的 handler `EH`，在编译或者优化的过程中 `EH = eh2 + eh3 + ...` 被分成了多个连续的区间，而 `eh1` 和 `EH` 是存在着包含关系的。

## 跨方法异常分析

TBD
