+++
date = '2025-08-16T12:37:55+08:00'
draft = true
title = 'ZIP 二进制格式解析'
categories = ['Main Sections']
mermaid = true
+++

ZIP 是一种支持无损数据压缩的归档文件格式。一个 ZIP 文件可包含一个或多个经过压缩的文件或目录。 ZIP 归档中的文件是独立压缩的，因此可以单独提取或添加新文件，而无需对整个归档进行压缩或解压处理。这与 tar 文件格式形成对比，后者难以实现这种随机访问处理。

本文主要针对程序员或其他技术人员，概述 ZIP 文件二进制格式。本文的范围仅限于 ZIP 文件格式本身，不涉及可选的压缩或加密算法。

## ZIP 的总体结构
假设某 ZIP 文件中含有 a.txt 和 b.txt 两个文件，ZIP 的总体结构如下图：

<div>
    <svg width="400" height="350" viewBox="0 0 400 350" xmlns="http://www.w3.org/2000/svg">
    <defs>
        <marker id="arrowhead" markerWidth="10" markerHeight="10" refX="9" refY="3" orient="auto">
        <path d="M0,0 L0,6 L9,3 z" fill="#555"/>
        </marker>
    </defs>
    <g font-family="Noto Sans SC" font-size="16" text-anchor="middle">
        <rect x="25" y="10" width="250" height="40" fill="#f6f6f6" stroke="#707070" stroke-dasharray="5,2" stroke-width="1"/>
        <rect x="25" y="50" width="250" height="40" fill="#ffe7cd" stroke="#bc9e50" stroke-width="1"/>
        <rect x="25" y="90" width="250" height="40" fill="#fff2cc" stroke="#caba8a" stroke-width="1"/>
        <rect x="25" y="130" width="250" height="40" fill="#ffe7cd" stroke="#bc9e50" stroke-width="1"/>
        <rect x="25" y="170" width="250" height="40" fill="#fff2cc" stroke="#caba8a" stroke-width="1"/>
        <rect x="25" y="210" width="250" height="40" fill="#dbe5f9" stroke="#7c90b5" stroke-width="1"/>
        <rect x="25" y="250" width="250" height="40" fill="#dbe5f9" stroke="#7c90b5" stroke-width="1"/>
        <rect x="25" y="290" width="250" height="40" fill="#bfd6fe" stroke="#7c90b5" stroke-width="1"/>
        <text x="150" y="35">未定义</text>
        <text x="150" y="75">本地文件头 (a.txt)</text>
        <text x="150" y="115">a.txt 的数据</text>
        <text x="150" y="155">本地文件头 (b.txt)</text>
        <text x="150" y="195">b.txt 的数据</text>
        <text x="150" y="235">中央目录文件头 (a.txt)</text>
        <text x="150" y="275">中央目录文件头 (b.txt)</text>
        <text x="150" y="315">中央目录结束记录</text>
    </g>
    <g stroke="#555" stroke-width="1.5" fill="none" marker-end="url(#arrowhead)">
        <path d="M275,315 H300 V211 L275,211"/>
        <path d="M275,275 H310 V131 L275,131"/>
        <path d="M275,235 H320 V51 L275,51"/>
    </g>
    </svg>
</div>

* **本地文件头** : 包含单个文件的元数据，后面紧接文件数据。每个文件可独立进行压缩和加密。
* **中央目录文件头** : 包含单个文件的元数据以及指向该文件本地文件头的偏移量。这些文件头组成的列表形成了中央目录，可用于快速枚举存档中的所有文件。
* **中央目录结束记录** : 包含定位首个中央目录文件头位置的信息。

中央目录结束记录位于文件末尾的原因在于，我们可以方便地对内部的文件进行增加、修改、删除。

根据文章 [The ZIP File Format](https://medium.com/@felixstridsberg/the-zip-file-format-6c8a160d1c34) 所言，中央目录结束记录位于文件末尾，可以不用覆写地对内部的文件进行增加，但笔者认为这个说法有点不对。首先，这样会增加 ZIP 的体积；其次，无论怎么样，都要对磁盘写入相同的字节数量（新文件的内容）。当然，笔者并不是在否定“中央目录结束记录位于文件末尾”这个设计，这个设计确实可以方便地对内部的文件进行增加、修改、删除，但具体方式与文章 [The ZIP File Format](https://medium.com/@felixstridsberg/the-zip-file-format-6c8a160d1c34) 不同。

---

为 ZIP 增加文件 c.txt ，首先把中央目录内容保存到内存中：

<div>
    <svg width="400" height="350" viewBox="0 0 400 350" xmlns="http://www.w3.org/2000/svg">
    <defs>
        <marker id="arrowhead" markerWidth="10" markerHeight="10" refX="9" refY="3" orient="auto">
        <path d="M0,0 L0,6 L9,3 z" fill="#555"/>
        </marker>
    </defs>
    <g font-family="Noto Sans SC" font-size="16" text-anchor="middle">
        <rect x="25" y="10" width="250" height="40" fill="#f6f6f6" stroke="#707070" stroke-dasharray="5,2" stroke-width="1"/>
        <rect x="25" y="50" width="250" height="40" fill="#ffe7cd" stroke="#bc9e50" stroke-width="1"/>
        <rect x="25" y="90" width="250" height="40" fill="#fff2cc" stroke="#caba8a" stroke-width="1"/>
        <rect x="25" y="130" width="250" height="40" fill="#ffe7cd" stroke="#bc9e50" stroke-width="1"/>
        <rect x="25" y="170" width="250" height="40" fill="#fff2cc" stroke="#caba8a" stroke-width="1"/>
        <rect x="25" y="210" width="250" height="40" fill="#f6f6f6" stroke="#707070" stroke-dasharray="5,2" stroke-width="1"/>
        <rect x="25" y="250" width="250" height="40" fill="#f6f6f6" stroke="#707070" stroke-dasharray="5,2" stroke-width="1"/>
        <rect x="25" y="290" width="250" height="40" fill="#f6f6f6" stroke="#707070" stroke-dasharray="5,2" stroke-width="1"/>
        <text x="150" y="35">未定义</text>
        <text x="150" y="75">本地文件头 (a.txt)</text>
        <text x="150" y="115">a.txt 的数据</text>
        <text x="150" y="155">本地文件头 (b.txt)</text>
        <text x="150" y="195">b.txt 的数据</text>
        <text x="150" y="235">中央目录文件头 (a.txt) (弃置)</text>
        <text x="150" y="275">中央目录文件头 (b.txt) (弃置)</text>
        <text x="150" y="315">中央目录结束记录 (弃置)</text>
    </g>
    <g stroke="#555" stroke-width="1.5" fill="none" marker-end="url(#arrowhead)">
        <path d="M275,315 H300 V211 L275,211"/>
        <path d="M275,275 H310 V131 L275,131"/>
        <path d="M275,235 H320 V51 L275,51"/>
    </g>
    </svg>
</div>

从 b.txt 的数据出直接追加数据：

<div>
    <svg width="400" height="470" viewBox="0 0 400 470" xmlns="http://www.w3.org/2000/svg">
    <defs>
        <marker id="arrowhead" markerWidth="10" markerHeight="10" refX="9" refY="3" orient="auto">
        <path d="M0,0 L0,6 L9,3 z" fill="#555"/>
        </marker>
    </defs>
    <g font-family="Noto Sans SC" font-size="16" text-anchor="middle">
        <rect x="25" y="10" width="250" height="40" fill="#f6f6f6" stroke="#707070" stroke-dasharray="5,2" stroke-width="1"/>
        <rect x="25" y="50" width="250" height="40" fill="#ffe7cd" stroke="#bc9e50" stroke-width="1"/>
        <rect x="25" y="90" width="250" height="40" fill="#fff2cc" stroke="#caba8a" stroke-width="1"/>
        <rect x="25" y="130" width="250" height="40" fill="#ffe7cd" stroke="#bc9e50" stroke-width="1"/>
        <rect x="25" y="170" width="250" height="40" fill="#fff2cc" stroke="#caba8a" stroke-width="1"/>
        <rect x="25" y="210" width="250" height="40" fill="#ffe7cd" stroke="#bc9e50" stroke-width="1"/>
        <rect x="25" y="250" width="250" height="40" fill="#fff2cc" stroke="#caba8a" stroke-width="1"/>
        <rect x="25" y="290" width="250" height="40" fill="#dbe5f9" stroke="#7c90b5" stroke-width="1"/>
        <rect x="25" y="330" width="250" height="40" fill="#dbe5f9" stroke="#7c90b5" stroke-width="1"/>
        <rect x="25" y="370" width="250" height="40" fill="#dbe5f9" stroke="#7c90b5" stroke-width="1"/>
        <rect x="25" y="410" width="250" height="40" fill="#bfd6fe" stroke="#7c90b5" stroke-width="1"/>
        <text x="150" y="35">未定义</text>
        <text x="150" y="75">本地文件头 (a.txt)</text>
        <text x="150" y="115">a.txt 的数据</text>
        <text x="150" y="155">本地文件头 (b.txt)</text>
        <text x="150" y="195">b.txt 的数据</text>
        <text x="150" y="235">本地文件头 (c.txt)</text>
        <text x="150" y="275">c.txt 的数据</text>
        <text x="150" y="315">中央目录文件头 (a.txt)</text>
        <text x="150" y="355">中央目录文件头 (b.txt)</text>
        <text x="150" y="395">中央目录文件头 (c.txt)</text>
        <text x="150" y="435">中央目录结束记录</text>
    </g>
    <g stroke="#555" stroke-width="1.5" fill="none" marker-end="url(#arrowhead)">
        <path d="M275,435 H300 V291 L275,291"/>
        <path d="M275,395 H310 V211 L275,211"/>
        <path d="M275,355 H320 V131 L275,131"/>
        <path d="M275,315 H330 V51 L275,51"/>
    </g>
    <g stroke="#555" stroke-width="1.5" fill="none" marker-end="url(#arrowhead)">
        <path d="M400,211 L340,211"/>
    </g>
    <g font-family="Noto Sans SC" font-size="12" text-anchor="middle">
        <text x="370" y="200">在这里追加</text>
    </g>
    </svg>
</div>

这样，增加 c.txt 时，就不需要对 a.txt 和 b.txt 文件的相关内容做任何操作。

## 中央目录结束记录
中央目录结束记录(End of central directory record, EOCD)的各字段如下：

```mermaid
---
showBits = true
bitWidth = 32
---
packet
title End of central directory record
+4: "签名"
+2: "本分卷编号"
+2: "中央目录起始分卷"
+2: "此分卷上的中央目录记录数量"
+2: "中央目录记录总数"
+4: "中央目录的大小（以字节为单位）"
+4: "中央目录起始偏移量（以字节为单位）"
+2: "注释长度"
+4: "注释"
```

## References
1. [The ZIP File Format](https://medium.com/@felixstridsberg/the-zip-file-format-6c8a160d1c34)
1. [ZIP (file format) - Wikipedia](https://en.wikipedia.org/wiki/ZIP_(file_format))