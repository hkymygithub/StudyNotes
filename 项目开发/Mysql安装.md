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
    environment:
      MYSQL_ROOT_PASSWORD: "123456"
      TZ: "Asia/Shanghai" #解决时区问题
    volumes:
    - 
    command:
    -  "--server-id=1"
    -  "--character-set-server=utf8mb4"
    -  "--collation-server=utf8mb4_unicode_ci"
    -  "--log-bin=mysql-bin"
    -  "--sync_binlog=1"
  mysql-slave1:
    image: mysql
    container_name: mysql-slave1
    environment:
      MYSQL_ROOT_PASSWORD: "123456"
      TZ: "Asia/Shanghai" #设置时区
    volumes:
    - 
    command:
    -  "--server-id=2"
    -  "--character-set-server=utf8mb4"
    -  "--collation-server=utf8mb4_unicode_ci"
```

