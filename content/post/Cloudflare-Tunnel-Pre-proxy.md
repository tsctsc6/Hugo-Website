+++
date = '2025-05-13T16:00:00+08:00'
draft = false
title = '给 Cloudflare Tunnel 添加前置代理'
categories = ['Main Sections']
tags = ['Cloudflare']
mermaid = true
+++

引用文章: [Cloudflare Tunnel速度慢? 尝试给它加个前置代理提高速度](https://blog.xmgspace.me/archives/cloudflare-tunnel-via-proxy.html)

本文在此文章的基础上，结合实践，补充更多内容。

首先，此文章的方法的核心是使用 Linux 的路由表机制，而 Windows 原生是没有类似功能的，或者实现起来比较复杂。如果后端非要部署在 Windows 平台，那么可以这么做:

* 再额外配置一台 Linux 虚拟机，与宿主机一起在某一网络内。
* 把后端公开的那个网络中，通过 Linux 虚拟机访问后端，再转发到 Cloudflare Tunnel 。
* 配置 Cloudflare Tunnel 。

```mermaid
graph LR;
    A["Computer with backend"];
    B["Linux"];
    C["Cloudflare"];
    A <--> B;
    B <--> C;
```

## Step1. 安装 Xray-core
```Shell {name="Linux Machine"}
sudo bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install
```

众所周知，国内机器上安装Github上的软件是很痛苦的问题。如果实在连不上，就对 curl 使用代理：

```Shell {name="Linux Machine"}
sudo bash -c "$(curl -x "http://ip:port" http -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install
```

但是这还不够，因为这个脚本是会下载 Xray-core 的，会继续访问 github ，下不动了。于是我们可以访问 [https://github.com/XTLS/Xray-install](https://github.com/XTLS/Xray-install) ，查看它有什么命令。查看这个脚本的帮助：

```Shell {name="Linux Machine"}
sudo bash -c "$(curl -x "http://ip:port" http -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ help
```

发现可以使用代理 ( `-p 代理地址` ) 和从本地文件安装 (`-l Xray-core.zip`) 。

## Step2. 配置 Xray-core
Xray-core 默认被安装在 `/usr/local/bin/xray` 中。

在 Linux 中，配置文件 `config.json` 通常位于 `/etc/xray/` 或 `/usr/local/etc/xray/` 目录下。

打开 `config.json` ，填写以下内容:

```json {name="config.json"}
{
  "inbounds": [
    {
      "port": 12345,
      "protocol": "dokodemo-door",
      "settings": {
        "network": "tcp,udp",
        "followRedirect": true
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom",
      "settings": {}
    }
  ],
  "routing": {
    "domainStrategy": "AsIs",
    "domainMatcher": "mph"
  }
}
```

启动 xray:

```Shell {name="Linux Machine"}
xray run -c /usr/local/etc/xray/config.json
```

可以把 xray 包装为守护进程。

## Step3. 配置 iptables
添加下列iptables规则:

```Shell {name="Linux Machine"}
sudo iptables -t nat -A OUTPUT -d 198.41.192.0/24 -p tcp -j REDIRECT --to-ports 12345
sudo iptables -t nat -A OUTPUT -d 198.41.200.0/24 -p tcp -j REDIRECT --to-ports 12345
```

为了在重启后保留规则，需要保存 iptables 配置。可以安装 iptables-persistent:

```Shell {name="Linux Machine"}
sudo apt install iptables-persistent
```

安装过程中会提示保存当前规则，选择“是”即可。如果需要手动保存规则:

```Shell {name="Linux Machine"}
sudo netfilter-persistent save
```

## Step4. 调优
Cloudflare Tunnel 会时不时连接到下面几个域名来完成类似检查更新，汇报信息的功能，好在是使用 443(HTTPS)端口来进行通信，可以自选 IP 。

* api.cloudflare.com
* update.argotunnel.com
* <你的团队名称_需自己修改>.cloudflareaccess.com
* pqtunnels.cloudflareresearch.com

查看团队名称，可以在 Cloudflare 仪表盘 -> Zero Trust -> 设置 -> 自定义页面看到。

将上述域名放到 hosts 里面自选 IP ，或者直接在 hosts 中写死我们透明代理加速过的 198.41.192.0/24 和 198.41.200.0/24 中的 IP 也可以。

打开 hosts 文件:

```Shell {name="Linux Machine"}
sudo nano /etc/hosts
```

向文件添加以下内容，保存并退出编辑器。

``` {name="Linux Machine"}
104.17.18.19 api.cloudflare.com
104.17.18.19 update.argotunnel.com
104.17.18.19 pqtunnels.cloudflareresearch.com
104.17.18.19 <你的团队名称_需自己修改>.cloudflareaccess.com
```

至于 Cloudflared 的配置文件 `config.yml` ， ChatGPT 说位于 `/etc/cloudflared/` ，我并没有找到。

后来看到一篇博客：[Cloudflared Tunnel 隧道开启 HTTP2 和 IPv6 支持](https://blog.lufei.de/p/394/)，请参照该文章开启。

## Step5. 验收
重启cloudflared，查看效果:

```Shell {name="Linux Machine"}
sudo systemctl restart cloudflared
```

当然，由于网络和线路的不稳定性，网络波动可能会导致 Cloudflared 掉线。而似乎 Cloudflared 的断线重连做的不好，常常重连失败或者重连没有经过代理。最好的办法还是定时重启 Cloudflared ，辅以相对稳定的优化线路。

重启 Cloudflared 可以使用 crontab 实现:

```Shell {name="Linux Machine"}
crontab -e #编辑crontab
 
# 将下列代码粘贴进编辑器中，作用是每6小时重启一次cloudflareds
00 */6 *   *   *    sudo service cloudflared restart
```