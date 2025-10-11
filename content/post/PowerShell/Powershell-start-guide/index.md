+++
date = '2025-10-10T20:38:26+08:00'
lastmod = '2025-10-10T20:38:30+08:00'
draft = false
title = 'PowerShell 入门指南'
categories = ['Main Sections']
tags = ['PowerShell']
+++

## PowerShell 简介
PowerShell 是一个由微软开发的​​命令行外壳、脚本语言和配置管理框架​​。它构建于 .NET 框架之上，专为系统管理员和高级用户设计，用于实现任务的自动化和管理操作系统（包括 Windows, Linux, macOS）及其上运行的应用程序。

可以把它理解为 ​​“命令提示符（cmd.exe）的升级版”​​。

## PowerShell 的核心特点
### 基于对象，而非文本​
* 传统命令行（如 CMD, Bash）​​：输入命令，输出的是纯文本。如果你想对输出结果进行进一步处理，就需要使用像 grep, awk, sed 这样的文本处理工具来解析文本。
* PowerShell​​：输入命令，输出的是​​完整的 .NET 对象​​。对象自带了丰富的属性和方法。这意味着你可以直接访问对象的特定属性（如文件名、进程ID、服务状态），而无需进行复杂的文本解析。

### 管道​
管道 (`|`) 功能允许你将一个命令的输出传递给另一个命令的输入。

由于管道传递的是对象，而不仅仅是文本，这使得管道操作变得极其强大和灵活。你可以轻松地将一个命令的结果对象传递给另一个命令，进行过滤、排序、分组等操作。

### ​​别名(Alias)
为了方便从其他命令行（如 CMD 或 Unix shell）过渡的用户， PowerShell 为许多常用命令提供了别名。例如， Get-ChildItem 的别名是 dir 和 ls ； Copy-Item 的别名是 copy 和 cp 。你既可以使用熟悉的 ls ，也可以使用标准命令 Get-ChildItem 。还可以对命令进行自定义的别名。

### 脚本语言功能
PowerShell 是一门完整的脚本语言，支持变量、数组、哈希表、循环（for, foreach, while）、条件判断（if, switch）、函数、模块等编程结构，可以编写复杂的自动化脚本。

### 调用 \.NET 方法
PowerShell 可以创建 \.NET 类和调用 \.NET 静态类方法。可以理解为动态的 \.NET 脚本。

### 跨平台和开源
最初的 PowerShell 仅适用于 Windows 。 2016 年，微软发布了 ​​PowerShell Core​​（基于 .NET Core），这是一个开源、跨平台版本。

现在，PowerShell 可以运行在 Windows、macOS 和多种 Linux 发行版上。

PowerShell 的源代码地址：[https://github.com/PowerShell/PowerShell](https://github.com/PowerShell/PowerShell)

## PowerShell 核心概念
### 版本和安装
Windows 原生提供的 PowerShell 是基于 .NET Framework 的，较为旧版的 PowerShell 。建议还是安装较新的，基于 .NET Core 的 PowerShell 7.

[在 Windows、Linux 和 macOS 上安装 PowerShell - PowerShell | Microsoft Learn](https://learn.microsoft.com/zh-cn/PowerShell/scripting/install/installing-PowerShell)

Windows 原生的 PowerShell 的可执行文件名是"powershell.exe"， PowerShell 7 的可执行文件名是"pwsh"。

### Cmdlet
Cmdlet 是最小命令单元。每个 Cmdlet 都遵循“动词-名词”命名规范，例如：

| Cmdlet | 含义 |
| :--: | :--: |
| Get-Process | 获取进程信息 |
| Remove-Item | 删除文件等 |

### 帮助系统
使用 Get-Help 命令可以获取任何命令的详细说明、参数列表和示例。例如：

```PowerShell
Get-Help Get-Process -Examples
```

### 资源驱动器抽象
PowerShell 提供了一种统一的资源访问方式: Provider 模型，它把各种资源映射为虚拟驱动器（PSDrive），就像 C: 盘一样操作。

常见 Provider 类型：

| Provider | 示例 | 说明 |
| :--: | :--: | :--: |
| FileSystem | C:, D: | 本地文件系统 |
Registry | HKLM:, HKCU: | Windows 注册表
Environment | Env: | 环境变量
Certificate | Cert: | 证书存储
Function | Function: | 当前会话中的函数
Variable | Variable: | 当前定义的变量
Alias | Alias: | 命令别名

例如：
```PowerShell
# 查看文件夹内容
Get-ChildItem "C:\Program Files"

# 查看注册表项
Get-ChildItem HKLM:\Software\Microsoft
```

### 执行策略（Execution Policy）
为了防止恶意脚本执行，PowerShell 引入了 执行策略（Execution Policy），它限制 .ps1 脚本文件的执行行为。

| 策略 | 含义 |
| :--: | :--: |
| Restricted | 默认，不允许运行任何脚本 |
| RemoteSigned | 允许本地脚本运行，远程脚本需签名 |
| AllSigned | 所有脚本都需签名 |
| Bypass | 无限制，不建议在生产中使用 |

设置执行策略：

```PowerShell
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
```

## 基本语法
### 注释
```PowerShell
# 这是单行注释
<#
这是多行注释
#>
Get-ChildItem
```

### 输出文本
```PowerShell
Write-Host "Hello"
```

### 变量
```PowerShell
# 变量以 $ 符号开头，无需事先声明类型

# 字符串
$name = "Alice"

# 整数
$age = 14

# 浮点数
$pi = 3.14

# 布尔值
$x = $true

# 数组，可以混合类型
$array = @(1, 2, 3, "a")

# 字典，字符串有无双引号都行
$dic = @{Name="Tom"; "Age"=30}

# 可以使用 .GetType() 查看变量类型的详细信息：
$dic.GetType()
```

### 运算符
| 类别 | 示例 | 说明 |
| :--: | :--: | :--: |
| 算术运算 | + - * / % | 常见数学运算 |
| 比较运算 | -eq -ne -lt -gt | 等于、不等于、小于、大于 |
| 逻辑运算 | -and -or -not | 逻辑运算符 |
| 字符串 | -like -match -replace | 模式匹配和替换 |
| 包含运算 | -in -contains | 集合判断 |

示例：

```PowerShell
5 -eq 5       # True
"abc" -like "a*"  # True
```

### 条件判断
```PowerShell
if ($age -ge 18) {
    Write-Host "成年人"
} else {
    Write-Host "未成年人"
}

if ($score -ge 90) {
    "优秀"
} elseif ($score -ge 60) {
    "及格"
} else {
    "不及格"
}
```

### 循环
```PowerShell
for ($i = 1; $i -le 5; $i++) {
    Write-Host "第 $i 次循环"
}

$count = 0
while ($count -lt 3) {
    Write-Host $count
    $count++
}

$colors = @("Red", "Green", "Blue")
foreach ($color in $colors) {
    Write-Host "颜色：$color"
}
```

### 错误处理
```PowerShell
try {
    Get-Item "C:\NotExist.txt"
} catch {
    Write-Host "找不到文件"
} finally {

}
```

> 更多语法，请参考官方文档: [https://learn.microsoft.com/zh-cn/powershell/module/microsoft.powershell.core/about](https://learn.microsoft.com/zh-cn/powershell/module/microsoft.powershell.core/about/about)

## 字符串插值
简单类型的字符串插值：

```PowerShell
$count = 3
Write-Host "Count is $count"
# 输出: Count is 3
```

对于复杂类型的插值，以上的方法就不太管用了：

```PowerShell
$dic = @{Name="Tom"; "Age"=30}
Write-Host "$dic"
# 输出: System.Collections.Hashtable
```

这是因为， `$dic` 是复杂类型，默认提供的，自动转换为字符串的方法，就是输出类名。我们可以把它转换为 json:

```PowerShell
$dic = @{Name="Tom"; "Age"=30}
ConvertTo-Json $dic
<# 输出:
{
  "Name": "Tom",
  "Age": 30
}
#>
```

如果我们尝试输出 `$dic` 的 `Name` :

```PowerShell
$dic = @{Name="Tom"; "Age"=30}
Write-Host "Hello, $dic.Name"
# 输出: Hello, System.Collections.Hashtable.Name
```

显然，".Name"成了字符串常量。正确的方式是：

```PowerShell
$dic = @{Name="Tom"; "Age"=30}
Write-Host "Hello, $($dic.Name)"
# 输出: Hello, Tom
```
