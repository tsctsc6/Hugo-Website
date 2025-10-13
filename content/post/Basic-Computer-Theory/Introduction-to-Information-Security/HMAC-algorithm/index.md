+++
date = '2025-07-31T21:46:41+08:00'
lastmod = '2025-10-13T11:47:28+08:00'
draft = false
title = 'HMAC 算法'
categories = ['Sub Sections']
math = true
+++

HMAC（Hash-based Message Authentication Code） 是一种基于哈希函数的加密验证机制，它将一个密钥与数据组合后，用指定的哈希函数（如 SHA-256）来生成消息认证码（MAC）。其设计初衷是防止数据被篡改。

> HMAC 不是一个哈希算法，而是一个用哈希函数构建的算法结构。

HMAC 的计算公式如下（公式里的变量都是字节串）：

$$
HMAC(K, m) = H((K' \oplus opad) \mid\mid H((K' \oplus ipad) \mid\mid m))
$$

$$
K' = \begin{cases}
  H(K), & \text{if } K \text{ is large than block size} \\
  K \mid\mid 0^*, & \text{if } K \text{ is less than block size}
\end{cases}
$$

> * $H$ : 你选择的哈希函数（如 SHA-1, SHA-256, SHA-512 等）；
> * $K$ : 原始密钥（比如口令）；
> * $K'$ : 由 $K$ 生成的值，调整为哈希块大小（不足补零，超出则先哈希）（哈希块大小由哈希函数 $H$ 决定）；
> * $m$ : 输入消息；
> * $opad$ : 外部填充字节（全是 0x5c）；
> * $ipad$ : 内部填充字节（全是 0x36）；
> * $\mid\mid$ : 表示字节串拼接；
> * $\oplus$ : 按位异或。

注意！广义来说， HMAC 算法能用于[数字签名](../Message-Digest/index.md#数字签名)，但是签名和验证都使用同一个密钥（K），也就是只有持有该密钥的一方才能生成或校验签名。

参考连接: [HMAC - Wikipedia](https://en.wikipedia.org/wiki/HMAC)
