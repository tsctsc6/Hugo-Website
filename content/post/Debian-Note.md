+++
date = '2025-05-13T15:52:01+08:00'
lastmod = '2025-11-23T18:07:25+08:00'
draft = false
title = 'Debian 常用命令'
categories = ['Main Sections']
tags = ['Linux']
+++

## 新安装系统

### 开启 SSH

```shell
sudo apt install openssh-server
```

如果同一个 IP 的主机已经改变，可以使用以下命令来删除历史记录:

```
ssh-keygen -R {IP}
```

### 普通用户无法使用 sudo 提权

先通过普通用户登入， ssh 连接或者 Debian 本地终端也行，输入以下命令:

```shell
su -
```

然后输入 **root 的密码**。

查看文件:

```shell
nano /etc/sudoers
```

找到 `%sudo ALL=(ALL:ALL) ALL` 这行，在它的下一行我们输入一行文本:

```shell
{你要提权的账号} ALL=(ALL:ALL) ALL
```

这样就赋予账号可以使用 `sudo` 的权限。

### 移除 apt cd 源

在使用 apt 安装时，可能会从 cd 搜索软件包；但此时如果 cd 拔出，就会卡住。

打开源配置文件：

```shell
sudo nano /etc/apt/sources.list
```

把第一行（也就是写着 "cdrom" 的行）注释掉。

### 使用公钥远程登录

#### 生成密钥对

推荐使用 ed25519 算法。

```shell
# 在 Windows 和 Debian 都能使用
ssh-keygen -t ed25519 -C "<your@email>" -f ~/.ssh/<file-name>
```

执行该命令后，会询问 passphrase：强烈建议设置（为密钥加一个解锁口令）。

结果会生成 `~/.ssh/<file-name>`（私钥）和 `~/.ssh/<file-name>.pub`（公钥）。

#### 把公钥上传到 Debian （服务端）

把公钥文件的内容，放在 Debian 的 ~/.ssh/authorized_keys 文件中。

在拥有公钥的设备，输出公钥内容：

```shell
cat ~/.ssh/<file-name>.pub
```

```shell
# 假设已经通过密码远程登录 Debian
# 将公钥粘贴到 ~/.ssh/authorized_keys（每个密钥一行）
mkdir -p ~/.ssh
echo "ssh-ed25519 AAAA.... your@email" >> ~/.ssh/authorized_keys
# 设置公钥文件的权限，因为 Debian 对 ssh 公钥的权限要求严格
chmod 600 ~/.ssh/authorized_keys
# 设置 .ssh 文件夹的权限，因为 Debian 对 ssh 公钥的权限要求严格
chmod 700 ~/.ssh
chown -R <user>:<user> ~/.ssh
```

#### 验证公钥能否登录

```shell
ssh -i ~/.ssh/<file-name> <user>@<Debian-address>
```

#### 加固 ssh 配置

打开 ssh 配置文件：

```shell
sudo nano /etc/ssh/sshd_config
```

```planetext
# 允许公钥认证
PubkeyAuthentication yes

# 不允许空口令
PermitEmptyPasswords no

# 禁止 root 密码登录（保留 root 的密钥登录可选）
PermitRootLogin prohibit-password    # 或 'no' 更严格

# 关闭基于密码的认证（在确认公钥可用后启用）
PasswordAuthentication no
```

保存后重启 ssh 服务：

```shell
sudo systemctl restart ssh
```

> [!warning]
> 设置生效后，应保留当前 ssh 连接，另外再开一个 ssh 连接，尝试登录，以验证配置是否正确。

## 常用命令

### 命令行读写文本文件

```shell
sudo nano file.txt
```

### 设置代理

有些程序不会走系统代理，需要手动指定代理。大部分程序，会读取环境变量，来指定代理。

```shell
export HTTP_PROXY="http://127.0.0.1:10808"
export HTTPS_PROXY="http://127.0.0.1:10808"
export NO_PROXY="localhost,127.0.0.1,::1"
```

```powershell
$Env:HTTP_PROXY = "http://127.0.0.1:10808"
$Env:HTTPS_PROXY = "http://127.0.0.1:10808"
$Env:NO_PROXY = "localhost,127.0.0.1,::1"
```

> [!Warning]
> 这样设置仅在单个会话中有效。

想要持久化设置代理，编辑 ~/.bashrc 文件，在末尾加上：

```shell
export HTTP_PROXY="http://127.0.0.1:10808"
export HTTPS_PROXY="http://127.0.0.1:10808"
export NO_PROXY="localhost,127.0.0.1,::1"
```

### 图形界面的开关

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

### 开机

这里使用安装在 Windows 的 Vmvare 。

我们可以在 Vmvare 的安装目录找到 `vmrun.exe` 。

```shell
vmrun.exe start "{你的虚拟机.vmx文件路径}" nogui
```

### 关机

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

### 重启

```shell
sudo reboot
```

### 查看系统资源使用情况

总览

```shell
top
```

内存

```shell
free
```

### 使用 `{keyword}` 搜索进程

```shell
ps -ef | grep {keyword} | grep -v "grep"
```

### 使用服务

假设你有命令 `{command}` (命令的路径需要绝对路径)，创建服务:

```shell
sudo nano /etc/systemd/system/{service-name}.service
```

在文件中填写:

```{name = ".service"}
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

### 安装 crt 证书

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
