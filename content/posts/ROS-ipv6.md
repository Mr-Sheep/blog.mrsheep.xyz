---
title: "ROS Ipv6"
date: 2020-03-22T18:53:14+08:00
draft: false
---

h幫朋友升級了1000兆頻寬，順便就幫他上了雙栈，電信的配置真是惱火

<!--more-->

# 橋接

首先需要在光貓裏該成橋接，使用基地臺撥號，具體怎麼修改由很多介紹，我就不再寫了

[中国电信天翼光猫改桥接模式方法](https://www.yeboyzq.com/luyoujiaohuan/984.html)

[中国电信智能网关怎么更改桥接模式拨号啊 | V2EX](https://www.v2ex.com/t/468726)

# IPv6

## DHCP Client

> IPv6 -> DHCPv6 Client -> 新建

```
Interface: pppoe out
Request: prefix
Pool name: 隨意
Pool Prefix Length: 60/64/48

✅ Rapid Commit
✅ Add Default Route
```

## Address List

>  IPv6 -> Address -> 新建

```
Address: ::/64
From pool: 上一步Pool Name
Interface: bridge
Advertise：✅

如果報錯Duplicated Address Detected勾選
No DAD：✅
```

## DHCP Server

> IPv6 -> DHCP Address -> 新建

```
Name：隨意
Interface：bridge
Address Pool6: 第一步中的名字
Lease Time：小於第一步中的Prefix Expires 
✅ Allow Dual Stack Queue
```

