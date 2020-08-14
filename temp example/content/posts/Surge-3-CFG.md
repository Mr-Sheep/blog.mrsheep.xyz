---
title: "Surge 3 CFG"
date: 2019-11-27T09:35:37+08:00
draft: false
---

11-30-2019 更新：Surge as Gateway

05-03-2020 更新：移除overture部分

Surge是一個功能十分強大的網絡工具箱，可以作為高性能的HTTP和SOCKS代理服務器。可以完成抓包，分析等工作，
支持規則代理，支持多種代理（支持V2Ray）。並包括基於TLS的HTTP(s), Socks5等

<!--more-->

# 為甚麼使用Surge

我知道有一大把開源項目，比如做的非常好的Clash，並且有MacOS的Dashboard，但是由於目前仍出於Pre-Release階段，部分功能並不完善和穩定
相比較下，Surge作為一個付費軟件，客服支持做的更好（交了錢當然好）
但是更重要的，是Surge作為網關使用會比Clash更加的強大，用戶界面看著更加的完善，有完整的User Manual,支持MitM和抓包等
算了我編不下去了，反正好用就行了
關於購買這一塊兒，別去找盜版了，找不到的

## 無限試用

通過更改系統時間的方法可以獲得無限的使用，但是這樣做會無法啟動Dashboard,不清楚是不是因為試用版的原因

另，別想着去找盜版了~~（別問我怎麼知道的）~~

# 配置

基本的配置教程官方寫的十分的詳細，可以直接參考[nssurge manual][1]
我只寫一些，畢竟人家大佬寫了那麼多

## DNS

Surge支持DoH，使用方法為直接在`DNS`配置中添加，或者是直接修改配置文件
在頭部 `[General]` 部分添加:
`dns-server = 你指定的dns服務器`,
但是支持DoH的大多在CN沒有服務器，延遲比較高，但是解決了DNS投毒的問題,一下列部分支持DoH的服務商

1. https://cloudflare-dns.com/dns-query
2. https://1.1.1.1/dns-query //1.1.1.1這個地址很可能會被作為保留地址，比如說某為
3. https://dns.google.com/resolve
4. https://8.8.8.8/dns-query
5. https://dns.quad9.net/dns-query
6. https://9.9.9.9/dns-query
7. https://dns.rubyfish.cn/dns-query
   關於DoT和DoH誰更好已經爭論了很久了

## V2Ray

Surge從3.3.1版本（應該是）開始支持原生V2Ray，不再需要External-Proxy或者是前置代理實現
格式如下

```bash
你的代理名字 = vmess, 服務器地址, 端口, username=uuid, tls13=true, ws=true, ws-path=路徑
```

注：

1. `tls13 = true` 即強制使用tls1.3，使用前請**確認支持**，如果**不支持**,請將該命令改成`tls = true`來啟動tls
2. 如果你不需要驗證證書，可以把證書添加進入mac的**鑰匙串**,或者你自己研究怎麼跳過驗證
3. `ws-header`如果有需要，可以單獨設置，具體請參見官方[Community][3]

# Gateway

將Surge作爲本地網關對於我個人來說比使用[Clash作爲網關](https://blog.mrsheep.xyz/posts/clash-gateway/)好用的多，Clash封裝的軟件總是有神神奇奇的問題。Surge採用 Fake-ip 方案，即對於所有dns查詢都返回198.18.x.x的ip。

### 甚麼是網關

可以參考[維基百科](https://zh.wikipedia.org/zh-hant/%E7%BD%91%E5%85%B3)的詞條

簡單的來說需要實現網絡之間的通信，就必須要通過網關。

### 配置

#### Surge

Surge的配置十分的簡單，只需要打開`Enhanced mode`即可，同時請注意，你的mac的網關地址需要手動設定爲上級路由地址。

並且需要在surge中指定dns地址

根據[Surge作者的說法](https://medium.com/@Blankwonder/surge-mac-as-gateway-b0bd68464a4b)，“macOS 10.13 后完善了 Kernel 的转发机制”

使用Surge作爲網關的優勢由很多，比如

1. 支持tcp和udp代理
2. 搞定類似於Google Home這樣不支持手動配置代理的設備
3. Hands-Freed的全局代理
4. Apple TV使用代理會有莫名其妙的Bug

#### 客戶端

在網絡設置中將網關地址改成`mac ip`，dns改成`198.18.0.2`或者其他

#### 路由器

同上，僅需要在配置中將網關地址和dns更改即可。



[1]: https://manual.nssurge.com/
[2]: https://www.reddit.com/r/privacy/comments/89pr15/dnsoverhttps_vs_dns_overtls_vs_dnscrypt/
[3]: https://community.nssurge.com/