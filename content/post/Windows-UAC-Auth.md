+++
date = '2025-05-13T15:32:45+08:00'
draft = false
title = '如何给 Windows UAC 弹窗加上验证功能'
categories = ['Main Sections']
tags = ['Windows']
+++

## 方法一
1. 打开本地组策略编辑器
2. 计算机配置 -> Windows 设置 -> 安全设置 -> 本地策略 -> 安全选项 -> 用户帐户控制: 在管理审批模式下管理员的提升提示行为
3. 选择"在安全桌面上提示凭据"或"提示凭据"

## 方法二
1. 打开注册表编辑器
2. `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System`
3. 新建或修改 PromptOnSecureDesktop (DWORD)为 1
4. 新建或修改 ConsentPromptBehaviorAdmin (DWORD)为 1 或 3

ConsentPromptBehaviorAdmin 的值分别对应:

0 不提示，直接提升

1 在安全桌面上提示凭据

2 在安全桌面上同意提示

3 提示凭据

4 同意提示

5 非 Windows 二进制文件的同意提示
