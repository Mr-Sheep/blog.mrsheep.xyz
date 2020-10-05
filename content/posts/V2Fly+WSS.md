---
weight: 4
title: "V2Fly+WSS"
date: 2020-08-22T11:57:06+08:00
draft: false
author: "MrSheep"
authorLink: "https://mrsheep.xyz"
description: "重寫基於Caddy V2"
resources:
tags: ["V2ray", "HTML"]
categories: ["anti-censorship"]

lightgallery: false
---


之前寫過的[Caddy/Nginx + V2Ray](https://blog.mrsheep.xyz/v2ray-ws/)是基於 Caddy v1 以及一些古董配置寫的
偷個懶，直接重寫一份

<!--more-->

## 我的環境

環境 A

```bash
         _,met$$$$$gg.           root@
      ,g$$$$$$$$$$$$$$$P.        OS: Debian 9.13 stretch
    ,g$$P""       """Y$$.".      Kernel: x86_64 Linux 4.14.129-bbrplus
   ,$$P'              `$$$.      Uptime: 1d 23h 35m
  ',$$P       ,ggs.     `$$b:    Packages: 421
  `d$$'     ,$P"'   .    $$$     Shell: zsh 5.3.1
   $$P      d$'     ,    $$P     CPU: QEMU Virtual CPU version (cpu64-rhel6) @ 2.7GHz
   $$:      $$.   -    ,d$$'     RAM: 288MiB / 1005MiB
   $$\;      Y$b._   _,d$P'
   Y$$.    `.`"Y$$$$P"'
   `$$b      "-.__
    `Y$$
     `Y$$.
       `$$b.
         `Y$$b.
            `"Y$b._
                `""""
```

環境 B

```bash
                             OS: Pop 20.04 focal
                             Kernel: x86_64 Linux 5.4.0-7634-generic
         #####               Uptime: 3d 11h 53m
        #######              Packages: 10
        ##O#O##              Shell: zsh 5.8
        #######              Resolution: 3840x1080
      ###########            DE: GNOME 3.36.4
     #############           WM: Mutter
    ###############          WM Theme: Pop
    ################         GTK Theme: Nordic-bluish-accent [GTK2/3]
   #################         Icon Theme: Papirus-Dark
 #####################       Font: Fira Sans Semi-Light 10
 #####################       Disk: 612G / 933G (69%)
   #################         CPU: Intel Core i7-9750H @ 12x 4.5GHz [58.0°C]
                             GPU: GeForce RTX 2080 with Max-Q Design
                             RAM: 10329MiB / 40030MiB

```

## Caddy

### 安裝 Caddy

**注： 如果需要通配符 ssl 憑證，請直接轉到 [XCaddy](#xCaddy) 部分**
在 [github/caddyserver/caddy/releases](https://github.com/caddyserver/caddy/releases) 下載對應版本的 caddy

#### Debian 系

下載 release 裏面的 deb 包即可，以 2.1.1 版本爲例

```
wget https://github.com/caddyserver/caddy/releases/download/v2.1.1/caddy_2.1.1_linux_amd64.deb
sudo dpkg -i caddy_2.1.1_linux_amd64.deb
```

#### 非 Debian 系

如果是非 debian 系用戶

```bash
wget https://github.com/caddyserver/caddy/releases/download/v2.1.1/caddy_2.1.1_linux_amd64.tar.gz
tar xvzf caddy_2.1.1_linux_amd64.tar.gz
mv caddy /usr/local/bin/caddy
```

需要手動創建 `/lib/systemd/system/caddy.service`

```bash
Description=Caddy
Documentation=https://caddyserver.com/docs/
After=network.target

[Service]
User=caddy
Group=caddy
ExecStart=/usr/bin/caddy run --environ --config /etc/caddy/Caddyfile
ExecReload=/usr/bin/caddy reload --config /etc/caddy/Caddyfile
TimeoutStopSec=5s
LimitNOFILE=1048576
LimitNPROC=512
PrivateTmp=true
ProtectSystem=full
AmbientCapabilities=CAP_NET_BIND_SERVICE

[Install]
WantedBy=multi-user.target
```

### xCaddy

Caddy 官方提供 xcaddy 來允許用戶編譯自己的 caddy，需要安裝 go 環境

#### 配置 GO

已有 Golang 環境請跳過本部分,以下以`go1.15`爲例
[下載對應版本的 Go](https://golang.org/dl/)，解壓到`/usr/local`
編輯 `/etc/profile` 或 `$HOME/.profile`， 加入

```bash
export PATH=$PATH:/usr/local/go/bin
```

下載安裝完成之後`source 你編輯的文件`

#### 編譯 Caddy

Syntax:

```
$ xcaddy build [<caddy_version>]
    [--output <file>]
    [--with <module[@version][=replacement]>...]

    <caddy_version> is the core Caddy version to build; defaults to CADDY_VERSION env variable or latest. 你想要build的版本，默認值爲env中的Caddy_Version或最新版本
    --output changes the output file. 輸出文件名
    --with can be used multiple times to add plugins by specifying the Go module name and optionally its version, similar to go get. Module name is required, but specific version and/or local replacement are optional. 你需要添加的插件
```

我使用 Cloudflare 作爲我的 DNS provider, 所以我需要的是 [caddy-dns/cloudflare](https://github.com/caddy-dns/cloudflare)，其他 dns 也可以在[github/caddy-dns](https://github.com/caddy-dns/)尋找, 使用以下命令即可進行 build

```
xcaddy build --with github.com/caddy-dns/cloudflare
```

如果中途報錯，很可能不是你的問題，請善用 issue 查詢
如果你實在無法編譯，就乖乖的去用 per domain cert 吧
之後的步驟請參考[非 Debian](#非Debian系)部分創建 service

### 配置

#### 使用現有憑證：

```json
你的域名 {
	encode gzip zstd
	tls 證書.pem 金鑰.pem
	reverse_proxy /ws路徑 127.0.0.1:本地v2ray端口 {
		header_up Host {http.request.host}
        	header_up X-Real-IP {http.request.remote.host}
        	header_up X-Forwarded-For {http.request.remote.host}
        	header_up X-Forwarded-Port {http.request.port}
        	header_up X-Forwarded-Proto {http.request.scheme}
	}

}

```

#### 自動申請憑證

##### Per Domain

```json
你的域名 {
	encode gzip zstd
	tls 你的郵箱
	reverse_proxy /ws路徑 127.0.0.1:本地v2ray端口 {
		header_up Host {http.request.host}
        	header_up X-Real-IP {http.request.remote.host}
        	header_up X-Forwarded-For {http.request.remote.host}
        	header_up X-Forwarded-Port {http.request.port}
        	header_up X-Forwarded-Proto {http.request.scheme}
	}

}
```

##### 通配符

注：需要自行編譯 Caddy

```json
你的域名 {
    encode gzip zstd
    tls {
		dns cloudflare {cloudflare_api_key}
    }
    reverse_proxy /ws路徑 127.0.0.1:本地v2ray端口 {
        header_up Host {http.request.host}
        	header_up X-Real-IP {http.request.remote.host}
        	header_up X-Forwarded-For {http.request.remote.host}
        	header_up X-Forwarded-Port {http.request.port}
        	header_up X-Forwarded-Proto {http.request.scheme}
	}
}
```

## V2Fly (V2Ray)

V2Fly == V2Ray 但是 V2Ray !=== V2Fly

### 安裝 V2Ray

```bash
curl -O https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-release.sh
bash install-release.sh
```

會創建如下文件

```bash
installed: /usr/local/bin/v2ray
installed: /usr/local/bin/v2ctl
installed: /usr/local/share/v2ray/geoip.dat
installed: /usr/local/share/v2ray/geosite.dat
installed: /usr/local/etc/v2ray/config.json
installed: /var/log/v2ray/
installed: /var/log/v2ray/access.log
installed: /var/log/v2ray/error.log
installed: /etc/systemd/system/v2ray.service
installed: /etc/systemd/system/v2ray@.service
```

### 配置

#### 如何選擇適合的配置？

以下部分來自[v2fly/v2ray-examples](https://github.com/v2fly/v2ray-examples)的 README.md
![如何选取适合自己的配置](https://raw.githubusercontent.com/v2fly/v2ray-examples/master/how-to-choose/how-to-choose-a-v2ray-plan.png)
附加说明：
尽管 Websocket+TLS+Web 可能称得上是现阶段最好的方案，但绝对不是推荐新手一上来就尝试的方案，更不是 V2Ray 唯一的用法。
<br><br>同时，你应当了解，每个地区的网络状况不同 (主要指对不同协议的 QoS 程度)，你可以将所有配置都尝试一遍来寻找最适合自己的，尽量少问、最好不问“为什么我的 V2Ray 这么慢？”这样的问题。
本文將以 Websocket + TLS 爲例，如需要參考其他配置模板，請前往[v2fly/v2ray-examples](https://github.com/v2fly/v2ray-examples)

#### config.json

編輯`/usr/local/etc/v2ray/config.json`

```json
{
  "log": {
    "access": "",
    "error": "",
    "loglevel": "debug"
  },
  "dns": {
    "hosts": {
      "geosite:category-ads": "127.0.0.1"
    },
    "servers": [
      "https+local://cloudflare-dns.com/dns-query",
      "https://dns.google/dns-query",
      "https://1.1.1.1/dns-query",
      "127.0.0.1"
    ]
    //DNS部分不是必須，可以刪除，我爲了保證伺服器端的dns查詢不被污染，使用Cloudflare的DoH服務
  },
  "inbounds": [
    {
      "sniffing": {
        "enable": true,
        "destOverride": ["http", "tls"]
      },
      "port": 你的端口,
      "protocol": "vmess",
      "settings": {
        "clients": [
          {
            "id": "你的uuid",
            "alterId": 你的alterId,
            "email": "隨便寫"
          }
        ]
      },
      "streamSettings": {
        "network": "ws",
        "security": "auto",
        "wsSettings": {
          "path": "你的ws path",
          //注意"/ws/"和"/ws"是有區別的
          "connectionReuse": true,
          "headers": {
            "Host": ""
          }
        },
        "disableInsecureEncryption": true
      }
    }
  ],
  "outbound": {
    "sendThrough": "0.0.0.0",
    "protocol": "freedom",
    "settings": {
      "domainStrategy": "UseIP"
    }
  },
  "routing": {
    "domainStrategy": "AsIs",
    "rules": [
      {
        "type": "field",
        "outboundTag": "block",
        "protocol": ["bittorrent"]
        //這裏屏蔽了bt協議，不需要的話刪除就好
      }
    ]
  }
}
```

<br/>由於編寫本文時 v2ray 仍處於 v4.27，但在 4.28.0 以後如果把 `alterId` 设置为 `0` 则代表为启用 VMess AEAD。
<br/>詳見：[v2fly/v2fly-github-io/issues/20](https://github.com/v2fly/v2fly-github-io/issues/20)

## BBR
非生產環境可以嘗試bbrplus,生產環境建議使用原生bbr
<br/>BBR是什麼可以參考 [谷歌的博文](https://cloud.google.com/blog/products/gcp/tcp-bbr-congestion-control-comes-to-gcp-your-internet-just-got-faster)
<br/>也可以看[APNIC的博文](https://blog.apnic.net/2017/05/09/bbr-new-kid-tcp-block/)
### 原裝正品bbr
請確定內核版本 > 4.9
```
cat >>/etc/sysctl.conf << EOF
net.core.default_qdisc=fq
net.ipv4.tcp_congestion_control=bbr
EOF
```
之後 `sysctl -p`確認返回值
複覈是否生效
```
sysctl net.ipv4.tcp_available_congestion_control
lsmod | grep bbr
```
返回：
```
net.ipv4.tcp_available_congestion_control = reno cubic bbr
tcp_bbr                20480  41 
```

### bbr plus
```
wget -N --no-check-certificate "https://raw.githubusercontent.com/chiakge/Linux-NetSpeed/master/tcp.sh"
chmod +x tcp.sh
./tcp.sh
```
首先選擇你需要的bbr方案，替換內核重啓之後選擇開啓即可。

## 最後
完成以後重啓Caddy和V2Ray即可
```bash
systemctl restart caddy
systemctl restart v2ray
```
再不寫點東西，博客都長草了