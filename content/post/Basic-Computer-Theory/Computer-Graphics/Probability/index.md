+++
date = '2025-05-20T16:46:54+08:00'
draft = false
title = '概率论简单笔记'
categories = ['Sub Sections']
math = true
+++

## Random Variables(随机变量)
随机变量用 $X$ 表示，它会观测到随机值。

$$X \sim p(x) \tag{1}$$

$1$ 式表示，随机变量 $X$ 的观测值 $x$ 遵循概率密度函数(Probability Density Function, PDF) $p(x)$。 $p(x) \geq 0$ ，其积分或求和等于 $1$ 。

## Expected Value of a Random Variable(随机变量的期望值)
$$E[X] = \sum_{i} x_i p_i$$

$$E[X] = \int x p(x) ~ {\rm d}x$$

## Function of a Random Variable(随机变量的函数)
假设有:

$$X \sim p(x) \tag{1}$$

$$Y = f(X)$$

那么:

$$E[Y] = E[f(X)] = \int f(x) p(x) ~ {\rm d}x$$

## Monte Carlo Integration(蒙特卡洛积分)
蒙特卡洛积分是数值求解定积分的一种方法。对于一般的定积分，如果被积函数的不定积分容易求，那么定积分也容易求。但是如果被积函数的不定积分难求，就需要采用其他方法。

假设我们要求定积分: $\int_a^b f(x) ~ {\rm d}x ,~  \forall x \in [a, b] ,~ f(x) > 0$

### 算法1
首先使用一个大的矩形区域， $x$ 覆盖区间 $[a, b]$ ， $y$ 覆盖 $0$ 和 $f(x)$ 在区间 $[a, b]$ 的最大值。这个矩形的面积好求，是 $S$ 。在矩形区域内随机取多个点，统计在 $x$ 轴和 $f(x)$ 曲线之间的点的数量于点的总数的占比 $\alpha$ ，得出 $\int_a^b f(x) ~ {\rm d}x \approx \alpha S$

### 算法2
设一个随机变量服从均匀分布

$$X_i \sim p(x) = \frac{1}{b - a}$$

计算

$$\frac{b - a}{N} \sum_{i = 1}^{N} f(X_i)$$

即可。