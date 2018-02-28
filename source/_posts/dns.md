---
title: DNS 解析详解
tags: Networkfdf
date: 2018-02-28 16:36:20
---


大家都知道 DNS (Domain Name System, 域名系统), 是一个域名和 IP 地址映射系统. 通过 DNS 我们可以根据比较容易记住的域名而不是数字 IP 地址来访问网络资源.
本文主要使用 `dig` (ubuntu 上可以通过 `sudo apt install dnsutils` 安装) 工具来了解 DNS 的各方面.
<!-- more -->

## 0x01 CNAME 和 A 记录
在 DNS 中, A 记录值表示的是真正的 IP 地址, 而 CNAME 相当于域名的别名.
我们来看一个例子, 新浪的 DNS 解析:

```shell
$ dig www.sina.com.cn

; <<>> DiG 9.10.3-P4-Ubuntu <<>> www.sina.com.cn
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 45272
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4000
;; QUESTION SECTION:
;www.sina.com.cn.		IN	A

;; ANSWER SECTION:
www.sina.com.cn.	0	IN	CNAME	spool.grid.sinaedge.com.
spool.grid.sinaedge.com. 280	IN	A	222.73.28.96

;; Query time: 5 msec
;; SERVER: 192.168.1.6#53(192.168.1.6)
;; WHEN: Wed Feb 28 13:37:11 CST 2018
;; MSG SIZE  rcvd: 97
```

其中 `QUESTION SECTION:` 表示我们要查找的是 `www.sina.com.cn` 的 A 记录, `ANSWER SECTION:` 返回了记录的结果.
这个查询返回了三条记录, 每条记录有五个字段分别代表了: 主机域名, TTL, 类型(IN 表示 Internet), DNS 记录类型以及记录值. DNS 记录类型如下图所示 (图片来自阿里云截图):

![dns record](/images/dns/dns record.png)

`www.sina.com.cn` 有一个 CNAME和一个 A记录. 如果我们直接在浏览器中访问 CNAME 值或者 IP 地址, 会发现不能访问.

![error](/images/dns/error.png)

这是因为新浪服务器拒绝了非 `www.sina.com.cn` 域名访问. 我们可以构造一个 HTTP header 来实现直接连接 IP 访问新浪:
```shell
$ curl 222.73.28.96 -H "Host: www.sina.com.cn"
```
这样就能返回正常的页面.

## 0x02 另一个例子

再来看一个例子:

```
$ dig baidu.com                    

; <<>> DiG 9.10.3-P4-Ubuntu <<>> baidu.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 23153
;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4000
;; QUESTION SECTION:
;baidu.com.			IN	A

;; ANSWER SECTION:
baidu.com.		300	IN	A	111.13.101.208
baidu.com.		300	IN	A	123.125.114.144
baidu.com.		300	IN	A	220.181.57.216

;; Query time: 5 msec
;; SERVER: 192.168.1.6#53(192.168.1.6)
;; WHEN: Wed Feb 28 13:53:03 CST 2018
;; MSG SIZE  rcvd: 86
```

我们看到 `baidu.com` 返回了三个 A 记录地址. 那客户端应该选择哪一个作为服务器的 IP 地址呢?
这个需要看客户端的实现, 比如使用 `gethostbyname` 的话会选择一个 IP 地址返回,  `getaddrinfo` 会返回所有的地址.

## 0x03 NS 记录

NS 记录是域名服务器记录，用来指定域名由哪个DNS服务器来进行解析.

`dig` 可以添加 `+trace` 参数来获取完整 DNS 解析过程. 还是通过 `www.sina.com.cn` 作为例子.

```shell
$ dig www.sina.com.cn +trace

; <<>> DiG 9.10.3-P4-Ubuntu <<>> www.sina.com.cn +trace
;; global options: +cmd
.			3600	IN	NS	d.root-servers.net.
.			3600	IN	NS	e.root-servers.net.
.			3600	IN	NS	f.root-servers.net.
.			3600	IN	NS	g.root-servers.net.
.			3600	IN	NS	h.root-servers.net.
.			3600	IN	NS	i.root-servers.net.
.			3600	IN	NS	j.root-servers.net.
.			3600	IN	NS	k.root-servers.net.
.			3600	IN	NS	l.root-servers.net.
.			3600	IN	NS	m.root-servers.net.
.			3600	IN	NS	a.root-servers.net.
.			3600	IN	NS	b.root-servers.net.
.			3600	IN	NS	c.root-servers.net.
;; Received 1771 bytes from 192.168.1.6#53(192.168.1.6) in 0 ms

cn.			172800	IN	NS	b.dns.cn.
cn.			172800	IN	NS	ns.cernet.net.
cn.			172800	IN	NS	c.dns.cn.
cn.			172800	IN	NS	d.dns.cn.
cn.			172800	IN	NS	a.dns.cn.
cn.			172800	IN	NS	g.dns.cn.
cn.			172800	IN	NS	f.dns.cn.
cn.			172800	IN	NS	e.dns.cn.
cn.			86400	IN	DS	41470 8 2 3623FB6E3B1F69C6855DA1E48D3A38236DD2EDF0380FB018FF538650 EAC2C4DD
cn.			86400	IN	DS	57724 8 2 5D0423633EB24A499BE78AA22D1C0C9BA36218FF49FD95A4CDF1A4AD 97C67044
cn.			86400	IN	RRSIG	DS 8 1 86400 20180313050000 20180228040000 41824 . GbRg9UYKus5nqvJxCKVZTaX5j2WYaF2c3jH5XOEPzqgcGp23+U941ZHz nKAXTv8oJq2+dJiRuVnwAD7c+Ge8MBJbd+tpw0jcQ3zs3SiocVhWgF3/ Bjig8ouJsuKukEuF89tx+oqbYjRrau9PFNJBoN2zlVZP1JQYTukYaGeY aK91OtgRuC1yVVQqNstLhU+YWyi7gNKOd31SMSpyvUZDIf+8wQrR6j9y dUb7zex8dk+XFS9/NqOXi6kRQxOPdflXXGieNZRVAnHfKdgO3LsXYmsD 92CY9G7cwj9XShFdYq7GLSirh3c9LvUR4E0VKk6qcr3usgKbmDPumyAv aN+ZwA==
;; Received 754 bytes from 192.33.4.12#53(c.root-servers.net) in 173 ms

sina.com.cn.		86400	IN	NS	ns3.sina.com.cn.
sina.com.cn.		86400	IN	NS	ns2.sina.com.cn.
sina.com.cn.		86400	IN	NS	ns1.sina.com.cn.
sina.com.cn.		86400	IN	NS	ns4.sina.com.cn.
GICE14DNTMDN31G43AUGVRKTKALVB8QC.com.cn. 21600 IN NSEC3	1 1 10 AEF123AB HIO2MHL5BSKBHFFRA5I1J58SU91CDLLA NS SOA RRSIG DNSKEY NSEC3PARAM
GICE14DNTMDN31G43AUGVRKTKALVB8QC.com.cn. 21600 IN RRSIG	NSEC3 8 3 21600 20180318092826 20180216084403 48018 com.cn. iuUIwe/vd4QLsTo8behQVf8ZPWaU9JsP+gxrUHop+oybuZH+II+kvOBW wTfGHap/n3C7iSevN80Wa2eFeH0QBWif2A30+zfg9hCzVjEEUDulmc1a 4+ltDbv4UZVJpBPRU7n2AgW4UMK/q0vyWqV6oKwmIygj58fhrMkqcrpR CpU=
T1MQAIVAIU5JVK5ON55K8AOCE62H72MI.com.cn. 21600 IN NSEC3	1 1 10 AEF123AB UQROTQK62NOIM5U43DMF7AMC8JJFRM7T NS DS RRSIG
T1MQAIVAIU5JVK5ON55K8AOCE62H72MI.com.cn. 21600 IN RRSIG	NSEC3 8 3 21600 20180311113832 20180209104700 48018 com.cn. I1zxGBgSFJfq5GCrwlukCCkWNeQRcJJu9ydX5OgoH0mdYwVVLGoB2y1D htn8lGc4MMfdbY+zTdlnvYvBHdtFSS2+2eq+ficKzzZQ2CVtDrFm91Eo 0MK+BavvLcE5pkRpIfpI9FIIMrlNaj9cOBwWNR2g1yXfSYWSr5NUFKQG voE=
;; Received 679 bytes from 203.119.25.1#53(a.dns.cn) in 33 ms

www.sina.com.cn.	0	IN	CNAME	spool.grid.sinaedge.com.
;; Received 81 bytes from 61.172.201.254#53(ns2.sina.com.cn) in 5 ms

```

我们最上面域名为 `.` (根域名) 的 NS 记录, 一共有 13 条, 代表了 13 台根域名服务器. 接下来是在根域名服务器上查询 `cn.` (实际中最后一个点去掉) 得到的 8 条 NS 记录, 代表了顶级域名服务器. 然后是顶级域名服务器查询 `sina.com.cn` 得到的 4 条 NS 记录, 代表了授权域名服务器, 最后授权 DNS 返回了 `www.sina.com.cn` 的 CNAME 记录.

<div class="tip">
授权域名服务器: 我们知道计算机要上网的话, 必须要设置 DNS (除非只用 IP), 或者 ISP 自动分配 DNS. 这个 DNS 服务器是叫做 LocalDNS, 提供了缓存和递归查询服务. 如果查询一个域名在 LocalDNS 上已经有缓存了, LocalDNS 直接返回结果 (叫做非授权应答); 如果 LocalDNS 上没有这个域名的话, LocalDNS 就要对域名发起递归查找(类似 dig +trace 的过程), 就是从根域名 -> 顶级域名 -> 授权域名查找这个域名, 并将应答返回, 同时缓存应答(可以看出根域名, 顶级域名也是授权域名服务器). 一般宽带运行商和公共DNS服务提供的是 LocalDNS, 域名注册商提供的是授权DNS.
</div>

假如我在阿里云上注册了一个域名 (example.com), 其 NS 记录就是 dns14.hichina.com 和 dns13.hichina.com (万网的授权域名服务器), 我们还可以设置 NS 记录指向其他的授权域名服务器.
