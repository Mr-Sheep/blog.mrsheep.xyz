---
title: "Wireguard, Take Me Home"
date: 2021-01-16T22:16:50+08:00
draft: false
tags: ["VPN","WireGuard"]
categories: ["Networking"]
---

Setup a WireGuard server on Raspberry Pi 4, allow a remote client to access the local network
在樹莓派上配置WireGuard來遠程訪問內網


<!---more--->

# 要求
- 公網ip地址，並且擁有可以訪問的端口（只要是個端口就行）
- UDP連結
- Linux，不一定是樹莓派

本來是想直接安裝在我的Bugtik上的，但是6.48的stable版本暫時沒有WireGuard支持，又想試試裝Mac mini上，在看了[這篇文章](https://barrowclift.me/post/wireguard-server-on-macos) 以後，我就直接放棄了（太懶）
用來訪問的設備是iOS和MacOS，其他的設備同理

# 配置Wireguard Server
### 安裝WireGuard
寫文章的時候(01-16-2021)樹莓派的Raspbian的內核版本還是[5.4.83](https://downloads.raspberrypi.org/raspios_full_armhf/images/raspios_full_armhf-2021-01-12/2021-01-11-raspios-buster-armhf-full.zip)，而WireGuard是在5.6正式加入kernel，所以需要加上樹莓派官方的testing源


另外廢話一下raspbian的升級，更多內容請看[官網文檔](https://www.raspberrypi.org/documentation/raspbian/updating.md):
```
sudo apt update
sudo apt full-upgrade
sudo apt -y dist-upgrade
sudo apt clean
```


樹莓派雖然是Debian改出來的，但是並沒有backport...

```
echo "deb http://archive.raspbian.org/raspbian testing main" | sudo tee --append /etc/apt/sources.list.d/testing.list
sudo apt update
sudo apt -y install wireguard -y
```

### 開啓轉發：
```
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf && sysctl -p
echo "net.ipv6.conf.all.forwarding=1" >> /etc/sysctl.conf && sysctl -p
```

### 金鑰🔑️生成

```
cd /etc/wireguard
wg genkey | tee privatekey | wg pubkey > publickey

```

### 節點配置
```
[Interface]
  PrivateKey = Node1 私鑰🔑️
  Address = 10.89.64.1/24, fd86:ea04:1120::1/64
  ListenPort = 9999 // 你想要的通訊埠
  PostUp   = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
  PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
[Peer]
  PublicKey = Node2 公鑰🔑️
  AllowedIPs = 10.89.64.2/32, fd86:ea04:1120::2/128
[Peer]
  PublicKey = Node3 公鑰🔑️
  AllowedIps = 10.89.64.3/32, fd86:ea04:1120::3/128
```
```
systemctl enable wg-quick@wg0
systemctl start wg-quick@wg0
```
## 路由器配置
我是Mikrotik 4011, 需要轉發端口和開啓ddns服務
使用WebFig的Terminal或者SSH上去
```
/ip cloud set ddns-enabled=yes

/ip cloud print
// 獲取你的ddns地址

// 配置端口轉發
/ip firewall nat add action=dst-nat chain=dstnat disabled=no dst-port=9999 in-interface=你的pppoe撥號interface protocol=udp to-addresses=樹莓派ip to-ports=9999 
```

# 訪問設備配置
先下載 WireGuard

[MacOS](https://itunes.apple.com/us/app/wireguard/id1451685025?ls=1&mt=12)

[iOS](https://itunes.apple.com/us/app/wireguard/id1441195209?ls=1&mt=8)

[Android](https://play.google.com/store/apps/details?id=com.wireguard.android)

### 配置：

- 點擊Generate Keypair 生成私鑰🔑️和公鑰🔑️，替換上文中`節點配置`部分中的Node公鑰🔑️

```
Address: 10.89.64.2/32
ListenPort: 9999
PublicKey: 樹莓派的公鑰
Endpoint: 遠程地址，可以使ddns域名:端口
AllowedIPs: 10.89.64.0/24, fd86:ea04:1120::1/64, 192.168.88.0/24
PersistentKeepAlive: 25

```
**允許遠程設備訪問內網**

在配置中`AllowedIPs`中加上你的內網ip段，如`192.168.88.0/24`

最後開啓WireGuard即可


