+++
date = '2025-10-11T12:10:04+08:00'
lastmod = '2025-10-11T15:54:05+08:00'
draft = false
title = 'PowerShell 小技巧'
categories = ['Main Sections']
tags = ['PowerShell']
+++

## 命令换行
在 PowerShell 中，使用"`"进行命令的换行：

```PowerShell
dotnet publish`
xxx.csproj`
--runtime $runtime`
--configuration Release`
--output $output_dir
```

## 直接调用绝对路径的可执行程序
在 PowerShell 中，使用"&"直接调用绝对路径的可执行程序（不使用"&"就是一个字符串）：

```PowerShell
&"path\to\exe.exe"
```

## 字符串转义
在 PowerShell 中，使用"`"进行字符串中的字符的转义：

```PowerShell
"`"Alice`""
# 输出: "Alice"
```

## 查找可执行文件的位置
在 Windows 的 CMD 中，使用 `where` 查找可执行文件的位置。

但是，在 PowerShell 中，使用 `where` 是 Cmdlet `Where-Object` 的别名。所以，在 PowerShell 中，使用 `where` 不能查找可执行文件的位置。要使用 `where.exe` 或者 `Get-Command` (别名 `gcm` )。

`where.exe` 相当于 CMD 的 `where` ，它其实是 C:\WINDOWS\system32\ 下的一个可执行程序，仅能查找可执行文件的位置。

`gcm` 的功能更加强大，不但能查找可执行文件的位置，还能查找 Cmdlet 和 Cmdlet 的别名。

## 参数转递
首先看个例子，假设我们想要查看某文件夹下的所有 .txt 文件：

```PowerShell
Get-ChildItem -Path "C:\Example" -Filter "*.txt" -Recurse -ErrorAction SilentlyContinue
```

这玩意儿看起来挺长的，虽然我们可以使用"`"进行换行，不过还有另一种解决办法。

在 PowerShell 中，可以通过哈希表(Hashtable)来传递 Cmdlet 的参数，这种方法称为 Splatting 。

```PowerShell
# 定义参数哈希表
$params = @{
    Path        = "C:\Example"
    Filter      = "*.txt"
    Recurse     = $true
    ErrorAction = "SilentlyContinue"
}

# 通过 Splatting 传递参数给 Get-ChildItem
Get-ChildItem @params
```

> 哈希表的键值大小写不敏感。

使用 Splatting 传递参数，除了避免丑陋的长命令，还能复用 `$params` ，或动态地构建和修改参数。

Splatting 只能对 Cmdlet 有用，对非 PowerShell 可执行程序没有用。因为 Cmdlet 的参数格式是 `-Path` ，其他可执行程序不一定遵循这个格式。不过还是有替代方案：参数数组。

```PowerShell
# 创建参数数组
$arguments = @(
    "--input",
    "C:\files\input.txt",
    "--output",
    "C:\output\result.csv",
    "--verbose"
)

# 执行外部程序并传递参数
myprogram.exe @arguments
```

非要用字典也不是不行：

```PowerShell
$paramMap = @{
    Input = "C:\input.txt"
    Output = "C:\output.txt"
    Verbose = $true
}

$cmdLine = "myprogram.exe "
# 自定义展开的逻辑
foreach ($key in $paramMap.Keys) {
    $cmdLine += "--$key `"$($paramMap[$key])`" "
}

Invoke-Expression $cmdLine
```

## 特殊变量
| 变量名 | 描述 |
| :--: | :--: |
| $_ | 管道中当前处理的对象 |
| $? | 上一个命令是否成功 |
| $LASTEXITCODE | 上一个外部程序的退出码 |
| $PSVersionTable | PowerShell 的版本信息 |

### 环境变量
PowerShell 能通过 `$env:<name>` 访问环境变量，比如通过 `$env:Path` 获取 Path 环境变量（大小写不敏感）。

> 在 Windows 的 CMD 中，访问环境变量的方式是 `%<name>%` 。

## Json 解析
PowerShell 的 Cmdlet 是基于变量的，而非 PowerShell 可执行程序是基于字符串的。这就造成了隔阂。缓解这一隔阂的方法，就是让非 PowerShell 可执行程序返回 Json 字符串（别的数据交换格式也行，但 Json 最为广泛），然后 PowerShell 解析，以便处理。

### Json 字符串转换为对象
```PowerShell
$json_string = "{`"Name`": `"Alice`", `"Age`": 14}"
$obj = ConvertFrom-Json $json_string
```

`$obj` 的类型是 `PSCustomObject` 。

如果要转换为哈希表：

```PowerShell
$json_string = "{`"Name`": `"Alice`", `"Age`": 14}"
$obj = ConvertFrom-Json $json_string -AsHashtable
```

> JSON 标准允许重复键名称，但在 PSObject 和 哈希表 类型中是禁止的。 例如，如果 JSON 字符串包含重复键，则此 ConvertFrom-Json 仅使用最后一个键。

### 对象转换为 Json 字符串
```PowerShell
$dic = @{Name="Tom"; "Age"=30}
ConvertTo-Json $dic
```
