+++
date = '2025-10-11T13:22:26+08:00'
lastmod = '2025-10-11T13:22:36+08:00'
draft = false
title = 'PowerShell 管道'
categories = ['Main Sections']
tags = ['PowerShell']
+++

PowerShell 的 Cmdlet 都是输出对象的，通过管道符"|"就可以把上一个 Cmdlet 的结果，传到下一个 Cmdlet 作为输入。就像函数式编程那样。

## 一个使用管道的例子
比如，使用 `Get-Process` 可以获取一系列进程的信息:

```PowerShell
Get-Process -Name msedge
```

输出:

```
 NPM(K)    PM(M)      WS(M)     CPU(s)      Id  SI ProcessName
 ------    -----      -----     ------      --  -- -----------
     83    33.34      40.68       0.00   18732   5 msedge
......
```

使用管道运算符和 `Select-Object` 可以对输出结果进行**转换**:

```PowerShell
Get-Process -Name msedge | Select-Object -Property Id
```

输出:

```
   Id
   --
18732
```

使用管道运算符和 `Where-Object` 可以对输出结果进行**筛选**:

```PowerShell
Get-Process | Where-Object { $_.ProcessName -eq "msedge" }
```

> `Where-Object` {} 里面的 `$_` ，就是指当前处理的对象。

输出:

```
 NPM(K)    PM(M)      WS(M)     CPU(s)      Id  SI ProcessName
 ------    -----      -----     ------      --  -- -----------
     83    33.34      40.68       0.00   18732   0 msedge
```

使用管道运算符和 `ForEach-Object` 可以对输出结果进行**遍历**:

```PowerShell
Get-Process -Name msedge | ForEach-Object { $myPid = $_.Id; Get-NetTCPConnection | Where-Object { $_.OwningProcess -eq $myPid } }
```

输出:

```
LocalAddress                        LocalPort RemoteAddress                       RemotePort State       AppliedSetting OwningProcess
------------                        --------- -------------                       ---------- -----       -------------- -------------
0.0.0.0                             55113     0.0.0.0                             0          Bound                      20756
0.0.0.0                             52461     0.0.0.0                             0          Bound                      20756
172.16.0.10                         55113     47.106.194.220                      443        Established Internet       20756
```

以上代码，先根据进程名称 msedge 得到 pid ，再根据 pid 获取其占用的端口号。
