---
title: "Run Ur Own Doh Server"
date: 2020-05-03T01:57:04+08:00
draft: false
---

你的DNS Query不會被加密，直接以明文的方式傳輸，這樣ISP或者網路中的任何人都可以清楚的瞭解到你瀏覽了哪些網站。所以就來折騰一下Caddy DoH Server （Caddy + DoH server + overture）

雖然說Cloudflare等有提供免費的DoH DNS服務，但是查詢DNS的使用自己的ip/域名總是要好玩那麼一點。

同時在翻了一邊網上的資料以後，發現基本都是基於Nginx的，麻煩死了，Caddy他不好嗎？

> Unfortunately, by default, DNS is usually slow and insecure. Your  ISP, and anyone else listening in on the Internet, can see every site  you visit and every app you use — even if their content is encrypted.  Creepily, some DNS providers sell data about your Internet activity or  use it to target you with ads.

DoH的原理可以參見：https://blog.mrsheep.xyz/posts/dot-vs-doh/#dns-over-https

<!--more-->

# 架構

圖是我畫的，不想做圖。。。

GoodNote真是好東西（剛更換貼膜見諒）

![photo_2020-05-03_02-12-17.jpg](https://i.loli.net/2020/05/03/LARIq3theD62yrm.jpg)

好吧你要是覺得我畫的丑，你可以看看Nginx版本的或者下面的圖（都是抄的）

![Architecture](https://www.aaflalo.me/wp-content/uploads/2018/10/DoH_architecture.svg)

```
+--------------+                                +------------------------+
| Application  |                                |  Recursive DNS Server  |
+-------+------+                                +-----------+------------+
        |                                                   |
+-------+------+                                +-----------+------------+
| Client side  |                                |      doh-server        |
| cache (nscd) |                                +-----------+------------+
+-------+------+                                            |
        |         +--------------------------+  +-----------+------------+
+-------+------+  |    HTTP cache server /   |  |   HTTP service muxer   |
|  doh-client  +--+ Content Delivery Network +--+ (Apache, Nginx, Caddy) |
+--------------+  +--------------------------+  +------------------------+
```

# 服務端

## Overture

> Overture means an orchestral piece at the beginning of a classical music composition, just like DNS which is nearly the first step of surfing the Internet.
>
> 序曲是古典音樂創作開始時的管弦樂作品，就像DNS一樣，這幾乎是上網的第一步。

我用overture來向upstream轉發我的請求

### 下載overture

前往[shawn1m/overture](https://github.com/shawn1m/overture/releases/latest)獲取最新編譯版本,以1.6.1版本amd64爲例：

```bash
wget https://github.com/shawn1m/overture/releases/download/v1.6.1/overture-linux-amd64.zip
mv overture-linux-amd64.zip /usr/local/bin/overture && unzip overture-linux-amd64.zip
```

### 配置

Overture本身是支持分流的，這裏只貼出我自己的配置文件，具體可以參見作者github readme或者我的[另一篇文章](https://blog.mrsheep.xyz/posts/overture-cfg/)

```
{
  "BindAddress": ":53",
  "DebugHTTPAddress": "127.0.0.1:5555",
  "PrimaryDNS": [
    {
      "Name": "DNSPod",
      "Address": "119.29.29.29:53",
      "Protocol": "udp",
      "SOCKS5Address": "",
      "Timeout": 6,
      "EDNSClientSubnet": {
        "Policy": "disable",
        "ExternalIP": "",
        "NoCookie": true
      }
    },
    {
      "Name": "AliDNS",
      "Address": "223.5.5.5:53",
      "Protocol": "udp",
      "SOCKS5Address": "",
      "Timeout": 6,
      "EDNSClientSubnet": {
        "Policy": "auto",
        "ExternalIP": "",
        "NoCookie": true
      }
    }
  ],
  "AlternativeDNS": [
    {
      "Name": "CloudFlareDNS DoT",
      "Address": "one.one.one.one:853",
      "Protocol": "tcp-tls",
      "SOCKS5Address": "",
      "Timeout": 6,
      "EDNSClientSubnet": {
        "Policy": "disable",
        "ExternalIP": "",
        "NoCookie": true
      }
    },
    {
      "Name": "CloudFlareDNS DoH",
      "Address": "https://cloudflare-dns.com/dns-query",
      "Protocol": "https",
      "SOCKS5Address": "",
      "Timeout": 6,
      "EDNSClientSubnet": {
        "Policy": "disable",
        "ExternalIP": "",
        "NoCookie": true
      }
    },
    {
      "Name": "GoogleDNS",
      "Address": "dns.google:853",
      "Protocol": "tcp-tls",
      "SOCKS5Address": "",
      "Timeout": 6,
      "EDNSClientSubnet": {
        "Policy": "disable",
        "ExternalIP": "",
        "NoCookie": true
      }
    },
    {
      "Name": "OpenDNS",
      "Address": "208.67.222.222:443",
      "Protocol": "tcp",
      "SOCKS5Address": "",
      "Timeout": 6,
      "EDNSClientSubnet": {
        "Policy": "auto",
        "ExternalIP": "",
        "NoCookie": true
      }
    }
  ],
  "OnlyPrimaryDNS": false,
  "IPv6UseAlternativeDNS": false,
  "AlternativeDNSConcurrent": false,
  "PoolIdleTimeout": 15,
  "PoolMaxCapacity": 15,
  "WhenPrimaryDNSAnswerNoneUse": "PrimaryDNS",
  "IPNetworkFile": {
    "Primary": "./china_ip_list.txt",
    "Alternative": ""
  },
  "DomainFile": {
    "Primary": "",
    "Alternative": "./gfwlist_domain.txt",
    "Matcher":  "regex-list"
  },
  "HostsFile": {
    "HostsFile": "./hosts_sample",
    "Finder": "full-map"
  },
  "MinimumTTL": 0,
  "DomainTTLFile" : "./domain_ttl_sample",
  "CacheSize" : 99,
  "RejectQType": [255]
}

```

完成之後可以dig一下測試

## DoH-Server

### 安裝

因爲海綿寶寶找我有事情（其實就是懶），我直接用了[Antoine Aflalo](https://www.aaflalo.me)編譯好的[2.0.1](https://www.aaflalo.me/doh-server/doh-server_2.0.1_amd64.deb)版本.

```
wget https://www.aaflalo.me/doh-server/doh-server_2.0.1_amd64.deb
dpkg -i doh-server_2.0.1_amd64.deb
```

當然我建議你自己編譯一下，你需要一個Go環境。[Golang官方寫的Instruction](https://golang.org/doc/install)

```
git clone https://github.com/m13253/dns-over-https
cd dns-over-https
make
make install
```

### 配置

配置文件位於`/etc/dns-over-https/doh-server.conf`

```
# HTTP listen port
# 監聽的端口，上面v4下面v6
listen = [
    "127.0.0.1:8053",
    "[::1]:8053",
]
# 這個部分可以忽略，等下用Caddy解決
# TLS certification file
# If left empty, plain-text HTTP will be used.
# You are recommended to leave empty and to use a server load balancer (e.g.
# Caddy, Nginx) and set up TLS there, because this program does not do OCSP
# Stapling, which is necessary for client bootstrapping in a network
# environment with completely no traditional DNS service.
cert = ""

# TLS private key file
key = ""

# HTTP path for resolve application
path = "/dns-query"

# 上游dns伺服器，我用overture
# Upstream DNS resolver
# If multiple servers are specified, a random one will be chosen each time.
upstream = [
    "127.0.0.1:53",
]

# Upstream timeout
timeout = 60

# Number of tries if upstream DNS fails
tries = 10

# Only use TCP for DNS query
tcp_only = false

# Enable logging
verbose = false

```

```
systemctl restart doh-server
systemctl status doh-server
```

## Caddy

```
域名:443 {
   gzip
   tls 證書 金鑰
   proxy /dns-query localhost:8053 {
	transparent
   }

}
```

### 測試可用性

```
root@lax:~# curl "https://域名/dns-query?ct=application/dns-json&name=baidu.com&type=A"

{"Status":0,"TC":false,"RD":true,"RA":true,"AD":false,"CD":false,"Question":[{"name":"baidu.com.","type":1}],"Answer":[{"name":"baidu.com.","type":1,"TTL":527,"Expires":"Sat, 02 May 2020 18:50:25 UTC","data":"220.181.38.148"},{"name":"baidu.com.","type":1,"TTL":527,"Expires":"Sat, 02 May 2020 18:50:25 UTC","data":"39.156.69.79"}]}
```

# 參考

[Github m13253/dns-over-https](https://github.com/m13253/dns-over-https)

[Tutorial to setup your own DNS-over-HTTPS (DoH) server](https://www.aaflalo.me/2018/10/tutorial-setup-dns-over-https-server/)