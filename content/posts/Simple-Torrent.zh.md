---
title: "Simple Torrent"
date: 2020-02-22T15:25:34+08:00
draft: false
---

[Simple-Torrent](https://github.com/boypt/simple-torrent)基於[Cloud-Torrent](https://github.com/jpillora/cloud-torrent)修改而來，鑑於Cloud-Torrent作者已经不再维护，同時Simple-Torrent添加了很多必要的新Features。個人感覺相比較之下，Simple torrent會非常的方便

<!--more-->

# New-Features

**摘自[boypt/simple-torrent](https://github.com/boypt/simple-torrent)**

- Individual file download control (1.1.3+)
- Run extrenal program on tasks completed: `DoneCmd`
- Stops task when seeding ratio reached: `SeedRatio`
- Download/Upload speed limiter: `UploadRate`/`DownloadRate`
- Detailed transfer stats in web UI.
- [Torrent Watcher](https://github.com/boypt/simple-torrent/wiki/Torrent-Watcher)
- K8s/docker health-check endpoint `/healthz`
- Extra trackers from extrenal source
- Protocol Handler to `magnet:`
- Magnet RSS subscribing supported
- Flexible config file accepts multiple formats (.json/.yaml/.toml) ([by spf13/Viper](https://github.com/spf13/viper/)) (1.2.0+)

Also:

- Single binary
- Cross platform
- Embedded torrent search
- Real-time updates
- Mobile-friendly
- Fast [content server](http://golang.org/pkg/net/http/#ServeContent)
- IPv6 out of the box
- Updated torrnet engine from [anacrolix/torrent](https://github.com/anacrolix/torrent)

# 安装

### 一鍵安裝指令：

` bash <(wget -qO- https://raw.githubusercontent.com/boypt/simple-torrent/master/scripts/quickinstall.sh)`

这个命令会完成如下：

	1. 下载Simple-torrent bin
 	2. 创建自动启动文件/etc/systemd/system/cloud-torrent.service
 	3. 创建配置文件cloud-torrent.json

完成之后默认端口为`3000`

```
systemctl start cloud-torrent
systemctl restart cloud-torrent
systemctl stop cloud-torrent
systemctl status cloud-torrent
```

### 一些建議配置：

#### TLS

個人會建議開啓https支持，畢竟你不希望下載部分資源的時候使用明文傳輸

可以自簽一張憑證，如下命令一鍵生成即可

```visual basic
openssl req -x509 -nodes -sha256 -newkey \ rsa:2048 -keyout server.key -out server.crt \ -config openssl.cnf -extensions v3_req
```

之後請編輯`/etc/systemd/system/cloud-torrent.service`

在啓動參數`ExecStart`中加上   --key-path /金鑰位置 --cert-path /證書位置,並更改端口爲443

#### Tracker

推薦[ngosang/trackerslist](https://github.com/ngosang/trackerslist),所有的Tracker均爲每日更新

- trackers_best (20 trackers) => [link](https://ngosang.github.io/trackerslist/trackers_best.txt) / [mirror](https://raw.githubusercontent.com/ngosang/trackerslist/master/trackers_best.txt)
- trackers_all (82 trackers) => [link](https://ngosang.github.io/trackerslist/trackers_all.txt) / [mirror](https://raw.githubusercontent.com/ngosang/trackerslist/master/trackers_all.txt)
- trackers_all_udp (48 trackers) => [link](https://ngosang.github.io/trackerslist/trackers_all_udp.txt) / [mirror](https://raw.githubusercontent.com/ngosang/trackerslist/master/trackers_all_udp.txt)
- trackers_all_http (30 trackers) => [link](https://ngosang.github.io/trackerslist/trackers_all_http.txt) / [mirror](https://raw.githubusercontent.com/ngosang/trackerslist/master/trackers_all_http.txt)
- trackers_all_https (4 trackers) => [link](https://ngosang.github.io/trackerslist/trackers_all_https.txt) / [mirror](https://raw.githubusercontent.com/ngosang/trackerslist/master/trackers_all_https.txt)
- trackers_all_ws (3 trackers) => [link](https://ngosang.github.io/trackerslist/trackers_all_ws.txt) / [mirror](https://raw.githubusercontent.com/ngosang/trackerslist/master/trackers_all_ws.txt)

# 最後

API 、遠程調用的我就不去嘗試了，由於涉及DCMA，伺服器位置偏僻，線路質量堪憂

Simple Torrent對於我來說最主要的就是下載4k電影之後回傳到本地

好啦，其實這個也是一篇水文，最近沒有什麼可以更新的內容