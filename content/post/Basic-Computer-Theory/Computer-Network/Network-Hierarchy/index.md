+++
date = '2025-05-20T16:27:20+08:00'
draft = false
title = '计算机网络-网络层次'
categories = ['Sub Sections']
+++

<style>
table.center tr, table.center td, table.center th {
    text-align: center;
}
</style>

网络的目的，是让两台（多台）计算机，能够相互通信。两台计算机之间的通信，是一个复杂的过程。对于这个复杂的过程，我们将其划分为多个步骤，每一个步骤只做一类事情。一个步骤就是一个层次。

## 网络层次的划分
网络层次的划分有以下几种：

<table align="center" valign="center" class="center">
    <tr>
        <th>OSI/RM 7 层模型</th>
        <th>TCP/IP 5 层模型</th>
        <th>TCP/IP 4 层模型</th>
    </tr>
    <tr>
        <td>应用层</td>
        <td rowspan="3">应用层</td>
        <td rowspan="3">应用层</td>
    </tr>
    <tr>
        <td>表示层</td>
    </tr>
    <tr>
        <td>会话层</td>
    </tr>
    <tr>
        <td>传输层</td>
        <td>传输层</td>
        <td>传输层</td>
    </tr>
    <tr>
        <td>网络层</td>
        <td>网络层(IP层)</td>
        <td>网络层(IP层)</td>
    </tr>
    <tr>
        <td>数据链路层</td>
        <td>数据链路层</td>
        <td rowspan="2">网络接口层</td>
    </tr>
    <tr>
        <td>物理层</td>
        <td>物理层</td>
    </tr>
</table>

OSI/RM 7 层模型其实有点啰嗦了，在实际应用中，往往采用 TCP/IP 5 层模型或 TCP/IP 4 层模型。

每一层中都要自己的专属协议，完成自己相应的工作以及与上下层级之间进行沟通。我们以 OSI/RM 7 层模型为例一一介绍。

<table align="center" valign="center" class="center">
    <tr>
        <th>OSI/RM 7 层模型</th>
        <th>协议</th>
        <th>重要设备</th>
        <th>说明</th>
    </tr>
    <tr>
        <td>应用层</td>
        <td rowspan="3">HTTP, SMTP, FTP, DNS, RPC, SSH 等</td>
        <td rowspan="3">无特定设备，由软件解析</td>
        <td>为操作系统或网络应用程序提供访问网络服务的接口。</td>
    </tr>
    <tr>
        <td>表示层</td>
        <td>表示层对上层数据或信息进行变换以保证一个主机应用层信息可以被另一个主机的应用程序理解。表示层的数据转换包括数据的加密、压缩、格式转换等。</td>
    </tr>
    <tr>
        <td>会话层</td>
        <td>会话层管理主机之间的会话进程，即负责建立、管理、终止进程之间的会话。会话层还利用在数据中插入校验点来实现数据的同步。</td>
    </tr>
    <tr>
        <td>传输层</td>
        <td>TCP, UDP</td>
        <td>网关</td>
        <td>负责主机到主机之间的数据传输。</td>
    </tr>
    <tr>
        <td>网络层</td>
        <td>IP, 与之配套的协议有 ICMP, ARP, RARP 等</td>
        <td>路由器</td>
        <td>负责设备寻址以及数据传输路径的选择。</td>
    </tr>
    <tr>
        <td>数据链路层</td>
        <td>网络接口协议</td>
        <td>网桥，交换机</td>
        <td>将数据可靠地传输到相邻设备。</td>
    </tr>
    <tr>
        <td>物理层</td>
        <td>以太网，令牌环等</td>
        <td>中继器，放大器</td>
        <td>确保原始的数据可在各种物理媒体上传输。</td>
    </tr>
</table>

PS1: 在计算机网络中，套接字(Socket)是网络通信过程中端点的抽象表示，包含进行网络通信所需的五种信息：连接使用的协议(TCP or UDP)，本地节点的 IP 地址，本地进程的端口号，远程节点的 IP 地址和远程进程的端口号。传输层及以下由操作系统以及网卡实现。在一般的编程中，编程语言提供的最底层的网络接口就是套接字。编程语言中的套接字，就是通过调用操作系统提供的接口实现的。

PS2: 其实网络层只有一种协议，就是 IP 。即使后来出现了 IPv6 ，那也是 IP 。所以有两句话：Everything is over IP! 和 IP over Everything!

PS3: 传输层、网络层、数据链路层和物理层不使用任何安全机制，传输明文数据。除了应用层的部分协议有安全机制(如 HTTPS, SSH)。