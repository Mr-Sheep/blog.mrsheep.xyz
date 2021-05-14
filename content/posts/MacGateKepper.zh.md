---
title: "MacGateKepper"
date: 2020-02-27T12:10:31+08:00
draft: false
---

記錄一下mac提示安裝包損壞或來自未知開發者的解決方法

其實MacOS 從很早之前就引入了一種安全機制：[維基百科：Gatekeeper](https://en.wikipedia.org/wiki/Gatekeeper_(macOS))， [Apple：About Gatekeeper](https://support.apple.com/en-us/HT202491)

解決方法也很簡單：`sudo spctl --master-disable`

