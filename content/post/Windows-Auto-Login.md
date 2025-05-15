+++
date = '2025-05-13T16:20:00+08:00'
draft = false
title = 'Windows 自动登录'
categories = ['Main Sections']
tags = ['Windows']
+++

使用 `Windows + R` 快捷键调出「运行」对话框，输入 `netplwiz` 打开「用户账户」设置窗口。

取消勾选「要使用本计算机，用户必需输入用户名和密码」，然后点击「应用」按钮。

此时会弹出「自动登录」对话框，在此输入密码两次，确认无误后点击「确定」即可。

设置完成后，在登录时会自动帮你验证密码，但密码并未删除。如果你更改了账户密码，需要按以上步骤重新设置。

***

如果没有显示「要使用本计算机，用户必需输入用户名和密码」，可以更改注册表。

使用 `Windows + R` 快捷键打开「运行」，输入 `regedit` 打开注册表编辑器。

浏览到以下路径:

```
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon
```

在右侧窗口中并双击名为 `AutoAdminLogon` 的字符串值(REG_SZ)(如果没有就新建一个)，并将其值设置为:

* `0` 禁用自动登录
* `1` 启用自动登录

这时就可以回去看看，是否出现了「要使用本计算机，用户必需输入用户名和密码」。