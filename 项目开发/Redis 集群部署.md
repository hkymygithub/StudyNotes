# Redis部署

## Docker Compose

### Redis 单节点

#### 部署环境

服务器: Linux

#### 创建相应挂载目录

```bash
mkdir -p ./redis/conf
mkdir -p ./redis/data
```

### 创建 redis.conf 文件

```bash
cd redis/conf
vim redis.conf
```

```text
# 端口
port 6379

# 登录密码
requirepass ******

# 设置外部网络连接redis服务，开启需配置bind ip或者设置访问密码，关闭此时外部网络可以直接访问
protected-mode no

# 不作为守护进程运行
daemonize no

# 开启持久化
appendonly yes
```

#### 创建 docker-compose.yml 文件

```bash
cd redis
vim docker-compose.yml
```

```yml
version: "3"

services:
  redis:
    image: redis
    container_name: redis
    restart: always
    network_mode: host
    volumes:
      - ./conf/redis.conf:/usr/local/etc/redis/redis.conf
      - ./data:/data
    command: redis-server /usr/local/etc/redis/redis.conf
```

#### 运行 Redis

```bash
docker-compose up -d
```

#### 检查 Redis 是否启动

```bash
docker ps
```

### Redis 集群部署

#### 部署环境

服务器：均为 Linux

| 发行版本       | IP（假设）  |
| -------------- | ----------- |
| Ubuntu 18.04.6 | 192.168.0.1 |
| Centos 8       | 192.168.0.2 |
| Centos 8       | 192.168.0.3 |

注意事项：

每台服务器均装有 Docker 和 Docker Compose

本次部署共三台服务器，redis 集群采取三主三从模式。

个服务器需要开放相应端口：

  192.168.0.1 开放 6371，16371；6372，16372

  192.168.0.2 开放 6373，16373；6374，16374

  192.168.0.3 开放 6375，16375；6376，16376

#### 创建相应挂载目录

##### 192.168.0.1服务器

```bash
mkdir -p ./redis-cluster/6371/conf
mkdir -p ./redis-cluster/6371/data
mkdir -p ./redis-cluster/6372/conf
mkdir -p ./redis-cluster/6372/data
```

##### 192.168.0.2服务器

```bash
mkdir -p ./redis-cluster/6373/conf
mkdir -p ./redis-cluster/6373/data
mkdir -p ./redis-cluster/6374/conf
mkdir -p ./redis-cluster/6374/data
```

##### 192.168.0.3服务器

```bash
mkdir -p ./redis-cluster/6375/conf
mkdir -p ./redis-cluster/6375/data
mkdir -p ./redis-cluster/6376/conf
mkdir -p ./redis-cluster/6376/data
```

#### 创建相应配置文件

##### 192.168.0.1 服务器

```bash
vim redis-cluster/6371/conf/redis.conf
```

具体内容：

```text
port 6371 #端口号
requirepass ****** #访问认证（访问密码）
masterauth ****** #访问认证（访问密码）（如果主节点开启了访问认证，从节点访问主节点需要认证）
protected-mode no #关闭保护模式
daemonize no #是否以守护线程的方式启动（后台启动），默认 no
appendonly yes  #开启 AOF 持久化模式
cluster-enabled yes  #开启集群模式
cluster-config-file nodes.conf  #集群节点信息文件
cluster-node-timeout 15000  #集群节点连接超时时间
cluster-announce-ip 192.168.0.1  #集群节点 IP（这里为192.168.0.1服务器）
cluster-announce-port 6371  #集群节点映射端口
cluster-announce-bus-port 16371  #集群节点总线端口
```

------

```bash
vim redis-cluster/6372/conf/redis.conf
```

具体内容：

```test
port 6372
requirepass ******
masterauth ******
protected-mode no
daemonize no
appendonly yes
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 15000
cluster-announce-ip 192.168.0.1
cluster-announce-port 6372
cluster-announce-bus-port 16372
```

##### 192.168.0.2 服务器

```bash
vim redis-cluster/6373/conf/redis.conf
```

具体内容：

```text
port 6373
requirepass ******
masterauth ******
protected-mode no
daemonize no
appendonly yes
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 15000
cluster-announce-ip 192.168.0.2
cluster-announce-port 6373
cluster-announce-bus-port 16373
```

------

```bash
vim redis-cluster/6374/conf/redis.conf
```

具体内容：

```text
port 6374
requirepass ******
masterauth ******
protected-mode no
daemonize no
appendonly yes
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 15000
cluster-announce-ip 192.168.0.2
cluster-announce-port 6374
cluster-announce-bus-port 16374
```

##### 192.168.0.3 服务器

```bash
vim redis-cluster/6375/conf/redis.conf
```

具体内容：

```text
port 6375
requirepass ******
masterauth ******
protected-mode no
daemonize no
appendonly yes
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 15000
cluster-announce-ip 192.168.0.3
cluster-announce-port 6375
cluster-announce-bus-port 16375
```

------

```bash
vim redis-cluster/6376/conf/redis.conf
```

具体内容：

```text
port 6376
requirepass ******
masterauth ******
protected-mode no
daemonize no
appendonly yes
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 15000
cluster-announce-ip 192.168.0.3
cluster-announce-port 6376
cluster-announce-bus-port 16376
```

#### 创建相应 docker-compose.yml 文件

##### 192.168.0.1 服务器

```bash
vim redis-cluster/docker-compose.yml
```

具体内容：

```yml
version: "3"

services:
  redis-6371:
    image: redis
    container_name: redis-6371
    restart: always
    network_mode: host
    volumes:
      - ./6371/conf/redis.conf:/usr/local/etc/redis/redis.conf
      - ./6371/data:/data
    command: redis-server /usr/local/etc/redis/redis.conf

  redis-6372:
    image: redis
    container_name: redis-6372
    restart: always
    network_mode: host
    volumes:
      - ./6372/conf/redis.conf:/usr/local/etc/redis/redis.conf
      - ./6372/data:/data
    command: redis-server /usr/local/etc/redis/redis.conf
```

##### 192.168.0.2 服务器

```bash
vim redis-cluster/docker-compose.yml
```

具体内容：

```yml
version: "3"

services:
  redis-6373:
    image: redis
    container_name: redis-6373
    restart: always
    network_mode: host
    volumes:
      - ./6373/conf/redis.conf:/usr/local/etc/redis/redis.conf
      - ./6373/data:/data
    command: redis-server /usr/local/etc/redis/redis.conf

  redis-6374:
    image: redis
    container_name: redis-6374
    restart: always
    network_mode: host
    volumes:
      - ./6374/conf/redis.conf:/usr/local/etc/redis/redis.conf
      - ./6374/data:/data
    command: redis-server /usr/local/etc/redis/redis.conf
```

##### 192.168.0.3 服务器

```bash
vim redis-cluster/docker-compose.yml
```

具体内容：

```yml
version: "3"

services:
  redis-6375:
    image: redis
    container_name: redis-6375
    restart: always
    network_mode: host
    volumes:
      - ./6375/conf/redis.conf:/usr/local/etc/redis/redis.conf
      - ./6375/data:/data
    command: redis-server /usr/local/etc/redis/redis.conf

  redis-6376:
    image: redis
    container_name: redis-6376
    restart: always
    network_mode: host
    volumes:
      - ./6376/conf/redis.conf:/usr/local/etc/redis/redis.conf
      - ./6376/data:/data
    command: redis-server /usr/local/etc/redis/redis.conf
```

#### 启动各服务器容器

```bash
docker-compose up -d
```

#### 创建集群

```bash
#进入任意一个容器节点
docker exec -it redis-6371 /bin/bash
```

```bash
# 切换至指定目录
cd /usr/local/bin/
```

```bash
#输入命令创建集群（各服务器IP需对应，此处每个主节点有一个从节点）
redis-cli -a hkyredis --cluster create 192.168.0.1:6371 192.168.0.1:6372 192.168.0.2:6373 192.168.0.2:6374 192.168.0.3:6375 192.168.0.3:6376 --cluster-replicas 1
```

#### 部署完成
