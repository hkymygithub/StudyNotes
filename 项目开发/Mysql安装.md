# Mysql安装

### Mysql集群部署——docker-compose

#### 部署环境：

服务器：均为Linux

| 发行版本       | IP（假设）  |
| -------------- | ----------- |
| Ubuntu 18.04.6 | 192.168.0.1 |
| Centos 8       | 192.168.0.2 |

注意事项：

每台服务器均装有docker和docker-compose

本次部署共两台服务器，mysql集群采取一主一从模式。

个服务器需要开放相应端口：

  192.168.0.1开放

  192.168.0.2开放

#### 创建相应挂载目录

##### 192.168.0.1服务器(master节点)

```
mkdir -p ./mysql-cluster/master/log
mkdir -p ./mysql-cluster/master/conf
mkdir -p ./mysql-cluster/master/data
```

##### 192.168.0.2服务器(slave节点)

```
mkdir -p ./mysql-cluster/slave/log
mkdir -p ./mysql-cluster/slave/conf
mkdir -p ./mysql-cluster/slave/data
```

#### 创建配置文件my.cnf

##### 192.168.0.1服务器(master节点)

```
vim ./mysql-cluster/master/conf/my.cnf
```

```
# Copyright (c) 2017, Oracle and/or its affiliates. All rights reserved.
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; version 2 of the License.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA  02110-1301 USA

#
# The MySQL  Server configuration file.
#
# For explanations see
# http://dev.mysql.com/doc/mysql/en/server-system-variables.html

[mysqld]
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /var/lib/mysql
secure-file-priv= NULL

# Custom config should go here
!includedir /etc/mysql/conf.d/
```

##### 192.168.0.2服务器(slave节点)

```
vim ./mysql-cluster/slave/conf/my.cnf
```

```
# Copyright (c) 2017, Oracle and/or its affiliates. All rights reserved.
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; version 2 of the License.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA  02110-1301 USA

#
# The MySQL  Server configuration file.
#
# For explanations see
# http://dev.mysql.com/doc/mysql/en/server-system-variables.html

[mysqld]
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /var/lib/mysql
secure-file-priv= NULL

# Custom config should go here
!includedir /etc/mysql/conf.d/
```

#### 创建docker-compose.yml文件

##### 192.168.0.1服务器(master节点)

```
cd mysql-cluster
vim docker-compose.yml
```

```
version: '3'
services:
  mysql-master:
    image: mysql
    container_name: mysql-master
    restart: always
    network_mode: host
    environment:
      MYSQL_ROOT_PASSWORD: "******"    #设置root账号密码
      TZ: "Asia/Shanghai"
    volumes:
      - ./master/log:/var/log/mysql  
      - ./master/conf/my.cnf:/etc/mysql/my.cnf
      - ./master/data:/var/lib/mysql
    command:
      - "--server-id=1"
      - "--character-set-server=utf8mb4"
      - "--collation-server=utf8mb4_unicode_ci"
      - "--log-bin=mysql-bin"
      - "--sync_binlog=1"
```

##### 192.168.0.2服务器(slave节点)

```
cd mysql-cluster
vim docker-compose.yml
```

```
version: '3'
services:
  mysql-slave:
    image: mysql
    container_name: mysql-slave
    restart: always
    network_mode: host
    environment:
      MYSQL_ROOT_PASSWORD: "******"    #设置root账号密码
      TZ: "Asia/Shanghai"
    volumes:
      - ./slave/log:/var/log/mysql  
      - ./slave/conf/my.cnf:/etc/mysql/my.cnf
      - ./slave/data:/var/lib/mysql 
    command:
      - "--server-id=2"
      - "--character-set-server=utf8mb4"
      - "--collation-server=utf8mb4_unicode_ci"
```

#### 启动镜像

##### 192.168.0.1服务器(master节点)

```
cd mysql-cluster
docjer-compose up -d
```

##### 192.168.0.2服务器(slave节点)

```
cd mysql-cluster
docjer-compose up -d
```

#### 进入master容器，创建主从同步账号

```
docker exec -it mysql-master /bin/bash
#用密码登录客户端，“******”为之前设置的密码
mysql -uroot -p******
#创建同步账号，并设置密码
CREATE USER 'sync_root'@'%' IDENTIFIED WITH mysql_native_password BY '******';
#设置权限
GRANT REPLICATION SLAVE ON *.* TO 'sync_root'@'%';
#刷新权限
FLUSH PRIVILEGES;
#查看master的binlog信息
SHOW MASTER STATUS;
#显示
mysql> SHOW MASTER STATUS;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000004 |      628 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```

#### 进入slave容器，实现主从同步

```
docker exec -it mysql-slave /bin/bash
#用密码登录客户端，“******”为之前设置的密码
mysql -uroot -p******
# master_host就是master的ip，master_user就是创建的用于同步的账号，master_log_file和master_log_pos就是通过show master status获得到的
CHANGE MASTER TO MASTER_HOST='192.168.0.1',MASTER_USER='sync_root',MASTER_PASSWORD='******',MASTER_LOG_FILE='mysql-bin.000004',MASTER_LOG_POS=628;
#开启同步
START SLAVE;
#查看slave同步状态
SHOW SLAVE STATUS\G
#若连接成功则完成搭建
```

#### 测试集群是否同步

进入maste结点，创建test数据库

```
mysql> create database test;
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| test               |
+--------------------+
5 rows in set (0.01 sec)
```

进入salve节点查看数据库

```
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| test               |
+--------------------+
5 rows in set (0.01 sec)
```

同步成功
