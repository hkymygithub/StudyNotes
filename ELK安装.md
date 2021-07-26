# ELK安装

### 服务器环境：CentOS 7.6

### docker部署

#### 1.下载镜像

```bash
docker pull elasticsearch:7.13.3
docker pull kibana:5.6.11
docker pull logstash:5.6.15
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
