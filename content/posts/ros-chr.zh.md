---
title: "ROS CHR"
date: 2020-04-23T18:48:40+08:00
draft: false
---

本文主要是記錄一下如何dd安裝ROS CHR並配置好IPv4和IPv6

<!--more-->

# 安裝ROS

## SystemRescueCD

在你的vps提供商的面板處上傳SystemRescueCD的鏡像，複製連接地址即可：

| Release      | SystemRescueCd-6.1.3 for amd64                               | SystemRescueCd-6.1.3 for i686                                |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Download ISO | [systemrescuecd-amd64-6.1.3.iso](https://osdn.net/projects/systemrescuecd/storage/releases/6.1.3/systemrescuecd-amd64-6.1.3.iso) | [systemrescuecd-i686-6.1.3.iso](https://osdn.net/projects/systemrescuecd/storage/releases/6.1.3/systemrescuecd-i686-6.1.3.iso) |

或者例如vultr這類服務商會提供SystemRescueCD

## DD系統

1. VNC登入VPS，選擇`Boot SystemRescueCd using default options`

2. 進入系統後使用`passwd`重置密碼，可以使用ssh連線伺服器（方便複製粘貼）

   ssh登入以後，執行

```
root@sysresccd ~]# wget https://download.mikrotik.com/routeros/6.45.8/chr-6.45.8.img.zip
# 請在mikrotik官網獲取最新版本CHR Raw Disk Image

[root@sysresccd ~]# unzip chr-6.45.8.img.zip 
[root@sysresccd ~]# dd if=chr-6.45.8.img of=/dev/vda 
[root@sysresccd ~]# mount /dev/vda1 /mnt/
[root@sysresccd ~]# cat > /mnt/rw/autorun.scr << EOF
# Auto configure script on RouterOS first boot
# feel free to customize it if you really need
/ip address add address=[你的vps ip]/22 interface=ether1 network=route
# 在ipinfo.io/your_ip可以獲得對應的route
/ip route add distance=1 gateway=閘道
/ip service
set telnet port=23023
set ftp disabled=yes
set ssh disabled=yes
set api disabled=yes
set api-ssl disabled=yes
/ip dns set servers=1.1.1.1,1.0.0.1
EOF
[root@sysresccd ~]# umount /mnt/
[root@sysresccd ~]# reboot 
# 重啓後即安裝完成

```

**注意：請記得移除之前上傳的iso**

# IPv6

ROS沒有辦法正常找到類似vultr等的默認gateway，ping一下`ff02::2`，fe80開頭的地址就是你的網關了

打開terminal

```
/ipv6 settings set forward=no
/ipv6 address add interface=ether1 address=你的ipv6/64
/ipv6 route add dst-address=::/0 gateway=網關地址%ether1
/ipv6 nd set [ find default=yes ] hop-limit=64 interface=ether1 managed-address-configuration=yes other-configuration=yes
```

# 參考

[Install MikroTik RouterOS on a Vultr VPS](https://www.vultr.com/docs/install-mikrotik-routeros-on-a-vultr-instance)

[vyos 配置和 bgp 宣告 ipv6 (vyos: 从入门到放弃](https://blog.ni-co.moe/public/572.html)