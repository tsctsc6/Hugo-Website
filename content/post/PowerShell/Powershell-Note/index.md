+++
date = '2025-05-13T16:00:00+08:00'
lastmod = '2025-10-10T20:39:02+08:00'
draft = false
title = 'PowerShell 常用命令'
categories = ['Main Sections']
tags = ['PowerShell']
+++

## Cmdlet 与别名
```PowerShell
# 根据 Cmdlet 查询别名
Get-Alias -Definition "Cmdlet名称"
```

```PowerShell
# 根据别名查询 Cmdlet
Get-Command "别名"
```

## 校验文件哈希
定义变量 `fileHash` :

```PowerShell
$fileHash = Get-FileHash "program.exe" -Algorithm SHA256
```

对比哈希值：

```PowerShell
$fileHash.Hash -eq "HashValue"
```

## Base64
```PowerShell
# 将字符串编码为 Base64
$string = "Hello, PowerShell!"
$bytes = [System.Text.Encoding]::UTF8.GetBytes($string)
$base64Encoded = [System.Convert]::ToBase64String($bytes)

Write-Output $base64Encoded

# 将 Base64 解码为字符串
$base64String = "SGVsbG8sIFBvd2VyU2hlbGwh"
$decodedBytes = [System.Convert]::FromBase64String($base64String)
$decodedString = [System.Text.Encoding]::UTF8.GetString($decodedBytes)

Write-Output $decodedString
```

## 读取文件
使用 `Get-Content` Cmdlet 读取文本内容。

```PowerShell
$content = Get-Content file.txt
```

这样读取文件， `$content` 的类型是 `Object[]` ，一行为一个 `Object` 。

如果想读取为完整的字符串：

```PowerShell
$content = Get-Content file.txt -Raw
```

如果想读取为字节数组：

```PowerShell
$content = Get-Content file.txt -Raw -AsByteStream
```
