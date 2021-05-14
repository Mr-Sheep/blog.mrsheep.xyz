---
title: "Firewalld"
date: 2020-03-14T18:42:37+08:00
draft: false
---

為了省事，可以直接上Firewalld，而且服務器就只需要開放個端口，firewalld完全夠用（其實就是太懶不想去配置iptables）
這篇文章僅紀錄一下
<!--more-->

### 安裝

CentOS默認即為Firewalld，debian需要額外安裝。
CentOS：
`yum install firewalld -y`
Debian:
` apt -y install firewalld`

### 命令

1. 查看開放的端口
   `firewall-cmd --list-ports`
2. 開放端口
   `firewall-cmd --zone=public --add-port=$port/$protocol --permanent`
   其中：
   - `--zone`：作用域
   - `$port`：需要開放的端口
   - `$protocol`：協議
   - `--permanent`：永久生效
3. 🈲️端口
   `firewall-cmd --zone=public --remove-port=$port/$protocol --permanent`
   其中：
   - `--zone`：作用域
   - `$port`：需要開放的端口
   - `$protocol`：協議
   - `--permanent`：永久生效
4. 重載配置
   `firewall-cmd --reload`
5. 停止
   `systemctl stop firewalld`
6. 開機啟動
   `systemctl enable firewalld`

