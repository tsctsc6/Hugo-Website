+++
date = '2025-05-20T16:44:19+08:00'
draft = false
title = '线性代数简单笔记'
categories = ['Sub Sections']
math = true
+++

## 向量
$$ \vec{a} = \begin{bmatrix} a_x \\\\ a_y \end{bmatrix} $$
$$ \vec{b} = \begin{bmatrix} b_x \\\\ b_y \end{bmatrix} $$
$$ \mid \vec{a} \mid = \sqrt{a_x^2 + a_y^2} (仅在直角坐标系有效) $$

## 向量加法
$$ \vec{a} + \vec{b} = \begin{bmatrix} a_x + b_x \\\\ a_y + b_y \end{bmatrix} $$
$$ \vec{a} + \vec{b} = \vec{b} + \vec{a} $$
$$ \vec{a} + (\vec{b} + \vec{c}) = (\vec{a} + \vec{b}) + \vec{c}  $$

## 向量点乘
$$ \vec{a} \cdot \vec{b} = \mid \vec{a} \mid \mid \vec{b} \mid \cos{\theta} = a_x b_x + a_y b_y $$
$$ \vec{a} \cdot \vec{b} = \vec{b} \cdot \vec{a} $$
$$ \vec{a} \cdot (\vec{b} + \vec{c}) = \vec{a} \cdot \vec{b} + \vec{a} \cdot \vec{c} $$
$$ (k \vec{a}) \cdot \vec{b} = \vec{a} \cdot (k \vec{b}) = k(\vec{a} \cdot \vec{b}) $$

## 向量叉乘
$$ \vec{d} = \vec{a} \times \vec{b} $$
$$ \mid \vec{d} \mid = \mid \vec{a} \mid \mid \vec{b} \mid \sin{\theta} $$
$ \vec{d} $ 的方向同时垂直于 $ \vec{a} $ 和 $ \vec{b} $ ，符合右手螺旋定则。

$$ \vec{a} \times \vec{b} = \begin{bmatrix} a_y b_z - a_z b_y \\\\ a_z b_x - a_x b_z \\\\ a_x b_y - a_y b_x \end{bmatrix} $$

$$ \vec{a} \times \vec{b} = \begin{vmatrix} \hat{\imath} & \hat{\jmath} & \hat{k} \\\\ a_x & a_y & a_z \\\\ b_x & b_y & b_z \end{vmatrix} $$

$$ \vec{a} \times \vec{b} = - \vec{b} \times \vec{a} $$
$$ \vec{a} \times \vec{a} = \vec{0} $$
$$ \vec{a} \times (\vec{b} + \vec{c}) = \vec{a} \times \vec{b} + \vec{a} \times \vec{c} $$
$$ (k \vec{a}) \times \vec{b} = \vec{a} \times (k \vec{b}) = k(\vec{a} \times \vec{b}) $$

## 矩阵相乘
( $M$ 行 $N$ 列)( $N$ 行 $P$ 列) = ( $M$ 行 $P$ 列)

$$ A = \begin{bmatrix} a_{1,1} & a_{1,2} & \cdots & a_{1,n} \\\\ a_{2,1} & a_{2,2} & \cdots & a_{2,n} \\\\ \vdots & \vdots & \ddots & \vdots \\\\ a_{m,1} & a_{m,2} & \cdots & a_{m,n} \end{bmatrix} $$
$$ B = \begin{bmatrix} b_{1,1} & b_{1,2} & \cdots & b_{1,p} \\\\ b_{2,1} & b_{2,2} & \cdots & b_{2,p} \\\\ \vdots & \vdots & \ddots & \vdots \\\\ b_{n,1} & b_{n,2} & \cdots & b_{n,p} \end{bmatrix} $$
$$ C = AB $$
$$ C[r, c] = \sum_{i = 1}^{n} a_{r,i} \cdot b_{i,c} $$

$ AB \neq BA $ in general.

$$ A(BC) = (AB)C $$
$$ A(B+C) = AB+AC $$
$$ (A+B)C = AC+BC $$

$$ AB = (B^T A^T)^T $$

$$ I = \begin{bmatrix} 1 && 0 && \cdots && 0 \\\\ 0 && 1 && \cdots && 0 \\\\ \vdots && \vdots && \ddots && \vdots \\\\ 0 && 0 && \cdots && 1 \end{bmatrix} $$

$$ AA^{-1} = A^{-1}A = I $$
$$ AB = (B^{-1}A^{-1})^{-1} $$
