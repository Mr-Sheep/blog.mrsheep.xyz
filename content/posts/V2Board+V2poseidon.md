---
title: "V2Board+V2poseidon"
date: 2020-04-05T22:48:01+08:00
draft: false
---

本文目的在於教你如何使用免費[^1]的前(後)端成爲一名~~奸商~~ 機場主

<!--more-->

# 開始之前

- 採用[V2Board](https://github.com/v2board/v2board/)作爲前端，[V2-poseidon](https://github.com/ColetteContreras/v2ray-poseidon)作爲伺服器端
- 部分節點多個v2ray共存
- 其實配置很簡單

### 所需工具：

Debian 9 + SSH + 腦子 + 電腦

# V2Board

## 最低要求

官方推薦使用aapanel部署，環境要求如下：

| 环境  | 版本   |
| :---- | :----- |
| php   | >=7.3  |
| nginx | >=1.17 |
| mysql | 5.x    |
| redis | ☑️      |

最低配置512m1c即可,其實完全可以不用aaPanel也可以順利的完成部署

## aaPanel

這個部分大多來自官方文檔：https://docs.v2board.com/deploy/aapanel.html

1. 安裝參考另一篇文章：[aaPanel](https://blog.mrsheep.xyz/posts/aapanel/)

2. 安裝完成後請安裝

   ☑️ Nginx 1.17
   ☑️ MySQL 5.6
   ☑️ PHP 7.3  

   之後請繼續安裝

   ☑️ Redis
   ☑️ PM2
   並在aaPanel 面板 > App Store > 找到PHP 7.3点击Setting > Disabled functions 将 `putenv` `proc_open` `pcntl_alarm` `pcntl_signal` 从列表中删除

3. 創建新的站點

   aaPanel 面板 > Website > Add site。

   > 在 Domain 填入你指向服务器的域名
   > 在 Database 选择MySQL
   > 在 PHP Verison 选择PHP-73  

4. 僞靜態

   ```nginx
   location /downloads {
   }
   
   location / {  
       try_files $uri $uri/ /index.php$is_args$query_string;  
   }
   
   location ~ .*\.(js|css)?$
   {
       expires      1h;
       error_log off;
       access_log /dev/null; 
   }
   ```

   

## 安裝V2Board

```bash
# cd到網站根目錄
cd /www/wwwroot/example.com

# 删除目录下文件
chattr -i .user.ini
rm -rf .htaccess 404.html index.html .user.ini

# 执行命令从 Github 克隆到当前目录。
git clone https://github.com/v2board/v2board.git ./

# 执行命令下载 composer.phar 到当前目录。
wget https://getcomposer.org/download/1.9.0/composer.phar

# 执行命令进行包安装。
php composer.phar install
```

安裝過程報錯請分配swap，詳見文章：[Swap](https://blog.mrsheep.xyz/posts/linux-swap/)

最後

```php
php artisan v2board:install
```

會要求輸入數據庫地址，數據庫名，數據庫賬戶名，數據庫密碼，管理帳號，管理密碼

之後會提示安裝成功即可

## 定時任務 + 隊列服務

### 定時任務

aaPanel 面板 > Cron。

> 在 Type of Task 选择 Shell Script
> 在 Name of Task 填写 v2board
> 在 Period 选择 N Minutes 1 Minute
> 在 Script content 填写 php /www/wwwroot/domain.com/artisan schedule:run  

根据上述信息添加每1分钟执行一次的定时任务。

或者crontab -e加入

```cron
1 * * * * php /www/wwwroot/domain.com/artisan schedule:run >/dev/null 2>&1
```

### 隊列服務

V2board的邮件系统强依赖队列服务，你想要使用邮件验证及群发邮件必须启动队列服务。下面以aaPanel中nodejs的PM2服务来守护队列服务作为演示。

aaPanel 面板 > App Store > Deployment

找到PM2 Manager 4.2.2进行安装，安装完成后按照如下填写

> 在 Project root directory 选择站点目录
> 在 Startup file name 填写 pm2.yaml
> 在 project name 填写 v2board  

或者自行安裝pm2:

```
wget -qO- https://getpm2.com/install.sh | bash
```

然後在網站目錄下

```
pm2 start pm2.yaml --name V2Board
```

## .env

在.env中設定郵件服務

至此部署完成



# V2-Poseidon

## 使用Caddy/Nginx接管SSL

簡單的說就是：父ws，子ws+tls就行。然后caddy弄tls

### 新建父節點

- 關閉tls

- 連接端口=服務端口

- websocket配置

  ```json
      {
        "path": "/example",
        "headers": {
          "Host": "example.com"
        }
      }
  ```

### 新建子節點

- 開啓tls

- 連接端口443

- 服務端口同上

- WebSocket配置

  ```json
   {
        "path": "/example",
        "headers": {
          "Host": "example.com"
        }
      }
  
  ```

- 父節點：上面配置的父節點



## 直接運行

### 安裝

```bash
curl -L -s https://raw.githubusercontent.com/ColetteContreras/v2ray-poseidon/master/install-release.sh | sudo bash
```

### 配置文件

和v2ray一樣，位於`/etc/v2ray/config.json`

#### TCP

```
{
  "poseidon": {
    "panel": "v2board",         
    "nodeId": 1,                // 你的节点 ID 和 v2board 里的一致
    "checkRate": 60,            // 每隔多长时间同步一次配置文件、用户、上报服务器信息
    "webapi": "http or https://YOUR V2BOARD DOMAIN",// v2board 的域名信息
    "token": "v2board token",   // v2board 和 v2ray-poseidon 的通信密钥

    "speedLimit": 0, // 节点限速 单位 字节/s 0 表示不限速
    "user": {
      "maxOnlineIPCount": 0,  // 用户同时在线 IP 数限制 0 表示不限制
      "speedLimit": 0         // 用户限速 单位 字节/s 0 表示不限速
    },

    "localPort": 10084 // 本地 api, dokodemo-door,　监听在哪个端口，不能和服务端口相同
  }
}
```



#### WS + TLS

```
{
  "poseidon": {
    "panel": "v2board",         // 这一行必须存在，且不能更改
    "nodeId": 1,                // 你的节点 ID 和 v2board 里的一致
    "checkRate": 60,            // 每隔多长时间同步一次配置文件、用户、上报服务器信息
    "webapi": "http or https://YOUR V2BOARD DOMAIN",// v2board 的域名信息
    "token": "v2board token",   // v2board 和 v2ray-poseidon 的通信密钥

    "speedLimit": 0, // 节点限速 单位 字节/s 0 表示不限速
    "user": {
      "maxOnlineIPCount": 0,  // 用户同时在线 IP 数限制 0 表示不限制
      "speedLimit": 0         // 用户限速 单位 字节/s 0 表示不限速
    },

    "localPort": 10084 // 本地 api, dokodemo-door,　监听在哪个端口，不能和服务端口相同
  }
}
```

##### 憑證

憑證需要保存在/home/下，命名爲 v2ray.crt v2ray.key

如果用caddy/nginx直接接管ssl部分可以忽略這點

### 啓動

```
service v2ray restart
```

## Docker

```bash
curl -fsSL https://get.docker.com | bash
curl -L "https://github.com/docker/compose/releases/download/1.25.3/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod a+x /usr/local/bin/docker-compose
systemctl start docker
```

可選擇创建个软链接，以后用 dc 命令来代替 docker-compose

```
rm -f `which dc`
# 若系统中存在 dc 则删除，这个 dc 就是个计算器，完全没有用
ln -s /usr/local/bin/docker-compose /usr/bin/dc
```

### Clone docker 和配置文件

默認你有安裝git

```bash
git clone https://github.com/ColetteContreras/v2ray-poseidon.git
cd v2ray-poseidon/docker/v2board
```

#### TCP 模式

```visual basic
cd tcp
# 1. 修改 config.json 中的 poseidon 下面的配置
# 2. 修改 docker-compose.yml 的服务端口
# 启动
dc up -d 
```

#### CloudFlare CDN + WebSocket CDN

1. v2board 中节点的连接端口和服务端口都得为 443 
2. CloudFlare 打开代理
3. 修改配置文件并启动 docker

```
cd ws
# 修改 config.json 中的 poseidon 下面的 nodeId, webapi, 和 token
# 启动
dc up -d
```

#### WebSocket-TLS 無CDN，Caddy不接管

```
cd ws-tls
# 1. 修改 config.json 中的 poseidon 下面的配置
# 2. 修改 docker-compose.yml 中的配置，根据你的 DNS 信息
# 启动
dc up -d使

```

#### WebSocket-TLS 無CDN,用Caddy/Nginx接管SSL

```
services:
      v2ray-ws-tls:
        image: v2cc/poseidon
        container_name: v2ray-ws-tls
        restart: always
        ports:
          # 这里服务端口与上文的服务端口一致
          - "服务端口:服务端口"
        volumes:
          - ./config.json:/etc/v2ray/config.json
        environment:
          - USE_CADDY_TLS=false
```

其實與直接WS類似

## 關於限速

以500b/s爲例

| 單位              | 數值   |
| ----------------- | ------ |
| 位每秒 (bit/s)    | 4000   |
| 千位每秒 (kbit/s) | 4      |
| 兆位每秒 (Mbit/s) | 4×10-3 |
| 吉位每秒 (Gbit/s) | 4×10-6 |
| 字節每秒 (B/s)    | 500    |
| 千字節每秒 (kB/s) | 0.5    |
| 兆字節每秒 (MB/s) | 5x10-4 |
| 吉字節每秒 (GB/s) | 5x10-7 |

# 備註

[^1]: 後端專業版需要付費