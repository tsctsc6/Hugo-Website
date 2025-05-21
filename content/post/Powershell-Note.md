+++
date = '2025-05-13T16:00:00+08:00'
draft = false
title = 'Powershell 常用命令'
categories = ['Main Sections']
tags = ['Powershell']
+++

## 对一系列结果的处理
比如，使用 `Get-Process` 可以获取一系列进程的信息:

```Powershell
Get-Process -Name alist
```

输出:

```
 NPM(K)    PM(M)      WS(M)     CPU(s)      Id  SI ProcessName
 ------    -----      -----     ------      --  -- -----------
     83    33.34      40.68       0.00   18732   0 alist
```

使用管道运算符和 `Select-Object` 可以对输出结果进行**转换**:

```Powershell
Get-Process -Name alist | Select-Object -Property Id
```

输出:

```
   Id
   --
18732
```

使用管道运算符和 `Where-Object` 可以对输出结果进行**筛选**:

```Powershell
Get-Process | Where-Object { $_.ProcessName -eq "alist" }
```

输出:

```
 NPM(K)    PM(M)      WS(M)     CPU(s)      Id  SI ProcessName
 ------    -----      -----     ------      --  -- -----------
     83    33.34      40.68       0.00   18732   0 alist
```

使用管道运算符和 `ForEach-Object` 可以对输出结果进行**遍历**:

```Powershell
Get-Process -Name alist | ForEach-Object { $myPid = $_.Id; Get-NetTCPConnection | Where-Object { $_.OwningProcess -eq $myPid } }
```

输出:

```
LocalAddress                        LocalPort RemoteAddress                       RemotePort State       AppliedSetting OwningProcess
------------                        --------- -------------                       ---------- -----       -------------- -------------
::                                  30040     ::                                  0          Listen                     18732
0.0.0.0                             53343     0.0.0.0                             0          Bound                      18732
0.0.0.0                             53340     0.0.0.0                             0          Bound                      18732
0.0.0.0                             53334     0.0.0.0                             0          Bound                      18732
127.0.0.1                           53343     127.0.0.1                           5432       Established Internet       18732
127.0.0.1                           53334     127.0.0.1                           5432       Established Internet       18732
127.0.0.1                           30040     127.0.0.1                           54628      Established Internet       18732
```

以上代码，先根据进程名称 alist 得到 pid ，再根据 pid 获取其占用的端口号。

## 校验文件哈希
定义变量 `fileHash` :

```Powershell
$fileHash = Get-FileHash "program.exe" -Algorithm SHA256
```

对比哈希值：

```Powershell
$fileHash.Hash -eq "HashValue"
```
