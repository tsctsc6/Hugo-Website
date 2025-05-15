+++
date = '2025-05-13T16:00:00+08:00'
draft = false
title = '配置 Windows CLI'
categories = ['Main Sections']
tags = ['Windows']
+++

## Windows 修改 CMD 默认编码为 UTF-8
1. win 键 + R，输入 `regedit` ，打开注册表。
2. 找到 `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Command Processor` 。
3. 新建“字符串值”(REG_SZ)，命名为 `autorun` ，数值数据为 `chcp 65001` 。

不过，在 Windows 11 中，直接改系统编码为 UTF-8 ，好像也可以。

## Windows 在 CMD 中提升为管理员权限
在 Windows 11 24H2 中，添加了 `sudo` 这个程序，可以直接提升为管理员权限了，用法和 Linux 的 `sudo` 差不多。不过在 `sudo` 后面只能使用 cmd 的命令，而不能是 Powershell 的命令。

## Windows 11 以管理员权限运行 Windows Terminal
### 方法一
1. 普通地打开 Windows Terminal 。
2. 添加标签，右键，"以管理员权限运行"。

### 方法二
1. 点击右键，显示菜单。
2. 按住 `Shift` `Ctrl` ，左键点击"在终端打开"。

注意: 在普通窗口打开新管理员权限窗口，路径会继承。但是在管理员权限窗口打开新管理员权限窗口，路径不会继承。

## Windows 11 创建右键菜单项
[https://github.com/qianfanguojin/windowsterminal-menu](https://github.com/qianfanguojin/windowsterminal-menu)

## Windows CMD 切换目录
1. 直接填写盘符，跳转到对应盘。如: `E:` 。
2. 输入 `cd {path}` 以跳转路径。其中， `./` 表示当前文件夹， `../` 表示父文件夹。

## Windows 11 以下系统，在右键菜单添加“在此处打开 CMD ”
1. 编写 .reg 文件

```reg
Windows Registry Editor Version 5.00
[HKEY_LOCAL_MACHINE\SOFTWARE\Classes\Directory\background\shell\cmd_here]
@="CMD Here"
"Icon"="cmd.exe"
[HKEY_LOCAL_MACHINE\SOFTWARE\Classes\Directory\background\shell\cmd_here\command]
@="\"C:\\Windows\\System32\\cmd.exe\""
[HKEY_LOCAL_MACHINE\SOFTWARE\Classes\Folder\shell\cmdPrompt]
@="CMD Here"
[HKEY_LOCAL_MACHINE\SOFTWARE\Classes\Folder\shell\cmdPrompt\command]
@="\"C:\\Windows\\System32\\cmd.exe\" \"cd %1\""
[HKEY_LOCAL_MACHINE\SOFTWARE\Classes\Directory\shell\cmd_here]
@="CMD Here"
"Icon"="cmd.exe"
[HKEY_LOCAL_MACHINE\SOFTWARE\Classes\Directory\shell\cmd_here\command]
@="\"C:\\Windows\\System32\\cmd.exe\""
```

2. 使用注册表编辑器打开。

或者，可以安装 Windows Terminal 。