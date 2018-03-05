# IPsec VPN 服务器一键安装脚本

[![Build Status](https://travis-ci.org/hwdsl2/setup-ipsec-vpn.svg?branch=master)](https://travis-ci.org/hwdsl2/setup-ipsec-vpn) [![GitHub Stars](https://img.shields.io/github/stars/hwdsl2/setup-ipsec-vpn.svg?maxAge=86400)](https://github.com/hwdsl2/setup-ipsec-vpn/stargazers) [![Docker Stars](https://img.shields.io/docker/stars/hwdsl2/ipsec-vpn-server.svg?maxAge=86400)](https://github.com/hwdsl2/docker-ipsec-vpn-server/blob/master/README-zh.md) [![Docker Pulls](https://img.shields.io/docker/pulls/hwdsl2/ipsec-vpn-server.svg?maxAge=86400)](https://github.com/hwdsl2/docker-ipsec-vpn-server/blob/master/README-zh.md)

使用 Linux 脚本一键快速搭建自己的 IPsec VPN 服务器。支持 IPsec/L2TP 和 Cisco IPsec 协议，可用于 Ubuntu/Debian/CentOS 系统。你只需提供自己的 VPN 登录凭证，然后运行脚本自动完成安装。

IPsec VPN 可以加密你的网络流量，以防止在通过因特网传送时，你和 VPN 服务器之间的任何人对你的数据的未经授权的访问。在使用不安全的网络时，这是特别有用的，例如在咖啡厅，机场或旅馆房间。

我们将使用 <a href="https://libreswan.org/" target="_blank">Libreswan</a> 作为 IPsec 服务器，以及 <a href="https://github.com/xelerance/xl2tpd" target="_blank">xl2tpd</a> 作为 L2TP 提供者。

<a href="https://blog.ls20.com/ipsec-l2tp-vpn-auto-setup-for-ubuntu-12-04-on-amazon-ec2/" target="_blank">**&raquo; 相关教程： IPsec VPN Server Auto Setup with Libreswan**</a>

*其他语言版本: [English](README.md), [简体中文](README-zh.md).*

#### 目录

- [快速开始](#快速开始)
- [功能特性](#功能特性)
- [系统要求](#系统要求)
- [安装说明](#安装说明)
- [下一步](#下一步)
- [重要提示](#重要提示)
- [升级Libreswan](#升级libreswan)
- [问题和反馈](#问题和反馈)
- [卸载说明](#卸载说明)
- [另见](#另见)
- [授权协议](#授权协议)

## 快速开始

首先，在你的 Linux 服务器[*](#quick-start-note) 上全新安装一个 Ubuntu LTS, Debian 或者 CentOS 系统。

使用以下命令快速搭建 IPsec VPN 服务器：

```bash
wget https://git.io/vpnsetup -O vpnsetup.sh && sudo sh vpnsetup.sh
```

如果使用 CentOS，请将上面的地址换成 `https://git.io/vpnsetup-centos`。

你的 VPN 登录凭证将会被自动随机生成，并在安装完成后显示在屏幕上。

如需了解其它安装选项，以及如何配置 VPN 客户端，请继续阅读以下部分。

<a name="quick-start-note"></a>
\* 一个专用服务器或者虚拟专用服务器 (VPS)。OpenVZ VPS 不受支持。

## 功能特性

- **新:** 增加支持更高效的 `IPsec/XAuth ("Cisco IPsec")` 模式
- **新:** 现在可以下载 VPN 服务器的预构建 <a href="https://github.com/hwdsl2/docker-ipsec-vpn-server/blob/master/README-zh.md" target="_blank">Docker 镜像</a>
- 全自动的 IPsec VPN 服务器配置，无需用户输入
- 封装所有的 VPN 流量在 UDP 协议，不需要 ESP 协议支持
- 可直接作为 Amazon EC2 实例创建时的用户数据使用
- 包含 `sysctl.conf` 优化设置，以达到更佳的传输性能
- 已测试： Ubuntu 16.04/14.04， Debian 9/8 和 CentOS 7/6

## 系统要求

一个新创建的 <a href="https://aws.amazon.com/ec2/" target="_blank">Amazon EC2</a> 实例，使用这些映像 (AMIs):
- <a href="https://cloud-images.ubuntu.com/locator/" target="_blank">Ubuntu 16.04 (Xenial) or 14.04 (Trusty)</a>
- <a href="https://wiki.debian.org/Cloud/AmazonEC2Image" target="_blank">Debian 9 (Stretch) or 8 (Jessie)</a>
- <a href="https://aws.amazon.com/marketplace/pp/B00O7WM7QW" target="_blank">CentOS 7 (x86_64) with Updates</a>
- <a href="https://aws.amazon.com/marketplace/pp/B00NQAYLWO" target="_blank">CentOS 6 (x86_64) with Updates</a>

请参见 <a href="https://blog.ls20.com/ipsec-l2tp-vpn-auto-setup-for-ubuntu-12-04-on-amazon-ec2/#vpnsetup" target="_blank">详细步骤</a> 以及 <a href="https://aws.amazon.com/cn/ec2/pricing/" target="_blank">EC2 定价细节</a>。

**-或者-**

一个专用服务器，或者基于 KVM/Xen 的虚拟专用服务器 (VPS)，全新安装以上操作系统之一。OpenVZ VPS 不受支持，用户可以另外尝试比如 <a href="https://shadowsocks.org" target="_blank">Shadowsocks</a> 或者 <a href="https://github.com/Nyr/openvpn-install" target="_blank">OpenVPN</a>。

这也包括各种公共云服务中的 Linux 虚拟机，比如 <a href="https://blog.ls20.com/digitalocean" target="_blank">DigitalOcean</a>, <a href="https://blog.ls20.com/vultr" target="_blank">Vultr</a>, <a href="https://blog.ls20.com/linode" target="_blank">Linode</a>, <a href="https://cloud.google.com/compute/" target="_blank">Google Compute Engine</a>, <a href="https://amazonlightsail.com" target="_blank">Amazon Lightsail</a>, <a href="https://azure.microsoft.com" target="_blank">Microsoft Azure</a>, <a href="https://www.ibm.com/cloud-computing/bluemix/virtual-servers" target="_blank">IBM Bluemix</a>, <a href="https://www.ovh.com/us/vps/" target="_blank">OVH</a> 和 <a href="https://www.rackspace.com" target="_blank">Rackspace</a>。

<a href="azure/README-zh.md" target="_blank"><img src="docs/images/azure-deploy-button.png" alt="Deploy to Azure" /></a> <a href="一个专用服务器，或者基于 KVM/Xen 的虚拟专用服务器 (VPS)，全新安装以上操作系统之一。OpenVZ VPS 不受支持，用户可以另外尝试比如 <a href="https://shadowsocks.org" target="_blank">Shadowsocks</a> 或者 <a href="https://github.com/Nyr/openvpn-install" target="_blank">OpenVPN</a>。

这也包括各种公共云服务中的 Linux 虚拟机，比如 <a href="https://blog.ls20.com/digitalocean" target="_blank">DigitalOcean</a>, <a href="https://blog.ls20.com/vultr" target="_blank">Vultr</a>, <a href="https://blog.ls20.com/linode" target="_blank">Linode</a>, <a href="https://cloud.google.com/compute/" target="_blank">Google Compute Engine</a>, <a href="https://amazonlightsail.com" target="_blank">Amazon Lightsail</a>, <a href="https://azure.microsoft.com" target="_blank">Microsoft Azure</a>, <a href="https://www.ibm.com/cloud-computing/bluemix/virtual-servers" target="_blank">IBM Bluemix</a>, <a href="https://www.ovh.com/us/vps/" target="_blank">OVH</a> 和 <a href="https://www.rackspace.com" target="_blank">Rackspace</a>。

<a href="azure/README-zh.md" target="_blank"><img src="docs/images/azure-deploy-button.png" alt="Deploy to Azure" /></a> <a href="http://dovpn.carlfriess.com/" target="_blank"><img src="docs/images/do-install-button.png" alt="Install on DigitalOcean" /></a> <a href="https://www.linode.com/stackscripts/view/37239" target="_blank"><img src="docs/images/linode-deploy-button.png" alt="Deploy to Linode" /></a>
>>>>>>>+master
YOUR_USE一个专用服务器，或者任何基于 KVM/Xen 的虚拟专用服务器 (VPS)，全新安装以上系统之一。另外也可用 Debian 7 (Wheezy)，但是必须首先运行 <a href="extras/vpnsetup-debian-7-workaround.sh" target="_blank">另一个脚本</a>。 OpenVZ VPS 用户可以尝试使用 Shadowsocks ( <a href="https://github.com/shadowsocks/shadowsocks-libev" target="_blank">libev</a> | <a href="https://github.com/breakwa11/shadowsocks-rss" target="_blank">rss</a> ) 或者 <a href="https://github.com/Nyr/openvpn-install" target="_blank">OpenVPN</a>。
>>>>>>>-bb61197
将 `https://git.io/vpnsetup` 换成 `https://git.io/vpnsetup-centos`。

## 下一步

配置你的计算机或其它设备使用 VPN 。请参见：

<a href="docs/clients-zh.md" target="_blank">**配置 IPsec/L2TP VPN 客户端**</a>

<a href="docs/clients-xauth-zh.md" target="_blank">**配置 IPsec/XAuth ("Cisco IPsec") VPN 客户端**</a>

<a href="docs/ikev2-howto-zh.md" target="_blank">**如何配置 IKEv2 VPN: Windows 和 Android**</a>

如果在连接过程中遇到错误，请参见 <a href="docs/clients-zh.md#故障排除" target="_blank">故障排除</a>。

开始使用自己的专属 VPN ! :sparkles::tada::rocket::sparkles:

## 重要提示

<<<<<<< master
*其他语言版本: [English](README.md#important-notes), [简体中文](README-zh.md#重要提示).*

**Windows 用户** 在首次连接之前需要<a href="docs/clients-zh.md#windows-错误-809" target="_blank">修改注册表</a>，以解决 VPN 服务器 和/或 客户端与 NAT（比如家用路由器）的兼容问题。

同一个 VPN 账户可以在你的多个设备上使用。但是由于 IPsec/L2TP 的局限性以及一个在 Libreswan 中的<a href="https://github.com/libreswan/libreswan/issues/166" target="_blank">问题</a>，现在还不支持同时连接在同一个 NAT（比如家用路由器）后面的多个设备。

对于有外部防火墙的服务器（比如 <a href="https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-network-security.html" target="_blank">EC2</a>/<a href="https://cloud.google.com/compute/docs/vpc/firewalls" target="_blank">GCE</a>），请为 VPN 打开 UDP 端口 500 和 4500。

如果需要添加，修改或者删除 VPN 用户账户，请参见 <a href="docs/manage-users-zh.md" target="_blank">管理 VPN 用户</a>。

在 VPN 已连接时，客户端配置为使用 <a href="https://developers.google.com/speed/public-dns/" target="_blank">Google Public DNS</a>。如果偏好其它的域名解析服务，请编辑 `/etc/ppp/options.xl2tpd` 和 `/etc/ipsec.conf` 并替换 `8.8.8.8` 和 `8.8.4.4`。然后重启服务器。

使用 L2TP 内核支持有助于提高 IPsec/L2TP 性能。它在以下系统上可用： Ubuntu 16.04, Debian 9, CentOS 7 和 6。 Ubuntu 16.04 用户需要安装 `` linux-image-extra-`uname -r` `` 软件包并且重启 `xl2tpd` 服务。

在 VPN 已连接时，客户端配置为使用 <a href="https://developers.google.com/speed/public-dns/" target="_blank">Google Public DNS</a>。如果偏好其它的域名解析服务，请编辑 `/etc/ppp/options.xl2tpd` 和 `/etc/ipsec.conf` 并替换 `8.8.8.8` 和 `8.8.4.4`。然后重启服务器。

如果需要在安装后更改 IPTables 规则，请编辑 `/etc/iptables.rules` 和/或 `/etc/iptables/rules.v4` (Ubuntu/Debian)，或者 `/etc/sysconfig/iptables` (CentOS)。然后重启服务器。

在使用 `IPsec/L2TP` 连接时，VPN 服务器在虚拟网络 `192.168.42.0/24` 内具有 IP `192.168.42.1`。

这些脚本在更改现有的配置文件之前会先做备份，使用 `.old-日期-时间` 为文件名后缀。

## 升级Libreswan

提供两个额外的脚本 <a href="extras/vpnupgrade.sh" target="_blank">vpnupgrade.sh</a> 和 <a href="extras/vpnupgrade_centos*其他语言版本: [English](README.md#important-notes), [简体中文](README-zh.md#重要提示).*
>>>>>>>+master
href="ht**Windows 用户** 在首次连接之前需要<a href="docs/clients-zh.md#regkey" target="_blank">修改一次注册表</a>，以解决 VPN 服务器 和/或 客户端与 NAT （比如家用路由器）的兼容问题。如果在连接过程中遇到错误，请参见 <a href="docs/clients-zh.md#故障排除" target="_blank">故障排除</a>。
>>>>>>>-bb61197
-O vpnupgrade.sh
```

## 问题和反馈

- 有问题需要提问？请先搜索已有的留言，在 <a href="https://gist.github.com/hwdsl2/9030462#comments" target="_blank">这个 Gist</a> 以及 <a href="https://blog.ls20.com/ipsec-l2tp-vpn-auto-setup-for-ubuntu-12-04-on-amazon-ec2/#disqus_thread" target="_blank">我的博客</a>。
- VPN 的相关问题可在 <a href="https://lists.libreswan.org/mailman/listinfo/swan" target="_blank">Libreswan</a> 或 <a href="https://lists.strongswan.org/mailman/listinfo/users" target="_blank">strongSwan</a> 邮件列表提问，或者参考这些网站： <a href="https://libreswan.org/wiki/Main_Page" target="_blank">[1]</a> <a href="https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Security_Guide/sec-Securing_Virtual_Private_Networks.html" target="_blank">[2]</a> <a href="https://wiki.strongswan.org/projects/strongswan/wiki/UserDocumentation" target="_blank">[3]</a> <a href="https://wiki.gentoo.org/wiki/IPsec_L2TP_VPN_server" target="_blank">[4]</a> <a href="https://wiki.archlinux.org/index.php/L2TP/IPsec_VPN_client_setup" target="_blank">[5]</a>。
- 如果你发现了一个可重复的程序漏洞，请提交一个 <a href="https://github.com/hwdsl2/setup-ipsec-vpn/issues?q=is%3Aissue" target="_blank">GitHub Issue</a>。

## 卸载说明

请参见 <a href="docs/uninstall-zh.md" target="_blank">卸载 VPN</a>。

## 另见

- <a href="https://github.com/hwdsl2/docker-ipsec-vpn-server/blob/master/README-zh.md" target="_blank">IPsec VPN Server on Docker</a>
- <a href="https://github.com/gaomd/docker-ikev2-vpn-server" target="_blank">IKEv2 VPN Server on Docker</a>
- <a href="https://github.com/jlund/streisand" target="_blank">Streisand</a>
- <a href="https://github.com/trailofbits/algo" target="_blank">Algo VPN</a>
- <a href="https://github.com/Nyr/openvpn-install" target="_blank">OpenVPN Install</a>

## 授权协议

版权所有 (C) 2014-2017 <a href="https://www.linkedin.com/in/linsongui" target="_blank">Lin Song</a> <a href="https://www.l如果需要在安装后更改 IPTables 规则，请编辑 `/etc/iptables.rules` 和/或 `/etc/iptables/rules.v4` (Ubuntu/Debian)，或者 `/etc/sysconfig/iptables` (CentOS)。然后重启服务器。

在使用 `IPsec/L2TP` 连接时，VPN 服务器在虚拟网络 `192.168.42.0/24` 内具有 IP `192.168.42.1`。
>>>>>>>+master
tp://creativecommons.org/licenses/by-sa/3.0/" target="_blank">知识共享署名-相同方式共享3.0</a> 许可协议授权。   
必须署名： 请包括我的名字在任何衍生产品，并且让我知道你是如何改善它的！
