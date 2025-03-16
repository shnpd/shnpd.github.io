---
title: DNS记录
date: 2024-04-08 13:36:43
toc: true
categories: 杂项
tags: DNS
---

## DNS记录

DNS 记录（又名区域文件）是位于 DNS 服务器中的指令，提供一个域的相关信息，包括哪些 IP 地址与该域关联，以及如何处理对该域的请求。<!-- more -->这些记录由一系列以所谓的 DNS 语法编写的文本文件组成。DNS 语法是用作命令的字符串，这些命令告诉 DNS 服务器执行什么操作。此外，所有 DNS 记录都有一个 “TTL”，其代表生存时间，指示 DNS 服务器多久刷新一次该记录。

<!-- more -->

### A记录

> 保存域的IP地址的记录

“A”代表“地址”，这是最基础的 DNS 记录类型：它用来指定给定域名的 IP 地址。

A 记录只保存 IPv4 地址。如果一个网站拥有 IPv6 地址，它将改用“AAAA”记录。

A记录示例
|example.com|record type|value|TTL|
|---|---|---|---|
|@|A|192.0.2.1|14400|

其中@表示example.com域名本身没有任何前缀

绝大多数网站只有一个 A 记录，但也可以有多个。一些高知名度网站可能有数个不同的 A 记录，将请求流量分配到多个 IP 地址中的一个，实现负载均衡

### AAAA记录

> 包含域的 IPv6 地址的记录（与 A 记录相反，A 记录列出的是 IPv4 地址）

DNS AAAA 记录将域名与 IPv6 地址进行匹配。DNS AAAA 记录与 DNS A 记录完全一样，只是它们存储域的 IPv6 地址，而非 IPv4 地址。

AAAA记录示例：
|example.com|record type|value|TTL|
|---|---|---|---|
|@|AAAA|2001:0db8:85a3:0000:0000:8a2e:0370:7334|14400|

### CNAME记录

> 将一个域或子域转发到另一个域，不提供 IP 地址。

一个"canonical name"（CNAME）记录从一个别名域指向一个"规范名称"域，当一个域或子域是另一个域的别名时，CNAME记录被用来代替A记录(一个域名不能同时设置A记录和CNAME记录)。所有CNAME记录都必须指向一个域名，而不是指向一个IP地址。想象一下，在一个寻宝游戏中，每条线索都指向另一条线索，而最后的线索则指向宝藏。 一个有CNAME记录的域名就像一条线索，可以把你指向另一条线索（另一个有CNAME记录的域名）或宝藏（一个有A记录的域名）

例如，假设 blog.example.com 的 CNAME 记录的值为“example.com”（没有“blog”）。这意味着当 DNS 服务器点击 blog.example.com 的 DNS 记录时，它实际上会触发另一个对 example.com 的 DNS 查找，并通过其 A 记录返回 example.com 的 IP 地址。在这种情况下，我们会说 example.com 是 blog.example.com 的规范名称（或真实名称）。

一个常见的误解是 CNAME 记录必须始终解析为其指向的域所在的网站，但事实并非如此。CNAME 记录仅将客户端指向与根域相同的 IP 地址。客户端访问该 IP 地址后，Web 服务器仍将相应地处理 URL。例如，blog.example.com 可能有一个 CNAME 指向 example.com，从而将客户端定向到 example.com 的 IP 地址。但是，当客户端实际连接到该 IP 地址时，Web 服务器将查看 URL，发现它是 blog.example.com，并且提供博客页面而不是主页。

CNAME记录示例：
|blog.example.com|record type|value|TTL|
|---|---|---|---|
|@|CNAME|is an alias of example.com|32600|

**一个CNAME记录可以指向另一个CNAME记录吗?**
将 CNAME 记录指向另一个 CNAME 记录十分低效，因为它需要在加载域之前进行多次 DNS 查找——这会降低用户体验，不过这个过程是可以实现的。例如，blog.example.com 可以有一个 CNAME 记录指向 www.example.com 的 CNAME 记录，然后后者指向 example.com 的 A 记录。
blog.example.com 的 CNAME：
|blog.example.com|record type|value|TTL|
|---|---|---|---|
|@|CNAME|is an alias of www.example.com|32600|

它指向 www.example.com 的 CNAME：

|www.example.com|record type|value|TTL|
|---|---|---|---|
|@|CNAME|is an alias of example.com|32600|

**CNAME记录的限制**
MX 和 NS 记录不能指向 CNAME 记录，它们必须指向 A 记录（对于 IPv4）或 AAAA 记录（对于 IPv6）（即该记录指向的域名设置的记录不能为CNAME）。MX 记录是邮件交换记录，将电子邮件指向一个邮件服务器。NS 记录是“名称服务器”记录，表明哪个 DNS 服务器是该域的权威。

### MX记录

> 将邮件定向到电子邮件服务器。

DNS“邮件交换”(MX) 记录将电子邮件定向到邮件服务器。MX 记录指示如何根据简单邮件传输协议（SMTP，所有电子邮件的标准协议）路由电子邮件。与 CNAME 记录类似，MX 记录必须始终指向另一个域。

MX 记录示例：
|example.com|record type|优先级|value|TTL|
|---|---|---|---|---|
|@|MX|10|mailhost1.example.com|45000|
|@|MX|20|mailhost2.example.com|45000|

这些 MX 记录的域前面的“优先级”数字表示优先权，较低的“优先级”值是首选。服务器将始终先尝试mailhost1，因为 10 小于 20。当消息发送失败时，服务器将默认使用 mailhost2。

**查询MX记录的过程**
邮件传输代理 (MTA) 软件负责查询 MX 记录。当用户发送电子邮件时，MTA 会发送一个 DNS 查询，以确定电子邮件收件人的邮件服务器。MTA 与这些邮件服务器建立 SMTP 连接，从优先级高的域开始（在上面的第一个示例中，即为 mailhost1）。

MX 记录必须直接指向服务器的 A 记录或 AAAA 记录。定义 MX 记录运作原理的 [RFC](https://datatracker.ietf.org/doc/html/rfc2181) 文档禁止 MX 记录指向 CNAME

### TXT 记录

> 可让管理员在记录中存储文本注释。这些记录通常用于电子邮件安全。

DNS“文本”(TXT) 记录允许域管理员将文本输入到域名系统 (DNS) 中。TXT 记录最初的目的是用作存放人类可读笔记的地方。但是，现在也可以将一些机器可读的数据放入 TXT 记录中。一个域可以有许多 TXT 记录。

TXT 记录示例：
|example.com|record type|value|TTL|
|---|---|---|---|
|@|TXT|This is an awesome domain! Definitely not spammy.|32600|

如今，DNS TXT 记录的两个最重要用途是防止垃圾邮件和域名所有权验证，尽管 TXT 记录最初并非为这些用途而设计。

**TXT 记录如何帮助防止垃圾邮件？**

垃圾邮件发送者经常试图伪造或假冒他们发送电子邮件的域。TXT 记录是几种不同的电子邮件验证方法的关键组成部分，它可帮助电子邮件服务器确定邮件是否来自可信的来源。

常见的电子邮件身份验证方法包括域密钥识别邮件 (DKIM)、发送方策略框架 (SPF) 以及基于域的邮件身份验证、报告和一致性 (DMARC)。通过配置这些记录，域运营商可以使垃圾邮件发送者更难伪造他们的域，并且可以跟踪此类尝试。

SPF 记录：SPF TXT 记录列出了所有被授权从一个域发送电子邮件的服务器。

DKIM 记录：DKIM 的工作原理是使用一个公钥-私钥对，对每封电子邮件进行数字签名。这有助于验证电子邮件确实来自它所声称的域。公钥被存放在与域关联的 TXT 记录中。（了解有关公钥加密的更多信息）。

DMARC 记录：DMARC TXT 记录引用域的 SPF 和 DKIM 政策。它应该存储在标题 _marc.example.com 下，“example.com”用实际域名代替。记录的“值”是域的 DMARC 政策

**TXT 记录如何帮助验证域所有权？**

虽然域所有权验证最初不是 TXT 记录的一个功能，但这种方法已经被一些网站管理员工具和云提供商采用。

管理员可以通过上传包含特定信息的新 TXT 记录，或编辑当前的 TXT 记录，来证明他们控制着该域。工具或云提供商可以检查 TXT 记录，并看到它已按要求进行了更改。这有点像用户通过打开并点击发送到该电子邮件的链接来确认其电子邮件地址，证明他们拥有该地址

### NS记录

> 存储 DNS 条目的域名服务器。

NS 代表“域名服务器”，域名服务器记录指示哪个 DNS 服务器对该域具有权威性（即，哪个服务器包含实际 DNS 记录）。基本上，NS 记录告诉互联网可从哪里找到域的 IP 地址。一个域通常会有多个 NS 记录，这些记录可指示该域的主要和辅助域名服务器。倘若没有正确配置的 NS 记录，用户将无法加载网站或应用程序。

NS 记录示例：
|example.com|record type|value|TTL|
|---|---|---|---|
|@|NS|ns1.exampleserver.com|21600|

请注意，NS 记录永远不能指向规范名称（CNAME）记录。

**什么是域名服务器**

域名服务器是一种 DNS 服务器，上面存储了域的所有 DNS 记录，包括 A 记录、MX 记录或 CNAME 记录。

几乎所有域都依靠多个域名服务器来提高可靠性：如果一个域名服务器出现故障或不可用，DNS 查询可以转到另一个域名服务器。通常有一个主要域名服务器和几个次要域名服务器，后者包含只读的区域文件副本，这意味着它们不能被修改。它们不是从本地文件中获取信息，而是在称为“区域传输”的通信过程中从主服务器接收相关信息。

使用多个域名服务器时（大多数情况下），NS 记录应列出不止一个服务器。

### SOA记录

> 存储域的管理信息。

DNS ‘start of authority’ (SOA) 记录存储了关于域名和区域的重要信息，如管理员的电子邮件地址、域名上次更新的时间，以及服务器在刷新之间应等待的时长。

所有 DNS 区域都需要一个 SOA 记录，以符合 IETF 标准。SOA 记录对区域传输也很重要。

SOA 记录示例：
|name|example.com|
|---|---|
|record type|SOA|
|MNAME|ns.primaryserver.com|
|RNAME|admin.example.com|
|SERIAL|1111111111|
|REFRESH|86400|
|RETRY|7200|
|EXPIRE|4000000|
|TTL|11200|

- RNAME：管理员的电子邮件地址，这可能会造成混淆，因为它缺少“@”符号；但在 SOA 记录中，admin.example.com 等效于 admin@example.com。
- SERIAL：区域序列号，表示SOA记录的版本号，当区域文件中的序列号发生更改时，这会提醒辅助名称服务器，它们应当通过区域传输更新其区域文件的副本。
- MNAME：这是区域的主要名称服务器的名称。维护该区域 DNS 记录副本的辅助服务器会从该主要服务器接收对该区域的更新
- REFRESH：辅助服务器在向主要服务器询问 SOA 记录以查看其是否已更新之前应等待的时间长度（秒）。
- RETRY：服务器再次向无响应的主要名称服务器请求更新前应等待的时间长度。
- EXPIRE：如果辅助服务器在该时间段内没有得到主要服务器的响应，则应该停止响应对该区域的查询。


**什么是DNS区域？**
DNS 被分成许多不同的区域。这些区域可区分在 DNS 命名空间中以不同方式管理的区域。DNS 区域是 DNS 命名空间的一部分，由特定组织或管理员加以管理。DNS 区域是一个管理空间，其可实现对权威性域名服务器等 DNS 组件的更精细控制，DNS 区域可包含多个子域，并且多个区域可存在于同一服务器上。
例如，想象一下 cloudflare.com 域及其以下三个子域：support.cloudflare.com、community.cloudflare.com 和 blog.cloudflare.com。假设blog是一个强健的独立站点，该站点需要单独管理，但是support和community与cloudflare.com 更紧密地关联，并且可在与主域相同的区域中加以管理。在这种情况下，cloudflare.com 以及support站点和community站点都将在一个区域中，而blog.cloudflare.com将存在于其自己的区域中。
<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/20240408160401.png"  width="50%">

**什么是区域传输？**
DNS 区域传输是将 DNS 记录数据从一个主要名称服务器发送到一个辅助名称服务器的过程。SOA 记录将首先被传输。序列号将告知辅助服务器其版本是否需要更新。区域传输通过 TCP 协议进行。

### PTR记录

> DNS 指针记录（简称 PTR）提供与 IP 地址关联的域名。DNS PTR 记录与“A”记录完全相反，它提供与域名关联的 IP 地址



## 相关术语

### 胶水记录

[胶水记录介绍](https://laona.dev/post/glue-record/)
[死循环问题](http://www.ricearth.com/article/5019046192866861)
胶水记录（Glue Record）是DNS（域名系统）中的一个重要概念，尤其是在自建DNS环境中。其英文名为Glue Record，中文也有不同的称呼，比如一些运营商可能称之为“自定义DNS Host”或“注册DNS服务器”。

```
# dig @202.12.27.33 jd.com

; <<>> DiG 9.9.4-RedHat-9.9.4-61.el7 <<>> @202.12.27.33 jd.com
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 29264
;; flags: qr rd; QUERY: 1, ANSWER: 0, AUTHORITY: 13, ADDITIONAL: 27
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;jd.com.                                IN      A

;; AUTHORITY SECTION:
com.                    172800  IN      NS      d.gtld-servers.net.
com.                    172800  IN      NS      f.gtld-servers.net.
com.                    172800  IN      NS      e.gtld-servers.net.
com.                    172800  IN      NS      g.gtld-servers.net.
com.                    172800  IN      NS      k.gtld-servers.net.
com.                    172800  IN      NS      a.gtld-servers.net.
com.                    172800  IN      NS      c.gtld-servers.net.
com.                    172800  IN      NS      j.gtld-servers.net.
com.                    172800  IN      NS      l.gtld-servers.net.
com.                    172800  IN      NS      b.gtld-servers.net.
com.                    172800  IN      NS      m.gtld-servers.net.
com.                    172800  IN      NS      i.gtld-servers.net.
com.                    172800  IN      NS      h.gtld-servers.net.

;; ADDITIONAL SECTION:
a.gtld-servers.net.     172800  IN      A       192.5.6.30
b.gtld-servers.net.     172800  IN      A       192.33.14.30
c.gtld-servers.net.     172800  IN      A       192.26.92.30
d.gtld-servers.net.     172800  IN      A       192.31.80.30
e.gtld-servers.net.     172800  IN      A       192.12.94.30
f.gtld-servers.net.     172800  IN      A       192.35.51.30
g.gtld-servers.net.     172800  IN      A       192.42.93.30
h.gtld-servers.net.     172800  IN      A       192.54.112.30
i.gtld-servers.net.     172800  IN      A       192.43.172.30
j.gtld-servers.net.     172800  IN      A       192.48.79.30
k.gtld-servers.net.     172800  IN      A       192.52.178.30
l.gtld-servers.net.     172800  IN      A       192.41.162.30
m.gtld-servers.net.     172800  IN      A       192.55.83.30
a.gtld-servers.net.     172800  IN      AAAA    2001:503:a83e::2:30
b.gtld-servers.net.     172800  IN      AAAA    2001:503:231d::2:30
c.gtld-servers.net.     172800  IN      AAAA    2001:503:83eb::30
d.gtld-servers.net.     172800  IN      AAAA    2001:500:856e::30
e.gtld-servers.net.     172800  IN      AAAA    2001:502:1ca1::30
f.gtld-servers.net.     172800  IN      AAAA    2001:503:d414::30
g.gtld-servers.net.     172800  IN      AAAA    2001:503:eea3::30
h.gtld-servers.net.     172800  IN      AAAA    2001:502:8cc::30
i.gtld-servers.net.     172800  IN      AAAA    2001:503:39c1::30
j.gtld-servers.net.     172800  IN      AAAA    2001:502:7094::30
k.gtld-servers.net.     172800  IN      AAAA    2001:503:d2d::30
l.gtld-servers.net.     172800  IN      AAAA    2001:500:d937::30
m.gtld-servers.net.     172800  IN      AAAA    2001:501:b1f9::30

;; Query time: 211 msec
;; SERVER: 202.12.27.33#53(202.12.27.33)
;; WHEN: Thu Aug 01 18:13:03 CST 2019
;; MSG SIZE  rcvd: 831
```
这是根服务器返回的.com服务器的信息，其中ADDITIONAL SECTION就是胶水记录

普通DNS记录保存在权威服务器上，胶水记录保存在域名注册局的DNS服务器上

胶水记录可以看作是和DNS查询结果一起顺便返回的一组域名和IP地址的映射

作用：

- 减少递归查询次数，加快DNS递归查询，在向根域名和顶级域名服务器查询时他们通常返回的是下一级要查询的服务器的域名，此时胶水记录会同时附带要查询的域名的IP地址，如果没有胶水记录则会重新启动一次新的查询去查询该域名，因此胶水记录可以减少递归查询次数，加快 DNS 递归查询。
- 如果域名的NS服务器是子域名那么解析过程可能会陷入死循环。以一个简单的例子来说明：假设我们有一个域名b.com，并希望指定b.com的NS地址是a.b.com（无法直接指定ip地址）。在这种情况下，我们就必须指定a.b.com的A记录，否则在解析a.b.com时，系统又会去解析b.com，导致循环而无法得出结果。这就需要我们添加一条胶水记录去解析a.b.com

### 缓存投毒

[一次出人意料而名留青史的DNS投毒攻击](https://zhuanlan.zhihu.com/p/92899876)
攻击者通过精心构造DNS报文，在LDNS查询某个域名时，冒充真正的权威DNS做出回应，使得LDNS得到一个虚假响应。如果LDNS接受了这个虚假响应并写入缓存，LDNS就会中毒。
