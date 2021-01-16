---
title: "Aapanel"
date: 2020-04-05T11:54:14+08:00
draft: false
tags: ["AApanel","BTpanel","寶塔"]
categories: ["hosting"]
---

aaPanel是寶塔面板的國際版本，我本來是很討厭面板的，安全性不能夠保證。但是必須上就沒辦法了，本文致力於將該面板的安全性進一步增強

<!--more-->

# 安裝

Centos :

```
yum install -y wget && wget -O install.sh http://www.aapanel.com/script/install_6.0_en.sh && bash install.sh
```

Ubuntu/Deepin :

```
wget -O install.sh http://www.aapanel.com/script/install-ubuntu_6.0_en.sh && sudo bash install.sh
```

Debian :

```
wget -O install.sh http://www.aapanel.com/script/install-ubuntu_6.0_en.sh && bash install.sh
```

# 安全改進

### SSL

**開啓之前請綁定域名**

開啓ssl之後是無法檢測到Security Entrance的路徑的

寶塔面板的ssl憑證僅僅能夠選擇自簽/Let's encrypt自動獲取，不支持在面板直接指定用戶上傳的憑證。爲了使用自己已經擁有的泛域名憑證，可以直接替換面板憑證

替換`/www/server/panel/ssl`路徑下的`certificate.pem`和`privateKey.pem`爲個人的即可

### IP白名單

擁有固定ip地址建議開啓白名單模式

### CDN

Cloudflare支持以下非常規端口

HTTP ports supported by Cloudflare:

```
- 80
- 8080
- 8880
- 2052
- 2082
- 2086
- 2095
```

HTTPS ports supported by Cloudflare:

```
- 443
- 2053
- 2083
- 2087
- 2096
- 8443
```

配置cdn需要在寶塔中新建網站，設置域名爲你的面板域名（PHP純靜態）

反向代理127.0.0.1:端口