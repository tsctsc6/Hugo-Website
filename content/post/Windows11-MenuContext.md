+++
date = '2025-10-10T17:50:56+08:00'
lastmod = '2025-10-10T17:51:18+08:00'
draft = false
title = '自定义 Windows 11 新版右键菜单'
categories = ['Main Sections']
tags = ['Windows']
+++

使用这个开源项目: [ContextMenuForWindows11](https://github.com/ikas-mc/ContextMenuForWindows11)

根据这个开源项目的文档，就能自定义 Windows 11 新版右键菜单了。

但是，由于 Windows 11 新版右键菜单的限制，一个独立的应用包只能同时存在一个菜单，如果在一个应用中弄多个右键菜单，会自动折叠为二级菜单。

解决这个问题的方法，就是多安装几个 ContextMenuForWindows11 应用。 Windows 的 MSIX 安装包，里面有一个 Guid ，通过它来判断是不是同一个应用。 ContextMenuForWindows11 的作者提供了[用户自己修改安装包的方式](https://github.com/ikas-mc/ContextMenuForWindows11/wiki/Create-Custom-Package)，不过说的不算特别清楚。本文的主要内容，就是清楚说明如何修改安装包。

## 下载相关工具
* [MSIX Packaging Tool](https://learn.microsoft.com/en-us/windows/msix/packaging-tool/package-editor)
* 带有"Any"的安装包: [下载地址](https://github.com/ikas-mc/ContextMenuForWindows11/releases)。

## 创建证书
带有"Any"的安装包是不能安装的，因为 MSIX 安装需要签名。在这个场景下，使用自签名证书即可。

* 先创建证书(pfx 格式)。
* 创建证书后，需要把证书安装到“本地计算机/受信任人”这个地方。
* 生成两个 Guid ，等下要用。

使用 PowerShell 脚本完成上述步骤，脚本如下：

```PowerShell
# 检查是否以管理员身份运行
if (-not ([Security.Principal.WindowsPrincipal] [Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole] "Administrator")) {
   sudo pwsh -NoProfile -ExecutionPolicy Bypass -File $PSCommandPath
   exit
}

$password_sec = Read-Host "Password" -AsSecureString
$pfx_path = ".\cmc-dev-vscode.pfx"

$params = @{
    Type = 'Custom'
    Subject = 'CN=CMC-DEV-VSCode'
    FriendlyName = 'cmc-dev-vscode'
    CertStoreLocation = 'Cert:\CurrentUser\My'
    KeyUsage = 'DigitalSignature'
    TextExtension = @(
        '2.5.29.37={text}1.3.6.1.5.5.7.3.3',
        '2.5.29.19={text}' )
}

$cert = New-SelfSignedCertificate @params

$cert_thumbprint = $cert.Thumbprint;

Export-PfxCertificate -cert "Cert:\CurrentUser\My\$cert_thumbprint" -FilePath $pfx_path -Password $password_sec

$params = @{
    FilePath = $pfx_path
    CertStoreLocation = 'Cert:\LocalMachine\TrustedPeople'
    Password = $password_sec
}

Import-PfxCertificate @params

New-Guid;
New-Guid;

Pause;
```

## 编辑 MSIX 包
使用 MSIX Packaging Tool 编辑安装包。

首先对安装包签名，在"Signing preference"处，选择"sign with a cert(.pfx)", select 生成的 pfx 文件。

然后，在"Manifest file"处，点击"Open file", 修改文件：

* 修改 `Identity` 标签的 Id 为一个新的 Guid 。
* 修改 `com:Class` 标签的 Id 为一个新的 Guid 。
* 修改 `Custom Context Menu (Any)` 为 `Custom Context Menu (<你的右键菜单程序名称>)` 。

以上这些，可以通过文本搜索替换简单完成。

保存 Manifest file ，导出安装包。

## 安装及使用
见[官方 wiki](https://github.com/ikas-mc/ContextMenuForWindows11/wiki)
