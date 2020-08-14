---
title: "DoT vs DoH"
date: 2019-11-28T14:54:10+08:00
draft: false
---

DoH （[RFC 8484][1]）和 DoT（[RFC 7858](https://tools.ietf.org/html/rfc7858)）以及先前的DNSCrypt和DNSSEC都是進行安全化的域名解析的方案，Mozilla從Firefox 62開始支持DoH（[官方博文](https://blog.nightly.mozilla.org/2018/06/01/improving-dns-privacy-in-firefox/)）

[DoH][1]是最新的方案，衆多大佬都開始支持了，~~就假裝不是草案了唄，其實人家只是Proposed Standard~~

這篇文章是想廢話一下這些方案的區別

<!--more-->

# 普通的DNS

最初的DNS (Domain Name System)協議於1987年定稿，但是那個時候的Internet基本不需要考慮安全問題（~~能不能夠跑起來都是問題~~）所以最初的DNS協議是直接明文的。按照[CloudFlare](https://www.cloudflare.com/learning/ddos/glossary/domain-name-system-dns/)的話來說，DNS可以被理解爲“互聯網的電話黃頁（phonebook of the internet）”,下圖來自CloudFlare

![ddos-dns-request](https://github.com/Mr-Sheep/Mr-Sheep.github.io/raw/master/resource/img/ddos-dns-request.webp)

但是明文傳輸的設計在現在看來有很多的不足，這也就是爲什麼GFW可以清楚的知道你訪問了甚麼網站，因爲你查詢的域名是明文傳輸的，同時還可以對你的DNS查詢進行投毒，篡改結果。以GFW爲例，會將黑名單內的網址解析到Facebook等公司的ip上去。比如說我的域名 mrsheep.xyz。

鑑於以上這些情況，更加安全的DNS查詢與傳輸顯得非常重要，於是在2011年的時候，全球的根服務器以及大多數的頂級域名都部署了[DNSSEC](https://tools.ietf.org/html/rfc4035)

## DNSSEC

上面說道的[DNSSEC](https://tools.ietf.org/html/rfc4035)僅僅在最初的DNS協議上加入了數字簽名，以此校驗查詢結果的合法性（不是正確性），也就是說，GFW還是可以達到DNS污染的目的

## DNSCrypt

官網在這裏:[DNSCrypt](https://dnscrypt.info)

原理：[DNSCrypt/Protocol](https://dnscrypt.info/protocol/)

維基詞條：[DNSCrypt](https://zh.wikipedia.org/wiki/DNSCrypt)

簡單來說DNSCrypt會對你的DNS查詢進行加密以及數字簽名驗證，但是DNS的到達時間可以被識別

openDNS和Quad 9均有部署該協議

# DNS over TLS

建立於TLS之上的DNS查詢，使用端口`853`，因爲使用TLS的原因，TLS採用CA認證，所以DoT同時具備“合法性”和”真實性“。

由於DNS是建立於TLS之上的，這代表着DoT會混在一堆tls連接中，無法被提出來特別針對

同時，大多數的瀏覽器並不支持DoT，可以使用Overture等軟件實現。



# DNS over HTTPS

最新的協議，[RFC 8484][1]，簡單的來說就是在DoT上套了一層http協議，可以理解爲DNS-over-HTTP-over-TLS，以此達到僞裝的效果，從而繞過運營商的重定向代理，實現方式其實蠻丑的。。。

由於中華人民共和國網絡出口常年故障，導致使用境外的DoH服務商會大幅增加解析時間

從本質上來講和DoT沒什麼區別，但是相對於DoT來說有更多的大佬支持，比如Mozilla，下圖基本說明了DoH的作用，圖自Mozilla

![DoH](https://github.com/Mr-Sheep/Mr-Sheep.github.io/raw/master/resource/img/DoH.png)



[1]:https://datatracker.ietf.org/doc/rfc8484/

# 參考文章

[[对比4种强化域名安全的协议——DNSSEC，DNSCrypt，DNS over TLS，DNS over HTTPS](https://program-think.blogspot.com/2018/10/Comparison-of-DNS-Protocols.html)]

