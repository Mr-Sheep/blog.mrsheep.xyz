---
title: "V2Ray Force Ipv4"
date: 2019-09-15T02:11:05+08:00
draft: false
tags: ["V2Ray","V2Fly","Anti-Censorship"]
categories: ["networking","Proxy"]
---

我在机器上开了6to4的隧道，本來是準備用來廣播我自己的ipv6，但是這樣子导致V2Ray流量会被转发到HE的隧道ip，最后NF会直接ban掉HE的ip
所以我不废话了...

<!--more-->

# Freedom

需要用到的就是Freedom是這個協議，從V2ray官網上有如下解釋：
**Freedom 是一个出站协议，可以用来向任意网络发送（正常的） TCP 或 UDP 数据**
在目标地址为域名时，Freedom 可以直接向此域名发出连接（"AsIs"），或者将域名解析为 IP 之后再建立连接（"UseIP"、"UseIPv4"、"UseIPv6"）。解析 IP 的步骤会使用 V2Ray 内建的 DNS。默认值为"AsIs"。

1. (V2Ray 4.6+) 当使用 **"UseIP"** 模式，并且出站连接配置中指定了sendThrough时，Freedom 会根据sendThrough的值自动判断所需的IP类型，IPv4 或 IPv6。
2. (V2Ray 4.7+) 当使用 **"UseIPv4"** 或 **"UseIPv6"** 模式时，Freedom 会只使用对应的 IPv4 或 IPv6 地址。当sendThrough指定了不匹配的本地地址时，将导致连接失败

需要用到這個協議:[V2Ray文檔][1]
關於配置文檔格式:[V2Ray文檔][2]

# 具體配置

一點都不複雜，和本部分沒有關係的都已經去掉

```
  "inbounds": [
    {
      "port": 29708,
      "protocol": "vmess",
      "settings":{
        "clients": [
        {
          "id": "uuid",
          "alterId": 16,
        }
      ]
    },
      "streamSettings": {
        "network": "ws",
        "security": "auto",
        "wsSettings": {
          "path": "/",
          "connectionReuse":true,
          "headers":{
          }
        },
      "disableInsecureEncryption": true
    }
  }
],
  "outbound":{
      "sendThrough": "0.0.0.0",
      "protocol": "freedom",
      "settings": {
          "domainStrategy": "UseIPv4",
          "userLevel": 0
      }
    }
}

```

其實這又是水文一篇

# 參考

V2ray.com

[1]: https://www.v2ray.com/chapter_02/protocols/freedom.html
[2]: https://www.v2ray.com/chapter_02/01_overview.html#outboundobject