https://juejin.cn/post/6963816831844876301 

架构：

![image-20221012163252982](D:\Tech\linux\System\.assets\image-20221012163252982.png)

## 1.集群版本

- Java： openjdk version "1.8.0_345"
- zookeeper： 3.5.6
- Kafka： 2.12-2.3.0
- elasticsearch：7.10.1
- kibana：
- logstash：

## 2.服务环境说明

| IP地址         | 主机名     | 配置  | 角色                        |
| -------------- | ---------- | ----- | --------------------------- |
| 192.168.254.51 | elk-master | 2C 4G | es-master、kafka+zookeeper1 |
| 192.168.254.52 | elk-node1  | 2C 4G | es-node1、kafka+zookeeper2  |
| 192.168.254.53 | elk-node2  | 2C 4G | es-node2、kafka+zookeeper3  |
|                |            |       |                             |

## 3.部署zookeeper

注意：3个节点都要部署

```shell
$ wget -P /data https://archive.apache.org/dist/zookeeper/zookeeper-3.5.6/apache-zookeeper-3.5.6-bin.tar.gz
$ cd /data
$ tar -xf apache-zookeeper-3.5.6-bin.tar.gz
$ mv apache-zookeeper-3.5.6-bin zookeeper
$ mkdir /data/zookeeper/zkdata   # 数据存放目录
$ mkdir /data/zookeeper/zklogs   # 日志存放目录
```

```shell
$ vim conf/zoo.cfg
# 服务器之间或客户端与服务器之间维持心跳的时间间隔
# tickTime以毫秒为单位。
tickTime=2000 
# 集群中的follower服务器(F)与leader服务器(L)之间的初始连接心跳数
initLimit=10
# 集群中的follower服务器与leader服务器之间请求和应答之间能容忍的最多心跳数
syncLimit=5
# 数据保存目录
dataDir=/data/zookeeper/zkdata
# 日志保存目录
dataLogDir=/data/zookeeper/zklogs
# 客户端连接端口
clientPort=2181
# 客户端最大连接数。# 根据自己实际情况设置，默认为60个
maxClientCnxns=60
# 三个接点配置，格式为： server.服务编号=服务地址、LF通信端口、选举端口
server.1=192.168.254.51:2888:3888
server.2=192.168.254.52:2888:3888
server.3=192.168.254.53:2888:3888
```

**写入节点标记：**

```shell
$ echo "1" > /data/zookeeper/zkdata/myid   # master节点
$ echo "2" > /data/zookeeper/zkdata/myid   # node1节点
$ echo "3" > /data/zookeeper/zkdata/myid   # node2节点

```

**启动zookeeper集群**

```shell
$ cd /data/zookeeper/bin/
$ ./zkServer.sh start
$ ./zkServer.sh status   #检查集群状态
```

**配置systemctl启动**

```shell
$ vim /etc/systemd/system/zookeeper.service

[Unit]
Description=zookeeper.service
After=network.target
ConditionPathExists=/data/zookeeper/conf/zoo.cfg

[Service]
Type=forking
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
User=root
Group=root
ExecStart=/data/zookeeper/bin/zkServer.sh start
ExecStop=/data/zookeeper/bin/zkServer.sh stop
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

## 4.部署kafka

注意：3个节点都要部署

```shell
$ wget -P /data https://archive.apache.org/dist/kafka/2.3.0/kafka_2.12-2.3.0.tgz
$ tar -xf kafka_2.12-2.3.0.tgz
$ mv kafka_2.12-2.3.0.tgz kafka

# 创建数据存储目录
$ mkdir /data/kafka/kfkdata
```

```shell
$ vim /data/kafka/config/server.properties
############################# Server Basics ############################# 
# broker的id，值为整数，且必须唯一，在一个集群中不能重复
broker.id=1

############################# Socket Server Se：ttings ############################# 
# kafka默认监听的端口为9092 (默认与主机名进行连接)
listeners=PLAINTEXT://192.168.254.51:9092

# 处理网络请求的线程数量，默认为3个
num.network.threads=3

# 执行磁盘IO操作的线程数量，默认为8个 
num.io.threads=8

# socket服务发送数据的缓冲区大小，默认100KB
socket.send.buffer.bytes=102400

# socket服务接受数据的缓冲区大小，默认100KB
socket.receive.buffer.bytes=102400

# socket服务所能接受的一个请求的最大大小，默认为100M
socket.request.max.bytes=104857600

############################# Log Basics ############################# 
# kafka存储消息数据的目录
log.dirs=/data/kafka/kfkdata

# 每个topic默认的partition数量
num.partitions=3

# 在启动时恢复数据和关闭时刷新数据时每个数据目录的线程数量
num.recovery.threads.per.data.dir=1

############################# Log Flush Policy ############################# 

# 消息刷新到磁盘中的消息条数阈值
#log.flush.interval.messages=10000

# 消息刷新到磁盘中的最大时间间隔,1s
#log.flush.interval.ms=1000

############################# Log Retention Policy ############################# 

# 日志保留小时数，超时会自动删除，默认为7天
log.retention.hours=168

# 日志保留大小，超出大小会自动删除，默认为1G
#log.retention.bytes=1073741824

# 日志分片策略，单个日志文件的大小最大为1G，超出后则创建一个新的日志文件
log.segment.bytes=1073741824

# 每隔多长时间检测数据是否达到删除条件,300s
log.retention.check.interval.ms=300000

############################# Zookeeper ############################# 
# Zookeeper连接信息，如果是zookeeper集群，则以逗号隔开
zookeeper.connect=192.168.254.51,192.168.254.52,192.168.254.53

# 连接zookeeper的超时时间,6s
zookeeper.connection.timeout.ms=6000
```

**修改broker.id**

分别在三个节点依次修改`/data/kafka/config/server.properties`配置文件

```shell
broker.id=1   # master节点
broker.id=2   # node1节点
broker.id=3   # node2节点
listeners=PLAINTEXT://192.168.254.x:9092
```

**启动kafka集群**

```shell
$ cd /data/kafka/bin/
$ ./kafka-server-start.sh ../config/server.properties   #启动测试
$ ./kafka-server-start.sh -daemon ../config/server.properties   #放入后台
```

**配置systemctl启动**

```shell
$ vim /etc/systemd/system/kafka.service

[Unit]
Description=kafka
After=network.target zookeeper.service

[Service]
Type=simple
ExecStart=/data/kafka/bin/kafka-server-start.sh /data/kafka/config/server.properties
ExecStop=/data/kafka/bin/kafka-server-stop.sh
Restart=on-failure
User=root
Group=root

[Install]
WantedBy=multi-user.target
```

**测试**

```shell
# 创建队列
$ ./kafka-topics.sh \
--create \
--zookeeper 192.168.254.51:2181,192.168.254.52:2181,192.168.254.53:2181 \
--topic demo01 \
--partitions 3 \
--replication-factor 1

# 查看队列
$ ./kafka-topics.sh --zookeeper 192.168.254.51:2181,192.168.254.52:2181,192.168.254.53:2181 --list
```

```shell
# 生产消息
$ ./kafka-console-producer.sh \
--broker-list 192.168.254.51:9092,192.168.254.52:9092,192.168.254.53:9092 \
--topic demo01
```

```shell
# 消费消息
$ ./kafka-console-consumer.sh \
--bootstrap-server 192.168.254.51:9092,192.168.254.52:9092,192.168.254.53:9092 \
--topic demo01 \
--from-beginning   # 从开始消费消息
```

## 5.部署elasticsearch

### 1.手动安装

**1.安装es**

https://www.elastic.co/cn/downloads/past-releases#elasticsearch

```shell
$ wget -P /data/ https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.10.1-linux-x86_64.tar.gz
$ tar -xf elasticsearch-7.10.1-linux-x86_64.tar.gz
$ mv elasticsearch-7.10.1 elasticsearch
```

**2.创建用户，因为es不能在root用户下运行**

```shell
$ useradd es
$ chown -R es.es /data/elasticsearch/
```

**3.修改配置文件**

修改主节点配置文件如下:

```shell
$ vim config/elasticsearch.yml
#集群名
cluster.name: es-cluster
#node名
node.name: es01
# 是否有资格被选举为master
node.master: true
# 是否存储数据
node.data: true
#最⼤集群节点数，为了避免脑裂，集群节点数最少为 半数+1
node.max_local_storage_nodes: 3
# 数据目录
path.data: /data/elasticsearch/data
# log目录
path.logs: /data/elasticsearch/logs
#
bootstrap.memory_lock: false
bootstrap.system_call_filter: false
# 修改 network.host 为 0.0.0.0，表示对外开放，如对特定ip开放则改为指定ip
network.host: 0.0.0.0
# 设置对外服务http端口，默认为9200，可更改端口不为9200
http.port: 9200
# 内部节点之间沟通端⼝
transport.tcp.port: 9300
# 写⼊候选主节点的设备地址，在开启服务后可以被选为主节点
discovery.seed_hosts: ["192.168.254.51:9300", "192.168.254.52:9301", "192.168.254.53:9302"]
# 设置集群中N个节点启动时进行数据恢复，默认为1
gateway.recover_after_nodes: 3
# 初始化⼀个新的集群时需要此配置来选举master
cluster.initial_master_nodes: ["es01", "es02", "es03"]
# 下面的两个配置在安装elasticsearch-head的时候会用到
# 开启跨域访问支持，默认为false
http.cors.enabled: true
# 跨域访问允许的域名地址，(允许所有域名)以上使用正则
http.cors.allow-origin: "*"
#关闭xpack
xpack.security.enabled: false
```

其他节点配置文件需要修改node.name和监听端口

**4.启动es**

```shell
$ su - es
$ cd /data/elasticsearch/
$ ./bin/elasticsearch -d    # -d参数在后台启动
```

```shell
# troubleshooting

a.报错：max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]

# 解决办法：
$ vim /etc/sysctl.conf
添加：
vm.max_map_count=262144
```

配置systemctl启动：

```shell
$ vim /etc/systemd/system/elasticsearch.service

[Unit]
Description=elasticsearch
After=network.target

[Service]
Type=simple
User=es
Group=es
ExecStart=/data/elasticsearch/bin/elasticsearch
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

**5.安装head插件（生成环境不建议使用，可以使用google插件）**

```shell
#下载插件
$ cd /data/elasticsearch/plugins
$ git clone https://github.com/mobz/elasticsearch-head.git

#下载nodejs
$ curl --silent --location https://rpm.nodesource.com/setup_10.x | bash -
$ yum install -y nodejs
$ node -v
$ npm -v

#安装grunt
$ npm install -g grunt-cli
$ npm install   #进入elasticsearch-head目录执行，否则会报错找不到package.json文件
```

```shell
#修改head配置 打开elasticsearch-head-master/Gruntfile.js，找到connect属性
$ vim /data/elasticsearch/plugins/elasticsearch-head/Gruntfile.js

server: {
        options: {
                hostname: '192.168.254.51',
                port: 9100,
                base: '.',
                keepalive: true
        }
}
```

```shell
#启动Head插件
$ grunt server
```

访问：http://192.168.254.51:9100

### 2.docker安装

```shell
- 临时安装生产文件：

$ docker run -d --name elasticsearch  -p 9200:9200 -p 9300:9300 -e  "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms256m -Xmx256m" elasticsearch:7.17.1

参数说明:
-d 后台启动
–name 起别名即：NAMES
-p 9200:9200 将端口映射出来
elasticsearch的9200端口是供外部访问使用；9300端口是供内部访问使用集群间通讯
-e "discovery.type=single-node"单节点启动
-e ES_JAVA_OPTS="-Xms256m -Xmx256m" 限制内存大小
```

```shell
- 设置外部数据卷

$ mkdir -p /data/elasticsearch/{config,data,logs,plugins}

将容器文件拷贝出来
$ docker cp elasticsearch:/usr/share/elasticsearch/config /data/elasticsearch
$ docker cp elasticsearch:/usr/share/elasticsearch/logs /data/elasticsearch
$ docker cp elasticsearch:/usr/share/elasticsearch/data /data/elasticsearch
$ docker cp elasticsearch:/usr/share/elasticsearch/plugins /data/elasticsearch

$ vim /data/elasticsearch/config/elasticsearch.yml
cluster.name: "docker-cluster"
network.hosts:0.0.0.0
# 跨域
http.cors.allow-origin: "*"
http.cors.enabled: true
http.cors.allow-headers: Authorization,X-Requested-With,Content-Length,Content-Type
```

```
docker stop elasticsearch
docker rm elasticsearch
```

```shell
$ docker run -d --name elasticsearch \
-p 9200:9200 \
-p 9300:9300 \
-e "discovery.type=single-node" \
-e ES_JAVA_OPTS="-Xms256m -Xmx256m" \
-v /data/elasticsearch/logs:/usr/share/elasticsearch/logs \
-v /data/elasticsearch/data:/usr/share/elasticsearch/data \
-v /data/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
-v /data/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
elasticsearch:7.17.1
```

## 6.安装logstash

### 1.手动安装

首先，安装java环境

```shell
$ wget https://artifacts.elastic.co/downloads/logstash/logstash-7.10.1-x86_64.rpm
$ rpm -ivh logstash-7.10.1-x86_64.rpm

```

```shell
$ vim /etc/logstash/logstash.yml

http.host: "0.0.0.0"
# 指发送到Elasticsearch的批量请求的大小，值越大，处理则通常更高效，但增加了内存开销
pipeline.batch.size: 3000
# 指调整Logstash管道的延迟，过了该时间则logstash开始执行过滤器和输出
pipeline.batch.delay: 200
```

```shell
# 配置文件（filebeat直接发送到logstash）
$ vim /etc/logstash/conf.d/logstash.conf
input {
  beats {
    codec => plain{ charset => "UTF-8" }
    port => 5044
  }
}

filter {
    multiline {
        pattern => "^\d"
        negate => true
        what => "previous"
        }
}

output {
  amazon_es {
    codec => plain{ charset => "UTF-8" }
    hosts => ["aws es地址"]
    region => "ap-northeast-1"
    aws_access_key_id => 'ADKJFOQIOELAKDJFLAJL'
    aws_secret_access_key => 'AKHhGgLjlLJJlkjlJlkjlJLkjlkjlkJLjljLJlkjlkjligtdfhrtyrYTry'
    index => "%{[fields][log_topics]}-%{+YYYY.MM.dd}"
    document_type => "%{[@metadata][type]}"
  }
  stdout {
      codec => json_lines
  }
}

### 配置参数讲解 #####
# pattern: 这个是用来匹配文本的表达式，也可以是grok表达式
# what: 如果pattern匹配成功的话，那么匹配行是归宿于上一个事件，还是归属于下一个事件。
# previous: 归属于上一个事件
# next: 归属于下一个事件
# negate:是否对 pattern 的结果取反
# false: 不取反，是默认值。
# true: 取反。
```

```shell
# 从kafka获取日志
$ vim /etc/logstash/conf.d/logstash.conf

input {                                   # 输入组件
  kafka {                                 # 从kafka消费数据
    bootstrap_servers => ["192.168.254.51:9092,192.168.254.52:9092,192.168.254.53:9092"]
    codec => "json"                       # 数据格式
    #topics => ["nginxinfo"]              # 使用kafka传过来的topic
    topics_pattern => "wsdy-prod-.*"      # 使用正则匹配topic
    consumer_threads => 3                 # 消费线程数量
    decorate_events => true               # 可向事件添加Kafka元数据，比如主题、消息大小的选项，这将向logstash事件中添加一个名为kafka的字段
    auto_offset_reset => "latest"         # 自动重置偏移量到最新的偏移量
    #group_id => "logstash-node"          # 消费组ID，多个有相同group_id的logstash实例为一个消费组
    #client_id => "logstash1"             # 客户端ID
    fetch_max_wait_ms => "1000"           # 指当没有足够的数据立即满足fetch_min_bytes时，服务器在回答fetch请求之前将阻塞的最长时间
  }
}

filter{
   # 当非业务字段时，无traceId则移除
   #if ([message] =~ "traceId=null") {     # 过滤组件，这里只是展示，无实际意义，根据自己的业务需求进行过滤
   #   drop {}
   #}
mutate {
    convert => ["Request time", "float"]
    }
        if [ip] != "-" {
        geoip {
                       source => "ip"
                        target => "geoip"
                       # database => "/usr/share/GeoIP/GeoIPCity.dat"
                        add_field => [ "[geoip][coordinates]", "%{[geoip][longitude]}" ]
                        add_field => [ "[geoip][coordinates]", "%{[geoip][latitude]}"  ]
                }
                   mutate {
                        convert => [ "[geoip][coordinates]", "float"]
                }
        }
}


output {            # 输出组件
  elasticsearch {   # Logstash输出到es
    hosts => ["192.168.254.51:9200","192.168.254.52:9201","192.168.254.53:9202"]
    index => "wsdy-prod-%{[fields][log_topics]}-%{+YYYY-MM-dd}"  # 直接在日志中匹配
    #index => "%{[@metadata][topic]}-%{+YYYY-MM-dd}"  # 以日期建索引
    #user => "elastic"
    #password => "123"
  }
  #stdout {
  #    codec => rubydebug
 #}
}

```

配置hosts：

```shell
$ vim /etc/hosts
192.168.254.51 elk-master
192.168.254.52 elk-node1
192.168.254.53 elk-node2
# 不配置hosts,logstash无法消费kafka消息
```

### 2.docker安装

```shell
$ docker run -d --name=logstash logstash:7.14.0

# 进入容器安装插件
$ docker exec -it 容器ID /bin/bash
bash-4.2$ bin/logstash-plugin install logstash-output-stomp
bash-4.2$ bin/logstash-plugin install logstash-filter-multiline

$ docker cp logstash:/usr/share/logstash /data/logstash       # 将目标拷贝到本地
$ chmod 777 -R /data/logstash/
$ mkdir /data/logstash/config/conf.d
```

```shell
$ vim /data/logstash/config/conf.d/logstash.conf
.....
```

```shell
# 配置conf
$ vim /data/logstash/config/pipelines.yml
- pipeline.id: main
  path.config: "/usr/share/logstash/config/conf.d/logstash.conf"   

# path也是在docker里的绝对路径

# 配置java
$ vim /data/logstash/config/jvm.options
-Xms512m
-Xmx512m
```

```shell
$ vim /data/logstash/config/logstash.yml
http.host: "0.0.0.0"
#xpack.monitoring.elasticsearch.hosts: [ "http://elasticsearch:9200" ]   
将连接es这行注释
```

```shell
# 启动
$ docker run -d --name=logstash -v /data/logstash:/usr/share/logstash -p 5044:5044 logstash:7.14.0
```

## 7.安装kibana

https://www.elastic.co/cn/downloads/past-releases#kibana  安装包地址

```shell
$ wget -P /data https://artifacts.elastic.co/downloads/kibana/kibana-7.10.1-linux-x86_64.tar.gz
$ tar -xf kibana-7.10.1-linux-x86_64.tar.gz
$ mv kibana-7.10.1-linux-x86_64 kibana

```

```shell
$ vim config/kibana.yml
server.port: 5601    #监听端口
server.host: "192.168.254.60"    #监听地址
elasticsearch.hosts: ["http://192.168.254.51:9200","http://xxxx:9200"] #elasticsearch服务器地址
i18n.locale: "zh-CN"    #修改为中文
```

启动

```shell
$ cd /data/kibana/bin
$ ./kibana --allow-root
# Kibana可以使用root来运行，只是必须明确指明--allow-root来运行，这是kibana的安全保护机制，所以我们尽可能的不要使用root来启动kibana
```

**配置systemctl启动：**

```shell
# Kibana与 ElasticSearch一样，不推荐使用root用户启动，所以我们可以创建一个专门的启动账户
$ useradd kibana
$ passwd kibana
$ chown -R kibana.kibana kibana
```

```shell
$ vim /etc/systemd/system/kibana.service
[Unit]
Description=kibana
After=network.target

[Service]
Type=simple
User=kibana
Group=kibana
ExecStart=/data/kibana/bin/kibana
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

访问：http://192.168.254.60:5601

## 8.安装filebeat

ubuntu：

```shell
# ubuntu禁用ipv6，否则可能会出现下一步wget卡住
$ vim /etc/sysctl.conf
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
$ sysctl -p    #生效配置

$ wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
$ apt-get install apt-transport-https
$ echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
$ apt-get update
$ apt install filebeat
```

输出到kafka集群：

```shell
$ vim /etc/filebeat/filebeat.yml

filebeat.inputs:

- type: log
  enabled: true
  paths:
    - /var/log/nginx/access.log
  fields:
    log_topics: nginx-info
- type: log
  enabled: true
  paths:
    - /var/log/nginx/error.log
  fields:
    log_topics: nginx-error

filebeat.config.modules:
  path: ${path.config}/modules.d/*.yml

output.kafka:
  enabled: true
  hosts: ["192.168.254.51:9092","192.168.254.52:9092","192.168.254.53:9092"]
  topic: 'wsdy-prod-%{[fields.log_topics]}'    # 发送到kafka时的主题名
  partition.hash:
    reachable_only: true
  compression: gzip
  max_message_bytes: 1000000
  required_acks: 1
# logging.to_files: true

# filebeat日志输出位置
logging.level: info 日志级别
logging.to_files: true
logging.files:
path: /var/log/filebeat
name: filebeat.log
keepfiles: 7         # 保留7天
permissions: 0644
```

```shell
$ vim /etc/hosts
192.168.254.51 elk-master
192.168.254.52 elk-node1
192.168.254.53 elk-node2
# 不配置hosts，会在kafka创建主题，但是无法消费；原因是kafka配置文件绑定了主机名。
```

## 9.es基本操作

**1.查询**

- 服务器端：

```shell
$ curl -XGET http://192.168.254.51:9200/_cat/indices?pretty
```

- dev tools：

```shell
# 查看集群健康状态
GET _cluster/health?pretty
{
  "cluster_name" : "es-cluster",
  "status" : "green",     #集群状态
  "timed_out" : false,
  "number_of_nodes" : 3,     #集群的节点数
  "number_of_data_nodes" : 3,    #集群的数据节点数
  "active_primary_shards" : 7,    #集群中所有活跃的主分片数
  "active_shards" : 14,       #集群中所有活跃的分片数
  "relocating_shards" : 0,
  "initializing_shards" : 0,    #正在初始化的分片
  "unassigned_shards" : 0,    #未分配的分片数，通常为0，当有某个节点的副本分片丢失该值就会增加
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 100.0   #集群分片健康度，活跃分片数占总分片数比例
}
```

```shell
# 集群状态信息
GET _cluster/stats?pretty
...
# indices.count: 索引总数
# indices.shards.total: 总分片数
# indices.shards.primaries: 主分片数
# nodes.count.total: 总节点数
# nodes.count.data: 数据节点数
# nodes.process.cpu.percent: 节点CPU使用率

```

```shell
# 查看节点指标
GET /_nodes/stats?pretty
```

```shell
# 查看所有索引
GET _cat/indices?pretty
# 查看单个索引
GET logstash-nginxinfo-2022-10-12/_settings
```

```shell
# 查看分片，模糊匹配
GET _cat/shards/logstash-nginxinfo*?v
# 查看状态为unassigned的es分片信息
GET _cat/shards | grep UNASSIGNED
```

**2.集群黄色处理**

```shell
# 查看异常原因
GET /_cluster/allocation/explain
```

