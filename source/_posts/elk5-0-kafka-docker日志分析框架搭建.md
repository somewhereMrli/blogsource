---
title: elk5.0+kafka+docker日志分析框架搭建
date: 2017-07-03 09:37:43
tags: elk,docker,docker-compose
---


## CentOS 7.0下Docker的安装

1. 查看内核版本(Docker需要64位版本，同时内核版本在3.10以上，如果版本低于3.10，需要升级内核)：

   ```shell
   uname -r
   ```

2. 更新yum包：

   ```shell
   yum update
   ```

3. 添加yum仓库：

   ```shell
   sudo tee /etc/yum.repos.d/docker.repo <<-'EOF'
   [dockerrepo]
   name=Docker Repository
   baseurl=https://mirrors.aliyun.com/docker-engine/yum/repo/main/centos/7/
   enabled=1
   gpgcheck=1
   gpgkey=https://mirrors.aliyun.com/docker-engine/yum/gpg
   EOF
   ```

4. 安装Docker

   ```shell
   yum install docker-engine
   ```

5. 启动Docker

   ```shell
   service docker start
   ```

6. 使用Docker国内镜像（为Docker镜像下载提速，非必须）

   ```shell
   curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://fe8a7d6e.m.daocloud.io
   ```

## Docker Compose的安装


Compose的安装有多种方式，例如通过shell安装、通过pip安装、以及将compose作为容器安装等等。本文讲解通过shell安装的方式。其他安装方式如有兴趣，可以查看Docker的官方文档：[https://docs.docker.com/compose/install/](https://docs.docker.com/compose/install/)


* 下载`docker-compose` ，并放到`/usr/local/bin/` 

```shell
curl -L https://mirrors.aliyun.com/docker-toolbox/linux/compose/1.9.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
```

* 为Docker Compose脚本添加执行权限

```shell
chmod +x /usr/local/bin/docker-compose
```

* 安装完成，测试：

```shell
docker-compose --version
```

## ELK 5.X 配置 yml
```yml
version: '2'
services:
  kibana:
    image: docker.elastic.co/kibana/kibana:5.4.2
    volumes:
      - "./kibana.yml:/usr/share/kibana/config/kibana.yml"
    restart: always
    ports:
      - "5601:5601"
    links:
      - elasticsearch
    depends_on:
      - elasticsearch
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:5.4.3
    environment:
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=true
      - xpack.security.enabled=false
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    mem_limit: 1g
    volumes:
      - "./esdata:/usr/share/elasticsearch/data"
    ports:
      - "9200:9200"
  logstash:
    image: docker.elastic.co/logstash/logstash:5.1.1
    volumes:
      - "./logstash.conf:/config-dir/logstash.conf"
    restart: always
    command: logstash -f /config-dir/logstash.conf
    ports:
      - "9600:9600"
      - "7777:7777"
    links:
      - elasticsearch
      - kafka1
      - kafka2
      - kafka3
  kafka1:
    image: wurstmeister/kafka
    depends_on:
      - zoo1
      - zoo2
      - zoo3
    links:
      - zoo1
      - zoo2
      - zoo3
    ports:
      - "9092:9092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ADVERTISED_PORT: 9092
      KAFKA_LOG_RETENTION_HOURS: "168"
      KAFKA_LOG_RETENTION_BYTES: "100000000"
      KAFKA_ZOOKEEPER_CONNECT:  zoo1:2181,zoo2:2181,zoo3:2181
      KAFKA_CREATE_TOPICS: "log:3:3"
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: 'false'
  kafka2:
    image: wurstmeister/kafka
    depends_on:
      - zoo1
      - zoo2
      - zoo3
    links:
      - zoo1
      - zoo2
      - zoo3
    ports:
      - "9093:9092"
    environment:
      KAFKA_BROKER_ID: 2
      KAFKA_ADVERTISED_PORT: 9092
      KAFKA_LOG_RETENTION_HOURS: "168"
      KAFKA_LOG_RETENTION_BYTES: "100000000"
      KAFKA_ZOOKEEPER_CONNECT:  zoo1:2181,zoo2:2181,zoo3:2181
      KAFKA_CREATE_TOPICS: "log:3:3"
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: 'false'
  kafka3:
    image: wurstmeister/kafka
    depends_on:
      - zoo1
      - zoo2
      - zoo3
    links:
      - zoo1
      - zoo2
      - zoo3
    ports:
      - "9094:9092"
    environment:
      KAFKA_BROKER_ID: 3
      KAFKA_ADVERTISED_PORT: 9092
      KAFKA_LOG_RETENTION_HOURS: "168"
      KAFKA_LOG_RETENTION_BYTES: "100000000"
      KAFKA_ZOOKEEPER_CONNECT:  zoo1:2181,zoo2:2181,zoo3:2181
      KAFKA_CREATE_TOPICS: "log:3:3"
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: 'false'
  zoo1:
    image: elevy/zookeeper:latest
    environment:
      MYID: 1
      SERVERS: zoo1,zoo2,zoo3
    ports:
      - "2181:2181"
  zoo2:
    image: elevy/zookeeper:latest
    environment:
      MYID: 2
      SERVERS: zoo1,zoo2,zoo3
    ports:
      - "2182:2181"
  zoo3:
    image: elevy/zookeeper:latest
    environment:
      MYID: 3
      SERVERS: zoo1,zoo2,zoo3
    ports:
      - "2183:2181"
  filebeat:
    image: docker.elastic.co/beats/filebeat:5.4.3
    mem_limit: 1g
    volumes:
      - "./filebeat.yml:/usr/share/filebeat/filebeat.yml:ro"
      - "./apache-logs:/apache-logs"
    links:
      - kafka1
      - kafka2
      - kafka3
    depends_on:
      - apache
      - kafka1
      - kafka2
      - kafka3
  apache:
    image: lzrbear/docker-apache2-ubuntu
    volumes:
      - "./apache-logs:/var/log/apache2"
    ports:
      - "8888:80"
    depends_on:
      - logstash
```

## 配置文件
### filebeat.yml:
```
filebeat.prospectors:
- paths:
    - /apache-logs/access.log
  tags:
    - testenv
    - apache_access
  input_type: log
  document_type: apache_access
  fields_under_root: true

- paths:
    - /apache-logs/error.log
  tags:
    - testenv
    - apache_error
  input_type: log
  document_type: apache_error
  fields_under_root: true

output.kafka:
  hosts: ["kafka1:9092", "kafka2:9092", "kafka3:9092"]
  topic: 'log'
  partition.round_robin:
    reachable_only: false
  required_acks: 1
  compression: gzip
  max_message_bytes: 1000000
```

### logstash.conf:
```
input {
  kafka {
    bootstrap_servers => "kafka1:9092,kafka2:9092,kafka3:9092"
    client_id => "logstash"
    group_id => "logstash"
    consumer_threads => 3
    topics => ["log"]
    codec => "json"
    tags => ["log", "kafka_source"]
    type => "log"
  }
}

filter {
  if [type] == "apache_access" {
    grok {
      match => { "message" => "%{COMMONAPACHELOG}" }
    }
    date {
      match => ["timestamp", "dd/MMM/yyyy:HH:mm:ss Z"]
      remove_field => ["timestamp"]
    }
  }
  if [type] == "apache_error" {
    grok {
      match => { "message" => "%{COMMONAPACHELOG}" }
    }
    date {
      match => ["timestamp", "dd/MMM/yyyy:HH:mm:ss Z"]
      remove_field => ["timestamp"]
    }
  }
}

output {
  if [type] == "apache_access" {
    elasticsearch {
         hosts => ["elasticsearch:9200"]
         index => "logstash-apache-access-%{+YYYY.MM.dd}"
    }
  }
  if [type] == "apache_error" {
    elasticsearch {
         hosts => ["elasticsearch:9200"]
         index => "logstash-apache-error-%{+YYYY.MM.dd}"
    }
  }
}
```
### kibana.yml:
```
server.name: kibana
server.host: "0"
elasticsearch.url: http://elasticsearch:9200
xpack.monitoring.ui.container.elasticsearch.enabled: false
```
