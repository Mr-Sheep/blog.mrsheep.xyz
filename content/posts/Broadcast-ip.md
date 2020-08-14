---
title: "廣播你的ip"
date: 2019-09-13T17:47:00+08:00
draft: false
---

我的AS號和IP都已經沾灰了，再不用就要徹底忘了。



在你開始之前，請確定你的ISP 为你分配了一个公网 IPv4, 並且你拥有 ASN 和一个至少为 / 48 的 ipv6 block

<!--more-->

# 開始之前的一點PY交易

準備Py之前，請確定如下內容(來自HE),不然你去PY也沒有意義

**Through this interface you can request a static, IPv6 BGP tunnel that will receive full transit routes from AS6939 and be able to announce your RIR allocated IPv6 space. To minimize any issues or delays in turnup please note the following:
    Your ASN must be allocated from a RIR. (Public ASNs only)
    We must be able to validate that this is your ASN/address space (SWIP entries or a LoA will do)
    We will prefix-filter your BGP session. If you are adding new announcements, please let us know via an email to ipv6@he.net.
    We do not filter traffic. However, we do reserve the right to filter at our sole discretion as needed.
    If this is your first tunnel for this ASN, you should receive a message to your account's email address. Please follow the instructions in the email to complete the verification process. During this period the tunnel and BGP session will be unavailable. If you have not heard from us within 48 hours then please email ipv6@he.net.
    地址：[tunnelbroker.net][1]**

然後你就可以去註冊一個帳號，之後创建一个 BGP 隧道，選擇最近的节点，填寫你的ASN以及IP段
最後你需要一封Authorize Letter（LoA）
提交24小時左右PY交易就會成功

# 創建隧道

訪問tunnelbroker

`Tunnel - Example Config - Linux net-tools`
配置大概張這個樣子

```ifconfig sit0 up
ifconfig sit0 inet6 tunnel ::0.0.0.0
ifconfig sit1 up
ifconfig sit1 inet6 add ::::/64
route -A inet6 add ::/0 dev sit1
```

# 廣播

### 安裝bird6

`yum install bird6 -y`

或者

`apt -y install bird`

### 編輯 `bird6.conf`

注意：

1. router id請寫本機ip
2. local as是你的asn
3. source add是HE給你的client ip
4. neighbor是 HE分配的 server ip,6939 为 he 的 asn 号码

```
router id 本機ip;
protocol bgp he
{
        local as 208884;
        source address Client_ip;
        import none;
        export all;
        graceful restart on;
        multihop 2;
        neighbor server_ip as 6939;
}
protocol static
{
    route  你的ip段 via cleint_ip;
}
protocol device
{
        scan time 5;
}
```

之後保存推出，重載bird配置，或者重啓bird即可
最後`birdc6 show proto all`
顯示Established即可

```
name     proto    table    state  since       info
he       BGP      master   up     17:40:47    Established   
  Preference:     100
  Input filter:   REJECT
  Output filter:  ACCEPT
  Routes:         0 imported, 1 exported, 0 preferred
  Route change stats:     received   rejected   filtered    ignored   accepted
    Import updates:           3457          0       3457          0          0
    Import withdraws:            0          0        ---       3457          0
    Export updates:              1          0          0        ---          1
    Export withdraws:            0        ---        ---        ---          0
  BGP state:          Established
    Neighbor address: 2001:470:d6:e::1
    Neighbor AS:      6939
    Neighbor ID:      64.71.128.26
    Neighbor caps:    refresh AS4
    Session:          external multihop AS4
    Source address:   2001:470:d6:e::2
    Hold timer:       179/180
    Keepalive timer:  52/60

static1  Static   master   up     17:40:42    
  Preference:     200
  Input filter:   ACCEPT
  Output filter:  REJECT
  Routes:         1 imported, 0 exported, 1 preferred
  Route change stats:     received   rejected   filtered    ignored   accepted
    Import updates:              1          0          0          0          1
    Import withdraws:            0          0        ---          0          0
    Export updates:              0          0          0        ---          0
    Export withdraws:            0        ---        ---        ---          0

device1  Device   master   up     17:40:42    
  Preference:     240
  Input filter:   ACCEPT
  Output filter:  REJECT
  Routes:         0 imported, 0 exported, 0 preferred
  Route change stats:     received   rejected   filtered    ignored   accepted
    Import updates:              0          0          0          0          0
    Import withdraws:            0          0        ---          0          0
    Export updates:              0          0          0        ---          0
    Export withdraws:            0        ---        ---        ---          0
```

# 宣告

創建虛擬網卡並分配ip

```visual basic
    ip link add dev dummy1 type dummy
    ip link set dummy1 up
    ip addr add dev dummy1 2001:2333:2333::1/128
```

編輯**bird6.conf**加入

```visual basic
protocol direct
{
    interface "dummy*";
    import all;
}
```

之後

```
systemctl restart bird6
```

這樣 bird 將查詢所有 dummy 开头的网卡的 ip 并宣告这些 ip
 `birdc6 show route` 查看

# 參考

[IP 广播：在不借助你的 ISP 进行任何操作的情况下，广播 (组播) 你的 IPv6][1]
[IP 广播：使用 bird 广播 (组播) ipv6][2]

[1]: https://blog.ni-co.moe/public/563.html
[2]: https://blog.ni-co.moe/public/560.html