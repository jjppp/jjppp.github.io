---
title: DB03 SQL
date: 2024-02-05 23:44:12
tags: Database
---



其实有点无聊，不过俺还从来没写过SQL，所以记一记

# Basics

常用的语法是

```sql
SELECT expr1, expr2 ...
FROM table1, table2 ...
WHERE SEL-predicate
GROUP BY col1, col2 ...
HAVING GB-predicate
```

语义写成伪代码是

```haskell
product(tables) |> (eval exprs <$>) |> (filter SEL-predicate)
                |> (toMap columns)  |> (filter GB-predicate)
```

注意两个 filter 的顺序

还有一些集合上的操作和类似let binding的语法，以及一些表达副作用的更新操作

# Others

上面感觉就已经足够cover绝大部分日常工作了

## View

我的理解就是带 lazy evaluation 的全局 let binding，因为 SELECT 非常的纯真所以可以延迟求值的同时做一些优化

这里把 cached view 叫做 materialized view

## Transaction

我的理解是副作用需要一个东西来管理，SQL 就提供了最简单的类似 TX-begin TX-end 的结构来保证若干个副作用满足原子性和顺序

## Constraints

书上说只有开销比较小的检查才支持，我感觉怪怪的.jpg

* not null 类似于类型修饰符，加在类型后面
* unique 表达 super key 限制，和 primary 的区别在于可以 null
* constraint check(Predicate) 相当于一些简单的 assertion

constraint 可以有自己的名字，并且可以随时添加/删除

## Foreign key

* 可以用 `foreign key (col_name) references talbe_name(ref_col_name)` 来表达引用的取值范围（也就是一个泛型 `ref<T>`）
* 可以用 `on delete/update cascade` 来保证当被引用的 tuple 发生变更时，对应的引用也一起变更（也就是一个GC）

### Cyclic reference

这个问题还挺常见的，办法有这么几个

* constraint 检查可以被推迟，例如出现相互引用的 foreign key 时可以在一个 transaction 内暂时不顾 referential integrity
* foreign key 可以设置为 null，变成 mutable 的形状

## User Defined Types

看关键字的意思是还可以 non final(有 subtype)。很裤，不说话

Domain 我的理解就是 duck typing，更裤了

## Others

还有很多很多细碎的

# PL+DB

一句话：在通用编程语言(PL)里访问和操作数据库(DB)

按照我的理解粗暴分成两类

* static/compiled/`PreparedStatement`：传给 DBMS 的查询是某种解析/优化过后的 IR
* dynamic/interpretive/`Statement`：直接给 DBMS 传 SQL 语句

这里的 static/dynamic 是相对于 DBMS 而言的，在 host 语言里可能全是动态行为(比如 JDBC 的字符串)，也可能是编译期行为(比如大名鼎鼎的 LINQ)

传 IR 的好处显而易见：

* 可以做优化
* 可以做安全性检查

<img src="eSQL.png" alt="eSQL" style="zoom:100%;" />

对着这张图看

* 最左是 LINQ 那种编译期可以做 lint 检查的 embedded SQL 做法
* 中间是 JDBC 那种运行期间拼接字符串产生 SQL 查询，调用 API 的做法
* 最右的两个箭头展示了 static 和 dynamic 的区别(以 JDBC 为例，就是 `PreparedStatement` 和 `Statement` 的区别)
