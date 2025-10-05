+++
date = '2025-10-05T13:34:57+08:00'
lastmod = '2025-10-05T13:35:00+08:00'
draft = false
title = '安装 Windows 11 指南'
categories = ['Main Sections']
tags = ['Windows']
+++

前往微软官方网站下载 Windows 11: [https://www.microsoft.com/zh-cn/software-download/windows11](https://www.microsoft.com/zh-cn/software-download/windows11)

## 通过刻录 U 盘安装
下载 [Rufus](https://github.com/pbatard/rufus).

选择下载的 iso 和 U 盘，刻录。

事实上， Rufus 提供了一个功能：就是绕过在线账号的登录，直接创建一个默认的本地账号。这个功能是通过“应答文件”实现的。然而，如果想把 Windows 11 安装到虚拟机上，就不能使用 Rufus 了，需要把应答文件写入到 iso 中。

## 编写应答文件
首先， Rufus 编写的应答文件内容如下：

```xml {name="unattend.xml"}
<?xml version="1.0" encoding="utf-8"?>
<unattend xmlns="urn:schemas-microsoft-com:unattend">
  <settings pass="disabled">
    <component name="Microsoft-Windows-Setup" processorArchitecture="amd64" language="neutral" xmlns:wcm="http://schemas.microsoft.com/WMIConfig/2002/State" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" publicKeyToken="31bf3856ad364e35" versionScope="nonSxS">
      <UserData>
        <ProductKey>
          <Key />
        </ProductKey>
      </UserData>
      <RunSynchronous>
        <!-- 跳过各种检查 -->
        <RunSynchronousCommand wcm:action="add">
          <Order>1</Order>
          <Path>reg add HKLM\SYSTEM\Setup\LabConfig /v BypassTPMCheck /t REG_DWORD /d 1 /f</Path>
        </RunSynchronousCommand>
        <RunSynchronousCommand wcm:action="add">
          <Order>2</Order>
          <Path>reg add HKLM\SYSTEM\Setup\LabConfig /v BypassSecureBootCheck /t REG_DWORD /d 1 /f</Path>
        </RunSynchronousCommand>
        <RunSynchronousCommand wcm:action="add">
          <Order>3</Order>
          <Path>reg add HKLM\SYSTEM\Setup\LabConfig /v BypassRAMCheck /t REG_DWORD /d 1 /f</Path>
        </RunSynchronousCommand>
      </RunSynchronous>
    </component>
  </settings>
  <settings pass="specialize">
    <component name="Microsoft-Windows-Deployment" processorArchitecture="amd64" language="neutral" xmlns:wcm="http://schemas.microsoft.com/WMIConfig/2002/State" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" publicKeyToken="31bf3856ad364e35" versionScope="nonSxS">
      <RunSynchronous>
        <!-- 跳过在线的界面(OBEE) -->
        <RunSynchronousCommand wcm:action="add">
          <Order>1</Order>
          <Path>reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\OOBE /v BypassNRO /t REG_DWORD /d 1 /f</Path>
        </RunSynchronousCommand>
      </RunSynchronous>
    </component>
  </settings>
  <settings pass="oobeSystem">
    <component name="Microsoft-Windows-Shell-Setup" processorArchitecture="amd64" language="neutral" xmlns:wcm="http://schemas.microsoft.com/WMIConfig/2002/State" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" publicKeyToken="31bf3856ad364e35" versionScope="nonSxS">
      <OOBE>
        <ProtectYourPC>3</ProtectYourPC>
      </OOBE>
      <TimeZone>中国标准时间</TimeZone>
      <UserAccounts>
        <LocalAccounts>
          <!-- 创建本地账号 -->
          <LocalAccount wcm:action="add">
            <Name>Admin</Name>
            <DisplayName>Admin</DisplayName>
            <Group>Administrators;Power Users</Group>
            <Password>
              <Value>UABhAHMAewB3AG8AcgBvAA==</Value>
              <PlainText>false</PlainText>
            </Password>
          </LocalAccount>
        </LocalAccounts>
      </UserAccounts>
      <FirstLogonCommands>
        <!-- 第一次使用无密码登录 -->
        <SynchronousCommand wcm:action="add">
          <Order>1</Order>
          <CommandLine>net user &quot;Admin&quot; /logonpasswordchg:yes</CommandLine>
        </SynchronousCommand>
        <SynchronousCommand wcm:action="add">
          <Order>2</Order>
          <CommandLine>net accounts /maxpwage:unlimited</CommandLine>
        </SynchronousCommand>
      </FirstLogonCommands>
    </component>
    <component name="Microsoft-Windows-International-Core" processorArchitecture="amd64" language="neutral" xmlns:wcm="http://schemas.microsoft.com/WMIConfig/2002/State" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" publicKeyToken="31bf3856ad364e35" versionScope="nonSxS">
      <InputLocale>00000804</InputLocale>
      <SystemLocale>zh-CN</SystemLocale>
      <UserLocale>zh-CN</UserLocale>
      <UILanguage>zh-CN</UILanguage>
      <UILanguageFallback>en-US</UILanguageFallback>
    </component>
    <component name="Microsoft-Windows-SecureStartup-FilterDriver" processorArchitecture="amd64" language="neutral" xmlns:wcm="http://schemas.microsoft.com/WMIConfig/2002/State" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" publicKeyToken="31bf3856ad364e35" versionScope="nonSxS">
      <PreventDeviceEncryption>true</PreventDeviceEncryption>
    </component>
    <component name="Microsoft-Windows-EnhancedStorage-Adm" processorArchitecture="amd64" language="neutral" xmlns:wcm="http://schemas.microsoft.com/WMIConfig/2002/State" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" publicKeyToken="31bf3856ad364e35" versionScope="nonSxS">
      <TCGSecurityActivationDisabled>1</TCGSecurityActivationDisabled>
    </component>
  </settings>
</unattend>
```

还可以使用应答文件生成工具：[https://schneegans.de/windows/unattend-generator/](https://schneegans.de/windows/unattend-generator/)

应答文件放在： `sources\$OEM$\$$\Panther\unattend.xml`

## 重新制作 iso
在 Windows 平台，安装 [Windows Assessment and Deployment Kit (ADK)](https://learn.microsoft.com/en-us/windows-hardware/get-started/adk-install)。安装时只需选择 "部署工具"。

使用管理员运行以下 powershell 脚本：

```powershell
# 定义路径
$SourceISOPath = "path\to\Win11_24H2_Chinese_Simplified_x64.iso"
$UnattendFilePath = ".\unattend.xml"
$NewISOPath = ".\Win11_24H2_Chinese_Simplified_x64.with_AutoUnattend.iso"
$TempExtractPath = ".\temp\Win11Extract"
$TargetUnattendFilePath = "$TempExtractPath" + '\sources\$OEM$\$$\Panther\unattend.xml'
$OscdimgPath = "C:\Program Files (x86)\Windows Kits\10\Assessment and Deployment Kit\Deployment Tools\amd64\Oscdimg"

# 创建临时目录
New-Item -ItemType Directory -Path $TempExtractPath -Force

# 挂载并复制 ISO 内容
$isoImage = Mount-DiskImage -ImagePath $SourceISOPath -PassThru
$driveLetter = ($isoImage | Get-Volume).DriveLetter + ":"
Copy-Item -Path "$driveLetter\*" -Destination $TargetUnattendPath $TempExtractPath -Recurse -Force
Dismount-DiskImage -ImagePath $SourceISOPath

# 复制应答文件
Copy-Item -Path $UnattendFilePath -Destination $TargetUnattendFilePath -Force

# 创建新 ISO
# UEFI
&"$OscdimgPath\oscdimg.exe" -m -u2 -udfver102 -b"$OscdimgPath\efisys.bin" -l"WIN11_UEFI" $TempExtractPath $NewISOPath
# BIOS
# &"$OscdimgPath\oscdimg.exe" -m -u2 -udfver102 -b"$OscdimgPath\etfsboot.com" -l"Win11_Install" $TempExtractPath $NewISOPath

Write-Host "新的 ISO 已创建: $NewISOPath" -ForegroundColor Green
    
Pause

# 清理
Remove-Item -Path $TempExtractPath -Recurse -Force
```

在创建新 ISO 的时候，需要指定引导文件。

* 对于 BIOS: 位于 ADK 的 oscdimg.exe 同级目录中: etfsboot.com ； Windows 11 ISO 中也有: \boot\etfsboot.com
* 对于 UEFI: 位于 ADK 的 oscdimg.exe 同级目录中: efisys.bin ； Windows 11 ISO 中也有: \efi\microsoft\boot\efisys.bin

这样， ISO 就制作完成了。
