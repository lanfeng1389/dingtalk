# 1、Centos7集群搭建minio步骤

## 一、参考文章

```shell
1、搭建文章
https://blog.csdn.net/AnJiaqi/article/details/146973331
https://blog.csdn.net/m0_71578648/article/details/132836904
2、官方文档
https://www.minio.org.cn/docs/minio/linux/operations/concepts/erasure-coding.html#minio-ec-parity
```

## 二、先行条件

### 2.1、关闭防火墙

```shell
所有节点
#关闭防火墙和内核防火墙
systemctl stop firewalld && systemctl disable firewalld
```

### 2.2、禁用SELinux安全

```shell
setenforce 0
sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config
```

### 2.3、同步时间

```shell
#同步时间
yum -y install ntpdate && ntpdate ntp.aliyun.com
```

### 2.4、服务器配置(最低配置)

| 服务器IP | cpu | 内存 | 磁盘最低要求 |
| --- | --- | --- | --- |
| 172.16.1.98 | 4C | 8G | 50G系统盘+200G数据盘 |
| 172.16.1.99 | 4C | 8G | 50G系统盘+200G数据盘 |
| 172.16.1.100 | 4C | 8G | 50G系统盘+200G数据盘 |
| 172.16.1.101 | 4C | 8G | 50G系统盘+200G数据盘 |
|  |  |  |  |
| 总物理存储容量 | 4 台服务器，每台 1 个数据盘，每个 200GB： 4 \* \* 200GB = 800GB。 |  |  |
| 纠删码分配 | 在 8 个磁盘中进行 2+2 配置，意味着 2 个磁盘用于存储数据，其余 2 个磁盘用于校验和冗余。 |  |  |
| 有效存储容量： | 存储效率为 2/4（约 50%），因此有效存储容量为：800GB \* (2/4) = 400GB。 |  |  |

## 三、部署步骤

### 3.1、在所有服务器下载文件

```shell
# 下载 MinIO 二进制文件
wget https://doc-private-software.oss-cn-hangzhou.aliyuncs.com/middle/minio

wget https://dl.min.io/server/minio/release/linux-amd64/minio
 
# 赋予执行权限
chmod +x minio
 
# 移动到 /usr/local/bin 目录，方便全局使用
sudo mv minio /usr/local/bin/
```

### 3.2、在所有服务器创建存储目录

*   创建存储目录：
    

```shell
mkdir -p /mnt/minio/data
```

*   服务器有单独的磁盘存储，比如 `/dev/vdb`，可以将其挂载
    
    ```shell
    1、minio节点格式化磁盘，所有驱动器需使用 XFS 文件系统，避免性能问题。
    mkfs.xfs /dev/vdb
    2、挂载
    mount /dev/vdb /mnt/minio/data
    ```
    
*   并在 `/etc/fstab` 添加自动挂载
    

```shell
/dev/vdb /mnt/minio/data xfs defaults 0 0
```

### 3.3、在所有服务器配置 MinIO

配置 minio文件 `/etc/default/minio`

```shell
# MinIO 监听的 IP 和端口
MINIO_VOLUMES="http://172.16.1.98:9000/mnt/minio/data http://172.16.1.99:9000/mnt/minio/data http://172.16.1.100:9000/mnt/minio/data http://172.16.1.101:9000/mnt/minio/data"
MINIO_OPTS="--console-address :9001"
MINIO_ROOT_USER="admin"
MINIO_ROOT_PASSWORD="DingDing@20250516"
# 以下2个配置仅仅适用4台服务器，每台1个数据盘，其他配置 这两个可不用写上。
MINIO_STORAGE_CLASS_STANDARD="EC:2"
MINIO_STORAGE_CLASS_RRS="EC:1"
```

### 3.4、在所有服务器配置 Systemd 服务

创建 Systemd 服务文件 `/etc/systemd/system/minio.service`：

```shell
[Unit]
Description=MinIO Object Storage
After=network.target
 
[Service]
User=root
Group=root
EnvironmentFile=/etc/default/minio
ExecStart=/usr/local/bin/minio server $MINIO_VOLUMES $MINIO_OPTS
Restart=always
LimitNOFILE=65536
 
[Install]
WantedBy=multi-user.target
```

### 3.5、在每台服务器上启动 MinIO

```shell
sudo systemctl daemon-reload
sudo systemctl enable minio
sudo systemctl start minio

检查运行状态：
sudo systemctl status minio
```

### 3.6、访问 MinIO

```shell
你可以通过 Web 控制台 访问：
http://172.16.1.98:9001

登录用户名：admin
密码：DingDing@20250516。
```

### 3.7、使用 MinIO Client（mc） 管理

*   下载 MinIO Client：
    

```shell
wget https://doc-private-software.oss-cn-hangzhou.aliyuncs.com/middle/mc
wget https://dl.min.io/client/mc/release/linux-amd64/mc
chmod +x mc
sudo mv mc /usr/local/bin/
```

*   添加 MinIO 服务器：
    

```shell
mc config host add myminio http://172.16.1.99:9000 admin DingDing@20250516
```

*   删除某个bucket下的所有数据：
    

```shell
mc rm --recursive --force myminio/docs-body/
```

*   查看纠删码
    

```shell
1、查看纠删码的标准
mc admin config get myminio storage_class

storage_class standard= rrs=EC:1 optimize=availability 
只允许1台服务器磁盘挂了，如果数据盘挂了2台，那么则不工作。

3、修改
mc admin config set myminio storage_class rrs=EC:2
```

![image.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/NpQlK5jZb7g9eqDv/img/b10bd56c-c10a-4778-b99f-b7b103dd8072.png)

*   一些基础命令
    

```shell
1、查看bucket的数据
mc ls myminio/docs-body/

2、迁移到OSS数据

```

## 四、一些概念

### 4.1、[**纠删码奇偶校验和存储效率**](https://www.minio.org.cn/docs/minio/linux/operations/concepts/erasure-coding.html#id12)

必须是4个总磁盘才能启用纠删码

![image.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/NpQlK5jZb7g9eqDv/img/af5d59e4-b07d-4b15-8598-d6a879c6515b.png)

![image.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/NpQlK5jZb7g9eqDv/img/9e6d0d25-5cbc-45c6-bbf4-f3c3444e0a1a.png)

### 4.2、例如：那么如果我采用4台机器，每台服务器的只有一块数据盘是500G，奇偶检验EC会采用EC：2，那么最大我能存多少500G的数据(2块数据盘+2块奇偶校验盘)

### 4.3、服务器的要求

![image.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/NpQlK5jZb7g9eqDv/img/8d4faa34-ff8b-46e9-aca5-7a36e40a4cac.png)