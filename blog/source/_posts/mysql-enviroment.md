---
title: mysql环境搭建，备份数据以及授权访问
categories: mysql
tags: mysql

---
主要介绍了mysql环境搭建，以及mysqldump和grant命令的使用.
<!--more-->
### mysql环境搭建

在linux环境下搭建mysql主要步骤包括：
1. 查看是否安装过mysql：yum list installed mysql\*
2. 安装mysql客户端：yum -y install mysql
3. 安装mysql服务器端：yum -y install mysql_server
4. 安装mysql开发库：yum -y install mysql-devel
5. 修改配置文件：vim /etc/my.cnf 添加：default-character-set=utf8
6. 启动mysql数据库: service mysqld start
7. 创建root密码：mysqladmin -u root password root

### 备份数据

mysql数据库的备份主要用的命令是 mysqldump.
1. 备份某个数据库的所有数据 

```
mysqldump -h 10.10.40.8 -u root -p root database_name > backup.sql
source backup.sql
```

### 授权访问
在多个机器访问一台机器上的数据库时，这时需要针对指定用户名情况下开放mysql的远程连接：
1. 授权root用户使用root密码从任何主机连接到mysql服务器
```
grant all privileges on *.* to 'root@%' identified by 'root' with grant option;
flush privileges;
```

2. 授权root用户用root密码从192.10.10.1的主机连接到mysql数据库的ja数据库
```
grant all privileges on ja.* to 'root@192.10.10.1' identified by 'root' with grant option;
flush privileges;
```

3.任何主机访问数据的权限
```
grant all privileges on *.* to 'root@%' with grant option;
flush privileges;
```
