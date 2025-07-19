+++
date = '2025-05-20T16:46:06+08:00'
draft = false
title = '计算机图形学-变换'
categories = ['Sub Sections']
math = true
+++

变换，Transformation。

## Linear Transformation (线性变换)
观察 $ A \vec{c} = \vec{b} $ ， $ \vec{c} $ 乘了 $ A $ 后，变成了 $ \vec{b} $ 。矩阵 $ A $ 就代表了一种线性变换。

比如，二维旋转矩阵:
$$ R_\theta = \begin{bmatrix} \cos{\theta} & -\sin{\theta} \\\\ \sin{\theta} & \cos{\theta} \end{bmatrix} \tag{2-1} $$
经过这个矩阵作用的二维向量，会绕原点逆时针旋转 $ \theta $ 角度。

## Homogeneous Coordinates (齐次坐标)
一般，我们对一个图形的操作有旋转，缩放，翻转，平移等等。我们来尝试表达一下平移操作:
$$ \vec{b} = \vec{c} + \vec{t} \tag{2-2} $$
显然，平移这个操作非常特殊，它不能通过矩阵与向量相乘来表示。我们可以通过齐次坐标来统一平移操作和其他操作。我们对向量扩展一个维度:
$$ \begin{bmatrix} b_x \\\\ b_y \\\\ 1 \end{bmatrix} = \begin{bmatrix} 1 & 0 & t_x \\\\ 0 & 1 & t_y \\\\ 0 & 0 & 1 \end{bmatrix} \begin{bmatrix} c_x \\\\ c_y \\\\ 1 \end{bmatrix} = \begin{bmatrix} c_x + t_x \\\\ c_y + t_y \\\\ 1 \end{bmatrix} \tag{2-3} $$
观察向量的 $x,y$ 分量，发现这就能用矩阵与向量相乘来表示平移操作了。这个变换叫仿射变换。

向量的 $z$ 分量也有意义。

二维的点: $ (x, y, 1)^T $

二维向量: $ (x, y, 0)^T $

向量 + 向量 = 向量

点 - 点 = 向量

点 + 向量 = 点

点 + 点: 那么向量的 $z$ 分量变成了 $2$ 。于是补充定义:

$ \begin{bmatrix} c_x \\\\ c_y \\\\ w \end{bmatrix} $ 是点 $ \begin{bmatrix} c_x / w \\\\ c_y / w \\\\ 1 \end{bmatrix} , w \neq 0 $

## 线性变换 + 平移
$$ \vec{b} = A\vec{c} + \vec{t} \tag{2-4} $$
$$ \begin{bmatrix} B \\\\ 1 \end{bmatrix} = \begin{bmatrix} A & T \\\\ O & 1 \end{bmatrix} \begin{bmatrix} C \\\\ 1 \end{bmatrix} \tag{2-5} $$
其中， $A$ 是矩阵； $T$ 是平移向量，即式子 $ (2-4) $ 中的 $ \vec{t} $ ； $C$ 是变换前的向量，即式子 $ (2-4) $ 中的 $ \vec{c} $ ； $B$ 是变换后的向量，即式子 $ (2-4) $ 中的 $ \vec{b} $ ； $O$ 是零矩阵。

## 关于二维旋转矩阵
绕原点逆时针旋转 $ \theta $ 角度:
$$ R_\theta = \begin{bmatrix} \cos{\theta} & -\sin{\theta} \\\\ \sin{\theta} & \cos{\theta} \end{bmatrix} \tag{2-6} $$
绕原点顺时针旋转 $ \theta $ 角度，也就是绕原点逆时针旋转 $ -\theta $ 角度，把 $ -\theta $ 带入 $(2-6)$ 式中:
$$ R_{-\theta} = \begin{bmatrix} \cos{\theta} & \sin{\theta} \\\\ -\sin{\theta} & \cos{\theta} \end{bmatrix} = R_\theta^T \tag{2-7} $$
从定义上来看，
$$ R_{-\theta} = R_\theta^{-1} \tag{2-8} $$
结合 $(2-7)$ 式和 $(2-8)$ 式，我们得到:
$$ R_\theta^{-1} = R_\theta^T \tag{2-9} $$
数学上，当一个矩阵的逆等于它的转置时，称这个矩阵为正交矩阵。

## 三维旋转矩阵
在此之前，先了解一下 [角速度 - 维基百科，自由的百科全书](https://zh.wikipedia.org/wiki/%E8%A7%92%E9%80%9F%E5%BA%A6) 的三维坐标系部分。

### 围绕三个坐标轴的旋转 (右手坐标系)
围绕 $x$ 轴的正方向旋转 $\theta$ 角度
$$ R_x(\theta) = \begin{bmatrix} 1 & 0 & 0 \\\\ 0 & \cos{\theta} & -\sin{\theta} \\\\ 0 & \sin{\theta} & \cos{\theta} \end{bmatrix} $$

围绕 $y$ 轴的正方向旋转 $\theta$ 角度
$$ R_y(\theta) = \begin{bmatrix} \cos{\theta} & 0 & \sin{\theta} \\\\ 0 & 1 & 0 \\\\ -\sin{\theta} & 0 & \cos{\theta} \end{bmatrix} $$

围绕 $z$ 轴的正方向旋转 $\theta$ 角度
$$ R_z(\theta) = \begin{bmatrix} \cos{\theta} & -\sin{\theta} & 0 \\\\ \sin{\theta} & \cos{\theta} & 0 \\\\ 0 & 0 & 1 \end{bmatrix} $$

其中的规律:

首先观察三个坐标轴:
$$ \vec{x} = \vec{y} \times \vec{z} \\\\ \vec{y} = \vec{z} \times \vec{x} \\\\ \vec{z} = \vec{x} \times \vec{y} $$
这就是轮换对称性。旋转矩阵的一行就代表了对应的坐标轴。观察上式的顺序，就不奇怪为什么 $R_y(\theta)$ 是反的。

### 一般旋转
Rodrigues's Formula (罗德里格斯公式)
$$ \textbf{R}(\vec{n}, \theta) = \cos{\theta}\textbf{I} + (1 - \cos{\theta}) \vec{n} \vec{n}^T + \sin{\theta} \underbrace{\begin{bmatrix} 0 & -n_z & n_y \\\\ n_z & 0 & -n_x \\\\ -n_y & n_x & 0 \end{bmatrix}}_{\textbf{N}} $$
其中， $\vec{n}$ 是旋转轴单位向量； $\textbf{N}$ 是向量 $\vec{n}$ 的反对称矩阵。

### 使用四元数表示一般旋转
了解旋转与万向锁：

[【【深度解读】引发“万向锁”的罪魁祸首？我在北极点找到了答案！——虚幻引擎UE免费教程】](https://www.bilibili.com/video/BV1nKKzz5EU7/)

请先看 3Blue1Brown 的视频

[【四元数的可视化】](https://www.bilibili.com/video/BV1SW411y7W1/)

[【四元数和三维转动，可互动的探索式视频（请看链接）】](https://www.bilibili.com/video/BV1Lt411U7og/)

假设要把向量 $\vec{m}$ 绕单位向量 $\vec{n}$ 旋转 $\theta$ 角度。构造四元数 $q = \cos{\frac{\theta}{2}} + \sin{\frac{\theta}{2}} (n_x i + n_y j + n_z k) , p = m_x i + m_y j + m_z k$ ，那么
$$ p' = qpq^{-1} $$
其中， $q^{-1}$ 是四元数 $q$ 的共轭，即把虚部各分量取相反数。

#### 为什么是 $\frac{\theta}{2}$ ?
如果看了 3Blue1Brown 的视频，你就会知道，四元数相乘的几何意义是，同时“沿着”旋转向量和“绕着”旋转向量转动。而我们需要的三维空间转动，只需要“绕着”旋转向量转动，不需要“沿着”旋转向量转动。

所以 $qp$ ，就是使用四维空间的右手定则，同时“沿着” $q$ 和“绕着” $q$ 转动 $\frac{\theta}{2}$ 。

$q^{-1}$ ，在三维空间的投影中，是和 $q$ 大小相等，方向相反的向量。而 $(qp)q^{-1}$ ，就是使用四维空间的左手定则，同时“沿着” $q^{-1}$ 和“绕着” $q^{-1}$ 转动 $\frac{\theta}{2}$ 。

于是，在三维空间的投影中，“沿着” $q$ 转动 $\frac{\theta}{2}$ 和“沿着” $q^{-1}$ 转动 $\frac{\theta}{2}$ 相抵消；“绕着” $q$ 转动 $\frac{\theta}{2}$ 和“绕着” $q^{-1}$ 转动 $\frac{\theta}{2}$ 相叠加。

这样就达成了“只需要‘绕着’旋转向量转动，不需要‘沿着’旋转向量转动”的目的了。

## View / Camera Transformation (视图变换)
视图变换其实就是拍照。

### 如何拍照？
1. 找地方放好物体。(Model Transformation)
2. 找地方放好照相机。(View Transformation)
3. 拍照。(Projection Transformation)

### 如何描述照相机位置
* 位置 $\vec{e}$
* Look-at / gaze direction 朝向 $\hat{g}$
* Up direction $\hat{t}$ (在朝向不变时，相机可以绕着 $\hat{g}$ 旋转，所以还需要一个向量来描述方向。它垂直于 $\hat{g}$)

### 照相机位置标准位置
根据相对运动原理，如果照相机和物体一起移动，那么拍出来的照片是相同的。

一般， $\vec{e} = \vec{0},\; \hat{g} = [0, 0, -1]^T,\; \hat{t} = [0, 1, 0]^T$

对于任意朝向的照相机，通过以下步骤转换到标准位置:
1. 把照相机平移到原点。
1. 把 $\hat{g}$ 旋转到 $[0, 0, -1]^T$ 。
1. 把 $\hat{t}$ 旋转到 $[0, 1, 0]^T$ 。
1. 把 $\hat{g} \times \hat{t}$ 旋转到 $[1, 0, 0]^T$ 。 

那么那个旋转矩阵 $\textbf{R}$ 如何求得？首先，这个矩阵同时满足上面的 2, 3, 4 要求，写成数学表达式:

$$\begin{bmatrix} 1 & 0 & 0 \\\\ 0 & 1 & 0 \\\\ 0 & 0 & 1 \end{bmatrix} = \textbf{R} \begin{bmatrix} \hat{g} \times \hat{t} & \hat{t} & -\hat{g} \end{bmatrix}$$
$$\textbf{R}^{-1} \begin{bmatrix} 1 & 0 & 0 \\\\ 0 & 1 & 0 \\\\ 0 & 0 & 1 \end{bmatrix} = \begin{bmatrix} \hat{g} \times \hat{t} & \hat{t} & -\hat{g} \end{bmatrix}$$
$$\textbf{R}^{-1} = \begin{bmatrix} \hat{g} \times \hat{t} & \hat{t} & -\hat{g} \end{bmatrix}$$

同时， $\textbf{R}$ 还是一个旋转矩阵，所以 $\textbf{R}$ 是正交矩阵，于是:
$$\textbf{R} = (\textbf{R}^{-1})^T = \begin{bmatrix} (\hat{g} \times \hat{t})^T \\\\ \hat{t}^T \\\\ (-\hat{g})^T \end{bmatrix}$$

### Projection Transformation
投影变换有两种，平行投影和透视投影。平行投影无论远近，大小一样；透视投影是近大远小。

注意，由于相机处于一般位置，所以近的点的坐标的 $z$ 坐标是大于远的点的坐标的 $z$ 坐标。

#### 平行投影
相机处于一般位置。我们只需要把各个点的坐标的 $z$ 分量去掉即可。

正式做法?:

首先选定一块区域进行投影，用六个数字来描述该区域（一个长方体）。

然后把这个长方体中心移动到原点，把这个长方体的大小拉伸到 $[-1, 1]^3$ 。

然后把各个点的坐标的 $z$ 分量去掉。

#### 透视投影
相机处于一般位置。

划定一个梯形体区域。近的，小的平面的 $z$ 分量为 $n$ ；远的，大的平面的 $z$ 分量为 $f$ 。

具体来说，这个梯形体区域是视锥的一部分。划定一个视锥，需要宽高比、垂直可视角度、水平可视角度这三个要素的其中两个即可。

我们要把大的平面收缩为小的平面， $z$ 分量不变，然后再使用平行投影。

使用相似三角形的原理，我们得到:
$$\textbf{P} \begin{bmatrix} x \\\\ y \\\\ z \\\\ 1 \end{bmatrix} = \begin{bmatrix} \frac{nx}{z} \\\\ \frac{ny}{z} \\\\ ? \\\\ 1 \end{bmatrix} = \begin{bmatrix} nx \\\\ ny \\\\ ? \\\\ z \end{bmatrix}$$
$$\textbf{P} = \begin{bmatrix} n & 0 & 0 & 0 \\\\ 0 & n & 0 & 0 \\\\ ? & ? & ? & ? \\\\ 0 & 0 & 1 & 0 \end{bmatrix}$$

那么怎么得到 $?$ 是什么呢。注意 $z$ 分量不变。把 $n$ 代入 $z$ ，所以:
$$\textbf{P} \begin{bmatrix} x \\\\ y \\\\ n \\\\ 1 \end{bmatrix} = \begin{bmatrix} x \\\\ y \\\\ n \\\\ 1 \end{bmatrix} = \begin{bmatrix} nx \\\\ ny \\\\ n^2 \\\\ n \end{bmatrix}$$
$$\begin{bmatrix} 0 & 0 & a & b \end{bmatrix} \begin{bmatrix} x \\\\ y \\\\ n \\\\ 1 \end{bmatrix} = n^2$$
把 $f$ 代入 $z$ ，所以:
$$\begin{bmatrix} 0 & 0 & a & b \end{bmatrix} \begin{bmatrix} x \\\\ y \\\\ f \\\\ 1 \end{bmatrix} = f^2$$
$$
\left\{ 
    \begin{array}{c}
        an + b &= n^2 \\\\ 
        af + b &= f^2
    \end{array}
\right.
$$
解得:
$$
\left\{ 
    \begin{array}{c}
        a &= n + f \\\\ 
        b &= -nf
    \end{array}
\right.
$$
所以:
$$\textbf{P} = \begin{bmatrix} n & 0 & 0 & 0 \\\\ 0 & n & 0 & 0 \\\\ 0 & 0 & n + f & -nf \\\\ 0 & 0 & 1 & 0 \end{bmatrix}$$

如果某个点的 $z = \frac{n + f}{2}$ ，那么在变换之后，它的 $z$ :
$$z' = \begin{bmatrix} 0 & 0 & n + f & -nf \end{bmatrix} \begin{bmatrix} x \\\\ y \\\\ \frac{n + f}{2} \\\\ 1 \end{bmatrix} = \frac{(n + f)^2}{2} - nf$$
$$\frac{(n + f)^2}{2} - nf = \frac{1}{2}((n+f)^2 - 2nf) = \frac{1}{2}(n^2 + f^2)$$

这是齐次坐标，第四维度是 $n$ ，所以:
$$z'_2 = \frac{z'}{n} = \frac{n}{2} + \frac{f^2}{2n} = \frac{n}{2} + \frac{f}{2} \cdot \frac{f}{n}$$

因为照相机处于一般位置， $0 > n > f$ ，所以 $\frac{f}{n} > 1$ ，所以 $z'_2 < z$ 。

结论: 经过变换后，中点离照相机更远了。

### 视图变换之后
经过视图变换之后，所有的东西都处于 $[-1, 1]^3$ 之中。

在平行投影中，显然物体经过了拉抻，失真了；而且视图变换并没有处理物体遮挡以及物体可见性。后续还会有步骤继续处理 (光栅化)。