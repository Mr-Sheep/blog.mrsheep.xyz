---
title: "Nftables Forward"
date: 2020-04-10T08:42:34+08:00
draft: false
---

之前寫過直接使用V2Ray的鏈式轉發實現中轉的文章，[傳送門](https://blog.mrsheep.xyz/posts/v2ray-fw/)

但是最近在嘗試配置v2Board的時候，我沒辦法直接編輯配置文件，所以就另想辦法

<!--more-->

# nftables

## 什麼是nftables？

> **nftables** replaces the popular **{ip,ip6,arp,eb}tables**. This software provides a new in-kernel packet classification framework  that is based on a network-specific Virtual Machine (VM) and a new **nft** userspace command line tool. **nftables** reuses the existing Netfilter subsystems such as the existing hook  infrastructure, the connection tracking system, NAT, userspace queueing  and logging subsystem.
>
> This software also provides **libnftables**, the high-level userspace library that includes support for JSON, see man (3)libnftables for more information.

Google Transalate結果如下：

>** nftables **取代了流行的** {ip，ip6，arp，eb}桌子**。 該軟件提供了一個新的內核數據包分類框架，該框架基於特定於網絡的虛擬機（VM）和新的** nft **用戶空間命令行工具。 ** nftables **重用了現有的Netfilter子系統，例如現有的hook基礎結構，連接跟踪系統，NAT，用戶空間排隊和日誌記錄子系統。
>
>該軟件還提供** libnftables **，這是高級用戶空間庫，其中包括對JSON的支持，有關更多信息，請參見man（3）libnftables。

## 配置

### 準備工作

#### 關閉原有防火牆

```
systemctl stop firewalld
systemctl stop iptables
systemctl disable firewalld
systemctl disable iptables
```

#### 禁用selinux（CentOS）

```
setenforce 0
sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config  
```

#### 開啓端口轉發

```
echo -e "net.ipv4.ip_forward=1" >> /etc/sysctl.conf && sysctl -p
```

### 手動配置（不推薦）

```
nft add set ip nat { type ipv4_addr\; }
nft add rule ip nat POSTROUTING ip daddr 目標ip tcp dport 443 counter snat to 本機ip
add rule ip nat POSTROUTING ip daddr 目標ip udp dport 443 counter snat to 本機ip
```

###  nftables nat规则生成工具(推薦)

Github項目： [nftables-nat-rust](https://github.com/arloor/nftables-nat-rust)

root運行以下命令

#### 優點

- 適合動態ip，自動解析ddns
- 支持域名
- 自動檢測本機ip
- 支持端口段

#### 下載安裝

```
wget -O /usr/local/bin/nat http://cdn.arloor.com/tool/dnat
chmod +x /usr/local/bin/nat
```

#### 進程守護

編輯：`/lib/systemd/system/nat.service`

```
[Unit]
Description=dnat-service
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/usr/local/bin/nat /etc/nat.conf
LimitNOFILE=100000
Restart=always
RestartSec=60

[Install]
WantedBy=multi-user.target
EOF
```

#### 配置文件

配置文件位於`/etc/nat.conf`

每行代表一個規則

```
SINGLE,本機端口,目標端口,mrsheep.xyz
RANGE,本機端口起,本機端口末,mrsheep.xyz
```

- SINGLE：单端口转发：本机49999端口转发到baidu.com:59999
- RANGE：范围端口转发：本机50000-50010转发到baidu.com:50000-50010

#### 注意事項

- 不支持多网卡

  

  

  

# 參考

[nftables-nat-rust](https://github.com/arloor/nftables-nat-rust)