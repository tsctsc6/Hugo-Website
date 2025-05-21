+++
date = '2025-05-20T16:55:20+08:00'
draft = false
title = '计算机图形学-仿真'
categories = ['Sub Sections']
math = true
+++

仿真，Simulation。

这里讲仿真的一些数学方法。

## Single Particle Simulation(单粒子仿真)
假设有一个粒子，我们知道它的速度场(是矢量场，根据位置矢量和时间得到速度矢量) ${\bf v}({\bf x}, t)$ ，也就是说:

$$\Large \frac{{\rm d}{\bf x}}{{\rm d}t} = \dot{\bf x} = {\bf v}({\bf x}, t)$$

这是一个一阶常微分方程(first-order ordinary differential equation)。要解这个方程，就是在求 ${\bf x}(t)$ ，我们需要给定初始位置 ${\bf x}_0$ 和初始速度 ${\bf v}_0$ 。

### Euler's Method(欧拉法)
又被称为 Forward Euler, Explicit Euler.

欧拉法的思想非常简单，就是我在一个比较微小的时间间隔内，把速度看作不变的，再把速度乘以时间加到位置中。

$$\Large {\bf x}(t + \Delta t) = {\bf x}(t) + \Delta t \dot{\bf x}(t)$$

欧拉法有两个问题:

* 误差(Error): $\Delta t$ 被称为步长。显然，步长越小，结果就越接近解析解。
* 不稳定性(Instability): 正常来说，步长越小，结果就越接近解析解。但是这并不是对所有的情况成立。有一些比较刁专的，计算结果会严重偏离解析解。因为在特定情况下，出现的误差会被严重放大。

其实只要是数值计算的方法都有这两个问题。误差的问题是比较好解决的，只要减小步长，就一定能接近解析解的。比较大的问题是不稳定性的问题。于是人们的欧拉法进行了各种改进，比如中点法、动态步长法、隐式欧拉法等等。

### 如何评估数值计算方法的好坏
首先介绍一些概念。

* Local Truncation Error(局部截断误差): 每一步的误差。
* Total Accumulated Error(累计误差): 所有步加起来的误差。

以上是绝对的数值，其实并不能反映什么问题。

我们知道，步长越小，误差就越小。那么误差是如何随着步长的减小而减小? 一次方的关系还是二次方的关系? 所以就类似于时间复杂度一样，提出了一个阶数。

### Midpoint Method(中点法)
首先使用欧拉法，计算出下一个步长所在的 ${\bf x}_a$ ，然后取 ${\bf x}(t)$ 与 ${\bf x}_a$ 的中点 ${\bf x}_b$ ，然后根据 ${\bf x}_b$ 取速度 ${\bf v}_b$ ，然后以 ${\bf x}(t)$ 为基础，根据 ${\bf v}_b$ ，计算出 ${\bf x}(t + \Delta t)$ .

$$\Large {\bf x}_{mid} = {\bf x}(t) + \frac{\Delta t}{2} {\bf v}({\bf x}(t), t)$$

$$\Large {\bf x}(t + \Delta t) = {\bf x}(t) + \Delta t {\bf v}({\bf x}_{mid}, t)$$

### Adaptive Step Size(动态步长法)
首先使用步长为 $\Delta t$ 的欧拉法，得到结果 ${\bf x}_a$ ；然后使用步长为 $\frac{\Delta t}{2}$ 的欧拉法，得到结果 ${\bf x}_b$ 。如果 ${\bf x}_a$ 和 ${\bf x}_b$ 足够接近(取决于你)，说明 $\Delta t$ 已经足够小了，不用再小了；否则就继续对半分。

### 隐式欧拉法
也被称为 Backward Euler.

其核心思想在于，显式欧拉法使用的是当前时间的导数，而隐式欧拉法使用的是下一个步长的导数。

$$\Large {\bf x}(t + \Delta t) = {\bf x}(t) + \Delta t \dot{\bf x}(t + \Delta t)$$

不过呢，其实下一个步长的导数，有时不好求。该方法稳定性较好。

隐式欧拉法是一阶的，局部截断误差 $O(h^2)$ ，全局截断误差 $O(h)$ .

### Runge-Kutta Families(龙格-库塔系列方法)
擅长解常微分方程(ordinary differential equation, ODE), 特别是非线性的。最为常用的是四阶龙格-库塔法(RK-4)。其实这个方法的思想和欧拉法差不多，就是在计算下一步长的结果那里，复杂了亿点点。

条件:

$$\Large \frac{{\rm d}y}{{\rm d}t} = f(t, y), \quad y(t_0) = y_0$$

RK4 解法:

$$\Large y(t + h) = y(t) + \frac{1}{6} h(k_1 + 2k_2 + 2k_3 + k_4)$$

$$\Large k_1 = f(t, y(t))$$

$$\Large k_2 = f(t + \frac{h}{2}, y(t) + \frac{hk_1}{2})$$

$$\Large k_3 = f(t + \frac{h}{2}, y(t) + \frac{hk_2}{2})$$

$$\Large k_4 = f(t + h, y(t) + hk_3)$$

### Position-Based / Verlet Integration
这是不基于物理原理的方法，而是通过定义一些简单规则，直接改变物体的位置。好处是简单，容易计算。坏处是不够真实。

## Rigid Body Simulation(刚体的仿真)
刚体，故名思意，就是不会发生形变的物体。也就是说，刚体的所有点，的运动状态是完全一致的。所以就和质点差不多，不过多了几种属性。

$$\Large \frac{{\rm d}}{{\rm d}t} \begin{pmatrix} {\bf X} \\\\ \theta \\\\ \dot{\bf X} \\\\ \omega \end{pmatrix} = \begin{pmatrix} \dot{\bf X} \\\\ \omega \\\\ \frac{\bf F}{m} \\\\ \frac{\bf \Gamma}{I} \end{pmatrix}$$

> 其中， $\bf \Gamma$ 是力矩， $I$ 是转动惯量。

## Fluid Simulation(流体的仿真)
### 使用 Position-Based 方法模拟
首先是几个基本假设:

* 水由许多刚体小球组成。
* 水不可压缩或者拉伸。也就是说，水的密度不会改变，且处处相等。

当小球动了起来，那么小球的分布就不均匀了，对应到水的密度就会变了。但是我们的基本假设说，水的密度不变，所以要做修正。

首先，我们考虑求密度。非常简单，要求空间中某一点的密度，只需要计算该点周围一圈的小球数量。也就是说，空间中某一点的密度和全部小球的位置有关。也就是说，空间中某一点的密度是小球的位置的函数。于是我们把某一点的密度对小球的位置求导，就得到了梯度。为了使密度恢复正常，我们只需要沿着梯度改变小球的位置，也就是所谓的梯度下降法。

这就是 Position-Based 方法，根据一些规则来直接更改小球的位置。

### Eulerian and Lagrangian
其实模拟流体的方法分为两大体系，欧拉法和拉格朗日法。

拉格朗日法，又叫质点法，基本思路就是上面提到的那个 Position-Based 方法，把流体看成多个粒子的集合。

欧拉法，又叫网格法，基本思路就是把空间划分为若干部分，计算每个部分在某时刻是什么状态。

现在还有一种把两种方法结合在一起的方法: Material Point Method(物质点方法, MPM).先通过网格法计算局部空间属性，再通过局部空间的属性更新粒子的属性。
