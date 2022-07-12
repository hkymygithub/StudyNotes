# Nacos 部署

## Docker Compose

### Nacos 单节点

#### 部署环境

服务器: Linux

#### 创建相应挂载目录

```bash
mkdir -p ./nacos/init.d
mkdir -p ./nacos/logs
```

#### 创建 custom.properties 文件

```bash
cd nacos/init.d
touch custom.properties
```

#### 创建 docker-compose.yml 文件

```yaml
version: "3"
services:
  nacos:
    image: nacos/nacos-server:latest
    container_name: nacos
    environment:
      # 单例模式
      - MODE=standalone
      # 开启鉴权(可不配置)
      - NACOS_AUTH_ENABLE=true
      # 数据存储值 MySQL
      - SPRING_DATASOURCE_PLATFORM=mysql
      # Mysql Host 地址
      - MYSQL_SERVICE_HOST= ip
      # Mysql 数据库名称
      - MYSQL_SERVICE_DB_NAME= databaseName
      # Mysql 端口
      - MYSQL_SERVICE_PORT= port
      # Mysql 用户
      - MYSQL_SERVICE_USER= user
      # Mysql 密码
      - MYSQL_SERVICE_PASSWORD= password
      # Mysql 连接属性
      - MYSQL_SERVICE_DB_PARAM=characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useSSL=false
    volumes:
      - ./logs/:/home/nacos/logs/
      - ./init.d/custom.properties:/home/nacos/init.d/custom.properties
    network_mode: host
    restart: always
```

#### 运行 Nacos

```bash
docker-compose up -d
```

#### 检查 Nacos 是否启动

```bash
docker ps
```