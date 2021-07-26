## Nginx安装

### 服务器环境：CentOS 7.6

### docker部署

#### 1.下载镜像

```bash
docker pull nginx
```

#### 2.创建挂载目录

```bash
mkdir -p nginx/{config,html,log}
```

#### 3.启动一个临时容器

```bash
docker run -d --name nginx -p 80:80 nginx
```

#### 4.复制配置文件

```bash
docker cp nginx:/etc/nginx/nginx.conf nginx/
docker cp nginx:/etc/nginx/conf.d/default.conf nginx/config/
```

#### 5.删除原来容器

```bash
docker stop nginx
docker rm nginx
```

#### 6.创建Nginx实例

```bash
docker run -d \
--name nginx -p 8501:80 \
-v /root/nginx/nginx.conf:/etc/nginx/nginx.conf \
-v /root/nginx/log:/var/log/nginx \
-v /root/nginx/html:/usr/share/nginx/html \
-v /root/nginx/config:/etc/nginx/conf.d \
--privileged=true \
nginx
```

