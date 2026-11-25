# 5.8、Centos7单机搭建minio

## 一、下载安装文件

### 1.1、在home目录下创建minio文件夹

```shell
mkdir /home/minio
```

### 1.2、进入/home/minio 文件夹

```shell
cd /home/minio
```

### 1.3、下载文件

```shell
wget https://doc-private-software.oss-cn-hangzhou.aliyuncs.com/middle/minio
```

[请至钉钉文档查看附件《minio》。](https://alidocs.dingtalk.com/i/nodes/R1zknDm0WR6XzZ4LtRdzAgDgWBQEx5rG?doc_type=wiki_doc&iframeQuery=anchorId%3DX02lvjc12e4mtdf5ohvv0f)

## 二、创建数据文件夹

```shell
mkdir /home/minio/data
```

## 三、启动

### 3.1、赋予权限

```shell
chmod 777 minio
或 
chmod +x minio
```

### 3.2、将Minio设置成服务 配置Systemd服务启动

#### 3.2.1、创建minIO 配置文件

```shell
vim /etc/default/minio

# 对应minio的安装目录数据文件夹
MINIO_VOLUMES="/home/minio/data" 
# 注意这里的ip要改成你们自己的ip地址, 这里的ip特别注意，需要内网ip,否则服务起不来。固定端口号设置，避免重启后 端口号随机
MINIO_OPTS="--address 172.16.0.230:9001 --console-address 172.16.0.230:9010"

# 设置用户名和密码
## 启动的时候看提示 新版本
MINIO_ROOT_USER=admin
MINIO_ROOT_PASSWORD=Xgp@135396
```

#### 3.2.2、创建service

```shell
vim /etc/systemd/system/minio.service

[Unit]
Description=Minio
Documentation=https://docs.minio.io
Wants=network-online.target
After=network-online.target
# minio文件具体位置
AssertFileIsExecutable=/home/minio/minio

[Service]
# 创建数据文件夹
WorkingDirectory=/home/minio/data
# User and group 用户/组
User=root
Group=root
PermissionsStartOnly=true

# 创建的配置文件 minio.conf
EnvironmentFile=/etc/default/minio

ExecStartPre=/bin/bash -c "[ -n \"${MINIO_VOLUMES}\" ] || echo \"Variable MINIO_VOLUMES not set in /etc/default/minio\""
# $MINIO_OPTS $MINIO_VOLUMES  //这其实就是minio服务启动命令 /root/minio是服务位置 后面是端口号和数据存放目录
ExecStart=/home/minio/minio server $MINIO_OPTS $MINIO_VOLUMES

StandardOutput=journal
StandardError=inherit

# Specifies the maximum file descriptor number that can be opened by this process*
LimitNOFILE=65536

# Disable timeout logic and wait until process is stopped*
TimeoutStopSec=0

# SIGTERM signal is used to stop Minio*
KillSignal=SIGTERM

SendSIGKILL=no
SuccessExitStatus=0

[Install]
WantedBy=multi-user.target
```

#### 3.2.3、重新加载 systemd

```shell
systemctl daemon-reload
```

#### 3.2.4、开机自动启动

```shell
systemctl enable minio.service
```

#### 3.2.5、启动服务

```shell
systemctl start minio.service
```

#### 3.2.6、查看服务

```shell
systemctl status minio.service

API: http://172.16.0.230:9001
WebUI: http://172.16.0.230:9010
```

## 四、访问(可忽略)

### 4.1、安装nginx(忽略)

### 4.2、通过域名访问

```shell
vim minio-ui.conf

server {
    listen 443 ssl;
    server_name minio-ui.teambition.net;
    ssl_certificate      /etc/nginx/server.pem;
    ssl_certificate_key  /etc/nginx/server.key;
    location / {
        #反向代理到的服务
        proxy_pass http://172.16.0.230:9010/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
    location /ws/ {
        #反向代理到的服务
        proxy_pass http://172.16.0.230:9010;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
server {
    listen 80;
    server_name minio-ui.teambition.net;
    # HTTP重定向到HTTPS
    rewrite ^(.*)$ https://$host$1 permanent;
}
```

### 4.3、api设置域名

```shell
vim minio-svc.conf

server {
    listen 443 ssl;
    server_name minio-svc.teambition.net;
    ssl_certificate      /etc/nginx/server.pem;
    ssl_certificate_key  /etc/nginx/server.key;
    location / {
        #反向代理到的服务
        proxy_pass http://172.16.0.230:9001/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

注意：这个api设置的域名即是配置的 endpoint

## 五、开放下载，设置永久访问链接(忽略)

很多情况下，我们的图片是需要浏览器直接访问的，这里通过[minio](https://so.csdn.net/so/search?q=minio&spm=1001.2101.3001.7020)的客户端进行设置。

### 5.1、下载客户端：

```shell
wget https://dl.minio.io/client/mc/release/linux-amd64/mc
```

[请至钉钉文档查看附件《mc》。](https://alidocs.dingtalk.com/i/nodes/R1zknDm0WR6XzZ4LtRdzAgDgWBQEx5rG?doc_type=wiki_doc&iframeQuery=anchorId%3DX02lvjc1ptm7vkr6t6wzio)

### 5.2、赋予权限

```shell
chmod 777 mc
```

### 5.3、添加server（注意自己的端口和账户名密码）

```shell
# ./mc config host add minio http://ip:端口/ 账户 密码
./mc config host add minio http://172.16.0.230:9001/ admin Xgp@135396


mc: Configuration written to `/root/.mc/config.json`. Please update your access credentials.
mc: Successfully created `/root/.mc/share`.
mc: Initialized share uploads `/root/.mc/share/uploads.json` file.
mc: Initialized share downloads `/root/.mc/share/downloads.json` file.
Added `minio` successfully.
```

### 5.4、设置需要开放下载的[**bucket**](https://so.csdn.net/so/search?q=bucket&spm=1001.2101.3001.7020), 注意需要带minio

```shell
# ./mc  anonymous  set  download  minio/桶名称
./mc  anonymous  set  download  minio/docs-body

Access permission for `minio/docs-body` is set to `public`
```

![image.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/3BMqYawQKg08qwZL/img/b9fcc5a0-9271-4c2b-b9da-fe49adb940e8.png)

## 六、结束