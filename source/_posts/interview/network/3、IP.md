---
title: 3、IP
toc: true
date: 2025-03-03 00:00:00
tags: ip
categories: 
	- 知识点整理
	- 计算机网络 

---

点击阅读更多查看文章内容<!--more-->

## IP基本知识

### IP与MAC之间的区别和关系

MAC 的作用则是实现「直连」的两个设备之间通信，而 IP 则负责在「没有直连」的两个网络之间进行通信传输。在数据包传输的过程中，源IP地址和目的IP地址是不会变化的（前提是没有使用NAT），只有源MAC地址和目的MAC地址一直在变化。

<img src="https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/3.jpg" alt="IP 的作用与 MAC 的作用" style="zoom:50%;" />

---

### 地址分类

<img src="https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/7.jpg" alt="IP 地址分类" style="zoom:50%;" />

主机号全0：用于表示**整个网络**，称为 **网络地址（Network Address）**。用于路由表中表示某个子网。

主机号全1：代表**该子网的所有主机**，称为 **广播地址（Broadcast Address）**。用于发送广播数据包，让该子网内所有主机接收。

---

### 私有IP地址

<img src="https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/22.jpg" alt="img" style="zoom:50%;" />

---

### IP分片与重组

每种数据链路的最大传输单元 MTU 都是不相同的，如 FDDI 数据链路 MTU 4352、以太网的 MTU 是 1500 字节等。 每种数据链路的 MTU 之所以不同，是因为每个不同类型的数据链路的使用目的不同。使用目的不同，可承载的 MTU 也就不同。 其中，我们最常见数据链路是以太网，它的 MTU 是 1500 字节。 

那么当 IP 数据包大小大于 MTU 时， IP 数据包就会被分片。 经过分片之后的 IP 数据报在被重组的时候，只能由目标主机进行，路由器是不会进行重组的。 

假设发送方发送一个 4000 字节的大数据报，若要传输在以太网链路，则需要把数据报分片成 3 个小数据报进行传输，再交由接收方重组成大数据报。

<img src="https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/26.jpg" alt="分片与重组" style="zoom:50%;" />

在分片传输中，一旦某个分片丢失，则会造成整个 IP 数据报作废，所以 TCP 引入了 MSS 也就是在 TCP 层进行分片不由 IP 层分片，那么对于 UDP 我们尽量不要发送一个大于 MTU 的数据报文。

---

## IP协议相关技术

### DNS

DNS负责将域名网址自动转换为具体的IP地址。

<img src="https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/33.jpg" alt="域名解析的工作流程" style="zoom:50%;" />

### ARP

在传输一个 IP 数据报的时候，确定了源 IP 地址和目标 IP 地址后，就会通过主机「路由表」确定 IP 数据包下一跳。然而，网络层的下一层是数据链路层，所以我们还要知道「下一跳」的 MAC 地址。 由于主机的路由表中可以找到下一跳的 IP 地址，所以可以通过 ARP 协议，求得下一跳的 MAC 地址。 那么 ARP 又是如何知道对方 MAC 地址的呢？ 简单地说，ARP 是借助 ARP 请求与 ARP 响应两种类型的包确定 MAC 地址的。

<img src="https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/34.jpg" alt="ARP 广播" style="zoom: 67%;" />

主机会通过广播发送 ARP 请求，这个包中包含了想要知道的 MAC 地址的主机 IP 地址。 当同个链路中的所有设备收到 ARP 请求时，会去拆开 ARP 请求包里的内容，如果 ARP 请求包中的目标 IP 地址与自己的 IP 地址一致，那么这个设备就将自己的 MAC 地址塞入 ARP 响应包返回给主机。 操作系统通常会把第一次通过 ARP 获取的 MAC 地址缓存起来，以便下次直接从缓存中找到对应 IP 地址的 MAC 地址。 不过，MAC 地址的缓存是有一定期限的，超过这个期限，缓存的内容将被清除。

### DHCP

<img src="https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/36.jpg" alt="DHCP 工作流程" style="zoom:67%;" />

先说明一点，DHCP 客户端进程监听的是 68 端口号，DHCP 服务端进程监听的是 67 端口号。 这 4 个步骤： 

- 客户端首先发起 DHCP 发现报文（DHCP DISCOVER） 的 IP 数据报，由于客户端没有 IP 地址，也不知道 DHCP 服务器的地址，所以使用的是 UDP 广播通信，其使用的广播目的地址是 255.255.255.255（端口 67） 并且使用 0.0.0.0（端口 68） 作为源 IP 地址。DHCP 客户端将该 IP 数据报传递给链路层，链路层然后将帧广播到所有的网络中设备。 
- DHCP 服务器收到 DHCP 发现报文时，用 DHCP 提供报文（DHCP OFFER） 向客户端做出响应。该报文仍然使用 IP 广播地址 255.255.255.255，该报文信息携带服务器提供可租约的 IP 地址、子网掩码、默认网关、DNS 服务器以及 IP 地址租用期。 
- 客户端收到一个或多个服务器的 DHCP 提供报文后，从中选择一个服务器，并向选中的服务器发送 DHCP 请求报文（DHCP REQUEST进行响应，回显配置的参数。 
- 最后，服务端用 DHCP ACK 报文对 DHCP 请求报文进行响应，应答所要求的参数。

一旦客户端收到 DHCP ACK 后，交互便完成了，并且客户端能够在租用期内使用 DHCP 服务器分配的 IP 地址。 如果租约的 DHCP IP 地址快期后，客户端会向服务器发送 DHCP 请求报文： 服务器如果同意继续租用，则用 DHCP ACK 报文进行应答，客户端就会延长租期。 服务器如果不同意继续租用，则用 DHCP NACK 报文，客户端就要停止使用租约的 IP 地址。 

可以发现，DHCP 交互中，全程都是使用 UDP 广播通信。 咦，用的是广播，那如果 DHCP 服务器和客户端不是在同一个局域网内，路由器又不会转发广播包，那不是每个网络都要配一个 DHCP 服务器？ 所以，为了解决这一问题，就出现了 DHCP 中继代理。有了 DHCP 中继代理以后，对不同网段的 IP 地址分配也可以由一个 DHCP 服务器统一进行管理。

<img src="https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/37.jpg" alt=" DHCP 中继代理" style="zoom:50%;" />

### NAT

NAT网络地址转换，私有网络对外部通信时，将私有IP地址转换成公有IP地址

<img src="https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/38.jpg" alt="NAT" style="zoom: 50%;" />

普通的NAT，N个私有地址就需要N个公有地址，没什么意义。 由于绝大多数的网络应用都是使用传输层协议 TCP 或 UDP 来传输数据的。 因此，可以把 IP 地址 + 端口号一起进行转换。 这样，就用一个全球 IP 地址就可以了，这种转换技术就叫网络地址与端口转换 NAPT。 很抽象？来，看下面的图解就能瞬间明白了。

<img src="https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/39.jpg" alt="NAPT" style="zoom:50%;" />

图中有两个客户端 192.168.1.10 和 192.168.1.11 同时与服务器 183.232.231.172 进行通信，并且这两个客户端的本地端口都是 1025。 此时，两个私有 IP 地址都转换 IP 地址为公有地址 120.229.175.121，但是以不同的端口号作为区分。 于是，生成一个 NAPT 路由器的转换表，就可以正确地转换地址跟端口的组合，令客户端 A、B 能同时与服务器之间进行通信。 这种转换表在 NAT 路由器上自动生成。例如，在 TCP 的情况下，建立 TCP 连接首次握手时的 SYN 包一经发出，就会生成这个表。而后又随着收到关闭连接时发出 FIN 包的确认应答从表中被删除。

---

### ICMP

ICMP 全称是 Internet Control Message Protocol，也就是互联网控制报文协议。 里面有个关键词 —— 控制，如何控制的呢？ 网络包在复杂的网络传输环境里，常常会遇到各种问题。 当遇到问题的时候，总不能死个不明不白，没头没脑的作风不是计算机网络的风格。所以需要传出消息，报告遇到了什么问题，这样才可以调整传输策略，以此来控制整个局面。 

ICMP 主要的功能包括：确认 IP 包是否成功送达目标地址、报告发送过程中 IP 包被废弃的原因和改善网络设置等。 在 IP 通信中如果某个 IP 包因为某种原因未能达到目标地址，那么这个具体的原因将由 ICMP 负责通知。

<img src="https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/40.jpg" alt="ICMP 目标不可达消息" style="zoom:50%;" />

如上图例子，主机 A 向主机 B 发送了数据包，由于某种原因，途中的路由器 2 未能发现主机 B 的存在，这时，路由器 2 就会向主机 A 发送一个 ICMP 目标不可达数据包，说明发往主机 B 的包未能成功。 ICMP 的这种通知消息会使用 IP 进行发送 。 因此，从路由器 2 返回的 ICMP 包会按照往常的路由控制先经过路由器 1 再转发给主机 A 。收到该 ICMP 包的主机 A 则分解 ICMP 的首部和数据域以后得知具体发生问题的原因。

ICMP类型：一类用于诊断的查询消息，也就是查询报文类型；另一类用于通知出错原因的错误消息，也就是差错报文类型。

<img src="https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/41.jpg" alt="常见的 ICMP 类型" style="zoom:50%;" />

---

### IGMP

在前面我们知道了组播地址，也就是 D 类地址，既然是组播，那就说明是只有一组的主机能收到数据包，不在一组的主机不能收到数组包，怎么管理是否是在一组呢？那么，就需要 IGMP 协议了。

<img src="https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/42.jpg" alt="组播模型" style="zoom:50%;" />

IGMP 是因特网组管理协议，工作在主机（组播成员）和最后一跳路由之间，如上图中的蓝色部分。 

- IGMP 报文向路由器申请加入和退出组播组，默认情况下路由器是不会转发组播包到连接中的主机，除非主机通过 IGMP 加入到组播组，主机申请加入到组播组时，路由器就会记录 IGMP 路由器表，路由器后续就会转发组播包到对应的主机了。 
- IGMP 报文采用 IP 封装，IP 头部的协议号为 2，而且 TTL 字段值通常为 1，因为 IGMP 是工作在主机与连接的路由器之间。

**常规查询与响应工作机制**

<img src="https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/43.jpg" alt=" IGMP 常规查询与响应工作机制" style="zoom:50%;" />

1. 路由器会周期性发送目的地址为 224.0.0.1（表示同一网段内所有主机和路由器） IGMP 常规查询报文。 主机1 和 主机 3 收到这个查询，随后会启动「报告延迟计时器」，计时器的时间是随机的，通常是 0~10 秒，计时器超时后主机就会发送 IGMP 成员关系报告报文（源 IP 地址为自己主机的 IP 地址，目的 IP 地址为组播地址）。如果在定时器超时之前，收到同一个组内的其他主机发送的成员关系报告报文，则自己不再发送，这样可以减少网络中多余的 IGMP 报文数量。
2. 路由器收到主机的成员关系报文后，就会在 IGMP 路由表中加入该组播组，后续网络中一旦该组播地址的数据到达路由器，它会把数据包转发出去。

**离开组播组工作机制**

一、网段中仍有该组播组

<img src="https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/44.jpg" alt=" IGMPv2 离开组播组工作机制 情况1" style="zoom:50%;" />

1. 主机 1 要离开组 224.1.1.1，发送 IGMPv2 离组报文，报文的目的地址是 224.0.0.2（表示发向网段内的所有路由器） 
2. 路由器 收到该报文后，以 1 秒为间隔连续发送 IGMP 特定组查询报文（共计发送 2 个），以便确认该网络是否还有 224.1.1.1 组的其他成员。 
3. 主机 3 仍然是组 224.1.1.1 的成员，因此它立即响应这个特定组查询。路由器知道该网络中仍然存在该组播组的成员，于是继续向该网络转发 224.1.1.1 的组播数据包。 

二、网段中没有该组播组

<img src="https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/45.jpg" alt=" IGMPv2 离开组播组工作机制 情况2" style="zoom:50%;" />

1. 主机 1 要离开组播组 224.1.1.1，发送 IGMP 离组报文。 
2. 路由器收到该报文后，以 1 秒为间隔连续发送 IGMP 特定组查询报文（共计发送 2 个）。此时在该网段内，组 224.1.1.1 已经没有其他成员了，因此没有主机响应这个查询。 
3. 一定时间后，路由器认为该网段中已经没有 224.1.1.1 组播组成员了，将不会再向这个网段转发该组播地址的数据包

---

## ping的工作原理

### ICMP

ping是基于ICMP协议的，ICMP主要的功能包括：确认 IP 包是否成功送达目标地址、报告发送过程中 IP 包被废弃的原因和改善网络设置等。

ICMP报文是封装在IP包里面的，它工作在网络层

<img src="https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/ping/5.jpg" alt="ICMP 报文" style="zoom:50%;" />

ICMP包头的类型字段，大致可以分为两类：

- 一类是用于诊断的查询消息，也就是「查询报文类型」 
- 另一类是通知出错原因的错误消息，也就是「差错报文类型」

<img src="https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/ping/6.jpg" alt="常见的 ICMP 类型" style="zoom:50%;" />

**查询报文类型**

> 回送消息 —— 类型 0 和 8

回送消息用于进行通信的主机或路由器之间，判断所发送的数据包是否已经成功到达对端的一种消息，**ping** 命令就是利用这个消息实现的

<img src="https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/ping/7.jpg" alt="ICMP 回送消息" style="zoom: 67%;" />

可以向对端主机发送回送请求的消息（ICMP Echo Request Message，类型 8），也可以接收对端主机发回来的回送应答消息（ICMP Echo Reply Message，类型 0）。

<img src="https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/ping/8.jpg" alt="ICMP 回送请求和回送应答报文" style="zoom:50%;" />

相比原生的 ICMP，这里多了两个字段： 

- 标识符：用以区分是哪个应用程序发 ICMP 包，比如用进程 PID 作为标识符； 
- 序号：序列号从 0 开始，每发送一次新的回送请求就会加 1， 可以用来确认网络包是否有丢失。 

在选项数据中，ping 还会存放发送请求的时间值，来计算往返时间，说明路程的长短

---

**差错报文类型**

- 目标不可达消息 —— 类型 为 3 
- 原点抑制消息 —— 类型 4 
- 重定向消息 —— 类型 5 
- 超时消息 —— 类型 11 

目标不可达消息（Destination Unreachable Message） —— 类型为 3 

IP 路由器无法将 IP 数据包发送给目标地址时，会给发送端主机返回一个目标不可达的 ICMP 消息，并在这个消息中显示不可达的具体原因，原因记录在 ICMP 包头的代码字段。 由此，根据 ICMP 不可达的具体消息，发送端主机也就可以了解此次发送不可达的具体原因。 举例 6 种常见的目标不可达类型的代码

<img src="https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/ping/9.jpg" alt="目标不可达类型的常见代码号" style="zoom:50%;" />

原点抑制消息（ICMP Source Quench Message） —— 类型 4 

在使用低速广域线路的情况下，连接 WAN 的路由器可能会遇到网络拥堵的问题。 ICMP 原点抑制消息的目的就是为了缓和这种拥堵情况。 当路由器向低速线路发送数据时，其发送队列的缓存变为零而无法发送出去时，可以向 IP 包的源地址发送一个 ICMP 原点抑制消息。 收到这个消息的主机借此了解在整个线路的某一处发生了拥堵的情况，从而增大 IP 包的传输间隔，减少网络拥堵的情况。 然而，由于这种 ICMP 可能会引起不公平的网络通信，一般不被使用。 

重定向消息（ICMP Redirect Message） —— 类型 5 

如果路由器发现发送端主机使用了「不是最优」的路径发送数据，那么它会返回一个 ICMP 重定向消息给这个主机。 在这个消息中包含了最合适的路由信息和源数据。这主要发生在路由器持有更好的路由信息的情况下。路由器会通过这样的 ICMP 消息告知发送端，让它下次发给另外一个路由器。 好比，小林本可以过条马路就能到的地方，但小林不知道，所以绕了一圈才到，后面小林知道后，下次小林就不会那么傻再绕一圈了。

超时消息（ICMP Time Exceeded Message） —— 类型 11 

IP 包中有一个字段叫做 TTL （Time To Live，生存周期），它的值随着每经过一次路由器就会减 1，直到减到 0 时该 IP 包会被丢弃。 此时，路由器将会发送一个 ICMP 超时消息给发送端主机，并通知该包已被丢弃。 设置 IP 包生存周期的主要目的，是为了在路由控制遇到问题发生循环状况时，避免 IP 包无休止地在网络上被转发

<img src="https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/ping/11.jpg" alt="ICMP 时间超过消息" style="zoom: 67%;" />

---

### ping —— 查询报文类型的使用

接下来，我们重点来看 ping 的发送和接收过程。 **同个子网**下的主机 A 和 主机 B，主机 A 执行ping 主机 B 后，我们来看看其间发送了什么？ 

ping 命令执行的时候，源主机首先会构建一个 ICMP 回送请求消息数据包。 ICMP 数据包内包含多个字段，最重要的是两个： 

- 第一个是类型，对于回送请求消息而言该字段为 8； 
- 另外一个是序号，主要用于区分连续 ping 的时候发出的多个数据包。 

每发出一个请求数据包，序号会自动加 1。为了能够计算往返时间 RTT，它会在报文的数据部分插入发送时间。 然后，由 ICMP 协议将这个数据包连同地址 192.168.1.2 一起交给 IP 层。IP 层将以 192.168.1.2 作为目的地址，本机 IP 地址作为源地址，协议字段设置为 1 表示是 ICMP 协议，再加上一些其他控制信息，构建一个 IP 数据包

接下来，需要加入 MAC 头。如果在本地 ARP 映射表中查找出 IP 地址 192.168.1.2 所对应的 MAC 地址，则可以直接使用；如果没有，则需要发送 ARP 协议查询 MAC 地址，获得 MAC 地址后，由数据链路层构建一个数据帧，目的地址是 IP 层传过来的 MAC 地址，源地址则是本机的 MAC 地址；还要附加上一些控制信息，依据以太网的介质访问规则，将它们传送出去。 

<img src="https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/ping/15.jpg" alt="主机 A 的 MAC 层数据包" style="zoom:50%;" />

主机 B 收到这个数据帧后，先检查它的目的 MAC 地址，并和本机的 MAC 地址对比，如符合，则接收，否则就丢弃。 

接收后检查该数据帧，将 IP 数据包从帧中提取出来，交给本机的 IP 层。同样，IP 层检查后，将有用的信息提取后交给 ICMP 协议。 

主机 B 会构建一个 ICMP 回送响应消息数据包，回送响应数据包的类型字段为 0，序号为接收到的请求数据包中的序号，然后再发送出去给主机 A。 

在规定的时候间内，源主机如果没有接到 ICMP 的应答包，则说明目标主机不可达；如果接收到了 ICMP 回送响应消息，则说明目标主机可达。 此时，源主机会检查，用当前时刻减去该数据包最初从源主机上发出的时刻，就是 ICMP 数据包的时间延迟

<img src="https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/ping/16.jpg" alt="主机 B 的 ICMP 回送响应报文" style="zoom:50%;" />

---

### traceroute —— 差错报文类型的使用

有一款充分利用 ICMP 差错报文类型的应用叫做 traceroute（在UNIX、MacOS中是这个命令，而在Windows中对等的命令叫做 tracert ）。 

**traceroute 作用一** 

traceroute 的第一个作用就是故意设置特殊的 TTL，来追踪去往目的地时沿途经过的路由器。 traceroute 的参数指向某个目的 IP 地址：`traceroute 192.168.1.100`

它的原理就是利用 IP 包的生存期限 从 1 开始按照顺序递增的同时发送 UDP 包，强制接收 ICMP 超时消息的一种方法。 比如，将 TTL 设置 为 1，则遇到第一个路由器，就牺牲了，接着返回 ICMP 差错报文网络包，类型是**时间超时**。 接下来将 TTL 设置为 2，第一个路由器过了，遇到第二个路由器也牺牲了，也同时返回了 ICMP 差错报文数据包，如此往复，直到到达目的主机。 这样的过程，traceroute 就可以拿到了所有的路由器 IP。 当然有的路由器根本就不会返回这个 ICMP，所以对于有的公网地址，是看不到中间经过的路由的。

发送方如何知道发出的 UDP 包是否到达了目的主机呢？ traceroute 在发送 UDP 包时，会填入一个不可能的端口号值作为 UDP 目标端口号：33434。然后对于每个下一个探针，它都会增加一个，这些端口都是通常认为不会被使用，不过，没有人知道当某些应用程序监听此类端口时会发生什么。 当目的主机，收到 UDP 包后，会返回 ICMP 差错报文消息，但这个差错报文消息的类型是**「端口不可达」**。 所以，当差错报文类型是端口不可达时，说明发送方发出的 UDP 包到达了目的主机。

**traceroute 作用二** 

traceroute 还有一个作用是故意设置不分片，从而确定路径的 MTU。 这么做是为了什么？ 这样做的目的是为了路径MTU发现。 因为有的时候我们并不知道路由器的 MTU 大小，以太网的数据链路上的 MTU 通常是 1500 字节，但是非以太网的 MTU 值就不一样了，所以我们要知道 MTU 的大小，从而控制发送的包大小。

它的工作原理如下： 首先在发送端主机发送 IP 数据报时，将 IP 包首部的分片禁止标志位设置为 1。根据这个标志位，途中的路由器不会对大数据包进行分片，而是将包丢弃。 随后，通过一个 ICMP 的不可达消息将数据链路上 MTU 的值一起给发送主机，不可达消息的类型为「需要进行分片但设置了不分片位」。 发送主机端每次收到 ICMP 差错报文时就减少包的大小，以此来定位一个合适的 MTU 值，以便能到达目标主机

---

### 断网还能ping通127.0.0.1吗

有网的情况下，ping 最后是通过网卡将数据发送出去的。 那么断网的情况下，网卡已经不工作了，ping 回环地址却一切正常，我们可以看下这种情况下的工作原理。从应用层到传输层再到网络层。这段路径跟ping外网的时候是几乎是一样的。到了网络层，系统会根据目的IP，在路由表中获取对应的路由信息，而这其中就包含选择哪个网卡把消息发出。 当发现目标IP是外网IP时，会从"真网卡"发出。 当发现目标IP是回环地址时，就会选择本地网卡。 本地网卡，其实就是个"假网卡"，它不像"真网卡"那样有个ring buffer什么的，"假网卡"会把数据推到一个叫 input_pkt_queue 的 链表 中。这个链表，其实是所有网卡共享的，上面挂着发给本机的各种消息。消息被发送到这个链表后，会再触发一个软中断。 专门处理软中断的工具人"ksoftirqd" （这是个内核线程），它在收到软中断后就会立马去链表里把消息取出，然后顺着数据链路层、网络层等层层往上传递最后给到应用程序。

<img src="https://cdn.xiaolincoding.com//mysql/other/c1019a8be584b27c4fc8b8abda9d3cf1.png" alt="图片" style="zoom:50%;" />

---

## 127.0.0.1、localhost、0.0.0.0、本机ip的区别

127.0.0.1：表示本地回环地址，127开头的都属于回环地址。

localhost：域名，对应127.0.0.1，可以在本地文件中配置

本机ip：ipconfig命令查询的本机ip

0.0.0.0：代表本机的所有ip，当启动服务监听0.0.0.0的地址时，上面所有的地址都可以访问到服务

