# 5.7.1、离线安装mysql5.7

## 一、部署前准备工作

### 1.1、查看并卸载系统自带的 Mariadb

```shell

[root@localhost /]# rpm -qa|grep mariadb
mariadb-libs-5.5.68-1.el7.x86_64
[root@localhost /]# rpm -e --nodeps mariadb-libs-5.5.68-1.el7.x86_64
[root@localhost /]# rpm -qa|grep mariadb
```

### 1.2、检查系统是否安装过 MySQL

```shell
rpm -qa | grep mysql
whereis mysql
find / -name mysql

找到的目录全部删掉
```

### 1.3、检查 mysql 用户组是否存在并创建

```shell
1、检查
cat /etc/group | grep mysql
cat /etc/passwd | grep mysql

2、创建
groupadd mysql
useradd -r -g mysql mysql
```

## 二、部署mysql5.7

### 2.1、安装包

```shell
https://dev.mysql.com/downloads/mysql/5.7.html
```

![image.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/r4mlQgYmwXmgqxow/img/9d7e2089-6664-4827-99d7-d3aee86209fa.png)

[请至钉钉文档查看附件《mysql-5.7.41-linux-glibc2.12-x86\_64.tar.gz》。](https://alidocs.dingtalk.com/i/nodes/Gl6Pm2Db8DMGQKRatgyL4yebWxLq0Ee4?doc_type=wiki_doc&iframeQuery=anchorId%3DX02lxyfgfoaq5yp0155trt)

### 2.2、解压下载的tar文件

```shell
1、下载
cd /usr/local/src 
wget https://downloads.mysql.com/archives/get/p/23/file/mysql-5.7.41-linux-glibc2.12-x86_64.tar
2、解压
tar -xf mysql-5.7.41-linux-glibc2.12-x86_64.tar
tar -xf mysql-5.7.41-linux-glibc2.12-x86_64.tar.gz
3、软链
mv mysql-5.7.41-linux-glibc2.12-x86_64 ../
ln -s /usr/local/mysql-5.7.41-linux-glibc2.12-x86_64 /usr/local/mysql
```

### 2.3、更改文件夹权限

为了避免权限问题，更改 mysql5.7 目录下所有文件夹所属的用户组、用户以及权限，一定要做

```shell
chown -R mysql:mysql /usr/local/mysql-5.7.41-linux-glibc2.12-x86_64
chmod -R 755 /usr/local/mysql-5.7.41-linux-glibc2.12-x86_64

chown -R mysql:mysql /usr/local/mysql
chmod -R 755 /usr/local/mysql
```

### 2.4、初始化mysql和获取密码

```shell
cd /usr/local/mysql/bin
./mysqld --initialize --user=mysql --datadir=/usr/local/mysql/data --basedir=/usr/local/mysql
```
```shell
2023-02-23T22:43:52.131986Z 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
...
...
2023-02-23T22:43:52.554378Z 1 [Note] A temporary password is generated for root@localhost: Gy>gfIiV8e_U # 临时密码一定要注意保存
```

### 2.5、修改 my.cnf 配置文件

```shell
1、编辑文件
vi /etc/my.cnf

[mysqld]
datadir=/usr/local/mysql/data
port=3306
max_connections=6000
innodb_file_per_table=1
lower_case_table_names=1
innodb_large_prefix=ON
character_set_server=utf8mb4
symbolic-links=0

2、增加权限
chmod -R 755 /etc/my.cnf
```

## 三、配置服务

### 3.1、设置mysql服务

```shell
/usr/local/mysql/support-files/mysql.server start
```

### 3.2、添加软连接，并重启服务

```shell
ln -s /usr/local/mysql/support-files/mysql.server /etc/init.d/mysql

ln -s /usr/local/mysql/bin/mysql /usr/bin/mysql

service mysql restart
```

## 四、使用并设置mysql

### 4.1 登录mysql

```shell
mysql -uroot -p
输入：Gy>gfIiV8e_U # 临时密码
```

### 4.2、修改密码

```shell
set password for root@localhost = password('Dingdoc@999');
```

### 4.3、开启mysql的远程访问权限

```shell
use mysql
update user set user.Host='%' where user.User='root';
flush privileges;
```

## 五、设置开机自启

```shell
cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld

chmod +x /etc/init.d/mysqld

chkconfig --add mysqld

chkconfig --list
```

![image.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/r4mlQgYmwXmgqxow/img/66fd0248-c699-4111-af34-6da88e5c173e.png)