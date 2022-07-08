# 服务器WireGuard安装与配置

#### 服务器环境：Ubuntu 18.04.6

#### 前置条件：已安装docker和docker-compose

#### 1.升级系统包版本

```
#此步骤为了确保系统支持WireGuard，如果确定支持，则可以跳过此步骤
#这里以ubuntu为例，其他版本Linux方式会不同
apt update
apt upgrade
```

#### 2.下载WireGuard包

```
#这里以ubuntu为例
apt install wireguard
```

#### 3.配置和启动WireGuard

这里采用docker-compose来快速配置和启动WireGuard

##### 3.1编写docker-compose.yml文件

```
version: "3.8"
services:
  wg-easy:
    environment:
      # 必填:
      # 改成服务器公网IP
      - WG_HOST=raspberrypi.local

      # 可选:
      # 设置密码，用于创建客户端文件（客户端登录时需要服务器生产的配置文件）
      # - PASSWORD=foobar123
      # 设置WireGuard端口默认为UDP:51820
      # - WG_PORT=51820
      # 设置默认地址，这里为服务器内网的网段,会将这个网段的IP分给客户端
      # - WG_DEFAULT_ADDRESS=10.8.0.x
      # 设置默认DNS服务器地址
      # - WG_DEFAULT_DNS=1.1.1.1
      # 设置客户端可访问的地址网段，可设置成服务器内网网段
      # - WG_ALLOWED_IPS=0.0.0.0/0, ::/0
      
    image: weejewel/wg-easy
    container_name: wg-easy
    
    #设置挂载目录，这里为当前目录
    volumes:
      - .:/etc/wireguard
    ports:
      - "51820:51820/udp"
      - "51821:51821/tcp"
    restart: unless-stopped
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    sysctls:
      - net.ipv4.ip_forward=1
      - net.ipv4.conf.all.src_valid_mark=1

```

##### 3.2运行docker-compose.yml，启动容器

```
docker-compose up -d
#运行成功后会在当前目录生成wg0.conf和wg0.json两个文件
```

#### 4.访问服务器51821端口

公网IP:51821

访问完成会进入WireGuard页面。

输入在docker-compose.ym中设置的密码。

点击创建客户端配置文件并下载。

下载完成后在本地使用WireGuard，添加下载的配置文件，点击连接。

成功后可利用服务器内网地址访问服务器。
