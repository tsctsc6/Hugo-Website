+++
date = '2026-03-03T11:07:04+08:00'
lastmod = '2026-03-03T18:55:42+08:00'
draft = false
title = 'LaTeX 入门'
categories = ['Main Sections']
tags = ['LaTeX']
+++

LaTeX 是一个排版系统，用于编写文档，论文，书籍等等。 LaTeX 于 Word 文档相比，有以下优势：

* 用户只需要关注写作的内容，而排版则完全交给软件自动完成。这让没有任何排版经验的普通人得以写出排版非常专业的论文或文章。
* LaTeX 代码是基于文本的，这使得它对 git 友好：
  * 可以清楚对比修改内容与原内容
  * 利于多人协作
* LaTeX 可以编写复杂的内容，包括但不限于图片，表格，数学公式，化学方程式，代码，伪代码，流程图，电路图，乐谱，化学结构式等。
* LaTeX 可以轻松和对话大模型(LLM)结合，有什么不会的，使用 LLM 生成 LaTeX 代码即可。

下面的视频是 LaTeX 的介绍（两个视频平台）：

{{<externalLinkCard title="为了让电脑上的数学公式更好看，这件事折腾了快50年" link="https://www.bilibili.com/video/BV1bU7JzAEbp/" cover="https://www.bilibili.com/favicon.ico">}}

{{<externalLinkCard title="为了让电脑上的数学公式更好看，这件事折腾了快50年" link="https://www.youtube.com/watch?v=RF9lcbSCfNg" cover="https://www.youtube.com/favicon.ico">}}

## 环境安装

需要安装的软件：

* VSCode ，作为文本编辑器
* 一个 TeX 发行版

| TeX 发行版 | 特点 |
| :--: | :--: |
| {{<externalLinkCard title="TeX Live" link="https://www.tug.org/texlive/" cover="auto">}} | 有完整的包；体积较大；安装时间较长 |
| {{<externalLinkCard title="MiKTeX" link="https://miktex.org/" cover="auto">}} | 体积较小；包按需下载 |

### TeX Live 安装

推荐通过 bt 下载 iso 文件。下载完成后，装载 iso ，运行 install-tl-windows.bat ，根据引导安装（建议选择全量安装）。安装时间较长（30min ~ 60 min），请耐心等待，或者干点别的。

> [!warning]
> 安装路径不要有中文。

验证安装：

```powershell
where.exe xelatex
xelatex --version
```

### VSCode 插件配置

{{<externalLinkCard title="LaTeX Workshop" link="https://marketplace.visualstudio.com/items?itemName=James-Yu.latex-workshop" cover="auto">}}

### 简单测试

新建文件 test.tex

```latex {name="test.tex"}
\documentclass{article}

\begin{document}

Hello LaTeX!

\end{document}
```

然后命令行：

```powershell
xelatex test.tex
```

会生成包括 pdf 在内的一系列文件，打开 pdf ，检查是否正常。

### VSCode 推荐配置

```json {name="./.vscode/settings.json"}
{
  "latex-workshop.latex.autoBuild.run": "never",

  "latex-workshop.latex.outDir": "out",

  "latex-workshop.latex.recipes": [
    {
      "name": "xelatex",
      "tools": ["xelatex"]
    },
    {
      "name": "xelatex draft",
      "tools": ["xelatex-draft"]
    }
  ],

  "latex-workshop.latex.tools": [
    {
      "name": "xelatex",
      "command": "xelatex",
      "args": [
        "-synctex=1",
        "-interaction=nonstopmode",
        "-file-line-error",
        "-output-directory=out",
        "-shell-escape",
        "%DOC%"
      ]
    },
    {
      "name": "xelatex-draft",
      "command": "xelatex",
      "args": [
        "-synctex=1",
        "-interaction=nonstopmode",
        "-file-line-error",
        "-draftmode",
        "-output-directory=out",
        "-shell-escape",
        "%DOC%"
      ]
    }
  ],

  "latex-workshop.latex.recipe.default": "xelatex"
}
```

其中：

* `"-output-directory=out"` 参数，表示生成的文件放到 out 文件夹中。
* `"-shell-escape"` 参数，允许在编译 LaTeX 过程中，调用操作系统的外部命令。某些宏包需要用到。
* `"-draftmode"` 参数，表示不输出 pdf 文件，用于检查 LaTeX 代码是否有错，节约时间。

## LaTeX 代码文件的基本结构

LaTex源文件分为导言区和正文区，以[简单测试](#简单测试)中的代码为例， `\begin{document}` 和 `\end{document}` 之间是正文区， `\begin{document}` 之前的是导言区。

LaTeX 使用 `%` 注释。

### 导言区

导言区进行全局设置，主要有以下几种设置。

#### 定义文件类型

根据文件类型的不同，会有不同的排版。

比如下面代码将定义文件类型为 article 。

```latex
\documentclass{article}
```

#### 文档元数据

定义一些文档的元数据：

```latex
\title{My Article}
\author{tsctsc6}
\date{\today}
```

可以在正文区使用 `\maketitle` 显示这些信息。

#### 导入宏包

使用以下代码导入宏包：

```latex
\usepackage{<package_name>} % 导入单个宏包
\usepackage{<package_name1>, <package_name2>, <package_name3>, ...} % 导入多个宏包
```

---

其他的，还有调整页眉页脚，字体大小等等。

### 正文区

```latex
\section{标题}
\subsection{一级标题}
\subsubsection{二级标题}
\noindent 这是一个新的段落，使用了 \texttt{\textbackslash noindent} 来取消首行缩进。
\paragraph{段落标题} 这是一个段落标题，下面是一些文本内容。
\newpage % 新起一页
```

### 将文档分散到多个文件

将文档分散到多个文件，主要有以下方法：

| 方法 | 自动分页 | 支持局部编译 | 子文件可单独编译 |
| :--: | :--: | :--: | :--: |
| `\input` | ❌ | ❌ | ❌ |
| `\include` | ✅ | ✅ | ❌ |
| subfiles 宏包 | ❌ | ✅ | ✅ |

以 `\input` 为例：

```latex {name="main.tex"}
\documentclass{book}

\begin{document}

\input{chapter1.tex}
\input{chapter2.tex}

\end{document}
```

```latex {name="chapter1.tex"}
\section{第一章}

这里是第一章内容。
```

## 常用宏包

### 中文支持

```latex
\usepackage[UTF8]{ctex}
```

### 数学公式

数学公式相关的宏包有许多，按需取用即可。下面是基本要用到的：

```latex
\usepackage{amsmath, amssymb}
```

如果嫌手写数学公式复杂，可以使用一些第三方工具辅助，比如 AxMath , 是一个 WYSIWYG 风格（所见即所得）的工具，直接在可视化界面编辑数学公式。

### 表格

表格的编辑也比较复杂，可以使用第三方工具编辑，比如：

{{<externalLinkCard title="Advanced Table Editor - LaTeX, Typst, HTML and more" link="https://www.latex-tables.com/" cover="auto">}}

### 交叉引用

交叉引用可以用几种不同的宏包来实现，这里用：

```latex
\usepackage{hyperref}
```

{{< tabs >}}

<!-- tab 表格 -->
在表格的 `\caption{}` （这是表格标题）后面加：

```latex
\label{tab:table_name}
```

<!-- tab 图片 -->
在图片的 `\caption{}` （这是图片标题）后面加：

```latex
\label{fig:figure_name}
```

{{< /tabs >}}

第一次生成 pdf 后，应该是引用的地方，可能会有 "??" 样子的字样，再编译一次即可。

### minted

这是一个用于展示代码的宏包。

需要安装 Python 环境。对于 TeX Live 2025 ，建议使用 Python 3.13 .

Python 需要安装以下包：

```powershell
pip install Pygments
```

例子：

```latex
\begin{minted}{rust}
fn main() {
    println!("Hello, world!");
}
\end{minted}
```

第一次生成 pdf 后，应该是代码的地方，可能会有 "\<minted\>" 样子的红色字样，再编译一次即可。

## 但是话又说回来

其实 Typst 也是挺好的，与 LaTeX 相比：

* Typst 本体非常轻量，大概十几 MB，与动辄十几 GB 的 TeX Live 相比，呵呵。
* Typst 的语法更加现代化，更易学；可以嵌入简单脚本来生成文档。
* Typst 还有实时预览的功能

{{<externalLinkCard title="Typst Github 地址" link="https://github.com/typst/typst" cover="auto">}}

## 参考链接

{{<externalLinkCard title="【LaTeX入门/速成】国赛省一写作手的LaTeX基本知识教学（第1期）" link="https://www.bilibili.com/video/BV1s2ZZBJEV2/" cover="https://www.bilibili.com/favicon.ico">}}
