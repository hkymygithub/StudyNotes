## Nginx反向代理Consul

### 服务器环境：CentOS 7.6

### 以docker方式实现

#### 事先准备

1. 已安装docker，并且创建了Nginx和Consul容器。
2. Nginx挂载目录：
   - 宿主机/root/nginx/config，容器/etc/nginx/conf.d
   - 宿主机/root/nginx/log，容器/var/log/nginx
   - 宿主机/root/nginx/html，容器/usr/share/nginx/html
   - 宿主机/root/nginx/nginx.conf，容器/etc/nginx/nginx.conf
3. Nginx映射端口：宿主机8501，容器80；Consul映射端口：宿主机8500，容器8500

#### 进入Nginx容器

```bash
docker exec -it nginx /bin/bash
```

#### 设置代理IP访问密码

```bash
#容器内部
#安装apache2-utils
apt-get update
apt-get install apache2-utils
#创建用户
htpasswd -c /etc/nginx/passwd.db ${用户名}
#输入密码
New password: 
Re-type new password: 
#退出容器
exit
```

#### 修改Nginx配置文件

**nginx.conf** (/etc/nginx/nginx.conf)与默认保持不变

**default.conf** （/etc/nginx/conf.default.conf）

```
server {
    listen       80;
    listen  [::]:80;
    server_name  localhost;
    location / {
        auth_basic "consul";
        auth_basic_user_file /etc/nginx/passwd.db;
        proxy_pass http://***.***.***.***:8500; #consul所在服务器IP
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```

#### 重启容器

```bash
docker restart nginx
```
