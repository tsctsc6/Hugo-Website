+++
date = '2025-05-20T16:53:08+08:00'
draft = false
title = '计算机图形学-辐射度量学'
categories = ['Sub Sections']
math = true
+++

辐射度量学，Radiometry。

## Why we need radiometry
在之前的 Blinn-Phong Reflectance Model 中，多少也涉及到了光照强度的计算，但这是简化的计算方式。如果我们需要更真实的渲染结果，就需要学习辐射度量学了。辐射度量学的主要内容是准确定义光照。

## 光的属性
### Radiant Energy
Radiant Energy 就是电磁辐射的能量，SI导出单位是焦耳(J, Joule)，一般用字母 Q 表示。

### Radiant Flux (Power)
Radiant Flux 是指单位时间的能量，又叫功率(Power)，SI导出单位是瓦(W, Watt)。也有另一种单位，流明(lm, lumen)，指光源有多亮。好吧，它们之间的区别只是换了个名字罢了。

$$\Phi \equiv \frac{{\rm d}Q}{{\rm d}t}$$

还可以从另一个角度来理解 Radiant Flux ，单位时间内通过感光平面的能量。

### Radiant intensity(辐射强度，光强)
定义: 单位立体角的功率。

$$I(\omega) \equiv \frac{{\rm d}\Phi}{{\rm d}\omega}$$

单位:

$$\rm \left [\frac{W}{sr}\right ]\left [\frac{lm}{sr} = cd = candela\right ]$$

那么什么是立体角和单位立体角呢?

首先来看看平面的角(angle)，它的定义是，弧长除以半径，单位是弧度(radians)。

$$\theta = \frac{l}{r}$$

圆形有 $2 \pi$ 弧度。

立体的角(solid angle)，它的定义是，面积除以半径的平方，单位是球面度(steradians)。

$$\omega = \frac{A}{r^2}$$

球面有 $4 \pi$ 球面度。

单位立体角: 就是 ${\rm d}\omega$ ，指的是，一个立体角，移动一点点，所造成的立体角差值。假设在球面上，使用球极坐标系，那么:

$${\rm d}\omega = \frac{{\rm d}A}{r^2} = \frac{(r \, {\rm d}\theta)(r \, \sin{\theta} \, {\rm d}\phi)}{r^2} = \sin{\theta} \, {\rm d}\theta \, {\rm d}\phi$$

对球面 $S$ 进行积分，可以计算出球面的球面度是 $4 \pi$ :

$$\omega = \iint_S{\rm d} \omega = \int^{2 \pi}_{0} \int^{\pi}_{0} \sin{\theta} \, {\rm d}\theta \, {\rm d}\phi = 4 \pi$$

进一步地，可以使用二重积分计算任意形状的球面度。

在辐射度量学中，使用 $\omega$ 来表示三维空间的方向。

Radiant intensity(辐射强度，光强)，就是指各个方向上的功率(亮度) 。假设某球形灯泡的流明是 $\Phi$ ，它往各个方向均匀地发出光线，光强是常数 $I$ ，可以得出以下等式:

$$\Phi = \iint_S I  \, {\rm d}\omega = 4 \pi I$$

$$I = \frac{\Phi}{4 \pi}$$

### Irradiance(辐照度)
指的是在一个表面上，单位面积上发射、反射、透射或接收的功率。

$$E({\bf x}) \equiv \frac{{\rm d} \Phi({\bf x})}{{\rm d}A}$$

单位:

$$\rm \left [\frac{W}{m^2}\right ]\left [\frac{lm}{m^2} = lux\right ]$$

如果光线不垂直于表面，需要投影到垂直于表面的方向。(向量点积)

### Radiance(辐射度，亮度)
指的是，表面在单位立体角、单位投影面积上发射、反射、透射或接收的功率。

$$L({\bf x}, \omega) \equiv \frac{{\rm d^2}\Phi({\bf x}, \omega)}{{\rm d}\omega \, {\rm d}A \, \cos{\theta}}, \quad \theta \text{是指表面法线和光线夹角}$$

单位:

$$\rm \left [\frac{W}{sr \cdot m^2}\right ]\left [\frac{cd}{m^2} = \frac{lm}{sr \cdot m^2} = nit\right ]$$

可以由 Radiant intensity 或 Irradiance 再微分一次，得到 Radiance:

$$L({\bf x}, \omega) = \frac{{\rm d}E({\bf x})}{{\rm d}\omega \, \cos{\theta}}$$

$$L({\bf x}, \omega) = \frac{{\rm d}I({\bf x}, \omega)}{{\rm d}A \, \cos{\theta}}$$

## Bidirectional Reflectance Distribution Function(双向反射分布函数, BRDF)
我们需要处理这么一件事情: 在物体的某个点上，接受到从某方向来的 Radiance ，然后往四面八方反射不同强度的 Radiance(就是镜面反射和漫反射)。而 BRDF 就是处理这件事情的。

反射可以理解为，物体吸收了光线，然后又发射了光线。

BRDF, 输入是物体表面某个点、入射角度、出射角度；输出是一个标量(单位是 $\left [\frac{1}{sr} \right ]$ )。

BRDF 描述了光线如何跟物体作用，物体的材质通过 BRDF 定义。

### The Reflection Equation(反射方程)
BRDF 得出的结果，乘入射的 Irradiance ，就得到出射的 Radiance 。对于全部入射方向进行积分(或者求和)。

这只是光线经过了一次反射，之后我们还需要考虑光线的多次反射，感觉可以使用递归的思想。

### The Rendering Equation(渲染方程)
如果物体自己会发光，那么只要在反射方程的基础上，加上物体自己发光的部分即可。

不过到目前为止，我们还没有说明如何解这两个方程，如何在计算机系统中实现。

### Use The Rendering Equation
摄像机看向了某一点。

如果只有一个点光源，就计算点光源的一次反射、这一点本身发的光。

如果有多个点光源，那么计算多个点光源的和、这一点本身发的光。

如果有面光源，那么计算面光源的积分，再加上其他项。

如果有其他物体的反射光到达了这一点，那么把其他物体也当成光源。那么其他物体发多少光呢? 递归地使用渲染方程，计算其他物体发出的光。

## Global illumination(全局光照)
可以通过一些奇怪的数学变换，可以把渲染方程写成以下算子形式:

$$\sf L = E + KL$$

其中， $\sf K$ 是算子， $\sf L$ 是 Radiance 。以上各个字母都是矩阵。

$$\sf IL - KL = E$$

$$\sf (I - K)L = E$$

$$\sf L = (I - K)^{-1} E$$

进行泰特展开，得到:

$$\sf L = (I + K + K^2 + K^3 + \cdots) E$$

$$\sf L = E + KE + K^2E + K^3E + \cdots$$

其中， $\sf E$ 是指直接看光源， $\sf KE$ 是指光源发出的光经过了一次反射， $\sf K^2E$ 是指光源发出的光经过了两次反射，以此类推。

光线反射多次的结果都加起来，就是全局光照。

光栅化可以轻松做到 $\sf E$ 和 $\sf KE$ ，后面的就比较难做了。

## 阅读材料
[基于物理着色：BRDF](Bibliography/基于物理着色BRDF/基于物理着色BRDF.md)