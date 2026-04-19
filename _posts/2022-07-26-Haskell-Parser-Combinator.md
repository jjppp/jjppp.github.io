---
title: Haskell Parser Combinator
date: 2022-07-26
permalink: /posts/2022/07/Haskell-Parser-Combinator/
tags:
  - haskell
---

欢迎指正本文的错误！

# Intro

> What is Parser Combinator?

传统的 Parsing 包括 Lexing 和 Parsing 两部分，指的是用算法来解析字符串以得到某些特定形态的数据结构(通常是AST这样的树形结构)

实际上 Parsing 有非常 general 的结构，因此平时真正的用法都是把特定的语法 $G$ 喂给一个程序 $P$，然后得到一个可以解析 $G$ 的程序 $P_G$。通常会用一个 DSL(Domain Specific Language) 来描述 $G$，这样的程序 $P$ 称为 Parser Generator。

* 好处：比较高效
* 坏处：生成代码可读性、换一个语法需要重写 DSL

Parser Combinator 则是另一条路子：一个大的 parser 可以看成是若干小 parser 组合得来的，这样就可以把 structural 的语法信息包含在 parser 里，同时追求 parser 作为组件的复用。

* 好处：清爽、调试方便、可以复用
* 坏处：性能问题

> Why Haskell?

实际上 Haskell 里的名字和符号都比较奇怪，我其实是看了 $\text{F}\sharp$ 的一个 talk 才恍然大悟的。用哪种语言都一样，Haskell 看起来更 Geek 一点。

# Single Char

首先要清楚 parser 是什么。我们的 parser 应当有至少一个函数 `runParser`，它会

1. 吃进一个字符串
2. 解析字符串
3. 如果解析成功，那么返回一个解析过的数据，以及剩下的输入
4. 如果解析失败，那么返回一个报错信息

在 OO 的语言中(例如 Java)很容易设计一个 `class Parser`，然后实现一个 `runParser()` 方法。在 Haskell 中则是通过设计一个带 field 的数据，然后令这个类型的数据为一个函数来实现的。

写成代码就是

```haskell
data Parser c a
    = Success {
        getTerm:: a,
        inputStream:: [c]
    }
    | Failure String
    | Parser {
        runParser:: [c] -> (Parser c a)
    }
```

这里比较巧妙的地方有两处：

1. 利用 Type Variable 可以得到更 general 的 parser，即我们创造的是解析类型 `c` 得到类型 `a` 的 parser。
2. Haskell data 的 field 本身可以是函数，这样就得到了类似 class + method 的结构。类似 OO 的 interface 多态也可以用这个办法实现。

如果考虑最简单的 parser，那么就是

1. 吃进一个字符串
2. 解析**一个字符**
3. 如果解析成功，那么返回**这个字符**，以及剩下的输入
4. 如果解析失败，那么返回一个报错信息

这就是一个最简单的 parser，也就是一个单字符的 lexer。

```haskell
satisfy:: (Eq c, Show c) => (c -> Bool) -> Parser c [c]
satisfy predicate = Parser go where
    go [] = Failure "EOF"
    go (x:xs)
        | predicate x = Success [x] xs
        | otherwise = Failure $ "Only to find " ++ (show x)

pChar:: Char -> Parser Char [Char]
pChar c = satisfy (== c)
```

同样是两个比较巧妙的地方

1. 可以用一个简短的 helper 函数搭配 `where` 做到看起来更清晰
2. type class 可以让 `satisfy` 很 general

这样 `pc = pChar 'c'` 就得到了一个可以解析单个字符 `'c'` 的 parser `pc` 了。

# Combination

可以回忆 parsing 过程中都会出现哪些结构：

1. 在解析 C 语言函数定义的时候，我们希望**先**解析一个类型，**再**解析一个标识符。即 `<type> <identifier> '(' <optional_args> ')' ';'` 这样的结构
2. 在解析 C 语言的基础类型时，可以是 `int` **或者** `float` **或者** `short` 等等。即 `<type1> | <type2> | <type3> ...` 这样的结构
3. 在解析 C 语言字面量的时候，我们希望把一个 `<int_literal>` 从字符串**变成**值储存在符号表中

上述三种 pattern 就能抽象出三种组合 parser 的基本方式

## `map`

关键就是把 `Parser c` 这个具有 `*->*` 的 Kind 看成 Functor，含义则是“包含了结果(类型为 `a`)的一个上下文”

那么结构3就是：给定一个包含了结果 `a1` 的 parser 和一个函数 `a1->a2`，我们希望得到一个能给出结果 `a2` 的新 parser，其中 `a2` 是通过函数作用于 `a1` 得来的。

注意到此处某个类型为 `Parser c a` 的数据的 field 实际上是一个函数 `[c] -> (Parser c a)`，这与逻辑上 `Parser c` 是解析结果的容器并不冲突。

```haskell
-- map
instance Functor (Parser c) where
    fmap f (Parser p) = Parser $ \input ->
        case p input of
            Failure err -> Failure err
            Success result rest -> Success (f result)  rest
```

## `andThen`

Haskell 是纯函数式的，这意味着 Haskell 中是没有副作用的。

这句话也意味着在 Haskell 中，我们不应当对函数参数的求值顺序有任何假设。但这显然是不能满足我们要求的，例如我们现在就想要实现**先**执行一个 parser，**再**执行另一个 parser。

这时就需要用到 Monad 了。

### Monad

Monad 的直觉就在于，我们虽然不能保证一个函数的各个参数的求值顺序，但是可以保证一个函数的参数必须在其结果之前求值。

这样就可以把若干个过程按顺序串起来，最后得到一个顺序复合(Sequential Composition)。包一层 Monad 只是规定了这个处于上下文内的值的使用方法，仅此而已。

```haskell
-- andThen
instance Applicative (Parser c) where
    pure x = Parser $ \input -> Success x input
    
    Parser f <*> Parser p = Parser $ \input ->
        case f input of
            Failure err1 -> Failure err1
            Success result1 rest1 -> case p rest1 of
                Failure err2 -> Failure err2
                Success result2 rest2 -> Success (result1 result2) rest2

instance Monad (Parser c) where
    return = pure
    
    Parser p >>= f = Parser $ \input ->
        case p input of
            Failure err -> Failure err
            Success result rest -> 
                p' rest where
                    Parser p' = f result
```

实际上我们只会用到一个 dummy Monad，即 `(Parser p1) >>= \_ -> (Parser p2)` 表示两个 parser 的顺序组合，前一个 parser 结果的获取是捕获得来的(或者说在 do-notation 中不需要考虑这个)。

有了 `andThen` 还可以得到很多有意思的组合子，例如

```haskell
pString:: [Char] -> (Parser Char [[Char]])
pString = traverse pChar
```

可以用来解析一个字符串，非常优雅

## `orElse`

需要一个新的 typeclass `Alternative`

然后就没啥好讲的了。

```haskell
-- orElse
class (Applicative f) => Alternative f where
    empty:: f a
    (<|>):: f a -> f a -> f a

instance Alternative (Parser c) where
    empty = Parser $ \input -> Failure "empty"

    Parser p1 <|> Parser p2 = Parser $ \input ->
        case p1 input of 
            Success result1 rest1 -> Success result1 rest1
            Failure err1 -> case p2 input of 
                Success result2 rest2 -> Success result2 rest2
                Failure err2 -> Failure $ err1
```

一个比较好玩的组合子是 `anyOf`

```haskell
anyOf:: [Parser c a] -> (Parser c a)
anyOf (x:[]) = x
anyOf (x:xs) = x <|> (anyOf xs)
```

即给定一串 parser，我们会得到一个新的 parser，新 parser 会挨个尝试这些链表中的 parser。

于是在 Haskell 里就可以非常优雅地写出字母的 lexer 了：

```haskell
pAlphabet = anyOf $ pChar <$> ['a'..'z']
```

# Untyped $\lambda$-Calculus

周末一直在写这个，事实上一直在写 Parsing 的部分

给出语法：

```
T ::= x
    | (λx.T)
    | (T T)
```

很显然加上了括号之后这是没有左递归的，并且也没有二义性

代码里 Parsing 的部分也非常简单：

```haskell
-- parsing λ-terms
pID = anyOf $ pChar <$> ['a'..'z']
pTerm = pVar <|> pAbs <|> pApp

pVar = do
    v <- pID
    return $ Var' v

pAbs = do
    pChar '('
    pChar 'λ'
    v <- pVar
    pChar '.'
    t <- pTerm
    pChar ')'
    return $ Abs' v t

pApp = do
    pChar '('
    m <- pTerm
    pChar ' '
    n <- pTerm
    pChar ')'
    return $ App' m n
```

# Summarize

这个 parser 总共也才写了 107 行，除去看起来困难实际上繁琐的 AFM 也才 66 行，属于非常轻量的代码。如果用像 MegaParse 这样的库还可以写得更爽一些，也能有更好的性能。

实际上还有很多可以挖掘

* 能不能改善报错？AFM的过程中可以看到很多时候都在写错误处理，这里是可以加入更多信息来让错误更可读的
* `Parser c a` 设计成结果和解析函数放在一起是否合适？用 `Either Error Result` 这样的类型是否更好？
* 还有哪些组合 parser 的方法？这些就是全部了吗？

但考虑到 tapl 后面的章节已经咕了很久，所以打算 move on 了。