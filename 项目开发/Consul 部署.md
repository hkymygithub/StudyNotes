# Consul 部署

## Docker Compose

### Consul 单节点

#### 部署环境

服务器: Linux

#### 创建相应挂载目录

```bash
mkdir -p consul/conf
mkdir -p consul/data
```

#### 创建 docker-compose.yml 文件

```bash
cd consul
vim docker-compose.yml
```

```yml
version: '3'

services:

  consul:
    image: hashicorp/consul:latest
    container_name: consul
    restart: always
    volumes:
      - ./config:/consul/config
      - ./data:/consul/data
    ports:
      - '8500:8500'
    command:
      - "agent"
      - "-server"
      - "-bootstrap"
      - "-ui"
      - "-node=consul-server"
      - "-client=0.0.0.0"
```

#### 运行 Consul

```bash
docker-compose up -d
```

#### 检查 Consul 是否启动

```bash
docker ps
```