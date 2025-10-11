+++
date = '2025-10-11T11:34:07+08:00'
lastmod = '2025-10-11T11:34:17+08:00'
draft = false
title = 'PowerShell 用户输入'
categories = ['Main Sections']
tags = ['PowerShell']
+++

## 普通的用户输入
等我们编写脚本，想要使用脚本的用户输入某些东西的时候，使用以下命令：

```PowerShell
Read-Host
```

这样，会等待用户输入内容，回车键结束输入。

如果想添加用户输入提示，可以这样：

```PowerShell
$name = Read-Host "Name"
```

> Name 后面会自动加冒号。

## 用户机密输入
如果想要用户输入密码，可以这样：

```PowerShell
$password = Read-Host "Password" -AsSecureString
```

用户输入密码时，密码会被"\*"代替。

同时， `$password` 不是字符串，而是 `System.Security.SecureString` 。

需要注意的是，有些 PowerShell Cmdlet 的某些参数要求是 `SecureString` ，不过对于某些外部应用，需要在参数中或环境变量输入明文密码。要把 `SecureString` 转换为明文，可以这样做：

```PowerShell
$password = Read-Host "Password" -AsSecureString
[System.Runtime.InteropServices.Marshal]::PtrToStringAuto(`
[System.Runtime.InteropServices.Marshal]::SecureStringToBSTR($password))
```
