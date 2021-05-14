---
title: "Personal Private Network with WireGuard"
date: 2021-01-08T17:53:43+08:00
draft: false
tags: ["VPN","WireGuard"]
categories: ["Networking"]
---
本文通過 WireGuard 設立虛擬局域網，將位於世界不同位置的伺服器連結起來，以實現 ~~裝13~~ 的目的

<!--more-->


# What is WireGuard?
WireGuard的目標是像SSH一樣易於配置和部署。 只需交換非常簡單的公鑰即可建立VPN連接，就像交換SSH密鑰一樣，其餘所有由WireGuard透明處理。 它甚至可以像Mosh一樣在IP地址之間漫遊。 無需管理連接，擔心狀態，管理守護程序。 WireGuard提供了一個非常基本但功能強大的界面。(其實就是官網的翻譯版)

>WireGuard 是一个易于配置、快速且安全的开源 VPN，它利用了最新的加密技术。目的是提供一种更快、更简单、更精简的通用 VPN，它可以轻松地在树莓派这类低端设备到高端服务器上部署。
>IPsec 和 OpenVPN 等大多数其他解决方案是几十年前开发的。安全研究人员和内核开发人员 Jason Donenfeld 意识到它们速度慢且难以正确配置和管理。这让他创建了一个新的开源 VPN 协议和解决方案，它更加快速、安全、易于部署和管理。
>WireGuard 最初是为 Linux 开发的，但现在可用于 Windows、macOS、BSD、iOS 和 Android。它仍在活跃开发中。
>[Linux中國](https://zhuanlan.zhihu.com/p/108365587)

![WireGuard](https://i1.wp.com/itsfoss.com/wp-content/uploads/2020/02/wireguard-logo.png?w=800&ssl=1)

**另外：WireGuard的特性導致其不適合用於翻牆，翻牆請使用v2ray等代理,如果對[WireGuard具體的實現感興趣可以參考WireGuard白皮書](https://www.wireguard.com/papers/wireguard.pdf)**

# Why
1. 爲了更安全的完成對無公網ip地址的伺服器的訪問，[避免因使用frp內網穿透導致內網伺服器中毒](https://www.v2ex.com/t/692012?p=1)
2. 伺服器過多需要更佳快捷的文件共享以及訪問
3. 易於配置和部署,安全
4. 更好的性能，已經加入 Linux Kernel 5.6
...

# 創建屬於你的虛擬局域網



![Slide1.jpeg](https://i.loli.net/2021/01/10/UtjgdhyNiBFLq49.jpg)

在這個私有的局域網下，Node1 作爲主節點存在，內部節點可以互相通訊，沒有公網ip的節點可以通過其他節點轉發

暫時沒有在中國大陸伺服器上部署WireGuard~~（CN2好貴，封不起）~~，香港節點由於機房維修暫時沒有加入虛擬網中

## 安裝WireGuard

本文所有節點採用的作業系統爲Debian 10

使用 ` apt -y install wireguard`即可安裝

如果提示 `not found`請在`/etc/apt/sources.list`中添加

```
deb https://deb.debian.org/debian buster-backports main contrib non-free
```

之後 `apt update`並重新執行安裝命令即可

## 配置WireGuard

### 金鑰🔑️生成

```
cd /etc/wireguard
wg genkey | tee privatekey | wg pubkey > publickey
```

### 節點配置

對於每個節點，編輯`/etc/wireguard/wg0.conf`

以Node 1（主節點）爲例

```
[Interface]
  PrivateKey = Node1 私鑰🔑️
  Address = 10.1.88.1/24, fd86:ea04:1120::1/64
  MTU = 1360
  ListenPort = 9999 // 你想要的通訊埠
  PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
  PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
  // 將PostUp和PostDown中的eth0修改爲你的interface，vultr是ens3
[Peer]
  PublicKey = Node2 公鑰🔑️
  AllowedIPs = 10.1.88.2/32, fd86:ea04:1120::2/128
[Peer]
  PublicKey = Node3 公鑰🔑️
  AllowedIps = 10.1.88.3/32, fd86:ea04:1120::3/128
[Peer]
  PublicKey = Node4 公鑰🔑️
  AllowedIps = 10.1.88.4/32, fd86:ea04:1120::4/128
[Peer]
  PublicKey = Node5 公鑰🔑️
  AllowedIps = 10.1.88.5/32, fd86:ea04:1120::5/128
```

其餘節點，以Node3爲例:

```
[Interface]
  PrivateKey = Node3 公鑰🔑️
  Address = 10.1.88.3/24, fd86:ea04:1120::3/64
  ListenPort = 9999 // 你想要的通訊埠
  PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
  PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
  // 將PostUp和PostDown中的eth0修改爲你的interface，vultr是ens3

[Peer]
  PublicKey = Node2 公鑰🔑️
  AllowedIps = 10.1.88.2/32, fd86:ea04:1120::2/64
  Endpoint = Node2公網ip:9999
  PersistentKeepalive = 25
[Peer]
  PublicKey = Node4 公鑰🔑️
  AllowedIps = 10.1.88.4/32, fd86:ea04:1120::4/128
  Endpoint = Node4公網ip:9999
  PersistentKeepalive = 25
[Peer]
  PublicKey = Node5 公鑰🔑️
  AllowedIps = 10.1.88.5/32, fd86:ea04:1120::5/128
  Endpoint = Node5公網ip:9999
  PersistentKeepalive = 25
[Peer]
  PublicKey = Node1 公鑰🔑️
  AllowedIps = 10.1.88.0/24, fd86:ea04:1120::1/64
  Endpoint = Node1公網ip:9999
  PersistentKeepalive = 25
```

> 其中，`AllowedIPs` 控制本地的路由表，可以根据情况修改，例如上面使用 `Node 1` 作为 Hub，访问所有非直连的节点（`Node 5` <–> `Node 2`） 使用 `Node 1` 作为中转 Hub 时，`AllowedIPs` 为 `10.1.88.0/24`。
>
> 同理，如果 `AllowedIPs` 为 `0.0.0.0/0`，则本机所有流量都会通过相应的 Peer 转发。

### 通过 systemd 管理 wireguard 服务

```
systemctl start wg-quick@wg0
systemctl enable wg-quick@wg0

查看狀態
wg

interface: wg0
  public key: (hidden)
  private key: (hidden)
  listening port: 50254

peer: 
  endpoint: (hidden):9999
  allowed ips: 10.1.88.5/32
  latest handshake: 1 minute, 4 seconds ago
  transfer: 528 B received, 272 B sent
  persistent keepalive: every 25 seconds

peer: 
  endpoint: (hidden):9999
  allowed ips: 10.1.88.4/32
  latest handshake: 1 minute, 57 seconds ago
  transfer: 472 B received, 648 B sent
  persistent keepalive: every 25 seconds

peer: 
  endpoint: (hidden):9999
  allowed ips: 10.1.88.0/24, fd86:ea04:1120::/64
  latest handshake: 1 minute, 57 seconds ago
  transfer: 184 B received, 648 B sent
  persistent keepalive: every 25 seconds

peer: 
  endpoint: (hidden):9999
  allowed ips: 10.1.88.3/32
  latest handshake: 2 minutes, 1 second ago
  transfer: 312 B received, 520 B sent
  persistent keepalive: every 25 seconds
```

### `RNETLINK answers: Operation not supported`解決

```
apt -y install libmnl-dev libelf-dev linux-headers-$(uname -r) build-essential pkg-config
```



# Credit 

['RNETLINK answers: Operation not supported'. (fresh Ubuntu, fresh Wireguard)](https://askubuntu.com/questions/973297/rnetlink-answers-operation-not-supported-fresh-ubuntu-fresh-wireguard)

[使用 Wireguard 构建全球私有局域网](https://blog.lipengwang.com/posts/wireguard/)