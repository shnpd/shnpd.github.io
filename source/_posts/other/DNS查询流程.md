---
title: DNS查询流程
date: 2024-04-08 21:06:26
toc: true
categories: 杂项
tags: DNS
---
## DNS查询流程


1. 客户端通过浏览器访问域名为www.baidu.com的网站，发起查询该域名的IP地址的 DNS 请求。<!-- more -->该请求发送到了本地DNS服务器上。本地DNS服务器会首先查询它的缓存记录，如果缓存中有此条记录，就可以直接返回结果。如果没有，本地DNS服务器还要向 DNS根服务器进行查询
2. 本地DNS服务器向根服务器请求域名www.baidu.com的IP地址。
3. 根服务器经过查询，没有记录该域名及 IP 地址的对应关系。但是会告诉本地 DNS 服务器，可以到顶级域名服务器(.com 服务器)上继续查询。
4. 本地 DNS 服务器向 .com 服务器发送 DNS 请求，请求域名www.baidu.com的 IP 地址。
5. com 服务器收到请求后也没有找到该域名及 IP 地址的对应关系，但是告诉本地DNS服务器，该域名可以在权威域名服务器 baidu.com上进行解析。
6. 本地 DNS 服务器向baidu.com域名服务器请求域名www.baidu.com的 IP 地址。
7. baidu.com服务器收到请求后，发现了该域名和 IP 地址的对应关系，并将 IP 地址返回给本地 DNS 服务器。
8. 本地 DNS 服务器将获取到与域名对应的 IP 地址返回给客户端，并且将域名和 IP 地址的对应关系保存在缓存中，以备下次别的用户查询时使用。

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/20240408163236.png"  width="50%">

其中客户端与本地DNS服务器之间是递归查询，本地DNS服务器与其他服务器直接是迭代查询

**高速缓存**
缓存的目的是将数据临时存储在某个位置，从而提高数据请求的性能和可靠性。DNS 高速缓存涉及将数据存储在更靠近请求客户端的位置，以便能够更早地解析 DNS 查询，并且能够避免在 DNS 查找链中进一步向下的额外查询，从而缩短加载时间并减少带宽/CPU 消耗。
由于域名到IP地址的映射关系并不是永久不变，为保持高速缓存中的内容正确，DNS中的每项记录都会设置对应的TTL，超过TTL就会删除这段记录。

**查询baidu.com的查询过程：**

```bash

C:\Users\19613>dig baidu.com +trace

; <<>> DiG 9.16.30 <<>> baidu.com +trace
;; global options: +cmd
.                       945     IN      NS      a.root-servers.net.
.                       945     IN      NS      e.root-servers.net.
.                       945     IN      NS      j.root-servers.net.
.                       945     IN      NS      i.root-servers.net.
.                       945     IN      NS      f.root-servers.net.
.                       945     IN      NS      h.root-servers.net.
.                       945     IN      NS      c.root-servers.net.
.                       945     IN      NS      m.root-servers.net.
.                       945     IN      NS      l.root-servers.net.
.                       945     IN      NS      b.root-servers.net.
.                       945     IN      NS      d.root-servers.net.
.                       945     IN      NS      g.root-servers.net.
.                       945     IN      NS      k.root-servers.net.
;; Received 783 bytes from 172.19.2.1#53(172.19.2.1) in 0 ms

com.                    172800  IN      NS      d.gtld-servers.net.
com.                    172800  IN      NS      f.gtld-servers.net.
com.                    172800  IN      NS      b.gtld-servers.net.
com.                    172800  IN      NS      j.gtld-servers.net.
com.                    172800  IN      NS      e.gtld-servers.net.
com.                    172800  IN      NS      m.gtld-servers.net.
com.                    172800  IN      NS      h.gtld-servers.net.
com.                    172800  IN      NS      a.gtld-servers.net.
com.                    172800  IN      NS      l.gtld-servers.net.
com.                    172800  IN      NS      c.gtld-servers.net.
com.                    172800  IN      NS      g.gtld-servers.net.
com.                    172800  IN      NS      i.gtld-servers.net.
com.                    172800  IN      NS      k.gtld-servers.net.
com.                    86400   IN      DS      19718 13 2 8ACBB0CD28F41250A80A491389424D341522D946B0DA0C0291F2D3D7 71D7805A
com.                    86400   IN      RRSIG   DS 8 1 86400 20240430200000 20240417190000 5613 . dB2Vlmf6lsKLbMiZZqDF2cHn0gF/dJguw4/786WLE9Z2iqtlBaULEMnr W3Qp9qHb2WqkatIxqKTktiGxHfklTdTz92MIyHZm8ow1wLPr9zDPZekM wk/Q29PnFonB0F71qWQI1vCLnljwWBFXYX6/CC6zOJalmqhThm4q13az 6qaOvvVTBICX5UwX/JXysFUdhquLBg/KsVN9D7vRzGuDaXmeVhZ8Zone 9RIAPJVHHntKAfjMEv6EDO/DOjf5wJCh9WWFVsPiYixaAIgIOK8JVW89 KYb8y+zKUZaTDiP4laXOtMN965vT/uEDJwKQW/4G64b3rMXnH31x2zJx YWzSng==
;; Received 1197 bytes from 2801:1b8:10::b#53(b.root-servers.net) in 19 ms

baidu.com.              172800  IN      NS      ns2.baidu.com.
baidu.com.              172800  IN      NS      ns3.baidu.com.
baidu.com.              172800  IN      NS      ns4.baidu.com.
baidu.com.              172800  IN      NS      ns1.baidu.com.
baidu.com.              172800  IN      NS      ns7.baidu.com.
CK0POJMG874LJREF7EFN8430QVIT8BSM.com. 86400 IN NSEC3 1 1 0 - CK0Q2D6NI4I7EQH8NA30NS61O48UL8G5 NS SOA RRSIG DNSKEY NSEC3PARAM
CK0POJMG874LJREF7EFN8430QVIT8BSM.com. 86400 IN RRSIG NSEC3 13 2 86400 20240422042504 20240415031504 4534 com. ecgUBNoIjF0/2NK5qbLLoESdr1gCp2UQeruasASkce/2OC1+tbNorpx2 zr2HsO8bFW7BEvnN11MdmPyQBU3Csg==
HPVV1UNKTCF9TD77I2AUR73709T975GH.com. 86400 IN NSEC3 1 1 0 - HPVVP23QUO0FP9R0A04URSICJPESKO9J NS DS RRSIG
HPVV1UNKTCF9TD77I2AUR73709T975GH.com. 86400 IN RRSIG NSEC3 13 2 86400 20240421050941 20240414035941 4534 com. KZ64CrlCZZr8BzEWnjMvpCDh6irfCyr9b6wtP29HpF/JlK+eIQXPEfKv 3lrPrlWgyhKKv7ONmdSctID2FUrMbg==
;; Received 653 bytes from 192.26.92.30#53(c.gtld-servers.net) in 40 ms

baidu.com.              600     IN      A       39.156.66.10
baidu.com.              600     IN      A       110.242.68.66
baidu.com.              86400   IN      NS      ns2.baidu.com.
baidu.com.              86400   IN      NS      ns3.baidu.com.
baidu.com.              86400   IN      NS      ns4.baidu.com.
baidu.com.              86400   IN      NS      ns7.baidu.com.
baidu.com.              86400   IN      NS      dns.baidu.com.
;; Received 356 bytes from 180.76.76.92#53(ns7.baidu.com) in 4 ms
```
