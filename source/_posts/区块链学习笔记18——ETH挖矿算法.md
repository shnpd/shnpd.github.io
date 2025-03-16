---
title: 区块链学习笔记18——ETH挖矿算法
toc: true
date: 2022-01-19 14:16:51
tags: 区块链 以太坊 数字货币
categories: 以太坊
---

​​点击阅读更多查看文章内容<!--more-->

本文转载自：[文章来源](https://superxlcr.github.io/2018/07/01/%E4%B8%8A%E7%BD%91%E9%99%90%E5%88%B6%E5%92%8C%E7%BF%BB%E5%A2%99%E5%9F%BA%E6%9C%AC%E5%8E%9F%E7%90%86/)
目前在国内基本访问不了google站点和android的站点，下载个gradle都要等很久。所以如果不翻墙很多工作都没办法正常做。所以在学习翻墙的同时也顺便了解了下目前限制网络访问的一些基本知识。

网络限制和监控应该说大家都有体会。比如很多公司都会限制一些网站的访问，比如网盘、视屏网站。有时也会对你访问的内容进行监控。还有一些公共WIFI，可能限制你只能访问80端口。在比如在国内无法访问google，facebook，android等网站。要想绕过这些限制，必须先知道他们是如何限制的。

本文主要是从技术角度来了解的网络限制方式和应对方式。并不做任何翻墙方式的推荐和指导。对于网络的知识，还是停留在上过网络课 的水平，文章内容也都是自己了解后总结的。可能会有错误和遗漏。会定期更新。

# DNS污染和劫持
以下解释来自百度百科：
某些网络运营商为了某些目的，对DNS进行了某些操作，导致使用ISP的正常上网设置无法通过域名取得正确的IP地址。
某些国家或地区出于某些目的为了防止某网站被访问，而且其又掌握部分国际DNS根目录服务器或镜像，也会利用此方法进行屏蔽。

目前我们访问网站主要都是通过域名进行访问，而真正访问这个网站前需要通过DNS服务器把域名解析为IP地址。而普通的DNS服务使用UDP协议，没有任何的认证机制。DNS劫持是指返回给你一个伪造页面的IP地址，DNS污染是返回给你一个不存在的页面的IP地址。

比如你使用电信、联通、移动的宽带，默认你是不需要设置任何DNS服务器的。这些DNS服务器由他们提供。一旦检测到你访问的网页是不允许的访问的，就会返回一个不存在的网页。而很多运营商也会使用DNS劫持来投放一些广告。

解决办法：

1. 使用OpenDNS（208.67.222.222）或GoogleDNS（8.8.8.8）（现在不太好用，被封锁，速度慢）
2. 使用一些第三方的DNS服务器
3. 自己用VPS搭建DNS服务器
4. 修改机器host文件，直接IP访问

# 封锁IP
通过上面一些方式，可以绕过DNS污染，通过IP地址访问无法访问的网页。但是目前针对IP进行大范围的封锁。虽然google这种大公司有很多镜像IP地址，但是目前基本全部被封锁掉，有漏网的可能也坚持不了多久。而且很多小公司的服务是部署在一些第三方的主机上，所以封锁IP有时会误伤，封锁一个IP导致主机上本来可以使用的页面也无法访问了。

不过目前不可能把所有国外的IP全部封锁掉，所以我们采用机会从国内连接到国外的VPS，进行翻墙。

解决办法：

1. 使用VPS搭建代理
2. 使用IPV6 （IPV6地址巨大，采用封地址不现实，但是目前国内只有部分高校部署了IPV6）

# 封锁HTTP代理
对于没有办法搭建VPS的人来说，最好的办法就是使用HTTP代理。客户端不在直接请求目标服务器，而是请求代理服务器，代理服务器在去请求目标服务器。然后返回结果。
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303162724427.png)
对于HTTP代理来说，封锁起来非常简单。因为HTTP协议是明文，Request Message中就带有要请求的URL或IP地址，这样很容易就被检测到。对于HTTPS来说，虽然通信是进行加密了，但是在建连之前会给代理服务器发送CONNECT方法，这里也会带上要访问的远端服务器地址。如果代理服务器在国外，在出去前就会被检测到。 如果代理服务器在国内，呵呵，你也出不去啊。

对于HTTP代理，因为是明文，所以很容易被服务器了解你的一些数据。**所以不要随便使用第三方的HTTP代理访问HTTP网站，而HTTPS虽然不知道你的数据，但是可以知道你去了那里。**

解决办法：

1. 使用VPS搭建VPN
2. 使用第三方VPN
# 封锁VPN
**虚拟专用网（英语：Virtual Private Network，简称VPN）**，是一种常用于连接中、大型企业或团体与团体间的私人网络的通讯方法。虚拟私人网络的讯息透过公用的网络架构（例如：互联网）来传送内联网的网络讯息。它利用已加密的通道协议（Tunneling Protocol）来达到保密、发送端认证、消息准确性等私人消息安全效果。
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303162734628.png)
正常网络通信时，所有网络请求都是通过我们的物理网卡直接发送出去。而VPN是客户端使用相应的VPN协议先与VPN服务器进行通信，成功连接后就在操作系统内建立一个虚拟网卡，一般来说默认PC上所有网络通信都从这虚拟网卡上进出，经过VPN服务器中转之后再到达目的地。
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303162736499.png)
通常VPN协议都会对数据流进行强加密处理，从而使得第三方无法知道数据内容，这样就实现了翻墙。翻墙时VPN服务器知道你干的所有事情（HTTP，对于HTTPS，它知道你去了哪)。

VPN有多种协议：OPENVPN、PPTP、L2TP/IPSec、SSLVPN、IKEv2 VPN，Cisco VPN等。其中的PPTP和L2TP是明文传输协议。只负责传输，不负责加密。分别利用了MPPE和IPSec进行加密。

对于VPN和其他一些加密的传输的协议来说，没有办法直接获取明文的请求信息，所以没有办法直接封锁，而是使用了监控的方式：

### 暴力破解
对于一些使用弱加密方式的协议来说，直接使用暴力破解检查传输内容。比如PPTP使用MPPE加密，但是MPPE是基于RC4，对于强大的防火墙背后的超级计算机集群，破解就是几秒钟的事情。

破解后明文中一旦包含了违禁内容，请求就会被封。而对应的IP可能会进入重点关怀列表。
### 特征检测
要想成功翻墙都必须与对应的远程服务器建立连接，然后再用对应的协议进行数据处理并传输。
而问题就出在这里：翻墙工具和远程服务器建立连接时，如果表现的很独特，在一大堆流量里很显眼，就会轻易被GFW识别出从而直接阻断连接，而VPN（尤其是OPENVPN）和SSH这方面的问题尤其严重。
### 流量监控
当一个VPN地址被大量人请求，并保持长时间连接时，就很容易引起关注。SSH接口有大量数据请求。一般会结合其他特征。
### 深度包检测
深度数据包检测（英语：Deep packet inspection，缩写为 DPI），又称完全数据包探测（complete packet inspection）或信息萃取（Information eXtraction，IX），是一种电脑网络数据包过滤技术，用来检查通过检测点之数据包的数据部分（亦可能包含其标头），以搜索不匹配规范之协议、病毒、垃圾邮件、入侵，或以预定之准则来决定数据包是否可通过或需被路由至其他不同目的地，亦或是为了收集统计数据之目的。

比如我们用HTTPS来访问一个网站，TLS/SSL协议在建连过程如下：
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303162713843.png)
很明显的会发送“client hello”和“server hello” 这种特诊很明显的信息。（当然不会根据这个就封掉，否则https没法用了）。而后续会有服务端证书发送，验证，客户端密钥协商等过程。有明显的协议特征。

下面是网上找的两张图：提醒大家最好不要随便用不安全的VPN来访问不合适的网页，开开android没啥问题。

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303162713865.png)
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303162756087.png)

# Socks代理
### Socks代理/SSH Socks
SOCKS是一种网络传输协议，主要用于客户端与外网服务器之间通讯的中间传递。SOCKS是”SOCKetS”的缩写。
当防火墙后的客户端要访问外部的服务器时，就跟SOCKS代理服务器连接。这个代理服务器控制客户端访问外网的资格，允许的话，就将客户端的请求发往外部的服务器。
这个协议最初由David Koblas开发，而后由NEC的Ying-Da Lee将其扩展到版本4。最新协议是版本5，与前一版本相比，增加支持UDP、验证，以及IPv6。
根据OSI模型，SOCKS是会话层的协议，位于表示层与传输层之间
### 与HTTP代理的对比
SOCKS工作在比HTTP代理更低的层次：SOCKS使用握手协议来通知代理软件其客户端试图进行的连接SOCKS，然后尽可能透明地进行操作，而常规代理可能会解释和重写报头（例如，使用另一种底层协议，例如FTP；然而，HTTP代理只是将HTTP请求转发到所需的HTTP服务器）。虽然HTTP代理有不同的使用模式，CONNECT方法允许转发TCP连接；然而，SOCKS代理还可以转发UDP流量和反向代理，而HTTP代理不能。HTTP代理通常更了解HTTP协议，执行更高层次的过滤（虽然通常只用于GET和POST方法，而不用于CONNECT方法）

Socks代理本身协议是明文传输，虽然相对HTTP有一些优势，但是明文也导致Socks代理很容易被封。所以可以考虑对Socks进行加密。所以出现了SSH Socks，对于MAC和Linux来说，不需要Client就可以进行访问。详细可以看：SSH隧道技术简介：端口转发&SOCKS代理

但是网上看有些地区好像会对一些VPS的SSH进行端口干扰。我在武汉好像SSH到我的VPS一会就会断。在上海一直没这问题。而且SSH一般是小流量数据，如果数据量特别大，也会被认为是翻墙，进入特别关怀列表。
 ### Shadowsocks
认准官网：https://shadowsocks.org/en/index.html （.com那个是卖账号的）

```bash
A secure socks5 proxy,
designed to protect your Internet traffic.
```

Shadowsocks 目前不容易被封杀主要是因为：

1. 建立在socks5协议之上，socks5是运用很广泛的协议，所以没办法直接封杀socks5协议
2. 使用socks5协议建立连接，而没有使用VPN中的服务端身份验证和密钥协商过程。而是在服务端和客户端直接写死密钥和加密算法。所以防火墙很难找到明显的特征，因为这就是个普通的socks5协议。
3. Shadowsock搭建也比较简单，所以很多人自己架设VPS搭建，个人使用流量也很小，没法通过流量监控方式封杀。
4. 自定义加密方式和密钥。因为加密主要主要是防止被检测，所以要选择安全系数高的加密方式。之前RC4会很容易被破解，而导致被封杀。所以现在推荐使用AES加密。而在客户端和服务端自定义密钥，泄露的风险相对较小。

所以如果是自己搭建的Shadosocks被封的概率很小，但是如果是第三方的Shadeowsocks，密码是server定的，你的数据很可能遭受到中间人攻击。
顺便说一下，Shadowssocks是天朝的clowwindy大神写的。不过Shadowsocks项目源码已经从github上删除了并停止维护了，但是release中还有源码可以下载。[https://github.com/shadowsocks/shadowsocks](https://github.com/shadowsocks/shadowsocks)

### Shadowsocks-rss
前面认为Shadowssocks特征并不是很明细，但是了解协议工作原理后会发现，SS协议本身还有有漏洞，可以被利用来检测特征，具体讨论看：ShadowSocks协议的弱点分析和改进。 里面中间那些撕逼就不用看了，我总结了下大致意思是：
协议过于简单，并且格式固定，很容易被发起中间人攻击。先看看协议结构

```bash
+--------------+---------------------+------------------+----------+
| Address Type | Destination Address | Destination Port |   Data   |
+--------------+---------------------+------------------+----------+
|      1       |       Variable      |         2        | Variable |
+--------------+---------------------+------------------+----------+
```
Possible values of address type are 1 (IPv4), 4 (IPv6), 3 (hostname). For IPv4 address, it’s packed as a 32-bit (4-byte) big-endian integer. For IPv6 address, a compact representation (16-byte array) is used. For hostname, the first byte of destination address indicates the length, which limits the length of hostname to 255. The destination port is also a big-endian integer.

The request is encrypted using the specified cipher with a random IV and the pre-shared key, it then becomes so-called payload.

结构很简单，上面解释也很清楚。Client每一个请求都是这种格式，然后进行加密。Server端解密然后解析。看起来没什么问题，没有密钥你无法模拟中间人攻击，也没什么明显特征。但是看看Server处理逻辑会发现存在一些问题：

Client数据在加密目前用的最多的是AES系列，加密后在协议数据前会有16位的IV。而Server段解析后，首先判断请求是否有效，而这个判断很简单：

判断的依据就是Address Type的那个字节，看它是不是在那三个可能取值，如果不是，立即断开连接，如果是，就尝试解析后面的地址和端口进行连接。

如果能发起中间人攻击，模拟Client请求，这个就是一个很明显的特征，如果把Address Type穷举各种情况，其中只有3种情况会连接成功。那么很可能就是一个Shadowsocks 服务器。

**所以只需要先劫持一条socks5的请求，因为AES加密后Address Type位置是固定的（第17位），篡改这一位，穷举256种情况（AES-256）**，然后发送给服务器。如果服务器在3种情况没有关闭连接，就说明这个很可能是Shadowsock服务。你这个IP很快就进入关怀列表了。

这里的关键就是AES加密明文和密文对应关系。密码学不是太懂，贴帖子里面一个回复：

```bash
举个例子，现在有一个协议包，共7个字节

0x01, 0x08, 0x08, 0x08, 0x08, 0x00, 0x50
对照socks5协议，很明显这是一个IPv4包(第一个字节是0x01)，目的地是8.8.8.8的80端口

被shadowsocks加密了以后（密码abc，加密方式aes-256-cfb），数据包就变成了这样

0xbb, 0x59, 0x1c, 0x4a, 0xb9, 0x0a, 0x91, 0xdc, 0x07, 0xef, 0x72, 0x05, 0x90, 0x42, 0xca, 0x0d, 0x4c, 0x3b, 0x87, 0x8e, 0xca, 0xab, 0x32
前16个字节，从0xbb到0x0d，都是iv，根据issue中提到的弱点和之前的总结，只需要修改0x4c，即真正密文中的第一个字节，就可要起到修改明文中的第一个字节的效果。

那就把0x4c修改成0x4d吧，解密以后的结果是

0x00, 0x08, 0x08, 0x08, 0x08, 0x00, 0x50
的确只有第一个字节被改掉了，根据breakwa11的理论，不难推出其他情况，其中合法的是

0x4e => 0x03 (Domain Name)
0x49 => 0x04 (IPv6)
```

所以目前Shadowsocks应该是比较容易被检测出来。但是为什么没有被封掉，呵呵，就不知道了。所以这个项目目的就是在SS基础上进行一些混淆。因为原有实现确实有漏洞。 不过目前这个项目好像也停止更新了。并且木有开源。

当然如果是自己用完全可以自己修改一个私有协议，这样就没法被检测到了。但是需要同时修改Server段，MAC Client，Windows Client， Android Client。 – -！

# GoAgent和GoProxy
Google App Engine是一个开发、托管网络应用程序的平台，使用Google管理的数据中心

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303162715345.png)
GoAgent的运行原理与其他代理工具基本相同，使用特定的中转服务器完成数据传输。它使用Google App Engine的服务器作为中传，将数据包后发送至Google服务器，再由Google服务器转发至目的服务器，接收数据时方法也类似。由于服务器端软件基本相同，该中转服务器既可以是用户自行架设的服务器，也可以是由其他人架设的开放服务器。

GoAgent其实也是利用GAE作为代理，但是因为他是连接到google的服务器，因为在国内现在google大量被封，所以GoAgent也基本很难使用。目前github上源码也已经删除。

但是GoAgent本身不依赖于GAE，而且使用Python编写，完全可以部署到VPS上进行代理。GoProxy是GoAgent的后续项目 [https://github.com/phuslu/goproxy](https://github.com/phuslu/goproxy)
还有一个XX-NET：[https://github.com/XX-net/XX-Net](https://github.com/XX-net/XX-Net) 有兴趣都可以去了解下。

# Tor
**Tor（The Onion Router，洋葱路由器）是实现匿名通信的自由软件**。Tor是第二代洋葱路由的一种实现，用户通过Tor可以在因特网上进行匿名交流。

[Tor:Overview](https://www.torproject.org/about/overview.html.en)
>The Tor network is a group of volunteer-operated servers that allows people to improve their privacy and security on the Internet. Tor’s users employ this network by connecting through a series of virtual tunnels rather than making a direct connection, thus allowing both organizations and individuals to share information over public networks without compromising their privacy. Along the same line, Tor is an effective censorship circumvention tool, allowing its users to reach otherwise blocked destinations or content. Tor can also be used as a building block for software developers to create new communication tools with built-in privacy features.

而关于Tor的漏洞和检测看这里：[Tor真的十分安全么其原理以及漏洞详解](https://network.pconline.com.cn/701/7017738_all.html)
目前有结合Tor+Shadowsocks前置代理使用的。