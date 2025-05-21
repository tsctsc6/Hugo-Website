+++
date = '2025-05-13T15:52:01+08:00'
draft = false
title = 'Debian 常用命令'
categories = ['Main Sections']
tags = ['Linux']
+++

## 开启 SSH
```shell
sudo apt install openssh-server
```

如果同一个 IP 的主机已经改变，可以使用以下命令来删除历史记录:

```
ssh-keygen -R {IP}
```

## 命令行读写文本文件
```shell
sudo nano file.txt
```

## 普通用户无法使用 sudo 提权
先通过普通用户登入， ssh 连接或者 Debian 本地终端也行，输入以下命令:

```shell
su -
```

查看文件:

```shell
sudo nano visudo
```

找到 `%sudo ALL=(ALL:ALL) ALL` 这行，在它的下一行我们输入一行文本:

```shell
{你要提权的账号} ALL=(ALL:ALL) ALL
```

这样就赋予账号可以使用 `sudo` 的权限。

## 图形界面的开关
查看当前的显示管理器

```shell
systemctl status display-manager
```

禁用显示管理器(如果显示管理器是 gdm3)

```shell
sudo systemctl disable gdm3
```

恢复图形界面(如果显示管理器是 gdm3)

```shell
sudo systemctl start gdm3
```

## 开机
这里使用安装在 Windows 的 Vmvare 。

我们可以在 Vmvare 的安装目录找到 `vmrun.exe` 。

```shell
vmrun.exe start "{你的虚拟机.vmx文件路径}" nogui
```

## 关机
```shell
sudo poweroff
```

或者

```shell
sudo shutdown -h now
```

或者(五分钟后关机)

```shell
sudo shutdown -h +5
```

## 查看系统资源使用情况
总览

```shell
top
```

内存

```shell
free
```

## 使用 `{keyword}` 搜索进程
```shell
ps -ef | grep {keyword} | grep -v "grep"
```

## 使用服务
假设你有命令 `{command}` (命令的路径需要绝对路径)，创建服务:

```shell
sudo nano /etc/systemd/system/{service-name}.service
```

在文件中填写:

``` {name = ".service"}
[Unit]
Description=Run my startup command
After=network.target

[Service]
ExecStart={command}
Restart=always

[Install]
WantedBy=multi-user.target
```

保存并退出，启动服务

```shell
sudo systemctl daemon-reload
sudo systemctl enable {service-name}.service
sudo systemctl start {service-name}.service
```

查看服务状态:

```shell
sudo systemctl status {service-name}.service
```

查看服务的最新输出:

```shell
sudo systemctl status {service-name}.service | tail
```

## 安装 crt 证书
将证书复制到系统固定的位置:

```shell
sudo cp /path/to/your/certificate.crt /usr/local/share/ca-certificates/
```

更新证书存储:

```shell
sudo update-ca-certificates
```

验证证书安装:

```shell
openssl s_client -connect example.com:443
```

如果证书安装成功，您将看到证书链中的证书包含了您安装的 CRT 证书。

某些应用可能需要重新启动才能识别新安装的证书，例如 Web 浏览器或 curl 等工具。
