# ELK安装

### 服务器环境：CentOS 7.6

### ELK部署方式：docker或docker-compose

### docker部署

#### 1.下载镜像

```bash
docker pull elasticsearch:7.13.3
docker pull kibana:7.13.3
docker pull logstash:7.13.3
```

#### 2.安装elasticsearch

##### 2.1创建挂载目录

```bash
mkdir -p elasticsearch/config
mkdir -p elasticsearch/data
```

##### 2.2创建elasticsearch.yml

```yaml
http.host: 0.0.0.0
http.cors.enabled: true
http.cors.allow-origin: "*"
http.cors.allow-headers: Authorization,X-Requested-With,Content-Length,Content-Type
xpack.security.enabled: true
xpack.license.self_generated.type: basic
xpack.security.transport.ssl.enabled: true
```

##### 2.3.创建elasticsearch实例

```bash
docker run \
--name elasticsearch -p 9200:9200 -p 9300:9300 \
-e "discovery.type=single-node" \
-e ES_JAVA_OPTS="-Xms256m -Xmx256m" \
-v /root/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
-v /root/elasticsearch/data:/usr/share/elasticsearch/data \
-d elasticsearch:7.13.3
```

##### 2.4设置ES密码

```bash
#进入容器
docker exec -it elasticsearch /bin/bash
#设置密码
./bin/elasticsearch-setup-passwords interactive
```

##### 2.5安装IK分词器

```bash
./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.13.3/elasticsearch-analysis-ik-7.13.3.zip
```

##### 2.6退出容器并重启

```
docker restart elasticsearch
```

#### 3.安装kibana

##### 3.1创建挂载目录

```bash
mkdir -p kibana/config
```

##### 3.2创建kibana.yml

```yaml
server.host: "0"
elasticsearch.hosts: [ "http://***.***.***.***:9200" ]
elasticsearch.username: "elastic"
elasticsearch.password: "********"
monitoring.ui.container.elasticsearch.enabled: true
#设置中文
i18n.locale: "zh-CN"
```

##### 3.3创建kibana实例

```bash
docker run \
--name kibana \
-v /root/kibana/config/kibana.yml:/usr/share/kibana/config/kibana.yml \
-e ELASTICSEARCH_URL=http://***.***.***.***:9200 \
-p 5601:5601 \
-d kibana:7.13.3
```

#### 4.安装Logstash

##### 4.1创建挂载目录

```bash
mkdir -p logstash/config
```

##### 4.2创建logstash.conf

```bash
vim /root/logstash/config/logstash.conf
```

```
input {
    tcp {
        port => 5044
        codec => json_lines
    }
}
output{
  elasticsearch {
        action => "index"
        hosts => ["http://***.***.***.***:9200"]
        index => "logstash-%{+YYYY.MM.dd}"
        user => "elastic"
        password => "********"
     }
  stdout { codec => rubydebug }
}
```

##### 4.4 创建logstash实例

```bash
docker run \
-d -p 5044:5044 \
-v /root/logstash/config:/usr/share/logstash/pipeline \
--name logstash \
logstash:7.13.3 
```

### docker-compose部署

#### 1.下载镜像

```
docker pull elasticsearch:7.13.3
docker pull kibana:7.13.3
docker pull logstash:7.13.3
```

#### 2.创建挂载路径

```
mkdir docker-compose

mkdir -p elasticsearch/data
mkdir -p elasticsearch/config
mkdir -p elasticsearch/plugins
chmod 777 elasticsearch/data
chmod 777 elasticsearch/config
chmod 777 elasticsearch/plugins

mkdir -p kibana/config
chmod 777 kibana/config

mkdir -p logstash/config
chmod 777 logstash/config
```

#### 3.创建elasticsearch.yml文件

```
touch elasticsearch/config/elasticsearch.yml
```

elasticsearch.yml内容：

```
cluster.name: elasticsearch
http.host: 0.0.0.0
http.cors.enabled: true
http.cors.allow-origin: "*"
http.cors.allow-headers: Authorization,X-Requested-With,Content-Length,Content-Type
```

#### 4.创建kibana.yml文件

```
touch kibana/config/kibana.yml
```

kibana.yml内容：

```
server.host: "0"
elasticsearch.hosts: [ "http://elasticsearch:9200" ]
monitoring.ui.container.elasticsearch.enabled: true
i18n.locale: "zh-CN"
```

#### 5.创建logstash.conf

```
touch logstash/config/logstash.conf
```

logstash.conf内容：

```
input {
    tcp {
        port => 5044
        codec => json_lines
    }
}
output{
  elasticsearch {
        action => "index"
        hosts => ["http://elasticsearch:9200"]
        index => "logstash-%{+YYYY.MM.dd}"
     }
  stdout { codec => rubydebug }
}
```

#### 创建docker-compose-elk.yml文件

```
version: "3"

services:
  elasticsearch:
    image: elasticsearch:7.13.3
    container_name: elasticsearch
    restart: always
    networks:
      - toolbox
    ports:
      - "9200:9200"
      - "9300:9300"
    volumes:
      - /root/docker-compose/elasticsearch/data:/usr/share/elasticsearch/data
      - /root/docker-compose/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
      - /root/docker-compose/elasticsearch/plugins:/usr/share/elasticsearch/plugins
    environment:
      - "ES_JAVA_OPTS=-Xms256m -Xmx256m"
      - discovery.type=single-node
  logstash:
    image: logstash:7.13.3
    container_name: logstash
    depends_on:
      - elasticsearch
    networks:
      - toolbox
    ports:
      - "5044:5044"
    volumes:
      - /root/docker-compose/logstash/config:/usr/share/logstash/pipeline

  kibana:
    image: kibana:7.13.3
    container_name: kibana
    restart: always
    depends_on:
      - elasticsearch
      - logstash
    networks:
      - toolbox
    ports:
      - "5601:5601"
    volumes:
      - /root/docker-compose/kibana/config/kibana.yml:/usr/share/kibana/config/kibana.yml

networks:
  toolbox:

```

#### 6.运行docker-compose-elk.yml

```
docker-compose -f docker-compose-elk.yml up -d
```

#### 7.查看是否部署成功

```
#在浏览器访问kibana
http://ip:5601
#在浏览器访问elasticsearch
http://ip:9
```

