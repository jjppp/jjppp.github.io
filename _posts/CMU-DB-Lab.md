---
title: CMU-DB Lab
date: 2023-01-10 23:54:11
tags: Database
---

# Intro

学校里的数据库实在是讲得太烂了，是听了十分钟就会想激情退课的水准

cmu这个感觉就 andy 比较有激情，21年代课的老师也比较闷

听说这个 lab 很牛逼（jyy和qlg强力推荐），于是做一做

我做的是2022fall的版本，这个评分的网站 gradescope 似乎没法注册，所以能通过本地测试就是胜利！

# Lab0-Trie

看说明 lab0 是用来劝退的，大概是要你写一个支持并发的字典树

实测并发是个酱油，一把大锁就能解决了，然后读写锁也都给了你...

比较关键的点就是 `TrieNodeWithValue` 继承了 `TrieNode`，然后有一个构造函数是 `TrieNodeWithValue(TrieNode &&other, T Value)` 用于 consume 一个普通节点，构造出一个带 value 的节点。插入的时候，节点类型的转换我是通过先删除后添加的方法做的...

以及一个之前没注意过的点是这样的：`make_unique<T>(ARGS)` 的参数 `ARGS` 将被用于构造 `T` 而不是作为 `unique_ptr` 的内容。这也意味着 `make_unique<T>()` 只能创造指向 `T` 的指针。

而如果想要实现多态的话，我们得这么做 `unique_ptr(new T())`，这样用一个裸指针来初始化。下面是一个我附会了的例子

```C++
class Base { public: virtual ~Base() = default; /* other stuff*/ };
class Derived : public Base { /* other stuff */ };

int main() {
    auto p1 = std::unique_ptr<Base>(new Derived());
    auto p2 = std::make_unique<Base>(Derived());
    
    std::cout << dynamic_cast<Derived *>(p1.get()) << std::endl;
    std::cout << dynamic_cast<Derived *>(p2.get()) << std::endl;
}
```

这里第一个是正常的指针，第二个是 `nullptr`。这是因为 

````c++
std::make_unique<Base>(Derived())
````

等价于

```cpp
std::unique_ptr<Base>(new Base(Derived()))
```

然后这里又有一个子类型的转换，于是

```cpp
std::unique_ptr<Base>(new Base((Base)Derived()))
```

c++还真是神奇啊
