---
title: "Clash/Surge Geoip"
date: 2020-02-25T17:13:17+08:00
draft: false
---

今天更新Country.mmdb文件的時候發現需要License Key，看過了[Maxmind官方的一份聲明](https://blog.maxmind.com/2019/12/18/significant-changes-to-accessing-and-using-geolite2-databases/)後發現,從2019年12月30日開始，要求用戶註冊Maxmind賬戶獲取免費的金鑰，來下載Geolite 2數據庫。

根據MM的說法，由於GDPR和CCPA的實施，限制了他們基於CC4.0的分享，解決方案就是提供資料的企業和用戶都遵循一定的數據隱私規定。有興趣可以查看官方的聲明

<!--more-->

# 解決方法

1. 註冊Maxmind帳號：https://www.maxmind.com/en/geolite2/signup，註冊完成之後會發送郵件到郵箱完成初始化設定
2. Service - My License Key - Generate new license key
3. https://download.maxmind.com/app/geoip_download?edition_id=GeoLite2-Country&license_key=$LICENSEKEY&suffix=tar.gz,將`$LICENSEKEY`替換爲上一步獲得的`license key`

## Surge

1.添加新的rewrite規則

```
^https:\/\/geolite\.maxmind\.com\/download\/geoip\/database\/GeoLite2-Country\.tar\.gz https://download.maxmind.com/app/geoip_download?edition_id=GeoLite2-Country&license_key=$LICENSEKEY&suffix=tar.gz 302
```

將`$LICENSEKEY`替換爲上一步獲得的`license key`

2.MitM添加新的規則

`geolite.maxmind.com`或者是` *.maxmind.com`

## Clash

Clash Cli版本推薦直接獲取Country.mmdb文件後替現有文件,CFW可以在面板中直接粘貼License Key或者使用如下URL

`https://download.maxmind.com/app/geoip_download?edition_id=GeoLite2-Country&license_key=$LICENSEKEY&suffix=tar.gz`

將`$LICENSEKEY`替換爲上一步獲得的`license key`

## 

# 參考

[Significant Changes to Accessing and Using GeoLite2 Databases](https://blog.maxmind.com/2019/12/18/significant-changes-to-accessing-and-using-geolite2-databases/)

[关于GeoIP的二三事|Clash/Surge如何更新GeoIP库](https://merlinblog.xyz/wiki/geoip.html)

