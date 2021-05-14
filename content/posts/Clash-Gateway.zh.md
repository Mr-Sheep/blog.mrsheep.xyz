---
title: "Clash 透明代理"
date: 2019-09-19T19:11:26+08:00
draft: false
tags: ["Proxy","Clash","Proxy"]
categories: ["Networking"]
---

水文一篇，因爲家裏的網絡中有類似小米盒子，Apple TV這樣的設備，自身配置不是很好，鏈接效率會被降低，還不如在RPi 3B配置一個透明代理
我本來的設想是v2ray + iptables + dnsmasq，然後突然想到Clash自帶了一個簡單的DNS以及透明代理功能，再加上CoreDNS岂不是美滋滋？最后配合ROS指定的ip走透明代理，還要什麼自行車？
<!--more-->

# 環境

需要準備的包,以apt为例

```bash
sudo apt-get install supervisor iptables-persistent net-tools curl wget vim  -y
```

開啓轉發

```bash
echo "net.ipv6.conf.all.forwarding = 1" >> /etc/sysctl.conf && sysctl -p
```

# Clash配置

隨便找一個目錄，cd進入
在[Clash的Release頁面][1]找到你需要的版本，

樹莓派3B的話是**clash-linux-armv6**

笔记本的话是**amd64**

我最終還是換成了Thinkpad X201t作爲網關

wget到本地，其實也可以下載到電腦再sftp上來，節約時間（github日常抽風）
如果電腦上已經有Clash，可以同時將**Country.mmdb**上傳，這樣clash就不需要重新下載了

```bash
gzip -d clash-linux-armv6-v0.15.0.gz
mv clash-linux-armv6-v0.15.0 clash-linux
chmod +x clash-linux

```

在該目錄下創建一個名爲**config.yaml**的配置文件

```json
port: 8001
socks-port: 1081
redir-port: 9280
allow-lan: true
mode: Rule
log-level: info
external-controller: 127.0.0.1:9090
secret: ""

dns:
  enable: true
  ipv6: false
  listen: 0.0.0.0:53
  enhanced-mode: redir-host
  nameserver:
      - tls://dns.rubyfish.cn:853
  fallback:
      - tls://1.1.1.1:853
      - tls://dns.google

Proxy:
# shadowsocks
- { name: "SS", type: ss, server: "地址", port: 端口, cipher: AEAD_CHACHA20_POLY1305, password: "密碼" }

# vmess
- { name: "HK", type: vmess, server: 地址, port: 端口, uuid: 你的uuid, alterId: 16, cipher: auto, network: ws, ws-path: ws目錄, tls: true, skip-cert-verify: true }

Proxy Group:
- { name: "Proxy", type: select, proxies: ["HK"] }

Rule:
# LAN
- DOMAIN-SUFFIX,local,DIRECT
- IP-CIDR,127.0.0.0/8,DIRECT
- IP-CIDR,172.16.0.0/12,DIRECT
- IP-CIDR,192.168.0.0/16,DIRECT
- IP-CIDR,10.0.0.0/8,DIRECT
- IP-CIDR,17.0.0.0/8,DIRECT
- IP-CIDR,100.64.0.0/10,DIRECT

# Final
- GEOIP,CN,DIRECT
- MATCH,Proxy
```

注意：

1. 必須配置`redir-port`因爲你要做透明代理
2. DNS那裏配置**監聽和enhanced-mode**
3. 開啓Allow Lan並開放對應防火牆

我配置的dns都是支持DoT的DNS服務器，國內的選擇紅魚，國外選擇CF，具體Clash請求DNS原理請參考Github頁面

# CoreDNS

参考文章 [CoreDNS配置][3]

CoreDNS需要开在高端口，或者修改iptables

请将上面的配置文件中的dns部分改为：

```json
dns:
  enable: true
  ipv6: false
  listen: 0.0.0.0:53
  enhanced-mode: redir-host
  nameserver:
      - 127.0.0.1:你的Coredns端口
```



# iPtables配置

我的本地網段是192.168.88.0/16

```
iptables -t nat -N Clash
//創建名爲Clash的規則
iptables -t nat -A Clash -d 192.168.88.0/16 -j RETURN
//不對本地進行代理
iptables -t nat -A Clash -p tcp -j REDIRECT --to-ports 9280
//這裏是你的redirport
iptables -t nat -A PREROUTING -p tcp -j Clash

```

然後`netfilter-persistent save`持久化

之後你就可以愉快的開啓Clash了
`./clash-linux -d .`
沒有Country.MMDB會自動下載，沒有報錯的話就說明配置完成了
可以把設備的DNS和Gateway都改成樹莓派的ip測試一下

### 守護進程 

#### Supervisor

新建一個supervisor配置
`vim /etc/supervisor/conf.d/clash.conf`
內容如下

```
[supervisord]
nodaemon=false

[program:clash]
priority=1
directory=clash所在的目錄
command=clash所在的目錄/clash-linux -d .
autorestart=true

```

#### PM2

官方推荐的进程管理器，十分的方便，安裝需要npm，天朝網路出口常年故障，能不能裝上隨緣

項目地址：  [github.com/Unitech/pm2][https://github.com/Unitech/pm2]

```bash
//安裝pm2
npm install pm2 -g
如果沒有nodejs
wget -qO- https://getpm2.com/install.sh | bash
//啓動程序
pm2 start app.js
//查看正在運行的程序
pm2 list
//查看日誌
pm2 logs
```

# ROS指定ip走透明代理

這部分的話參考[RouterOS指定内网IP走指定接口][2]
因爲不想改變現有的網絡結構，所以只讓指定的內網ip走樹莓派
在RouterOS中
IP - Firewall 中切換 Mangle 選項，添加一條新的規則
General 选项卡中，Chain 选择 prerouting ，Src.Address 寫小米盒子等需要代理的ip
如只需要特定协议通过指定接口就需要在protocol中选择对应的协议，全局则不选
Action 选项卡，在 Action 下拉菜单中选择 mark routing ，New Routing Mark 中填入该路由标记名称，Proxy_to_pi
如果多個設備的話，可以考慮配置個adress list，我暫時只有1個，就懶得啦
之後在IP - Routes配置路由，添加一條新的規則

但是我個人更加推薦全局應用，將dns和網關全部更改爲Clash的ip地址

# 參考文章

[RouterOS指定内网IP走指定接口][2]

[1]: https://github.com/Dreamacro/clash/releases/
[2]: http://www.myxzy.com/post-461.html
[3]: link-to-coredns