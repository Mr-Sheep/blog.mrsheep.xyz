---
title: "Overture 配置"
date: 2019-11-16T17:04:11+08:00
draft: false
---

Overture是一個由GO編寫的DNS，跨平台兼容，能夠很好的實現我分流的需求，相比与CoreDNS无论如何我都編譯不成功的proxy插件，實在是便捷了很多

<!--more-->

5-3-2020更新：Overture早就支持了DoH了，特地來更新一下

## Features

摘自[github.com/shawn1m/overture][1]

- Full IPv6 support
- Multiple DNS upstream 
  - Via UDP/TCP with custom port
  - Via SOCKS5 proxy (TCP only)
  - With EDNS Client Subnet (ECS) [RFC7871](https://tools.ietf.org/html/rfc7871)
- Dispatcher 
  - IPv6 record (AAAA) redirection
  - Custom domain
  - Custom IP network
- Minimum TTL modification
- Hosts (Random order if there are multiple answers)
- Cache with ECS

## 配置

從[Release][2]裡下載prebuilt版，並解壓

下載GFW和CN_ip列表,放置於同一目錄

```bash
wget https://cokebar.github.io/gfwlist2dnsmasq/gfwlist_domain.txt && wget https://raw.githubusercontent.com/17mon/china_ip_list/master/china_ip_list.txt
```



之後就可以魔改配置文件了

` vim config.json`

```json
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

個人比較推薦[PM2][3]作爲進程管理

```bash
npm install pm2 -g
#如果你沒nodejs可以使用 wget -qO- https://getpm2.com/install.sh | bash
pm2 start overture-linux -- -c ./config.json
```

你也可以創建service,以樹莓派爲例：

```
[Unit]
Description=overture

[Service]
WorkingDirectory=/home/pi
ExecStart=/home/pi/overture/overture -c /home/pi/overture/config.json
Restart=on-failure
RestartSec=3

[Install]
WantedBy=multi-user.target

```



## 測試

最後的話就dig測試一下就好了拉，

{{<  notice tip >}}

用我的域名測試的話，我的域名是被污染了的

{{< /notice >}}

```bash
; <<>> DiG 9.11.5-P4-5.1-Debian <<>> blog.mrsheep.xyz @127.0.0.1 -p 15353
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 18416
;; flags: qr rd ra ad; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;blog.mrsheep.xyz.              IN      A

;; ANSWER SECTION:
blog.mrsheep.xyz.       299     IN      A       104.27.165.30
blog.mrsheep.xyz.       299     IN      A       104.27.164.30

;; Query time: 535 msec
;; SERVER: 127.0.0.1#15353(127.0.0.1)
;; WHEN: Fri Nov 15 19:16:43 PST 2019
;; MSG SIZE  rcvd: 109
```



[1]: https://github.com/shawn1m/overture
[2]: https://github.com/shawn1m/overture/releases
[3]: https://github.com/Unitech/pm2

