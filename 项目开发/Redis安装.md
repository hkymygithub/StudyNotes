# Redis安装

### Redis集群部署——docker-compose

```
version: "3"

services:
  redis-6371:
    image: redis
    container_name: redis-6371
    restart: always
    network_mode: host
    volumes:
      - /root/redis-cluster/6371/conf/redis.conf:/usr/local/etc/redis/redis.conf
      - /root/redis-cluster/6371/data:/data
    command: redis-server /usr/local/etc/redis/redis.conf

  redis-6372:
    image: redis
    container_name: redis-6372
    restart: always
    network_mode: host
    volumes:
      - /root/redis-cluster/6372/conf/redis.conf:/usr/local/etc/redis/redis.conf
      - /root/redis-cluster/6372/data:/data
    command: redis-server /usr/local/etc/redis/redis.conf
```

```
version: "3"

services:
  redis-6373:
    image: redis
    container_name: redis-6373
    restart: always
    network_mode: host
    volumes:
      - /root/redis-cluster/6373/conf/redis.conf:/usr/local/etc/redis/redis.conf
      - /root/redis-cluster/6373/data:/data
    command: redis-server /usr/local/etc/redis/redis.conf

  redis-6374:
    image: redis
    container_name: redis-6374
    restart: always
    network_mode: host
    volumes:
      - /root/redis-cluster/6374/conf/redis.conf:/usr/local/etc/redis/redis.conf
      - /root/redis-cluster/6374/data:/data
    command: redis-server /usr/local/etc/redis/redis.conf
```

```
version: "3"

services:
  redis-6375:
    image: redis
    container_name: redis-6375
    restart: always
    network_mode: host
    volumes:
      - /root/redis-cluster/6375/conf/redis.conf:/usr/local/etc/redis/redis.conf
      - /root/redis-cluster/6375/data:/data
    command: redis-server /usr/local/etc/redis/redis.conf

  redis-6376:
    image: redis
    container_name: redis-6376
    restart: always
    network_mode: host
    volumes:
      - /root/redis-cluster/6376/conf/redis.conf:/usr/local/etc/redis/redis.conf
      - /root/redis-cluster/6376/data:/data
    command: redis-server /usr/local/etc/redis/redis.conf
```



```
port 6371
requirepass hkyredis
masterauth hkyredis
protected-mode no
daemonize no
appendonly yes
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 15000
cluster-announce-ip 106.54.118.77
cluster-announce-port 6371
cluster-announce-bus-port 16371

port 6372
requirepass hkyredis
masterauth hkyredis
protected-mode no
daemonize no
appendonly yes
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 15000
cluster-announce-ip 106.54.118.77
cluster-announce-port 6372
cluster-announce-bus-port 16372

port 6373
requirepass hkyredis
masterauth hkyredis
protected-mode no
daemonize no
appendonly yes
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 15000
cluster-announce-ip 121.36.29.146
cluster-announce-port 6373
cluster-announce-bus-port 16373

port 6374
requirepass hkyredis
masterauth hkyredis
protected-mode no
daemonize no
appendonly yes
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 15000
cluster-announce-ip 121.36.29.146
cluster-announce-port 6374
cluster-announce-bus-port 16374

port 6375
requirepass hkyredis
masterauth hkyredis
protected-mode no
daemonize no
appendonly yes
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 15000
cluster-announce-ip 1.15.181.135
cluster-announce-port 6375
cluster-announce-bus-port 16375

port 6376
requirepass hkyredis
masterauth hkyredis
protected-mode no
daemonize no
appendonly yes
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 15000
cluster-announce-ip 1.15.181.135
cluster-announce-port 6376
cluster-announce-bus-port 16376
```

