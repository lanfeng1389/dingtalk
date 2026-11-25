# 九、Slb是Nginx的情况下域名配置如下

## 一、背景

有些客户在部署POC的环境不提供Slb或者Slb是Nginx的情况下，有时候域名的配置总是出现问题，这里提供一个模版，供交付同学提供给客户，让他们进行参考

## 二、解决方案

### 2.1、主域名配置如下

```shell
server {
    listen 443 ssl;
    server_name docs.xxx.com;
    ssl_certificate      /etc/nginx/server.pem;
    ssl_certificate_key  /etc/nginx/server.key;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
    ssl_ciphers HIGH:!aNULL:!MD5:!3DES;
    location / {
        #反向代理到的服务
        proxy_pass http://ip:32080/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
server {
    listen 80;
    server_name docs.xxx.com;
    # HTTP重定向到HTTPS
    rewrite ^(.*)$ https://$host$1 permanent;
}
```

### 2.2、websocket域名配置如下

```shell
server {
    listen 443 ssl;
    server_name ws-docs.xxx.com;
    ssl_certificate      /etc/nginx/server.pem;
    ssl_certificate_key  /etc/nginx/server.key;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
    ssl_ciphers HIGH:!aNULL:!MD5:!3DES;
    location / {
        #反向代理到的服务
        proxy_pass http://ip:32080/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_read_timeout 300;
        proxy_send_timeout 300;
        keepalive_timeout 600;
        keepalive_requests 10000;
    }
}
server {
    listen 80;
    server_name ws-docs.xxx.com;
    # HTTP重定向到HTTPS
    rewrite ^(.*)$ https://$host$1 permanent;
}
```

### 2.3、vault代理域名配置如下（80和443共存，阿里云oss可不用）

```shell
server {
    listen 443 ssl;
    server_name docs-body.xxx.com;
    ssl_certificate      /etc/nginx/server.pem;
    ssl_certificate_key  /etc/nginx/server.key;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
    ssl_ciphers HIGH:!aNULL:!MD5:!3DES;
    location / {
        #反向代理到的服务
        proxy_pass http://ip:32080/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}

server {
    listen 80;
    server_name docs-body.xxx.com;
    location / {
        #反向代理到的服务
        proxy_pass http://ip:32080/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```