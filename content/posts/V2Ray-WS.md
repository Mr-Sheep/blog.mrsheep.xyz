---
title: "Caddy/Nginx + V2Ray"
date: 2019-09-15T13:37:25+08:00
draft: false
tags: ["V2Ray","V2Fly","Anti-Censorship"]
categories: ["networking","Proxy"]
---

相比較於之前的Nginx，Caddy v1是一個更加輕量級，安裝更加方便，配置超級簡單的web server，並且支持自動申請lets encrypt證書（雖然我用不到）
Caddy配置十分的簡單，幾分鐘即可以完成配置。本文使用Caddy V1實現，V2有一種開倒車的感覺，配置越來越複雜。

<!--more-->

# Caddy

### 下載安裝

[Caddy官網][1]

```bash
curl https://getcaddy.com | bash -s personal http.forwardproxy,http.proxyprotocol
```

創建service

```bash
curl -s https://raw.githubusercontent.com/GEM7/My_scripts/master/caddy/caddy.service -o /etc/systemd/system/caddy.service
```

```bash
[Unit]
#paste this to /usr/lib/systemd/system/caddy.service
Description=Caddy Service
After=network.target
Wants=network.target

[Service]
Type=simple
PIDFile=/var/run/caddy.pid
ExecStart=/usr/local/bin/caddy -conf /etc/Caddyfile
Restart=on-failure

[Install]
WantedBy=multi-user.target
```



### 配置

配置真心超級簡單，如果需要使用自己的證書，比如CF的證書，請將證書和私鑰保存本地，在配置文件中引用即可。

```json
你的域名:443 {
    root /www
    gzip
    index index.html
    tls 證書路徑 私鑰路徑
    //如果需要自動申請的話，請移除上面這行
    proxy 僞裝路徑 localhost:9999 {
           websocket
           header_upstream -Origin
    }
}

```

#### 關於證書

請參考另一篇文章：Caddy SSL

# Nginx

我之前使用是Nginx，但是由於體積大，配置複雜所以選用Caddy，但是這裏還是附上Nginx配置，僅供參考

```bash
server
{
    listen 80;
    listen 443 ssl http2;
    server_name 你的域名 *.你的域名;
    index index.php index.html index.htm default.php default.htm default.html;
    root /www/wwwroot/你的域名;
    
    #SSL-START SSL相关配置，请勿删除或修改下一行带注释的404规则
    #error_page 404/404.html;
    #HTTP_TO_HTTPS_START
    if ($server_port !~ 443){
        rewrite ^(/.*)$ https://$host$1 permanent;
    }
    #HTTP_TO_HTTPS_END
    ssl_certificate    /etc/letsencrypt/live/你的域名/fullchain.pem;
    ssl_certificate_key    /etc/letsencrypt/live/你的域名/privkey.pem;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;
    error_page 497  https://$host$request_uri;

    #SSL-END
    
    #ERROR-PAGE-START  错误页配置，可以注释、删除或修改
    error_page 404 /404.html;
    error_page 502 /502.html;
    #ERROR-PAGE-END
    
    #PHP-INFO-START  PHP引用配置，可以注释或修改
    include enable-php-72.conf;
    #PHP-INFO-END
    
    #REWRITE-START URL重写规则引用,修改后将导致面板设置的伪静态规则失效
    include /www/server/panel/vhost/rewrite/你的域名.conf;
    #REWRITE-END
    
    #禁止访问的文件或目录
    location ~ ^/(\.user.ini|\.htaccess|\.git|\.svn|\.project|LICENSE|README.md)
    {
        return 404;
    }
    
    #一键申请SSL证书验证目录相关设置
    location ~ \.well-known{
        allow all;
    }
    
    location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$
    {
        expires      30d;
        error_log off;
        access_log off;
    }
    
    location ~ .*\.(js|css)?$
    {
        expires      12h;
        error_log off;
        access_log off; 
    }
    access_log  /www/wwwlogs/你的域名.log;
    error_log  /www/wwwlogs/你的域名.error.log;
             location /ws/ {   
             proxy_redirect off;
             proxy_pass http://127.0.0.1:你的端口; 
             proxy_http_version 1.1;
             proxy_set_header Upgrade $http_upgrade;
             proxy_set_header Connection "upgrade";
             proxy_set_header Host $http_host;            
          }
}
```

證書申請之類的請自行參考使用**acme.sh**

### WebSocket解決Bad Request

轉自[从零开始安装爱国工具指南（十四）V2Ray补充、WebSockets模式补完](https://www.ivyseeds.cf/websockets2/)

#### 修改nginx.conf

文件位`于/usr/local/nginx/conf/`

在代码`http{}`段里添加一句

```
proxy_intercept_errors on;
```

保存退出即可

#### 修改vhost文件

文件位于`/usr/local/nginx/conf/vhost/`

在443端口的配置里加一句，比如說我跳轉回到[mrsheep.win](https://mrsheep.win)

```
error_page 400 = https://www.mrsheep.win;
```

保存退出重启nginx

```
service nginx restart
```



# V2Ray

## 安裝

### 官方安裝

**如果你想要使用管理腳本等，請直接跳過本節**
使用官方安裝腳本: [www.v2ray.com/chapter_00/install.html#linuxscript](https://www.v2ray.com/chapter_00/install.html#linuxscript)
**請使用root用戶運行一下腳本**
运行下面的指令下载并安装  V2Ray。当 yum 或 apt-get 可用的情况下，此脚本会自动安装 unzip 和 daemon。这两个组件是安装 V2Ray  的必要组件。如果你使用的系统不支持 yum 或 apt-get，请自行安装 unzip 和 daemon

```bash
bash <(curl -L -s https://install.direct/go.sh)
```

**此脚本会自动安装以下文件：**

`/usr/bin/v2ray/v2ray`：V2Ray 程序；
`/usr/bin/v2ray/v2ctl`：V2Ray 工具；
`/etc/v2ray/config.json`：配置文件；
`/usr/bin/v2ray/geoip.dat`：IP 数据文件
`/usr/bin/v2ray/geosite.dat`：域名数据文件
配置`/etc/v2ray/config.json`即可

### 腳本

這裏僅僅使用Multi-V2Ray，其他的請自行參考對應wiki

#### Multi-V2Ray

項目地址：https://github.com/Jrohy/multi-v2ray
運行命令安裝，

```bash
source <(curl -sL https://git.io/fNgqx)
```

部分VPS安裝完成之後提示找不到v2ray命令，請運行`/usr/local/multi-v2ray/v2ray`

##### 命令行參數：

```bash
   v2ray -h                   查看帮助
   v2ray -v                   查看版本信息
   v2ray start                启动 V2Ray
   v2ray stop                 停止 V2Ray
   v2ray restart              重启 V2Ray
   v2ray status               查看 V2Ray 运行状态
   v2ray log                  查看 V2Ray 运行日志
   v2ray update               更新 V2Ray 到最新Release版本
   v2ray update [version]     更新 V2Ray 到特定版本
   v2ray update.sh            更新 multi-v2ray 脚本
   v2ray update.sh [version]  更新 multi-v2ray 到特定版本
   v2ray add                  新增mkcp + 随机一种 (srtp | wechat-video | utp) header伪装的端口(Group)
   v2ray add [wechat|utp|srtp|dtls|wireguard|socks|mtproto|ss]     新增一种协议的组，端口随机,如 v2ray add utp 为新增utp协议
   v2ray del                  删除端口组
   v2ray info                 查看配置
   v2ray port                 修改端口
   v2ray tls                  修改tls
   v2ray tfo                  修改tcpFastOpen
   v2ray stream               修改传输协议
   v2ray stats                iptables流量统计
   v2ray clean                清理日志
```

更多請參考: https://github.com/Jrohy/multi-v2ray/wiki

## 配置

### TLS

證書的話可以直接使用Caddy自動申請Let‘s Encrypt的證書，3個月續命一次即可

其他的話如使用Cloudflare簽發的證書，請參考上面 [Caddy配置][配置]



### DNS

**推荐的DNS服务商（免费）：Geoscaling，Cloudflare（仅使用DNS）**
不建议使用国内解析服务，VPS提供的DNS服务（例如Conoha的DNS解析）

### BBR

#### 換核

如果你不需要，請直接跳過

```bash
//导入key
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
//安装 elrepo 的yum源
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm
//安装内核
yum --enablerepo=elrepo-kernel install kernel-ml-devel kernel-ml -y
//查看系统启动项：
cat /boot/grub2/grub.cfg |grep menuentry
//设置最新版本内核为默认启动：
grub2-set-default '最新的內核，如CentOS Linux (4.8.5-1.el7.elrepo.x86_64) 7 (Core)
```

**注：請一定等待安裝完成**

此时，查看默认启动的系统版本

```bash
grub2-editenv list
```

显示为'CentOS Linux (4.8.5-1.el7.elrepo.x86_64) 7 (Core)'或類似，reboot之后再次查看 `uname -r`



#### 開啓BBR

複製如下命令即可

```bash
cat >>/etc/sysctl.conf << EOF
net.core.default_qdisc=fq
net.ipv4.tcp_congestion_control=bbr
EOF
```

再继续内核参数：

```
sysctl -p
```

返回類似結果

```
net.core.default_qdisc = fq
net.ipv4.tcp_congestion_control = bbr
```

最后保险起见，复核一下BBR是否生效：

```
sysctl net.ipv4.tcp_available_congestion_control

lsmod | grep bbr
```

返回類似

```
net.ipv4.tcp_available_congestion_control = reno cubic bbr

tcp_bbr                20480  41 
```



#### CDN

個人不需要套CDN，當前地區封鎖不嚴重

#### 配置模板

我的部分配置可以參考

##### 服務端

```json
{
  "inbound": {
    "streamSettings": {
      "network": "ws", 
      "wsSettings": { 
        "headers": {
          "host": "你的域名"
        },
      "path": "/ws/"
      }
    }, 
    "protocol": "vmess", 
    "port": 你的端口, 
    "settings": {
      "clients": [
        {
          "alterId": 64, 
          "security": "auto", 
          "id": "你的uuid", 
          "level": 1
        }
      ]
    }
  }, 
  "outboundDetour": [
    {
      "tag": "blocked", 
      "protocol": "blackhole", 
      "settings": {}
    }
  ], 
  "outbound": {
    "protocol": "freedom", 
    "settings": {}
  }, 
  "log": {
    "access": "/var/log/v2ray/access.log", 
    "loglevel": "info", 
    "error": "/var/log/v2ray/error.log"
  }, 
  "routing": {
    "settings": {
      "rules": [
        {
          "ip": [
            "0.0.0.0/8", 
            "10.0.0.0/8", 
            "100.64.0.0/10", 
            "127.0.0.0/8", 
            "169.254.0.0/16", 
            "172.16.0.0/12", 
            "192.0.0.0/24", 
            "192.0.2.0/24", 
            "192.168.0.0/16", 
            "198.18.0.0/15", 
            "198.51.100.0/24", 
            "203.0.113.0/24", 
            "::1/128", 
            "fc00::/7", 
            "fe80::/10"
          ], 
          "type": "field", 
          "outboundTag": "blocked"
        }
      ]
    }, 
    "strategy": "rules"
  }
}
```

##### 客戶端

```json
{
  "policy" : {
    "levels" : {
      "0" : {
        "uplinkOnly" : 0
      }
    }
  },
  "dns" : {
    "servers" : [
      "localhost",
      "8.8.8.8"
    ]
  },
  "outboundDetour" : [
    {
      "protocol" : "freedom",
      "tag" : "direct",
      "settings" : {

      }
    }
  ],
  "inbound" : {
    "listen" : "0.0.0.0",
    "port" : 1081,
    "protocol" : "socks",
    "settings" : {
      "auth" : "noauth",
      "udp" : false,
      "ip" : "127.0.0.1"
    }
  },
  "inboundDetour" : [
    {
      "listen" : "0.0.0.0",
      "allocate" : {
        "strategy" : "always",
        "refresh" : 5,
        "concurrency" : 3
      },
      "port" : 8001,
      "protocol" : "http",
      "tag" : "httpDetour",
      "domainOverride" : [
        "http",
        "tls"
      ],
      "streamSettings" : {

      },
      "settings" : {
        "timeout" : 0
      }
    }
  ],
  "log" : {
    "error" : "\/var\/folders\/jt\/dc88j3317917cz1n8mxqj8jc0000gn\/T\/cenmrev.v2rayx.log.C2EAD579-EFCE-4BEC-91A2-61E4760E1219\/error.log",
    "loglevel" : "debug",
    "access" : "\/var\/folders\/jt\/dc88j3317917cz1n8mxqj8jc0000gn\/T\/cenmrev.v2rayx.log.C2EAD579-EFCE-4BEC-91A2-61E4760E1219\/access.log"
  },
  "routing" : {
    "strategy" : "rules",
    "settings" : {
      "domainStrategy" : "IPIfNonMatch",
      "rules" : [
        {
          "domain" : [
            "localhost"
          ],
          "type" : "field",
          "outboundTag" : "direct"
        },
        {
          "type" : "field",
          "outboundTag" : "direct",
          "ip" : [
            "geoip:private"
          ]
        }
      ]
    }
  },
  "outbound" : {
    "sendThrough" : "0.0.0.0",
    "mux" : {
      "enabled" : false,
      "concurrency" : 8
    },
    "protocol" : "vmess",
    "settings" : {
      "vnext" : [
        {
          "address" : "你的域名",
          "port" : 443,
          "users" : [
            {
              "id" : "你的UUID",
              "alterId" : 64,
              "security" : "auto",
              "level" : 0
            }
          ],
          "remark" : "這個"
        }
      ]
    },
    "streamSettings" : {
      "wsSettings" : {
        "path" : "\/ws\/",你的path
        "headers" : {
          "Host" : "你的域名"
        }
      },
      "tlsSettings" : {
        "serverName" : "你的域名",
        "allowInsecure" : false
      },
      "httpSettings" : {
        "path" : "",
        "host" : [
          ""
        ]
      },
      "tcpSettings" : {
        "header" : {
          "type" : "none"
        }
      },
      "kcpSettings" : {
        "header" : {
          "type" : "none"
        },
        "mtu" : 1350,
        "congestion" : false,
        "tti" : 20,
        "uplinkCapacity" : 5,
        "writeBufferSize" : 1,
        "readBufferSize" : 1,
        "downlinkCapacity" : 20
      },
      "network" : "ws",
      "security" : "tls"
    }
  }
}
```

# 參考鏈接

[V2Ray項目](https://www.v2ray.com)

[1]: https://caddyserver.com/download

