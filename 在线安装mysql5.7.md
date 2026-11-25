# 5.7、在线安装mysql5.7

## 一、前沿

一直有安装mysql的需求，这里记录一下，方便后续的使用。

## 二 .使用yum源方式安装

### 2.1、查看并卸载系统自带的 Mariadb

```shell
[root@localhost /]# rpm -qa|grep mariadb
mariadb-libs-5.5.68-1.el7.x86_64
[root@localhost /]# rpm -e --nodeps mariadb-libs-5.5.68-1.el7.x86_64
[root@localhost /]# rpm -qa|grep mariadb
```

### 2.2、下载并安装MySQL官方的 Yum

```shell
wget http://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
```

### 2.3、安装mysql的yum源

命令执行完成后会在 /etc/yum.repos.d/ 目录下生成两个repo文件

*   mysql-community.repo
    
*   mysql-community-source.repo
    

```shell
yum -y install mysql57-community-release-el7-11.noarch.rpm

[root@localhost ~]# ls /etc/yum.repos.d/
CentOS-Base.repo       CentOS-fasttrack.repo  CentOS-Vault.repo          mysql-community-source.repo
CentOS-CR.repo         CentOS-Media.repo      CentOS-x86_64-kernel.repo
CentOS-Debuginfo.repo  CentOS-Sources.repo    mysql-community.repo
```

### 2.4、使用yum方式安装mysql

```shell
yum -y install mysql-server
```

#### 2.4.1、安装过程中报错解决

*   问题描述
    

```shell
警告：/var/cache/yum/x86_64/7/mysql57-community/packages/mysql-community-common-5.7.41-1.el7.x86_64.rpm: 头V4 RSA/SHA256 Signature, 密钥 ID 3a79bd29: NOKEY
mysql-community-common-5.7.41-1.el7.x86_64.rpm 的公钥尚未安装


mysql-community-libs-compat-5.7.41-1.el7.x86_64.rpm 的公钥尚未安装

 失败的软件包是：mysql-community-libs-compat-5.7.41-1.el7.x86_64
 GPG  密钥配置为：file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
```

*   问题解决
    

```shell
rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022
```
```shell
-----BEGIN PGP PUBLIC KEY BLOCK-----

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
=Xqbo
-----END PGP PUBLIC KEY BLOCK-----
```

*   重新安装
    

```shell
yum -y install mysql-server
```

## 三、使用并设置mysql

### 3.1、启动mysql并查看状态

```shell
systemctl start mysqld.service
systemctl status mysqld.service
systemctl enable mysqld.service
```

### 3.2 获取临时密码

```shell
cat /var/log/mysqld.log | grep password

2024-04-25T02:47:45.675168Z 1 [Note] A temporary password is generated for root@localhost: Fyg#pfRIq656
```

### 3.3、登录mysql

```shell
mysql -u root -p
输入：Fyg#pfRIq656
```

### 3.4、修改登录密码

```shell
ALTER USER 'root'@'localhost' IDENTIFIED BY 'Dingdoc@999';
```

### 3.5、开启mysql的远程访问权限

```shell
grant all privileges on *.* to 'root'@'%' identified by 'Dingdoc@999';

flush privileges;
```

## 四、本地 tar 文件方式安装

## 五、参考文章

```shell
https://blog.csdn.net/weixin_54626591/article/details/133907510
```