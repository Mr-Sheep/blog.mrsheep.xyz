---
title: "Mac SSH W/ Private Key"
date: 2021-05-03T23:17:00+08:00
draft: false
tags: ["MacOS X","SSH"]
categories: ["Fun"]
---

I want to permanently disable SSH password authentication on OS X and use only  private keys for ssh remote logins.

<!--more-->

To gain more security when remotely logging into my home Mac mini, I thought it would be better to use only private keys to log in instead of using tunneled clear passwords.

To do so, one needs to edit `/etc/ssh/sshd_config`; note that you will need root privilege for this command.

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



References:

[How to use SSH keys and disable password authentication](https://apple.stackexchange.com/questions/225231/how-to-use-ssh-keys-and-disable-password-authentication)