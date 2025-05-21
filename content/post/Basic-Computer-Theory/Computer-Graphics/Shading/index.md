+++
date = '2025-05-20T16:49:18+08:00'
draft = false
title = '计算机图形学-着色'
categories = ['Sub Sections']
math = true
+++

在经过光栅化后，我们可以看到图像了。但是这是纯色的图像。所以接下来要引入明暗和颜色的变化，设就是着色。

着色(Shading)是对不同的材质物体应用不同颜色的过程。

材质的不同，和光线相互作用的效果就不同。

## 三角形内的插值
我们知道三角形三个顶点的属性值，现在我们希望在三角形内的任意一点，都有这个属性值的平滑过渡。

### 重心坐标
有 $\triangle{ABC}$ ，点 $X$ ，我们可以使用三角形三个顶点为基向量，表示点 $X$ 。
$$\vec{X} = \alpha\vec{A} + \beta\vec{B} + \gamma\vec{C}$$

$$\alpha + \beta + \gamma = 1 \iff \text{点} X \text{在平面} ABC \text{上}$$

$$\alpha, \beta, \gamma > 0 \iff \text{点} X \text{在} \triangle{ABC} \text{内}$$

$$\alpha = \frac{S_{\triangle{XBC}}}{S_{\triangle{ABC}}}$$

$$\beta = \frac{S_{\triangle{AXC}}}{S_{\triangle{ABC}}}$$

$$\gamma = \frac{S_{\triangle{ABX}}}{S_{\triangle{ABC}}}$$

$$S_{\triangle{ABC}} = \frac{\mid \vec{A} \times \vec{B} \mid}{2} = \frac{\mid \vec{B} \times \vec{C} \mid}{2} = \frac{\mid \vec{C} \times \vec{A} \mid}{2}$$

由于三角形经过投影后，其重心会发生变化，所以尽量在三维空间中做。

### 插值
对于 $\triangle{ABC}$ 内的点 $X$ ，有 $\vec{X} = \alpha\vec{A} + \beta\vec{B} + \gamma\vec{C}$ 。点 $A, B, C, X$ 有属性值 $V_A, V_B, V_C, V_X$ 。

$$V_X = \alpha V_A + \beta V_B + \gamma V_C$$

## Blinn-Phong Reflectance Model
一些概念:
* 一个物体，有三个部分，高光区，漫反射区，环境光区。
* 我们所关注的那个像素，称为"Shading Point"，法线 $\vec{n}$ ，观察方向 $\vec{v}$ ，光照方向 $\vec{l}$ ，它们都是单位向量。
* Shading is local: 着色的局部性，不考虑光线是否被其他物体挡住。

根据 Lambert's cosine law ，物体表面接收光的能量，正比于法线与光照方向角度的余弦。

显然，在某一点上，该点的来自点光源的能量密度反比于该点到点光源的距离的平方。

### 漫反射项(Diffse Term)
所以我们得到某个点的漫反射(光均匀反射到各个方向)的亮度的计算公式:
$$L_d = k_d \frac{I}{r^2} \max(0, \vec{n} \cdot \vec{l})$$

为什么 $\max(0, \vec{n} \cdot \vec{l})$ ? 因为不反射从背后射来的光。

其中 $k_d$ 就表示了不同材质的性质。它可以是一个向量，表示 RGB 。

### 高光项(Specular Term)
当观察方向 $\vec{v}$ 和 光照方向 $\vec{l}$ 的镜面反射方向接近时，我们会看到高光。

也就是说，当观察方向 $\vec{v}$ 和 光照方向 $\vec{l}$ 的半程向量 $\vec{h}$ 与法线 $\vec{n}$ 方向接近时，我们会看到高光。

半程向量是指，位于两个向量夹角中间的单位向量。十分容易计算。
$$bisector(\vec{a}, \vec{b}) = \frac{\vec{a} + \vec{b}}{\mid \vec{a} + \vec{b} \mid}$$

容易得到某个点的高光亮度的计算公式:
$$\begin{align*} L_s &= k_s \frac{I}{r^2} (\max(0, \cos{\alpha}))^p \\\\&= k_s \frac{I}{r^2} (\max(0, \vec{n} \cdot \vec{h}))^p \\\\&= k_s \frac{I}{r^2} (\max(0, \vec{n} \cdot (\frac{\vec{v} + \vec{l}}{\mid \vec{v} + \vec{l} \mid})))^p \end{align*}$$

其中，指数 $p$ 的作用是，当 $\cos{\alpha}$ 从 $1$ 下降的时候，快速地把值下降到 $0$ 。一般是 $[100,200)$ 。

### 环境光项(Ambient Term)
环境光，近似看作从所有方向，相等强度地照射。

$$L_a = k_a I_a$$

如果想要精确地计算环境光，需要开启光线追踪。

Blinn-Phong Reflectance Model 不考虑物体到观察点之间的光线能量损失。

### 加起来
最后把上述三个项加起来，得到:
$$L = L_d + L_s + L_a$$

## Shading Frequencies
### Flat Shading(着色每个三角型)
对每一个面，求法线(三角形的两个边叉积)，对该三角形的每一个点，使用那个法线进行 Shading 。即，对于三角形内部，亮度是不会变的。

### Gouraud Shading(着色每个顶点)
对于三角形，求三个顶点的法线，着色，于是得到三个顶点的颜色。然后通过[插值](#插值)的方法，求出三角形内所有点的颜色。

### Phong Shading(着色每个像素)
对于三角形，求三个顶点的法线。然后通过[插值](#插值)的方法，求出三角形内所有点的法线。然后着色。

在物体的面足够多的情况下， Flat Shading 也能有一个不错的效果。

### 计算顶点的法线
顶点的法线就是，该顶点相邻的所有面的法线的平均值。
$$\vec{n} = \frac{\sum_{i}\vec{n_i}}{\mid \sum_{i}\vec{n_i} \mid}$$

当然，可以根据三角形的面积，进行加权平均。这样更精确一点。
$$\vec{n} = \frac{\sum_{i}a_i\vec{n_i}}{\mid \sum_{i}a_i\vec{n_i} \mid}$$

那么如何得到顶点相邻的所有面? 以后再说。

## Graphics(Real-time Rendering) Pipeline
给定三维模型，给定光照条件，使用着色模型，得到渲染结果。这个过程就是图形管线。

* 三维空间中的一些点
    * 通过 Vertex Processing ，视图变换
* 在 $[-1, 1]^3$ 的一些点
    * 通过 Triangle Processing ，划定三角形
* 在 $[-1, 1]^3$ 的三角形
    * 通过 Rasterization ，光栅化
* Fragment (一个或多个 Fragment 对应了一个像素)
    * 通过 Shanding ，着色
* 像素图像

这些操作是由硬件完成的，比如显卡会做这些事情。有些操作时可编程的，比如 Shading 。控制 Shading 的代码叫 Shader 。

### Shader(着色器)
Shader 是自动对所有顶点或者像素操作的，所以不需要写 for 循环。如果一个 Shader 是对顶点操作的，称为 Vertex Shader ；如果一个 Shader 是对像素操作的，称为 Fragment Shader 或 Pixel Shader 。

现代技术不断发展， Shader 也还可以做更多的操作，而不仅仅用于 Shading 方面了。

## Texture Mapping(纹理映射)
纹理，是用来定义任意一个点的属性。比如说，现在要渲染一个球，我们可以使用 Shading Model 来计算光照。但是我们还需要定义球面上有什么图案。

首先，三维物体的表面是曲面。可以找到一个映射关系，把曲面映射到平面上。可以想象为，把一张有弹性的图，贴到物体的表面。

首先我们得知，三维物体表面有一些点，这些点对应了贴图平面的哪个点，是已经知道了。然后再把这些点连成三角形，再通过三角形内部的颜色，就可以渲染三维物体的表面纹理了。

那么我们如何知道三维物体表面要划分为哪些点，对应贴图平面哪个点呢? 别问，问就是美工干的。当然，也可以通过自动化来划分点和三角形。这个非常困难，是当前重大研究方向，叫参数化(Parameterization)。

在贴图平面上，坐标一般为 $(u,v) \in [0, 1]^2$ 。

A pixel on a texture -- a texel(纹理元素、纹素)

### Texture Magnification(1)
比如，一个纹理是 256x256 ，渲染分辨率是 4K ，那么多个像素对应一个 texel 。

最简单的办法是，什么也不做，像素颜色就直接取离它最近的 texel 。但是这样会出现明显的锯齿。

#### Bilinear(双线性插值)
首先介绍 Linear interpolation(线性插值)，定义符号: $\text{lerp}$ :
$$\text{lerp}(x, v_0, v_1) = v0 + x(v_1 - v_0), \quad x \in [0, 1],\, v_1 > v_0$$

取像素周围四个 texel ， $u_{00}, u_{01}, u_{10}, u_{11}$ 。像素与 $u_{00}$ 的水平、垂直距离分别为 $s, t$ 。

$$u_0 = \text{lerp}(s, u_{00}, u_{10})$$

$$u_1 = \text{lerp}(s, u_{01}, u_{11})$$

$$v = \text{lerp}(t, u_0, u_1)$$

上面是先水平地线性插值，再垂直地线性插值。这顺序对调也可以。

#### Bicubic(双三次插值)
这个算法考虑了像素周围16个 texel ，效果比 Bilinear 好，但是计算量加大。

这个算法比较复杂，公式略。

### Texture Magnification(2)
这次的问题就反过来了，一个像素对应多个 texel 。如果直接采样，会出现摩尔纹。这个问题的场景是，我的摄像机可以离一个纹理近(锯齿)，也可以离纹理远(摩尔纹)；甚至画面中同时出现了远近不同的物体。这样，就会同时出现像素和 texel 一对多和多对一的问题。

出现这种问题的根本原因，之前就已经说过，是采样频率和原始信号频率相差太远。我们当然可以使用之前讲过的思想，使用类似 MSAA 的方法，提高采样频率，可以有效去除摩尔纹和锯齿。但是这样计算量太大了。

这里介绍两个算法上的概念， Point Query and Range Query 。 Point Query 指的是类似上面提到的 Bilinear 算法； Range Query 指的是，类似"某个像素，覆盖了多个 texel ，那么如何快速地把这些 texel 的平均值(或者最大值、最小值等等)求出"的算法。

#### Mipmap
快速的，近似的，正方形的 Range Query 。首先，我们得到一个纹理，假设分辨率是 128x128 。把原始纹理图设为 Level0 。然后根据原始纹理图，生成分辨率为四分之一的纹理图(64x64)(求平均)，设为 Level1  。以此类推，直到分辨率为 1x1 。

取目标像素 $x$ 的上面 $u$ 和右边 $r$ 的顶点，和目标像素一起，求它们在 texel 的映射点 $x_t, u_t, r_t$ 。于是，我们可以得到在 texel 空间中，两点的最大距离 $L = \max(\mid u_t - x_t \mid, \mid r_t - x_t \mid)$ ，再据此求得 Level : $D = \log_{2}{L}$ 。最后，直接再 Level $D$ 中，取 $x_t$ 的值。

但是啊! 我们求得的一系列 texel 图的 Level 是离散的，而 $D = \log_{2}{L}$ 的结果是连续的，要怎么做呢?

我们在 Level $\lfloor D \rfloor$ 做一次 [Bilinear](#bilinear双线性插值) ，在 $\lceil D \rceil$ 做一次 [Bilinear](#bilinear双线性插值) ，最后把上面两个结果做一次 [Linear Interpolation](#bilinear双线性插值) 。这就是 Trilinear ，它可以得到非常连续的结果。

但是， Trilinear 有一个缺点，就是远处的纹理会过分模糊(Overblur)。因为，一个像素映射到纹理图中，可能是一个长条的区域。对于一个长条的区域，使用长边的长度来取值，当然会严重失真了。

#### Ripmap
有一个办法可以部分解决这个问题，各向异性过滤(Anisotropic Filtering)。

在 Mipmap 中，我们水平和竖直方向同时压缩二分之一。在 Ripmap 中，我们对纹理水平和竖直方向同时压缩二分之一、水平方向压缩二分之一、竖直方向压缩二分之一。这样，一个像素映射到纹理图中时，如果是一个水平或竖直的长条区域，就不会严重失真了。

对于其他更不规则的形状，还有一种 EWA Filtering 的方法。

### Environment Map(环境光映射)
纹理还可以描述环境光。使用一张纹理图来描述 360 度的环境光。 Environment Map 假设光线是从无限远处发出，所以贴图只记录光线的方向。

### Spherical Map
还可以把环境光记录在一个球上，然后记录这个球的纹理，以类似世界地图的形式展开。但是这样做，在某些地方会严重失真(指拉伸得十分严重)。

### Cube Map
把记录环境光的球，先映射到正方体之中，再把正方体的六个面展开，这样就没那么失真了。

### Bump Mapping(凹凸贴图)
纹理不仅可以用于保存物体表面的颜色信息，还可以保存物体表面的其他信息，比如高度。

通过凹凸贴图，改变高度，进而改变法线，进而改变 Shading 的结果，从而在简单物体表面产生粗糙感。

新的法线的计算方法: 先计算切线，再计算法线。

以下是二维的情形:

$$dp = c \cdot (h(p + 1) - h(p))$$

于是切线向量是 $(1, dp)$ ，把它旋转 90 度得到:

$$\vec{n}(p) = \frac{(-dp, 1)}{\mid (-dp, 1) \mid}$$

以下是三维的情形:

$$\frac{\partial p}{\partial u} = c_u \cdot (h(u + 1, v) - h(u, v))$$

$$\frac{\partial p}{\partial v} = c_v \cdot (h(u, v + 1) - h(u, v))$$

$$\vec{n}(u, v) = \frac{(-\frac{\partial p}{\partial u}, -\frac{\partial p}{\partial v}, 1)}{\mid (-\frac{\partial p}{\partial u}, -\frac{\partial p}{\partial v}, 1) \mid}$$

注意，以上的计算，都有一个前提，就是物体的表面的法线，是 $(0, 1)$ 或 $(0 , 0, 1)$ ，这称为"切线空间"。我们通过以上的方法算出来后，还需要把结果转换为"模型空间"的坐标。见[切线空间与 TBN 矩阵](#切线空间与-tbn-矩阵)。

### Normal Mapping(法线贴图)
法线贴图直接存储表面法线方向。分为模型空间(object-space)的法线贴图和切线空间(tangent-space)的法线贴图两种。

存储在模型空间中的法线信息是**绝对法线信息**，当我们尝试将这样的法线信息应用于UV动画或其它模型上时就会出错。因此在实际中我们常常会存储**相对法线信息**，即各个点在各自切线空间中的法线扰动方向。

### 切线空间与 TBN 矩阵
切线空间由三个互相正交的基向量 $\vec{t}, \vec{b}, \vec{n}$ 组成。切线空间定义在模型的每一个顶点上，切线空间的原点就是顶点本身; $z$ 轴是顶点的法线方向，也称为 $n$ 轴(normal); $x$ 轴是顶点的切线方向，也称为 $t$ 轴(tangent); $y$ 轴由法线和切线叉乘得到，也叫副切线，称为 $b$ 轴(bitangent)。

$n$ 轴可由顶点法线直接确定，而切线有无数条，如何确定 $t$ 轴和 $b$ 轴方向呢? 法线贴图的切线 $\vec{t}$ 与副切线 $\vec{b}$ 定义一般与纹理坐标的 $u, v$ 对齐，因此也利用这个特性来确定每个表面顶点切线空间的 $t, b$ 轴方向。

假设某点所在的三角形的三个顶点 $P_1, P_2, P_3$ 的模型空间坐标和贴图坐标已知。

取三角形的两条边，得到贴图坐标 $(\Delta U_1, \Delta V_1)$ 和 $(\Delta U_2, \Delta V_2)$ :

$$\vec{E_1} = P_2 - P_1 = \Delta U_1 \vec{T} + \Delta V_1 \vec{B}$$

$$\vec{E_2} = P_3 - P_2 = \Delta U_2 \vec{T} + \Delta V_2 \vec{B}$$

将向量展开为标量:

$$(E_{1x}, E_{1y}, E_{1z}) = \Delta U_1 (T_x, T_y, T_z) + \Delta V_1 (B_x, B_y, B_z)$$

$$(E_{2x}, E_{2y}, E_{2z}) = \Delta U_2 (T_x, T_y, T_z) + \Delta V_2 (B_x, B_y, B_z)$$

写成矩阵形式:

$$\begin{bmatrix} E_{1x} & E_{1y} & E_{1z} \\\\ E_{2x} & E_{2y} & E_{2z} \end{bmatrix} = \begin{bmatrix} \Delta U_1 & \Delta V_1 \\\\ \Delta U_2 & \Delta V_2 \end{bmatrix} \begin{bmatrix} T_x & T_y & T_z \\\\ B_x & B_y & B_z \end{bmatrix}$$

两边同乘逆矩阵:

$$\begin{bmatrix} T_x & T_y & T_z \\\\ B_x & B_y & B_z \end{bmatrix} = \begin{bmatrix} \Delta U_1 & \Delta V_1 \\\\ \Delta U_2 & \Delta V_2 \end{bmatrix}^{-1} \begin{bmatrix} E_{1x} & E_{1y} & E_{1z} \\\\ E_{2x} & E_{2y} & E_{2z} \end{bmatrix}$$

使用伴随矩阵法求解逆矩阵: $A^{-1} = \frac{A^*}{\text{det}(A)}$ :

$$\begin{bmatrix} T_x & T_y & T_z \\\\ B_x & B_y & B_z \end{bmatrix} = \frac{1}{\Delta U_1 \Delta V_2 - \Delta U_2 \Delta V_1} \begin{bmatrix} \Delta V_2 & - \Delta U_1 \\\\ - \Delta U_2 & \Delta U_1 \end{bmatrix} \begin{bmatrix} E_{1x} & E_{1y} & E_{1z} \\\\ E_{2x} & E_{2y} & E_{2z} \end{bmatrix}$$

至此，我们可以根据三角形三个顶点的模型坐标和贴图坐标来得到对齐 $u, v$ 轴的 $t,b$ 轴在模型坐标系中的方向了。

若要从模型空间向切线空间转换，根据基向量变换，只需左乘如下矩阵:

$$\begin{bmatrix} x_{tan} \\\\ y_{tan} \\\\ z_{tan} \end{bmatrix} = \begin{bmatrix} T_x & T_y & T_z \\\\ B_x & B_y & B_z \\\\ N_x & N_y & N_z \end{bmatrix} \begin{bmatrix} x_{obj} \\\\ y_{obj} \\\\ z_{obj} \end{bmatrix}$$

若要从切线空间向模型空间转换，已知该矩阵定义了一个正交的坐标空间，那么该矩阵为正交矩阵，正交矩阵的逆等于转置，这就得到了 TBN 矩阵:

$$\begin{bmatrix} x_{obj} \\\\ y_{obj} \\\\ z_{obj} \end{bmatrix} = \begin{bmatrix} T_x & B_x & N_x \\\\ T_y & B_y & N_y \\\\ T_z & B_z & N_z \end{bmatrix} \begin{bmatrix} x_{tan} \\\\ y_{tan} \\\\ z_{tan} \end{bmatrix}$$

应用上述方法虽然 $t$ 轴和 $b$ 轴可以与纹理坐标 $u$ 轴 $v$ 轴对齐，但 $u, v$ 不一定是相互垂直的，这就使得求得的 $t$ 轴不一定正确。所以有时候在求得 $t$ 轴后还需要做一步操作，施密特正交化。施密特正交化的几何意义就是将"任意坐标系"转换成"直角坐标系"。 $\vec{N}$ 已知，求得 $\vec{T}$ 后对其进行施密特正交化，再叉乘得到 $\vec{B}$ ，即可构造出正确的切线空间。

### Displacement Mapping(位移贴图)
凹凸贴图并没有实际改变模型表面，只是提供了一个视觉效果。而位移贴图会实际改变模型的表面的顶点的位置，使其变得凹凸不平。比如一个凹凸不平的球，如果使用凹凸贴图，这个球的轮廓依然是光滑的圆形；而使用位移贴图，这个球的轮廓也会呈现凹凸不平的效果。

但是，如果模型的面不太细分的话，效果就不好了。所以要求比较精细的面，但这样会对性能提出较高的要求。在某些图形学库中，比如 DirectX 中，可以进行动态曲面细分(Dynamic Tessellation)，根据位移贴图的频率，来动态地细分曲面。

### 三维纹理与体渲染(Volume Rendering)
纹理还可以是三维的。即，定义一个三维空间，三维空间中的每一个点都有一个值。实际上，也可以不生成纹理图，只记录一个解析式就可以了。比如 Perlin noice ，是一个三维噪声函数。我们可以使用 Perlin noice 来生成大理石纹理、山峰的起伏之类的。

一些物体的内部也有东西，可以理解为"透视"吧。

### 存储计算结果
比如可以把一些算起来比较麻烦的东西，提前算好，记录在纹理中。到时候直接应用纹理就可以了。比如环境光遮蔽。之后会讲。
