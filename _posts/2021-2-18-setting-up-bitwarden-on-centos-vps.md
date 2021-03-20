---
layout: post
title:  "在VPS(CentOS)上自建Bitwarden_rs密码管理服务"
date:   2021-02-18 22:25:00
categories: [Linux, VPS]
tags:   [Lifelong learning, Bitwarden, VPS, Linux, CentOS]
---

## 0. 背景

LastPass 个人用户手机端和电脑端同步要收费了。。。也一直种草 Bitwarden 很久了，想着反正有个服务器，不如自己整一个密码管理服务。上[Bitwarden官网](https://bitwarden.com/)一看，最小需求也比我这小破站大呀。。。

不过又看见有人用 Rust 重写了一个服务端[bitwraden_rs](https://github.com/dani-garcia/bitwarden_rs)，用的服务器资源并不多。索性自己也整一个，记录下过程。

## 1. 安装docker

由于bitwarden_rs使用Docker镜像安装，参照[Docker官方介绍](https://docs.docker.com/engine/install/centos/) 安装docker。

### 1.1 卸载旧版本

其实不需要。。。。。

```shell
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

### 1.2 安装docker

可自行选择一种方式安装，我选择设置 repository 的方式。

- 首先设置源。

```shell
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```
- 安装

```shell
sudo yum install docker-ce docker-ce-cli containerd.io
```

- 启动

```shell
sudo systemctl start docker
```

- 测试

```shell
sudo docker run hello-world
```

- 添加用户到 `docker` 组

```shell
sudo gpasswd -a king docker
```

## 2. 域名解析设置

依据自己的实际情况，设置域名解析。

如解析 https://bitwarden.robincn.com 到 server_ip。

## 3. 安装bitwarden_rs

参照 [bitwarden_rs官方介绍](https://github.com/dani-garcia/bitwarden_rs) 安装。

```shell
docker pull bitwardenrs/server:latest
```

## 4. 规划转发规则并设置nginx

根据我的小站情况，拟将所有访问 https://bitwarden.robincn.com 的全部转发至 8080 端口。docker 容器中运行 bitwarden_rs 服务并将容器内的80端口映射到主机的8080端口，容器内的3012端口映射到主机的3012端口。

### 4.1 ssl证书设置

和之前一样使用 `acme.sh` 生成并安装证书。

```shell
acme.sh --issue --dns dns_cf -d bitwarden.robincn.com --nginx
acme.sh --install-cert --nginx -d bitwarden.robincn.com --key-file /etc/nginx/bitwardenrobincn/key.pem --fullchain-file /etc/nginx/bitwardenrobincn/cert.pem --reloadcmd "service nginx force-reload"
```

### 4.2 nginx设置

修改 nginx 配置文件，将 bitwarden.robincn.com 的访问全部转发到主机的8080端口。

```vim
# vim /etc/nginx/nginx.conf
# http{ # 在http段内增加如下内容server

    upstream bitwardenrs-default { server localhost:8080;}
    upstream bitwardenrs-ws { server localhost:3012;}
	
	# 所有访问都走https
    server {	
        listen 80;
        listen [::]:80;
        server_name bitwarden.robincn.com;
        return 301 https://$host$request_uri;
    }

	# 转发对应端口到主机8080和3012
    server {
        listen 443 ssl http2;
        listen [::]:443 ssl http2;
        server_name bitwarden.robincn.com;

        ssl_certificate "/etc/nginx/passrobincn/cert.pem";
        ssl_certificate_key "/etc/nginx/passrobincn/key.pem";
        ssl_session_cache shared:SSL:1m;
        ssl_session_timeout  10m;
        ssl_ciphers PROFILE=SYSTEM;
        ssl_prefer_server_ciphers on;
        ssl_protocols TLSv1.1 TLSv1.2 TLSv1.3;

        proxy_intercept_errors on;  # for error page render
        large_client_header_buffers 2 2k;   # 解决 Android 客户端同步问题

        # 解决网页加载问题
        gzip off;
        proxy_max_temp_file_size 0;

        client_max_body_size 128M;

        location / {
                proxy_set_header Host  $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-Proto $scheme;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

                proxy_pass http://bitwardenrs-default;
        }
        location /notifications/hub/negotiate {
                proxy_set_header Host  $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-Proto $scheme;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

                proxy_pass http://bitwardenrs-default;
        }
        location /notifications/hub {
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header Connection $http_connection;

                proxy_pass http://bitwardenrs-ws;
        }

    }


```

## 5. 启动 Bitwarden 并测试

### 5.1 启动与测试

用命令行启动 Bitwarden 并通过网页添加用户进行基础测试。

**注意**本人将 Bitwarden 相关数据放在用户目录下。

```shell
docker run -d --name bitwarden \
-v /home/king/bitwarden/data/:/data/ \
-p 8080:80 \
-p 3012:3012 \
-e LOG_FILE=/data/bitwarden.log \
-e WEBSOCKET_ENABLED=true \
-e WEB_VAULT_ENABLED=true \
-e SIGNUPS_ALLOWED=true \
-e DOMAIN=https://bitwarden.robincn.com \
bitwardenrs/server:latest
# 返回docker的ID
```

字段说明如下
 - `WEB_VAULT_ENABLED=true`  开启网页端
 - `SIGNUPS_ALLOWED=true` 允许用户注册
 - 以上两字段首次使用注册用户需要，之后可改为 `false`

如遇到问题，可使用以下命令查看相关日志。

```shell
docker ps
# 成功可看到STATUS中的 healthy ，确认PORTS映射关系无误
docker logs *ID*  # 查看docker启动日志
docker stop *ID*  # 停止docker实例
docker rm *ID*  # 删除docker实例
```

### 5.2 新建用户与导入数据

启动成功后用浏览器访问 https://bitwarden.robincn.com 即可新建用户使用了。

参照[官方说明](https://bitwarden.com/help/article/import-data/)导入数据。

## 6. 安装Docker Compose并启动服务

###  6.1 安装

参照[官方介绍](https://docs.docker.com/compose/install/)安装 Docker Compose.

```shell
sudo curl -L "https://github.com/docker/compose/releases/download/1.28.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose

docker-compose --version
```

### 6.2 设置

在 `data` 目录下新建 Docker Compress 服务描述文件。

```vim
# vim /home/king/bitwarden/docker-compose.yml
version: "3"

services:
        bitwarden:
                image: bitwardenrs/server
                container_name: bitwardenrs
                restart: always
                ports:
                        - "8080:80"
                        - "3012:3012"
                volumes:
                        - ./bw-data:/data
                environment:
                        LOG_FILE: "/data/bitwarden.log"
                        WEBSOCKET_ENABLED: "true"
                        SIGNUPS_ALLOWED: "false"  # 关闭用户注册 
                        WEB_VAULT_ENABLED: "false"  # 关闭浏览器访问
                        DOMAIN: "https://bitwarden.robincn.com"
```

### 6.3 启动服务

**注意：首先关闭之前启动的Bitwarden**

```shell
docker stop *ID*
docker rm *ID*
```

使用 Docker Compose 启动服务。

```shell
cd /home/king/bitwarden/
docker-compose up -d  # 启动服务
docker-compose down		# 关闭服务
docker-compose restart  # 重启服务
```

## 7. 安装客户端与其他设置

### 7.1 安装客户端

参照[官网下载地址](https://bitwarden.com/download/)下载安装各平台的客户端和/或插件。

### 7.2 其他设置

参照[bitwarden_rs/wiki](https://github.com/dani-garcia/bitwarden_rs/wiki) 进行其他设置(如fail2ban设置等)。

