---
title: "Mac SSH W/ Private Key"
date: 2021-05-03T23:17:00+08:00
draft: false
tags: ["MacOS X","SSH"]
categories: ["Fun"]
---

僅使用私鑰通過ssh訪問MacOS

<!--more-->

爲了增加安全性，決定僅僅允許使用私鑰連結MacOS


編輯`/etc/ssh/sshd_config` （需要root權限）

```ssh
Port 22
ListenAddress 0.0.0.0
ListenAddress ::

PermitRootLogin no
PasswordAuthentication no
PermitEmptyPasswords no
ChallengeResponseAuthentication no

PubkeyAuthentication yes
AuthorizedKeysFile	.ssh/authorized_keys

```



參考:

[How to use SSH keys and disable password authentication](https://apple.stackexchange.com/questions/225231/how-to-use-ssh-keys-and-disable-password-authentication)