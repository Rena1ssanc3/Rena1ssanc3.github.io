---
title: HomeLab系列（二） - 网络
date: 2022-10-23 19:21:40
tags: HomeLab Network
---

# 搭建HomeLab的网络需要关心什么

## 网络拓扑
　　最简单的一种网络拓扑：一台路由器下面挂的设备，可以互相通过IP访问，这个网络可以用吗？可以。  
![](/static/images/2022/10/23/1.png)  
　　但是我们加一点需求，我人在公司使用笔记本，希望访问这台路由器下的设备，怎么办？这个时候笔记本需要通过公网访问路由器，路由器再通过内网访问设备。有几种方式可以实现？  
![](/static/images/2022/10/23/2.png)  
　　第一种方式就是端口映射，将希望访问的设备通过路由器的端口映射到公网，这样笔记本就可以通过公网访问路由器下的设备了。  
![](/static/images/2022/10/23/3.png)  
　　这种方式首先要求你有公网IP地址，其次是需要将路由器的端口映射到公网。直接暴露端口容易产生安全隐患，所以需要做好安全考量。想体验一下公网的黑暗森林，可以试着开一台虚拟机，将虚拟机的SSH端口映射到公网。一天下来，你可以收到来自世界各地无数狂热的登陆请求。  
　　第二种方式就是通过VPN。将笔记本连接到路由器的VPN。这样笔记本就可以通过VPN访问路由器下的设备了。  
![](/static/images/2022/10/23/4.png)
　　我们注意到这里仍旧需要要求你的路由器有公网IP。如果你的路由器没有公网IP，这个方案似乎就完全行不通了。那有没有不用公网IP的方案呢？这里就需要UDP打洞了，详见[Tailscale文档](https://tailscale.com/blog/how-nat-traversal-works/)。  
　　进一步的，当接入节点不再是一台Laptop，而是另一台Router的时候，我们就需要有一个新的网络方案，SD-WAN。  
![](/static/images/2022/10/23/5.png)

## 网络安全
　　不妨假设我们现在提供了一个接入方式，就需要考虑如何保障网络安全。首先是我们需要考虑对于接入者的鉴权能力，阻止非法访问。其次是在连接建立以后，需要防止第三方能够监听到我们的通信内容。这两点是HomeLab网络安全的重要考量因素。

## 网络性能
　　同一个内网下，设备间互访基本只通过交换机，这个时候吞吐量和延迟都是非常低的。但是当我们需要通过公网访问内网设备的时候，这个时候吞吐量和延迟就是问题了。我们会受到以下限制：
* 公网带宽
* 公网路由
* 路由器加解密性能
* 中转服务器能力  

　　如何尽可能的控制这些开销，是网络结构布置的时候需要考虑的。

# HomeLab网络搭建的几种主流方案

## OpenVPN
>    　　OpenVPN是一个开源的VPN软件，可以在Linux、Windows、MacOS、Android、iOS等平台上运行。OpenVPN的优点是可以在不同的平台上运行，而且可以通过OpenVPN Connect App在手机上运行。OpenVPN的缺点是需要自己搭建VPN服务器，而且需要自己管理证书。  

　　OpenVPN 是一个基于 OpenSSL 库的应用层 VPN 实现。和传统 VPN 相比，它的优点是简单易用。

## WireGuard
>    　　WireGuard ®是一种极其简单但快速且现代的 VPN，它利用了最先进的加密技术。它的目标是比 IPsec更快、更简单、更精简和更有用，同时避免令人头疼的问题。它打算比 OpenVPN 性能要高得多。WireGuard 被设计为在嵌入式接口和超级计算机等上运行的通用 VPN，适用于许多不同的环境。最初是为 Linux 内核发布的，现在它是跨平台的（Windows、macOS、BSD、iOS、Android）并且可广泛部署。它目前正在大力开发中，但它已经被认为是业内最安全、最容易使用和最简单的 VPN 解决方案。  

　　WireGuard是一款高性能的VPN软件，它的优点是简单易用，而且性能非常好。缺点是目前只支持Linux内核。

## TailScale

>Tailscale 是一项 VPN 服务，可让您在世界任何地方安全、轻松地访问您拥有的设备和应用程序。它使用开源WireGuard协议启用加密的点对点连接，这意味着只有您的私有网络上的设备才能相互通信。Tailscale还提供了一个简单的管理界面，您可以使用它来管理您的设备和网络。

　　TailScale是一款基于WireGuard的VPN软件。
![](/static/images/2022/10/23/6.png)
　　可惜的是，TailScale的免费版本只有如下支持：
>For personal & hobby projects  
1 user  
20 devices  
1 subnet router  
Secure, peer-to-peer connections  
SSO and MFA  
Sharing, MagicDNS, and more  

　　对于个人用户来说，这个限制还是太多了。

## ZeroTier

> ZeroTier 是一款适用于地球的智能以太网交换机。
它是一个分布式网络管理程序，构建在加密安全的全球对等网络之上。它提供与企业 SDN 交换机相当的高级网络虚拟化和管理功能，但跨局域网和广域网并连接几乎任何类型的应用程序或设备。

　　ZeroTier的免费版本支持就奢华的多了：
> Admin: 1      
Nodes: 25       
Networks: Unlimited                                         
Business SSO: n/a                                         
Support: Community           

　　更骚的是，他还有OpenSource版本，可以自己搭建服务器，支持不限Admin和Nodes……

# ZeroTier在HomeLab网络上的实施

　　首先我们需要根据情况准备几个Edge Router。我用了两台OrangePi R1 Plus LTS作为Edge Router。这两台设备都刷入OpenWrt系统，所以可以直接安装ZeroTier。然后为这两台Edge Router分配一下网段信息。其中一台使用这个网段172.26.0.1/16，另一台则使用172.28.0.1/16的网段。通过ZeroTier我们允许这两台Edge Router互相通信。  
　　在ZeroTier的管理界面上，通过Create A Network创建一个网络。别忘了将网络的Access Control设置为Private，这样节点只有通过授权才能加入这个网络。然后在Edge Router上操作，join到这个网络中。我们可以在Members里给两个Edge Router分别勾上Auth，节点就正式加入我们的网络了。现在，我们通过ZeroTier实现两台Edge Router之间的互联了。  
　　此时Edge Router之间是联通的，但我们Edge Router下的设备还是无法互联。所以我们需要在Edge Router上配置一下路由表。在ZeroTier的控制面板上，根据两台Edge Router的IP配置如下路由表：
>  172.26.0.0/16  via  $EdgeRoute1_IP  
  172.28.0.0/16  via  $EdgeRoute2_IP  

　　这样我们就拥有了一个可用的HomeLab网络了。

# HomeLab网络的各维度评估

## 拓扑
　　ZeroTier是通过UDP打洞的方式实现的。我们可以将Edge Router看作是一个个的ZeroTier节点。如果Edge Router之间的UDP打洞成功，那么就是互相直连的状态。如果打洞失败，那么就是通过ZeroTier的中心服务器进行转发的状态。这是我们不希望看到的，但是也是可以接受的。在Edge Router上，可以通过如下命令查看节点之间的连接状态：
```bash
zerotier-cli peers
```
　　如果link是DIRECT，那么就是互相直连的状态。

## 安全
　　ZeroTier的网络是通过加密的。
>If you don’t know much about cryptography you can safely skip this section. TL;DR: packets are end-to-end encrypted and can’t be read by roots or anyone else, and we use modern 256-bit crypto in ways recommended by the professional cryptographers that created it.  
Asymmetric public key encryption is Curve25519/Ed25519, a 256-bit elliptic curve variant.  
Every VL1 packet is encrypted end to end using (as of the current version) 256-bit Salsa20 and authenticated using the Poly1305 message authentication (MAC) algorithm. MAC is computed after encryption (encrypt-then-MAC) and the cipher/MAC composition used is identical to the NaCl reference implementation.

　　太长不看，Curve25519非对称加密用于点对点密钥交换，配合256bit的Salsa20+Poly1305加密数据包。很安全，放心用，但是没有前向安全性保证。
　　其次是对于Private网络，需要Admin账号在Control Panel上进行授权，这样才能加入网络。还有Flow Rules，可以对流量进行过滤。确实有需要做节点之间的流量限制的话，可以考虑使用这个功能。

## 性能
　　ZeroTier的性能是非常好的。如果两台Edge Router之间的UDP打洞成功，两台Edge Router之间的网络延迟和直接ping的延迟相差无几，带宽可以拉满公网上传峰值。除非什么时候开百兆上传，ZeroTier在一般情况下不是性能瓶颈所在。
> 正向ping:  
> 64 bytes from 172.26.0.1: seq=0 ttl=64 time=23.237 ms  
64 bytes from 172.26.0.1: seq=1 ttl=64 time=22.958 ms  
64 bytes from 172.26.0.1: seq=2 ttl=64 time=23.194 ms  
64 bytes from 172.26.0.1: seq=3 ttl=64 time=22.520 ms  
> 反向ping:  
>64 bytes from 172.28.0.1: seq=0 ttl=64 time=22.583 ms  
64 bytes from 172.28.0.1: seq=1 ttl=64 time=22.700 ms  
64 bytes from 172.28.0.1: seq=2 ttl=64 time=22.885 ms  
64 bytes from 172.28.0.1: seq=3 ttl=64 time=22.996 ms  
> 
> 正向iperf:  
> [  3] local xx.xx.xx.xx port 33238 connected with 172.28.0.1 port 5001  
[ ID] Interval       Transfer     Bandwidth  
[  3]  0.0-10.1 sec  28.6 MBytes  23.8 Mbits/sec  
> 反向iperf:  
> [  3] local xx.xx.xx.xx port 60216 connected with 172.26.0.1 port 5001  
[ ID] Interval       Transfer     Bandwidth  
[  3]  0.0-10.1 sec  27.6 MBytes  23.1 Mbits/sec  

## 扩展
　　我们这里以双节点的HomeLab网络搭建为例。如果我们需要扩展，只需要在Edge Router上join到这个网络中，进而在ZeroTier的Control Panel上勾选Auth，并配置新网段的路由，就可以实现新的节点加入了。  
　　任意设备接入到Edge Router后，分配到的IP地址，都可以在HomeLab网络内的任意设备上访问到。任意设备之间的互联就这样完成了。