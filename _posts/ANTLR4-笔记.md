---
title: ANTLR4 笔记
date: 2022-07-02 13:07:19
tags:
---

# 写在前面

最近需要用到这个工具，官方推荐的Definitive ANTLR Book写得比较杂乱，于是想记一下比较有用的一些点

# meta language

## Parser和Lexer

一条rule形如

```
`rule_name` : `product 1`
            | `product 2`
            | ...
            ;
```

若`rule_name`以小写字母开头则为Parser规则，否则为Lexer规则
存在二义性的时候ANTLR默认选择最先出现的Rule推导

## fragment

为了不在Parser中引入新的token name但是又要用一个名称来指代一些符号以简化书写（类似macro expansion）

```
NUM: DIGIT | DIGIT_NONZ DIGIT*;

fragment
DIGIT: [0-9];
DIGIT_NONZ: [1-9];
```

## assoc

指定符号的结合性（默认左结合）

```
expr: expr '^'<assoc=right> expr;
```

## RuleName

一对多的产生式，可以标记上名字以在用Parser Visitor的时候可以单独处理每一种语法推导的情况

```
expr: NUM             # Number
    | expr '*' expr   # Mult
    | expr '+' expr   # Add
    ;
```

这个时候`gen/`下的Parser代码就会区分出`NumberContext`、`MultContext`和`AddContext`以及对应的visit方法

## Member

可以在`.g4`文件里给Parser类声明field或method

```
@members {
  int count = 0;
  void helperMethod() { }
}
```

后续就可以继承生成的Parser类然后覆写这些方法了...

## return value

这个主要和visitor、listener的模式有关系

为了给树上的节点加上注解field，可以这么写

```
expr returns [int value]
    : expr '*' expr         # Mult
    | expr '+' expr         # Add
    | NUM                   # Number
    ;
```

此时`MultContext`会继承`exprContext`，而`exprContext`则会被ANTLR加上`int value`这个field

