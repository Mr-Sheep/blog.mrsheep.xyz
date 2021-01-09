---
title: "VLan IPtv"
date: 2020-04-07T22:57:20+08:00
draft: false
---

還是上次的朋友，他家裏的一間房間裏僅有1個RJ45口，但是想在不損失網線性能，即不拆分爲4芯線的同時播放中國電信的iptv和進行網頁瀏覽。

剛好他家裡有ROS和一臺本來用於監控的GS105E，那就搞個vlan來玩玩唄

<!--more-->
Update 01,10,2021: Update details

# 前言

## VLan？

根據維基詞條 [虛擬區域網路](https://zh.wikipedia.org/zh-tw/%E8%99%9A%E6%8B%9F%E5%B1%80%E5%9F%9F%E7%BD%91)的解釋：

> **虛擬區域網路**（**Virtual Local Area Network**或簡寫**VLAN**, **V-LAN**）是一種建構於區域網路交換技術（LAN Switch）的[網路管理](https://zh.wikipedia.org/wiki/網絡管理)的技術，網管人員可以藉此透過控制[交換器](https://zh.wikipedia.org/wiki/交換器)有效分派出入區域網的封包到正確的出入埠，達到對不同實體區域網中的裝置進行邏輯分群（Grouping）管理，並降低區域網內大量資料流通時，因無用封包過多導致壅塞的問題，以及提昇區域網的資訊安全保障

VLan是一個虛擬的Lan，將一個實體的Lan分割成多個Vlan使用，分割出來的Vlan各自獨立且不能相互通訊。本文基於802.1q

## 其他解決辦法

其實並不是沒有其他的解決方法，比如說：

- 8芯改爲4芯，僅100mbps速度
- 重新埋，想想就不現實
- EOC？算了吧，本來就沒有

# 設備

## 需要的設備

- 支持802.1q vlan劃分的交換機/路由器
- 網線
- 腦子
- 筆電

## 網路拓撲

因爲懶就不製圖了

簡單的來說就是將Modern的iptv口和internet口都接入到主路由上。iptv接入ether 9，ether5 作爲trunk口連結至需要的房間中的gs105e的port 5

gs105e的port 1留給iptv

# 配置

## Switch

### 配置程式

Netgear的交換機請前往[官方網站](https://www.netgear.com/support/product/GS105Ev2.aspx#docs)獲取 [ProSAFE Plus Config Utility](https://www.netgear.com/support/product/GS105Ev2.aspx#ProSAFE%20Plus%20Configuration%20Utility%20v2.7.7)

(這個真心難用)

### 關閉iGMP並開啓VLAN

系統 --> 組播 --> IGMP 偵聽 --> IGMP偵聽狀態： **關閉IGMP**

Vlan --> 802.1q VLAN --> 高級設定 -->**啓用** 

### 配置VLan

添加兩個新的Vlan id，id想多少就多少，例如 2，3

這裏將2作爲internet的vlan，3用語iptv

路由器ether5作爲與交換機連結的埠，ether9連結iptv

Vlan 成員：

Vlan 2:

| 端口   | 1    | 2    | 3    | 4    | 5    |
| ------ | ---- | ---- | ---- | ---- | ---- |
| 組操作 |      | U    | U    | U    | T    |

Vlan 3:

| 端口   | 1    | 2    | 3    | 4    | 5    |
| ------ | ---- | ---- | ---- | ---- | ---- |
| 組操作 | U    |      |      |      | T    |

關於tag和untagged，有興趣參考：[What are tagged and untagged ports](https://gtacknowledge.extremenetworks.com/articles/Q_A/What-are-tagged-and-untagged-ports) 以及 [[釐清網管型Switch 裡與VLAN + Tag + Untag 之間的觀念| The Will Will](https://blog.miniasp.com/post/2011/02/16/Understanding-Switch-VLAN-Tagged-Untagged)](https://blog.miniasp.com/post/2011/02/16/Understanding-Switch-VLAN-Tagged-Untagged)

### 配置PVID

Port VLAN ID需要和上面配置的Vlan id對應，即

| 端口 | 1    | 2    | 3    | 4    | 5    |
| ---- | ---- | ---- | ---- | ---- | ---- |
| PVID | 3    | 2    | 2    | 2    | 2    |

## ROS

- 在interface --> 新建 --> VLAN中新建兩條VLan，名字隨便,這裏我分別叫做IPTV-VLAN和LAN-VLAN。 vlan id分別爲Switch中配置兩個id，interface都選擇連結gs105e的端口，即ether 5 

- 將ether 5與ether 9從原有bridge中移除
- Bridge --> 新建 新建橋接，名稱隨意，這裏是**IPTV**
- bridge --> ports，將iptv vlan和 iptv接入的端口橋接在一起, 新建：
  - interface --> ether 9, bridge--> 你新建的橋接名字
  - interface --> iptv-vlan，bridge --> 橋接名字

<img src="https://i.loli.net/2021/01/10/V5SvmDar4WQb8hl.png" alt="Bridge" style="zoom:50%;" />

<img src="https://i.loli.net/2021/01/10/O57QhP1slmGkB8u.png" alt="Vlan" style="zoom:50%;" />

搞定

# 參考連結

[Mikrotik Manual:Basic VLAN switching](https://wiki.mikrotik.com/wiki/Manual:Basic_VLAN_switching)

[Configuring a Netgear GS105E Switch with LAN/WAN ports](https://www.arednmesh.org/content/configuring-netgear-gs105e-switch-lanwan-ports)

[Reddit: Netgear GS108Ev3 VLAN configuration](https://www.reddit.com/r/homelab/comments/7mpxcm/netgear_gs108ev3_vlan_configuration/)