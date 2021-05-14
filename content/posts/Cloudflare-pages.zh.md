---
title: "Cloudflare Pages"
date: 2021-03-06T02:42:10+08:00
draft: false
tags: ["CDN","Cloudflare"]
categories: ["Fun"]
---

很久之前Cloudflare推出的[Pages](https://developers.cloudflare.com/pages/), 看起來很好玩就直接申請了Beta測試

<!--more-->

幾個月過後，Cloudflare終於通過了我的請求（真快）

> Your account now has access to Cloudflare Pages.
> Cloudflare Pages
> Welcome to the Cloudflare Pages beta! Your account is enabled to use the product. You can start using Pages today by logging into the Cloudflare dashboard, and clicking on the Pages (beta) icon.


測試了一下沒看到部署靜態頁面的選項，那就試試部署基於hugo的博客
參考官方文檔中的教程，只需要進行以下設定：

| Configuration option | Value  |
| -------------------- | ------ |
| Production branch    | master |
| Build command        | hugo   |
| Build directory      | public |

另外Cloudflare默認的`Hugo`版本爲`0.54.0`,如果build出錯可以在` [settings] - [environment variables]`中添加 `HUGO_VERSION`,賦值 `0.81.0`





至於效果，個人覺得和Github Pages無太大區別