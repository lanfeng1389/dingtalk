# 5.12、linux下安装Elasticsearch-8.9.1

## 一、前沿

Elasticsearch（简称 ES）是一个基于 Apache Lucene 构建的开源分布式搜索与分析引擎。它支持实时数据处理，提供近实时的全文搜索能力，并通过 JSON 格式的 RESTful API 实现数据索引与检索。其分布式架构设计可水平扩展至数百节点，适用于日志分析、全文检索、大数据实时查询等场景，是 Elastic Stack（ELK）生态的核心组件。

本文是搭建高版本的ES是因为高版本的ES支持向量库

## 二、环境准备

```shell
操作系统：Centos7，架构：X86_64，ky10_aarch64
Java 版本：java 1.8
Elasticsearch 版本：8.9.1
```
> 注意：如果Elasticsearch为8.x的版本，需要jdk17及以上版本

## 三、优化服务器参数

### 3.1、修改limits.conf文件

```shell
vim /etc/security/limits.conf
 
# 在文件底部添加如下配置
* soft nofile 655350
* hard nofile 655350
* soft nproc 65535
* hard nproc 65535
```

### 3.2、调整vm.max\_map\_count参数

```shell
# 设置参数值至少为 262144
echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
 
# 查看参数值
sysctl vm.max_map_count
# 预期输出：vm.max_map_count = 262144
```

## 四、安装Elasticsearch

### 4.1、创建es安装目录

```shell
mkdir -p /elk/elastic
cd /elk/elastic
```

### 4.2、下载Elasticsearch安装包并解压

方式1-x86架构：通过本地oss来下载-x86架构

```shell
wget https://doc-private-software.oss-cn-hangzhou.aliyuncs.com/k8s/elasticsearch-8.9.1-linux-x86_64.tar.gz
```

方式2-arm64架构：通过本地oss来下载arm64架构

```shell
wget https://doc-private-software.oss-cn-hangzhou.aliyuncs.com/k8s/elasticsearch-8.9.1-linux-aarch64.tar.gz
```

方式2：通过官网来下载

```shell
https://www.elastic.co/downloads/past-releases/#elasticsearch
```

![image.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/r4mlQ5bKDNm6dlxo/img/f66ea7cc-9ddc-4cb3-a857-9da241b260ad.png)

```shell
https://www.elastic.co/downloads/past-releases/elasticsearch-8-9-1
选择自己需要的版本，因为自己的是X86的架构，所以选择X86_64
```

![image.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/r4mlQ5bKDNm6dlxo/img/d0128a74-4fb3-4a75-ae71-e7a1c303af5e.png)

### 4.3、解压安装包

```shell
tar -xf elasticsearch-8.9.1-linux-x86_64.tar.gz
```

### 4.4、修改jvm内存

8.X版本默认的堆内存大小是4G，如果服务器内存不够启动会报错，可以将堆内存设置小一点，如果服务器够可忽略这步骤。

```shell
vim /elk/elastic/elasticsearch-8.9.1/config/jvm.options
 
# 根据服务器的内存来设置，如果服务器内存小可以将值调小一点
-Xms4g
-Xmx4g
```

### 4.5、创建es的用户

es不允许通过root用户来启动。

```shell
// 创建用户组
groupadd esgroup
 
// 在esgroup用户组下创建es用户
useradd es -g esgroup
 
// 赋权限
chown -R es:esgroup /elk/elastic/elasticsearch-8.9.1
```

### 4.6、修改es配置文件

```shell
vim /elk/elastic/elasticsearch-8.9.1/config/elasticsearch.yml
 
# 集群名称
cluster.name: my-es-cluster
# 节点名称
node.name: node-1
# 运行远程访问
network.host: 0.0.0.0
http.port: 9200
# 单节点模式
discovery.type: single-node
 
# 跨域配置
http.cors.enabled: true
http.cors.allow-origin: "*"
http.cors.allow-methods: OPTIONS, HEAD, GET, POST, PUT, DELETE
http.cors.allow-headers: X-Requested-With, Content-Type, Content-Length
```

### 4.7、启动es

如果安装的es版本为8.x，es会默认开启安全安全配置

```shell
后台启动es
cd /elk/elastic/elasticsearch-8.9.1

sudo -u es bin/elasticsearch -d
```

![image.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/r4mlQ5bKDNm6dlxo/img/078f81e4-eb8a-4c6b-bab1-22b28c973bfd.png)

浏览器访问ip+端口(9200)结果如下表示安装成功

```shell
https://47.111.97.134:9200/
或者看一下9200端口是否存在
```

![image.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/r4mlQ5bKDNm6dlxo/img/3c331404-c668-4fe4-9dba-5719da209c9d.png)

### 4.8、Elasticsearch8.x版本设置密码

```shell
bin/elasticsearch-reset-password --username elastic -i
```

![image.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/r4mlQ5bKDNm6dlxo/img/20cbe14a-a768-459d-82d7-2acacd7c55f7.png)

![image.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/r4mlQ5bKDNm6dlxo/img/c9708baa-211f-4e6e-bba3-6e2d601fd533.png)

### 4.9、将es配置到systemd中

创建service文件

```shell
vim /etc/systemd/system/elasticsearch.service
 

[Unit]
Description=Elasticsearch Service
After=network.target

[Service]
Type=simple

# es 用户
User=es
# es 用户组（若需要）
#Group=esgroup

LimitNOFILE=100000
LimitNPROC=100000

# es 安装路径
ExecStart=/elk/elastic/elasticsearch-8.9.1/bin/elasticsearch
ExecReload=/bin/kill -HUP $MAINPID
Restart=always
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

刷新配置

```shell
systemctl daemon-reload
```

基础命令，启动前记得把上个通过后台启动es的进程给杀掉。

![image.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/r4mlQ5bKDNm6dlxo/img/908f6128-626d-4b6c-946b-33642ae52133.png)

```shell
systemctl status elasticsearch.service   # 查看状态
systemctl start elasticsearch.service    # 启动
systemctl stop elasticsearch.service     # 关闭
systemctl restart elasticsearch.service  # 重启
```

开启自动重启

```shell
systemctl enable elasticsearch.service
```

### 4.10、es关闭https ssl认证

```shell
vi /elk/elastic/elasticsearch-8.9.1/config/elasticsearch.yml


xpack.security.enabled: true  #  true表示开启安全认证，开启时访问es需要数据账号密码
 
xpack.security.enrollment.enabled: true
xpack.security.http.ssl:
  enabled: false # 是否启用https,默认启用，手动关闭,如果为true,服务es需要使用 https
  keystore.path: certs/elastic-certificates.p12
  truststore.path: certs/elastic-certificates.p12
xpack.security.transport.ssl:
  enabled: false
  verification_mode: certificate
  keystore.path: certs/elastic-certificates.p12
  truststore.path: certs/elastic-certificates.p12
```

![image.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/r4mlQ5bKDNm6dlxo/img/62c866d0-01cf-4a8f-b6ca-66454ac69af6.png)

重启一下es

```shell
systemctl restart elasticsearch.service
```

## 五、安装es的插件analysis-ik

### 5.1、官网下载地址

```shell
https://release.infinilabs.com/analysis-ik/stable/
```

![image.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/r4mlQ5bKDNm6dlxo/img/52d7ee1d-34e7-4b14-8c87-88ecda0937e3.png)

```shell
cd /tmp && wget https://release.infinilabs.com/analysis-ik/stable/elasticsearch-analysis-ik-8.9.1.zip

或者去我准备好的 本地oss上下载
cd /tmp && wget https://doc-private-software.oss-cn-hangzhou.aliyuncs.com/middle/elasticsearch-analysis-ik-8.9.1.zip
```

### 5.2、安装

```shell
/elk/elastic/elasticsearch-8.9.1/bin/elasticsearch-plugin install file:/tmp/elasticsearch-analysis-ik-8.9.1.zip

然后重启es

systemctl restart elasticsearch.service
```

![image.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/r4mlQ5bKDNm6dlxo/img/4b9c6597-92e5-454b-93b5-232adf1b6fc4.png)

可以发现在：/elk/elastic/elasticsearch-8.9.1/plugins/下多了目录analysis-ik