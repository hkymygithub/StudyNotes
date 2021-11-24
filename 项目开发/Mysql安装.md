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

##### 192.168.0.1服务器

```

```

##### 192.168.0.2服务器

```

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
      MYSQL_ROOT_PASSWORD: "******"
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



```
version: '3'
services:
  mysql-slave:
    image: mysql
    container_name: mysql-slave
    restart: always
    network_mode: host
    environment:
      MYSQL_ROOT_PASSWORD: "******"
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

```
CREATE USER 'sync_root'@'%' IDENTIFIED WITH mysql_native_password BY 'sync_pwd';
GRANT REPLICATION SLAVE ON *.* TO 'sync_root'@'%';
FLUSH PRIVILEGES;
#查看master的binlog信息
SHOW MASTER STATUS;
```

```
#启动容器,等待mysql正常启动
docker run --name slave -e MYSQL_ROOT_PASSWORD="123456" -d mysql:8.0.13 --server-id=2 --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
#进入容器，连接master，然后开启同步
docker exec -ti slave bash
mysql -uroot -p123456
# 这里master_host就是刚刚看到的master的ip，master_user就是我们创建用于同步的账号，master_log_file和master_log_pos就是通过show master status获得到的
CHANGE MASTER TO MASTER_HOST='1.15.181.135',MASTER_USER='sync_root',MASTER_PASSWORD='sync_pwd',MASTER_LOG_FILE='mysql-bin.000003',MASTER_LOG_POS=866;
#开启同步
START SLAVE;
#查看slave同步状态
SHOW SLAVE STATUS\G
```

```
ALTER mysql.user 'sync_root'@'%' IDENTIFIED WITH mysql_native_password BY 'root';
```

