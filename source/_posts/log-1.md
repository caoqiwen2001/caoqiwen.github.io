---
title: Filebeat+Logstash+ElasticSearch搭建
date: 2019-05-24 16:16:50
tags:
  -  日记搭建
categories:
    - 日记搭建
type: tags
---
### 前言
最近项目有很多日记需要记录，由于之前系统修改数据都是直接保存数据库，导致数据库单表的数据越来越多，后续查询越来越慢，在此基础上，急需一套日记采集系统来支撑。目前比较成熟的方案就是Filebeat+Logstash+ElasticSearch来实现，
笔者因此动手实现该系统，这套系统目前是基于elasticsearch-7.0.1实现的，在安装完三个主要软件之后，
接下来看看具体的配置文件：

### Filebeat的配置

```
filebeat.inputs:
   paths:
    - /opt/elk/log/info.log   ##指定输入的日记路径
  json.keys_under_root: true  
  json.overwrite_keys: true   ##覆盖相同的key，filebeat也有一个message的字段
output.logstash:
  # The Logstash hosts
  hosts: ["10.9.12.71:5044"] ##输出到logstash
```

###  Logstash的配置

```
input {
  beats {
    port => 5044
  }
}
filter {
    # 将message转为json格式
        json{
            source => "message"
            target => "message"
        }
}
output {
  elasticsearch {
    hosts => ["http://10.9.12.71:9200"]
    #index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
    index => "syslog-%{+YYYY.MM.dd}"
  }
}
output {
   stdout {
    codec => rubydebug
   }
}
```
这里重点在filter这里，由于filebeat收集到的message本身就是JSON字符串，所以这里要将json字符串进行转换，然后再输出到elasticsearch中。

###  elasticsearch 的配置

```
cluster.name: elasticsearch ##集群的名字
node.name: node-1 ##当前节点的名字
path.data: /opt/elk/elasticsearch-7.0.1/data
path.logs: /opt/elk/elasticsearch-7.0.1/logs
network.host: 0.0.0.0  ##允许所有用户访问
http.port: 9200    ##开放9200端口
http.cors.enabled: true
http.cors.allow-origin: "*"   ##允许head插件访问
```

### 启动方法

1. 后台启动logstash  
nohup  ./logstash -f  /opt/elk/logstash-7.0.1/config/logstash-sample.conf  &

1. 后台启动filebeat  
nohup  ./filebeat -e -c filebeat.yml -d "publish"  &

1. elasticsearch的启动
./elasticsearch -d   

如果测试filebeat，需要删除/opt/elk/filebeat-7.0.1-linux-x86_64/data下的registry的文件夹，因为它读取log每次是增量读取的。 	




