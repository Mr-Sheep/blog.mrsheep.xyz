---
title: "SSH Using PUBKEY"
date: 2020-03-12T15:37:46+08:00
draft: false
---

Secure Shell (SSH) 是一種加密的網絡傳輸協議，可在不安全的網絡中為網絡服務提供安全的傳輸環境

廢話不多，直接開始

<!--more-->

# 金鑰生成

## 算法選擇

建議Switch到[Ed25519](https://ed25519.cr.yp.to/)

- **🚨 DSA:** 不安全，OpenSSH version 7開始不再支持
- **⚠️ RSA:** 根據金鑰key size決定，3072和4096 😀. 其他的都需要升級，1024bit的會被認爲是unsafe的
- **👀 ECDSA:** ECDSA使用的NIST曲線上有一個[值得關注的問題](https://www.hyperelliptic.org/tanja/vortraege/20130531.pdf)
- **✅ Ed25519:** 最推薦的生成算法

## 金鑰生成

`ssh-keygen -o  -a 100 -t ed25519 -f ~/.ssh/id_ed25519 -C "一些comment"`

其中:

- -o: 儲存爲OpenSSH格式（在傳入了-t 參數後可以省略）
- -a: KDF（金鑰派生函數）重複的次數，理論上數值越大，暴力破解難度增加，但是會導致驗證速度變慢
- -t: 指定金鑰類型，如ed25519
- -f: 輸出文件以及位置
- -C: 註釋，隨便寫

## 安装公钥

键入以下命令，在服务器上安装公钥：

```
cd .ssh
cat id_rsa.pub >> authorized_keys
```

請修改權限

```
chmod 600 authorized_keys
chmod 700 ~/.ssh
```

编辑`/etc/ssh/sshd_config` 文件

```
RSAAuthentication yes
PubkeyAuthentication yes
```

給予root用戶登錄權限

```
PermitRootLogin yes
```

測試完成之後使用
`PasswordAuthentication no`
完成之後重啓ssh服務
`service sshd restart`