---
title: "V2Ray中轉"
date: 2019-05-10T16:51:13+08:00
draft: false
---

國內到我HE的小機線路峰值ping偏高，但是ip很清真，剛好我的HK小機的ISP和HE有transit
方案如下

1. 使用UUID1鏈接中轉服務器，將流量路由到`#1`服務器
2. 使用UUID2鏈接中轉服務器，將流量路由到`#2`服務器
3. 使用UUID3鏈接中轉服務器，直接使用當前服務器訪問

<!--more-->

# 服務器配置

### 末端服務器配置

參考[Caddy/Nginx + V2Ray][1]

### 中轉服務器

```json
  "log": {
    "access": "/var/log/v2ray/access.log",
    "error": "/var/log/v2ray/error.log",
    "loglevel": "warnings"
  },
   "inbounds": [
    {
      "port": 端口,
      "protocol": "vmess",
      "settings":{
        "clients": [
        {
          "id": "UUID1",
          "alterId": 16,
          "email": "Email1（隨意寫）"
        },
        {
          "id": "UUID2",
          "alterId": 16,
          "email": "Email2（隨意）"
        },
        {
          "id": "UUID3",
          "alterId": 16,
          "email": "Email3（隨意）"
        }
      ]
    },
    //鏈接中轉服務器的協議爲wss
      "streamSettings": {
        "network": "ws",
        "security": "auto",
        "wsSettings": {
          "path": "/你的path/",
          "connectionReuse":true,
          "headers":{
            "Host":"域名",
          }
        },
      "disableInsecureEncryption": true
    }
  }
],
  "outbound":{
  //使用UUID3，不進行中轉
      "protocol": "freedom",
      "settings": {}
    },
  "outboundDetour": [
    {
    //使用UUID1，路由至下一節點，下一節點使用普通tcp
      "protocol": "vmess",
      "settings": {
        "vnext": [
          {
            "address": "你的ip",
            "port": 端口,
            "users": [
              {
                "id": "和下一節點一致",
                "alterId": 16,
                "security": "auto"
              }
            ]
          }
        ]
      },
      "streamSettings": {
        "network": "tcp",
        "security": "none"
      },
      "tag": "tag1之後會用到"
    },
    //使用UUID2，路由至下一節點，採用wss
    {
      "protocol": "vmess",
      "settings": {
        "vnext": [
          {
            "address": "域名",
            "port": 443,
            "users": [
              {
                "id": "和下一節點一致",
                "alterId": 16,
                "security": "auto"
              }
            ]
          }
        ]
      },
      "streamSettings": {
        "network": "ws",
        "security": "tls",
        "wsSettings": {
          "connectionReuse":true,
          "path": "/你的path/",
          "headers":{
            "Host":"域名"
          }
        }
      },
      "tag": "tag2之後會用到，和上面的不一樣"
    }],
  "routing": {
  //路由策略
    "strategy": "rules",
    "settings": {
      "rules": [
        {
          "type": "field",
          "user": "和上面的Email1一致",
          "outboundTag": "上面寫的tag1"
        },
        {
          "type": "field",
          "user": "和上面的Email2一致",
          "outboundTag": "上面寫的tag2"
        }
      ]
    }
  }
}
```

### Client

配置不同的uuid即可

# 吐槽

英文單詞需要加s...

一定要记得

# contribute

[V2Ray 白话文教程][2]
[Project V][3]

































[1]: https://blog.mrsheep.xyz/2019/09/15/WebServer-V2Ray/
[2]: https://toutyrater.github.io/
[3]: https://v2ray.com

