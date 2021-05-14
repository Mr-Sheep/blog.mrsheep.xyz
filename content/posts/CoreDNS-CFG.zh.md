---
title: "CoreDNS配置"
date: 2019-09-19T19:27:40+08:00
draft: false
---

CoreDNS是一個由GO語言編寫的輕量級開源DNS服務，跨平臺支持，並有一下幾個特性：

- 插件
- 配置簡單
- 跨平臺
- 通用 DNS 方案

官網： [CoreDNS](https://coredns.io)

<!--more-->

# 配置

在[https://github.com/coredns/coredns/releases](https://github.com/coredns/coredns/releases)下載最新的pre-built版本

### 自行編譯

如果你需要用到`Proxy`這個plugin的話，你需要自行編譯一份CoreDNS，具體原因參見[ISSUE #3234](https://github.com/coredns/coredns/issues/3234),編譯十分的簡單，但是網路環境的原因，推薦使用境外服務器編譯,但是當然，你要是相信國內的鏡像，我也不會攔着你

```bash
git clone https://github.com/coredns/coredns
cd coredns
make
```

在`make`之前，請編輯位於文件夾內的`plugin.cfg`文件，添加

```
proxy:github.com/coredns/proxy
```

# Corefile

CoreDNS使用Corefile作爲配置文件（和Caddyfile類似），[官方的Manual](https://coredns.io/manual/toc/#configuration)是這麼說的

**That file consists of one or more Server Blocks. Each Server Block lists one or more Plugins. Those Plugins may be further configured with Directives.**

在這裏就廢話不多，需要其他的配置請參考Manual

