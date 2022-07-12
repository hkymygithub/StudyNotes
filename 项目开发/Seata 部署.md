# Seata 部署

## Docker Compose

### Seata 单节点

#### 部署环境

服务器: Linux

Nacos 注册中心

DB存储(Mysql)

#### 创建相应挂载目录

```bash
mkdir -p ./seata/config
```
#### 在 Mysql 中创建 Seata 所需的表
```sql
create database seata;

-- the table to store GlobalSession data
CREATE TABLE IF NOT EXISTS `global_table`
(
    `xid`                       VARCHAR(128) NOT NULL,
    `transaction_id`            BIGINT,
    `status`                    TINYINT      NOT NULL,
    `application_id`            VARCHAR(32),
    `transaction_service_group` VARCHAR(32),
    `transaction_name`          VARCHAR(128),
    `timeout`                   INT,
    `begin_time`                BIGINT,
    `application_data`          VARCHAR(2000),
    `gmt_create`                DATETIME,
    `gmt_modified`              DATETIME,
    PRIMARY KEY (`xid`),
    KEY `idx_status_gmt_modified` (`status` , `gmt_modified`),
    KEY `idx_transaction_id` (`transaction_id`)
    ) ENGINE = InnoDB
    DEFAULT CHARSET = utf8mb4;

-- the table to store BranchSession data
CREATE TABLE IF NOT EXISTS `branch_table`
(
    `branch_id`         BIGINT       NOT NULL,
    `xid`               VARCHAR(128) NOT NULL,
    `transaction_id`    BIGINT,
    `resource_group_id` VARCHAR(32),
    `resource_id`       VARCHAR(256),
    `branch_type`       VARCHAR(8),
    `status`            TINYINT,
    `client_id`         VARCHAR(64),
    `application_data`  VARCHAR(2000),
    `gmt_create`        DATETIME(6),
    `gmt_modified`      DATETIME(6),
    PRIMARY KEY (`branch_id`),
    KEY `idx_xid` (`xid`)
    ) ENGINE = InnoDB
    DEFAULT CHARSET = utf8mb4;

-- the table to store lock data
CREATE TABLE IF NOT EXISTS `lock_table`
(
    `row_key`        VARCHAR(128) NOT NULL,
    `xid`            VARCHAR(128),
    `transaction_id` BIGINT,
    `branch_id`      BIGINT       NOT NULL,
    `resource_id`    VARCHAR(256),
    `table_name`     VARCHAR(32),
    `pk`             VARCHAR(36),
    `status`         TINYINT      NOT NULL DEFAULT '0' COMMENT '0:locked ,1:rollbacking',
    `gmt_create`     DATETIME,
    `gmt_modified`   DATETIME,
    PRIMARY KEY (`row_key`),
    KEY `idx_status` (`status`),
    KEY `idx_branch_id` (`branch_id`),
    KEY `idx_xid_and_branch_id` (`xid` , `branch_id`)
    ) ENGINE = InnoDB
    DEFAULT CHARSET = utf8mb4;

CREATE TABLE IF NOT EXISTS `distributed_lock`
(
    `lock_key`       CHAR(20) NOT NULL,
    `lock_value`     VARCHAR(20) NOT NULL,
    `expire`         BIGINT,
    primary key (`lock_key`)
    ) ENGINE = InnoDB
    DEFAULT CHARSET = utf8mb4;

INSERT INTO `distributed_lock` (lock_key, lock_value, expire) VALUES ('AsyncCommitting', ' ', 0);
INSERT INTO `distributed_lock` (lock_key, lock_value, expire) VALUES ('RetryCommitting', ' ', 0);
INSERT INTO `distributed_lock` (lock_key, lock_value, expire) VALUES ('RetryRollbacking', ' ', 0);
INSERT INTO `distributed_lock` (lock_key, lock_value, expire) VALUES ('TxTimeoutCheck', ' ', 0);
```

#### 在各客户端即使用分布式事务的服务中建 undo_log 表

```sql
-- for AT mode you must to init this sql for you business database. the seata server not need it.
CREATE TABLE IF NOT EXISTS `undo_log`
(
    `branch_id`     BIGINT       NOT NULL COMMENT 'branch transaction id',
    `xid`           VARCHAR(128) NOT NULL COMMENT 'global transaction id',
    `context`       VARCHAR(128) NOT NULL COMMENT 'undo_log context,such as serialization',
    `rollback_info` LONGBLOB     NOT NULL COMMENT 'rollback info',
    `log_status`    INT(11)      NOT NULL COMMENT '0:normal status,1:defense status',
    `log_created`   DATETIME(6)  NOT NULL COMMENT 'create datetime',
    `log_modified`  DATETIME(6)  NOT NULL COMMENT 'modify datetime',
    UNIQUE KEY `ux_undo_log` (`xid`, `branch_id`)
) ENGINE = InnoDB
  AUTO_INCREMENT = 1
  DEFAULT CHARSET = utf8mb4 COMMENT ='AT transaction mode undo table';
```

#### 在 Nacos 中创建 Seata 配置文件

dataId: seata-server  
group: DEFAULT_GROUP

```yaml
# 存储模式
store:
  mode: db
  db:
    datasource: druid
    dbType: mysql
    # 需要根据mysql的版本调整driverClassName
    # mysql8及以上版本对应的driver：com.mysql.cj.jdbc.Driver
    # mysql8以下版本的driver：com.mysql.jdbc.Driver
    driverClassName: com.mysql.cj.jdbc.Driver
    # 注意根据生产实际情况调整参数host和port
    url: jdbc:mysql://ip:port/seata?useUnicode=true&characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useSSL=false
    # 数据库用户名
    user: user
    # 用户名密码
    password: password
```

#### 创建 registry.conf 文件

```bash
cd seata/registry.conf
vim registry.conf
```

```text
registry {
  type = "nacos"
  
  nacos {
    # seata服务注册在nacos上的别名，客户端通过该别名调用服务
    application = "seata-server"
    # 请根据实际生产环境配置nacos服务的ip和端口
    serverAddr = "ip:port"
    # nacos上指定的namespace
    namespace = "public"
    cluster = "default"
    group="DEFAULT_GROUP"
    # 如果 nacos 开启了鉴权则需要
    username = "username"
    password = "password"
  }
}

config {
  type = "nacos"
  
  nacos {
    # 请根据实际生产环境配置nacos服务的ip和端口
    serverAddr = "ip:port"
    # nacos上指定的 namespace
    namespace = "public"
    group = "DEFAULT_GROUP"
    # 如果 nacos 开启了鉴权则需要
    username = "username"
    password = "password"
    # 从v1.4.2版本开始，已支持从一个Nacos dataId中获取所有配置信息,你只需要额外添加一个dataId配置项
    dataId: "seata-server"
    file-extension = "yaml"
  }
}
```

#### 创建 docker-compose.yml 文件

```yaml
version: "3"
services:
  seata-server:
    image: seataio/seata-server:1.4.2
    container_name: seata-server
    hostname: seata-server
    network_mode: host
    restart: always
    environment:
      # 指定seata服务启动端口
      - SEATA_PORT= port
      # 注册到nacos上的ip。客户端将通过该ip访问seata服务。
      # 注意公网ip和内网ip的差异。
      - SEATA_IP= ip
      - SEATA_CONFIG_NAME=file:/root/seata-config/registry
    volumes:
      # 因为registry.conf中是nacos配置中心，只需要把registry.conf放到./seata-server/config文件夹中
      - ./config:/root/seata-config
```

#### 运行 Seata

```bash
docker-compose up -d
```

#### 检查 Seata 是否启动

```bash
docker ps
```