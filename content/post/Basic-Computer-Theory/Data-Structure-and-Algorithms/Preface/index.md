+++
date = '2025-05-20T16:13:00+08:00'
draft = false
title = '数据结构与算法-序言'
categories = ['Sub Sections']
math = true
+++

## 数据结构
数据结构(Data Structure)指的是，一系列数据的组织和存储的结构。也可以说是单个数据与其他数据之间的关系。数据结构有以下几大类型：

1. 线性结构
1. 哈希表
1. 树形结构
1. 图形结构

## 算法
算法(Algorithm)指的就是解决问题的方法。算法是一系列的运算步骤，这些运算步骤可以解决特定的问题。

算法追求的目标有 5 个：正确性、可读性、健壮性、所需运行时间更少（时间复杂度更低）、占用内存空间更小（空间复杂度更低）。

这里讨论的算法，是串行算法，不会同时进行多个操作。

### 算法复杂度
算法复杂度(Algorithm complexity)是指，在问题的输入规模为 $n$ 的条件下，算法执行时间的增长趋势和算法使用空间的增长趋势。

#### 时间复杂度
对于问题的输入规模 $n$ ，算法时间复杂度使用 $T(n)$ 表示。

某算法代码如下：

``` C#
public int algorithm(int n)
{
    int a = 1;
    int b = 2;
    int res = a * b + n;
    return res;
}
```

显然，这个算法的输入规模是 `int n` 。随着 `n` 的增大，这个算法所消耗的时间如何增大？显然，无论 `n` 如何增大，算法所使用的时间不变，于是 $T(n) = O(1)$ 。

> 其中， $O$ 是渐进上界符号。对于函数 $f(n)$ 和 $g(n)$ , $f(n) = O(g(n))$ 。存在常量 $c, n_0$ , 使得 $n > n_0$ 时，有 $0 \leq f(n) \leq c \cdot g(n)$ 。比如： $f(n) = 2n^2$ ， $f(n) = O(n^2)$ 。

#### 时间复杂度的计算
在计算时间复杂度的时候，我们经常使用 $O$ 渐进上界符号。因为我们关注的通常是算法用时的上界，而不用关心其用时的下界。求解时间复杂度一般分为以下几个步骤：

1. 找出算法中的基本操作（基本语句）：执行次数受到输入影响的语句。
1. 计算基本语句执行次数的数量级：只需要计算基本语句执行次数的数量级，即保证函数中的最高次幂正确即可。像最高次幂的系数和低次幂可以忽略。
1. 用渐进上界符号表示时间复杂度：将上一步中计算的数量级放入渐进上界符号 $O$ 中。

#### 空间复杂度
其实和时间复杂度差不多，只是计算的目标变为存储空间罢了。
