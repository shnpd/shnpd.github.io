---
title: DNSSEC详解
date: 2024-06-08 21:07:11
updated: 2024-06-08 21:07:11
toc: true
categories: 杂项
tags: DNS
description: 你要写的描述
---

## DNSSEC

> DNS 客户端不能确信来自给定 DNS 名称服务器的回复是真实的，且未被篡改。DNS 协议没有为客户端提供了一种机制来确保它不受中间人攻击。引入 DNSSEC 以解决使用 DNS 解析域名时缺少身份验证和完整性检查。<!--more-->它没有解决保密性的问题。DNSSEC 通过向现有 DNS 记录添加加密签名，确保域名系统的安全性。这些数字签名与 A、AAAA、MX、CNAME 等常见记录类型一起存储在域名服务器中。当用户发送DNS查询请求时，DNS服务器会返回数字签名和相应的资源记录，接收到DNS响应后，客户端会使用相应区域的公钥对数字签名进行验证，以确保DNS查询的完整性、真实性和认证性。

[参考资料](https://www.cloudflare.com/zh-cn/dns/dnssec/how-dnssec-works/)

## DNSSEC新增的资源记录

### RRSIG记录

即资源记录签名（英语：Resource Record Signature），资源记录除了A和AAAA之外，也包括DNSKEY、DS、NSEC等记录，RRSIG用于存放各个资源记录的签名，包括

- 算法类型
- 标签 (泛解析中原先 RRSIG 记录的名称)
- 原 TTL 大小
- 签名失效时间
- 签名签署时间
- Key 标签 (用来迅速判断应该用那个 DNSKEY 记录来验证的一个数值)
- 签名名称 (用于验证该签名的 DNSKEY 名称)
- 签名

### DNSKEY记录

该记录用于存放用于检查 RRSIG 的公钥。其包括

- 标识符 (Zone Key (DNSSEC密钥集) 以及 Secure Entry Point (KSK和简单密钥集))
- 协议 (固定值3 向下兼容)
- 算法类型
- 公钥内容

### DS记录

即委派签名者（英语：Deligated Signer），该记录用于存放 DNSKEY 中公钥的散列值 ，包括

- Key 标签：用来判断应该用哪个 DNSKEY 记录进行验证的一个数值
- 算法类型：常见的有RSASHA1、RSASHA256、ECDSAP256SHA256，具体可参考附录“算法类型列表”
- 摘要类型：创建摘要值的加密散列算法，主要使用SHA256，具体可参考附录“摘要类型列表”
- 摘要内容: 一串散列数据，由DNSKEY经由摘要类型算法得出

### NSEC和NSEC3记录

即下一个安全（英语：Next Secure）记录，用于明确表示特定域名的记录不存在。

### CDNSKEY和CDS记录

用于请求对父区域中的 DS 记录进行更新的子区域。

## RRSET

首先需要将DNS记录按照名称和类型分组为资源记录集（Resource record set，RRSet）中。例如，如果您的区域中有三个具有相同标签（如label.example.com）的 AAAA 记录，它们将全部捆绑到一个 AAAA RRset 中。

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/20240408193105.png"  width="50%">

实际上，是整个 RRset 获得数字签名，而不是单独的 DNS 记录获得。当然，这也意味着您必须从具有相同标签的区域中请求并验证所有 AAAA 记录，而不是仅验证其中一个。

## 区域签名密钥（ZSK）

DNSSEC 中的每个区域都有一个区域签名密钥对（ZSK）：私钥用于对区域中的RRset进行数字签名，公钥用于验证签名。区域管理员会使用ZSK私钥为每个RRset创建数字签名，并将其作为RRSIG记录存储在域名服务器中。
<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/20240408193620.png"  width="50%">
同时DNSSEC的区域管理员会将自己的ZSK公钥以DNSKEY记录的形式公布出来，这样解析器即可验证RRSIG的值。
当 DNSSEC 解析器请求特定的记录类型（例如 AAAA）时，名称服务器还会返回相应的 RRSIG。然后，解析器可以从域名服务器中提取包含公用 ZSK 的 DNSKEY 记录。RRset、RRSIG 和公共 ZSK 将一同用于验证响应。
<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/20240408193955.png"  width="50%">
如果我们信任 DNSKEY 记录中的区域签名密钥，则可以信任该区域中的所有记录。但是，如果区域签名密钥被泄露怎么办？我们需要一种方法来验证公开 ZSK。

## 密钥签名密钥（KSK）

除了区域签名密钥之外，域名服务器还具有密钥签名密钥（KSK）。KSK 验证 DNSKEY 记录的方式与上一节中描述的ZSK保护RRset的方式完全相同：它签署ZSK公钥（存储在 DNSKEY 记录中），从而为 DNSKEY 创建 RRSIG。

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/20240408195424.png"  width="50%">

与ZSK公钥一样，域名服务器会将KSK公钥存储在另一个DNSKEY记录中，ZSK公钥和KSK公钥共同组成了上面显示的 DNSKEY RRset 由KSK私钥签名。然后，解析器就可以使用KSK公钥来验证ZSK公钥。

解析器验证流程如下：

1. 请求所需的 RRset，系统还将返回相应的 RRSIG 记录。
2. 请求包含ZSK公钥和KSK公钥的DNSKEY记录，系统还将返回 DNSKEY RRset 的 RRSIG。
3. 用ZSK公钥验证所请求 RRset 的 RRSIG。
4. 用KSK公钥验证 DNSKEY RRset 的 RRSIG。
<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/20240408195915.png"  width="50%">

>区域签名密钥（ZSK）是一种短期密钥，它用于定期计算DNS记录的签名。每个DNS区域通常都有自己的一组ZSK，这些密钥会定期更换，以提高安全性。通过使用ZSK对DNS记录进行签名，DNSSEC能够确保这些记录在传输过程中未被篡改。
>密钥签名密钥（KSK）则是一种长期密钥，用于对区域签名密钥（ZSK）上的签名进行计算。也就是说，KSK实际上是对ZSK进行签名的密钥，这形成了一个签名链，增强了DNSSEC的安全性。KSK的更换频率通常低于ZSK，这是因为频繁更换KSK可能会对DNS系统的稳定性造成影响。

## 委派签名者记录（DS）

DNSSEC 引入了委派签名者（Delegation signer，DS）记录，以允许将信任从父区域转移到子区域建立起一个信任链模型。区域操作员对包含公共 KSK 的 DNSKEY 记录进行哈希处理，并将其提供给父区域以作为 DS 记录发布。

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/20240408200351.png"  width="50%">

每次将解析器引用到子区域时，父区域也会提供 DS 记录。此 DS 记录是解析器获知子区域启用 DNSSEC 的方式。为了检查子区域的公共 KSK 的有效性，解析器对其进行哈希处理并将其与父区域的 DS 记录进行比较。如果两者匹配，则解析器可以假定公共 KSK 未被篡改，这意味着它可以信任子区域中的所有记录。这就是在 DNSSEC 中建立信任链的方式。

> 请注意，KSK 的任何变更都需要更改父区域的 DS 记录。更改 DS 记录是一个多步骤的过程，如果执行不正确，最终可能会破坏该区域。首先，父级需要添加新的 DS 记录，然后需要等到原始 DS 记录的 TTL 过期后将其删除。这就是为什么换掉区域签名密钥比密钥签名密钥要容易得多。

## NSEC和NSEC3

Next Secure（NSEC）记录可用于确定特定区域中是否存在某个名称。它的出现主要是为了解决记录不存在于某个区域的问题（即“拒绝存在”认证问题）。攻击者可以利用这个漏洞，伪造网站主机名的NXDOMAIN响应使网站无法访问。

DNSSEC解决了这个问题，当查询收到一个不存在的名称时，通过将域名按字母顺序排列，它可以提供一条NSEC记录来代表区域中的下一个记录是真实存在的。

举例来说，如果“example.com”区域经过排序，让“beta.example.com”成为第一条记录，那么针对“alpha.example.com”的查询将导致NXDOMAIN，并产生一条指向“beta.example.com”的NSEC记录。NSEC记录与其他任何记录一样，都由ZSK签名，具备对应的RRSIG。因此对NXDOMAIN的响应需要具备经过认证的RRSIG NSEC记录才算有效。

NSEC3的诞生就是为了解决NSEC记录被用于列出区域中所有有效记录而造成的问题。NSEC3在行为上与NSEC完全相同，不过区域中的Next secure名称会显示为哈希而非明文。这有助于保护信息并防止区域遍历

## 信任链

现在，在区域内建立信任并将其连接到父区域的方法已经有了，但是我们如何信任 DS 记录呢？DS 记录就像其他任何 RRset 一样签署，这意味着它在父级中具有相应的 RRSIG。整个验证过程不断重复，直到获得父级的公共 KSK。为了验证父级的公共 KSK，我们需要转到父级的 DS 记录，以此类推，沿着信任链上行。
<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/20240408203735.png"  width="50%">
但是，当我们最终到达根 DNS 区域时，又有一个问题：没有父 DS 记录可用于验证。在这里，我们可以看到全球互联网非常人性的一面：
在“根区域签名仪式”上，来自世界各地的特定几人以公开且经严格审核的方式签署根 DNSKEY RRset。这次仪式会产生一个 RRSIG 记录，该记录可用于验证根名称服务器的公共 KSK 和 ZSK。我们不会由于父级的 DS 记录而信任公共 KSK，而是因为信任访问私有 KSK 所涉的安全性过程而假定其有效

## 总结

区域文件中会包含ZSK公钥和KSK公钥两条DNSKEY记录，用于验证签名，父区域DS记录包含子区域KSK的哈希用于建立信任




## 数据结构
