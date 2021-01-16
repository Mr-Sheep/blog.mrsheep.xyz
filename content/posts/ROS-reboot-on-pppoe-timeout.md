---
title: "ROS Reboot on ping Timeout"
date: 2021-01-16T23:07:34+08:00
draft: false
tags: ["Mikrotik","ROS"]
categories: ["Networking"]
---

通過ping檢測撥號是否掉線來重啓撥號/重啓設備
中國電信：世界加錢可得
<!--more-->
![ad90885ad1daf053.jpg](https://i.loli.net/2021/01/17/RBTbxi7csUFgGNt.jpg)
###
朋友電信的光纖非常的喜歡掉線，可能是改用了ROS撥號的緣故，重新撥號後無法訪問網路
就幫他寫了個簡單的netWatch腳本

1. 開啓netWatch

以114.114.114.114作爲測試對象，如果ping不通就執行`netWatchBot`腳本
```
/tool netwatch
add down-script="log info \"Netwatch missed a ping to 114.114.114.114 - starting 10 minute timeout script\" ;\
    \n/system script run netWatchBot" host=114.114.114.114
```
2. netWatchBot

邏輯：如果5分鐘一次都ping不通，則嘗試重啓pppoe-client，重啓後繼續嘗試5分鐘，還是不行就重啓設備

其中，CT-1000是我的pppoe interface的名字
```
/system script
add dont-require-permissions=no name=netWatchBot owner=admin policy=ftp,reboot,read,write,policy,test,password,sniff,sensitive,romon source=":if ([/ping 114.114.114.114 interval=5 count=60] =0) \
    do={\
    \n  /interface pppoe-client disable CT-1000;\
    \n  /interface pppoe-client enable CT-1000;\
    \n  :if ([/ping 114.114.114.114 interval=5 count=60] =0) do={\
    \n     /system reboot\
    \n  }\
    \n}"
```


Credit: 
[How to auto-reboot if remote IP down for 5 minutes](https://forum.mikrotik.com/viewtopic.php?f=9&t=56138)