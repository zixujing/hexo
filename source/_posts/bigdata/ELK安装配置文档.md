---
title: ELK安装配置文档
date: 2018-12-26 00:00:00
tags: 
- elasticsearch
- logstash
categories: 
- tool
---
> ELK 安装配置文档
<!-- more -->
<!-- toc -->
# elasticsearch 安装
## 环境配置
### linux环境配置
#### 创建elk用户并设置elk用户的密码
>elasticsearch默认不能在root用户下运行，所以需新创建一个elk用户，具体操作如下。
```bash
$ groupadd elk
$ useradd -g elk -m elk
$ passwd elk
```
#### 设置linux环境参数

1.	修改limits.conf配置文件
>limits.conf是会话级别，退出会话重新登录需改即可生效。
```bash
$ vim /etc/security/limits.conf
 * soft nofile 65536
 * hard nofile 131072
 * soft nproc 2048
 * hard nproc 4096
```
2.	修改20-nproc.conf配置文件
>某些版本的linux的max user processes配置来源是20-nproc.conf，还需修改20-nproc.conf（centos6 是90-nproc.conf）。
```bash
$ vim /etc/security/limits.d/20-nproc.conf
* soft nproc 4096
```
3.	设置sysctl.conf配置文件
```bash
$ vim /etc/sysctl.conf
vm.max_map_count=655360
$ sysctl -p
```
### java环境配置
>elasticsearch 6.5.*需要jdk版本1.8，如果系统中没有jdk或者是jdk版本不符合要求，那么需要安装jdk，以下是在elk用户下安装jdk的示例。
```bash
$ #以下是elk用户下
$ tar -xzvf jdk-8u192-linux-x64.tar.gz 
$ mv jdk1.8.0_192/ jdk
$ vi ~/.bash_profile
export JAVA_HOME=/home/elk/soft/jdk
PATH=$PATH:$HOME/.local/bin:$HOME/bin:/home/elk/soft/jdk/bin
$ source .bash_profile
```

## elasticsearch安装
```bash
$ tar -xzvf elasticsearch-6.5.2.tar.gz
$ cd elasticsearch-6.5.2/config
$ vi elasticsearch.yml 
cluster.name: log-dig
node.name: node1
#elasticsearch数据的存储路径
path.data: /xxx/data
#elasticsearch日志路径
path.logs: /xxx/logs
#绑定监听IP
network.host: 192.168.0.xxx
#设置对外服务的http端口,默认为9200
http.port: 9200
$ cd ..
$ bin/elasticsearch -d
$ #测试安装是否成功
$ curl 192.168.0.xxx:9200
{
  "name" : "node1",
  "cluster_name" : "log-dig",
  "cluster_uuid" : "EFpju3lYSeOL3Vs5bLM81A",
  "version" : {
    "number" : "6.5.3",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "159a78a",
    "build_date" : "2018-12-06T20:11:28.826501Z",
    "build_snapshot" : false,
    "lucene_version" : "7.5.0",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
  },
  "tagline" : "You Know, for Search"
}
$ #启动elasticsearch
$ bin/elasticsearch -d
$ #停止elasticsearch
$ jps
28410 Elasticsearch
$ kill -15 28410
```
***
# kibana安装
```bash
$ tar -xzvf kibana-6.5.2-linux-x86_64.tar.gz
$ cd kibana-6.5.3-linux-x86_64/
$ vi config/kibana.yml 
server.name: "namefordisplay"
#kinaba绑定ip设置
server.host: "192.168.0.xxx"
server.port: 5601
elasticsearch.url: "http://192.168.0.xxx:9200"
$ screen bin/kibana -c config/kibana.yml
CTRL+a d
$ #停止kibana
$ ps -ef | grep node
els        9514     1  1 Dec21 ?        01:30:02 ./../node/bin/node --no-warnings ./../src/cli
els       14516  9514  0 Dec21 ?        00:00:13 /home/es/kibana-6.5.3-linux-x86_64/node/bin/node --no-warnings /home/es/kibana-6.5.3-linux-x86_64/node_modules/x-pack/plugins/canvas/server/lib/route_expression/thread/babeled.js
$ kill -15 9514
```
>验证kibana是否安装成功，http://192.168.0.xxx:5601
<center>![Alt text](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/1545789425109.png)</center>

***
# kibana汉化
>Kibana_Hanization汉化需要python2.7环境，一般安装linux都会自带python2.7环境
```bash
$ unzip Kibana_Hanization-master.zip
$ #执行下面命令汉化kibana
$ cd Kibana_Hanization-master/
$ python main.py /path_for_kibana/kibana-6.5.2-linux-x86_64/
```
***
# kafka的搭建
## kafka安装
```bash
$ tar -xvzf kafka_2.11-2.0.0.tar.gz -C /usr/local/elk/
$ cd /usr/local/elk/kafka_2.11-2.0.0/config 
$ vim zookeeper.properties
# zookeeper的data路径
dataDir=/usr/local/elk/kafka_2.11-2.0.0/zookeeper/data
# zookeeper的log路径
dataLogDir=/usr/local/elk/kafka_2.11-2.0.0/zookeeper/logs
# zookeeper的端口号
clientPort=2181
# zookeeper的客户端连接数，设置成0为无限制连接。
maxClientCnxns=0
$ #配置server.properties文件，修改该配置文件中的以下参数：
$ vim server.properties
#服务端口，默认9092
port=9092
#监听地址
host.name=192.168.106.xxx
#日志文件存储路径
log.dirs=/usr/local/elk/kafka_2.11-2.0.0/kafka/logs
#Zookeeper quorum设置，如果有多个使用逗号分割。
zookeeper.connect=192.168.106.226:2181
$ #启动zookeeper
$ cd /usr/local/elk/kafka_2.11-2.0.0
$ bin/zookeeper-server-start.sh  config/zookeeper.properties 1>/dev/null 2>&1 &
$ #启动kafka
$ cd /usr/local/elk/kafka_2.11-2.0.0
$ bin/kafka-server-start.sh  config/server.properties 1>/dev/null 2>&1 &
$ #停止zookeeper
$ cd /usr/local/elk/kafka_2.11-2.0.0
$ bin/zookeeper-server-stop.sh  config/zookeeper.properties
$ #停止kafka
$ cd /usr/local/elk/kafka_2.11-2.0.0
$ bin/kafka-server-stop.sh  config/server.properties
```
## kafka测试
```bash
$ #执行下面的命令创建一个topic，名称为“testTopic”：
$ cd /usr/local/elk/kafka_2.11-2.0.0
$ bin/kafka-topics.sh --create --zookeeper 192.168.0.xxx:2181 --replication-factor 1 --partitions 1 --topic testTopic
$ #执行下面的命令查看topic：
$ bin/kafka-topics.sh --list --zookeeper 192.168.0.xxx:2181
$ #执行下面的命令测试生产消息
$ bin/kafka-console-producer.sh --broker-list 192.168.0.xxx:9092 --topic testTopic
this is test
$ #执行下面的命令测试消费消息
$ bin/kafka-console-consumer.sh --bootstrap-server 192.168.0.xxx:9092 --topic testTopic --from-beginning
```
***
# logstash安装
```bash
$ tar -xvzf logstash-6.5.2.tar.gz -C /usr/local/elk/
$ cd /usr/local/elk/logstash-6.5.2/config
$ vim logstash.conf
input {
        kafka {
                codec => "json"
                topics_pattern => "government_.*"
                bootstrap_servers => "192.168.0.xxx:9092"
                auto_offset_reset => "latest"
                group_id => "government"
               }
     }
output {
        #file {
        #       path => "/home/soft/elk/logstash-6.5.3/logstash.output"
        #     }
        elasticsearch {
                        hosts =>["192.168.0.xxx:9200"]
                        index => "%{[fields][log_topics]}"
                      }
        #stdout { codec => rubydebug }
      }
$ #后台启动logstash
$ nohup bin/logstash -f /usr/local/elk/logstash-6.5.2/configlogstash.conf &
$ #停止logstash
$ jps
30997 Jps
32026 Logstash
$ kill -15 32026 
```
***
# filebeat安装
```bash
$ tar -xvzf filebeat-6.5.3-linux-x86_64.tar.gz -C /usr/local/elk/
$ cd /usr/local/elk/filebeat-6.5.3-linux-x86_64
$ vim filebeat.yml
filebeat.prospectors:
#government_debug日志
- type: log
  enabled: true
  paths:
    - /usr/local/apache-tomcat-8.5.31/government_sllogs/debug.log
  fields:
    log_topics: government_debug

#government_info日志
- type: log
  enabled: true
  paths:
    - /usr/local/apache-tomcat-8.5.31/government_sllogs/info.log
  fields:
    log_topics: government_info

#government_warn日志
- type: log
  enabled: true
  paths:
    - /usr/local/apache-tomcat-8.5.31/government_sllogs/warn.log
  fields:
    log_topics: government_warn

#government_error日志
- type: log
  enabled: true
  paths:
- /usr/local/apache-tomcat-8.5.31/government_sllogs/error.log
  fields:
    log_topics: government_error
#多行日志合并
  multiline:
       pattern: '^\d{4}-\d{2}-\d{2}\s{1,5}[0-2][0-9]:[0-5][0-9]:[0-5][0-9]'
       negate:  true
       match:   after
       max_lines: 500
       timeout: 10s
  tail_files: true
#日志输出到kafka
output.kafka:
  enabled: true
  hosts: ["192.168.0.xxx:9092"]
  topic: '%{[fields][log_topics]}'
$ #启动filebeat
$ cd /usr/local/elk/filebeat-6.5.3-linux-x86_64
$ nohup ./filebeat -e -c filebeat.yml > filebeat.log &
$ #停止filebeat
$ ps -ef |grep filebeat
$ kill -15 pid
```
***
# logstash 配置示例
```dsconfig
input {
   #filebeat配置
   #filebeats配置
   beats {
      port => 5044
   }
   #tcp网络数据传输，回车是一行
   tcp {
	  port => 5045
	  type => "tcp"
   }
   #文件读取配置
   file {
        path => "/var/log/messages"
        type => "system"
        start_position => "beginning"
    }   
    file {
        path => "/var/log/secure"
        type => "secure"
        start_position => "beginning"
    }   
    file {
        path => "/var/log/httpd/access_log"
        type => "http"
        start_position => "beginning"
    }   
    file {
        path => "/usr/local/nginx/logs/elk.access.log"
        type => "nginx"
        start_position => "beginning"
    } 
}
filter {
   #如果字段logIndex的值为"nginx"
   if [fields][logIndex] == "nginx" {
      grok {
         match => {
            #grok 正则匹配
            grok {
		      "message" => "%{IPORHOST:addre} - - \[%{HTTPDATE:log_timestamp}\] \"%{WORD:verb} %{URIPATHPARAM:request} HTTP/%{NUMBER:httpversion}\" %{INT:status} %{INT:body_bytes_sent} \"%{GREEDYDATA:http_referrer}\" \"%{GREEDYDATA:message}\""
         }
      }
      urldecode {
         charset => "UTF-8"
         field => "url"
      }
      if [upstreamtime] == "" or [upstreamtime] == "null" {
         mutate {
            update => { "upstreamtime" => "0" }
         }
      }
      date {
         match => ["logtime", "dd/MMM/yyyy:HH:mm:ss Z"]
         target => "@timestamp"
      }
      mutate {
         convert => { 
            "responsetime" => "float"
            "upstreamtime" => "float"
            "size" => "integer"
         } 
         remove_field  => ["port","logtime","message"]
      }
   
   }
}
output {
  if [type] == "system" { 

        elasticsearch {
            hosts => ["192.168.1.202:9200"]
            index => "nagios-system-%{+YYYY.MM.dd}"
        }       
    }   
    if [type] == "secure" {
        elasticsearch {
            hosts => ["192.168.1.202:9200"]
            index => "nagios-secure-%{+YYYY.MM.dd}"
        }
    }
    if [type] == "http" {

        elasticsearch {
            hosts => ["192.168.1.202:9200"]
            index => "nagios-http-%{+YYYY.MM.dd}"
        }
    }
    if [type] == "nginx" {

        elasticsearch {
            hosts => ["192.168.1.202:9200"]
            index => "nagios-nginx-%{+YYYY.MM.dd}"
        }
    }
}
```
# logstash jdbc配置示例

```
input{  
    jdbc{  
        jdbc_connection_string => "jdbc:oracle:thin:@//192.168.106.174:1521/orcl"
        jdbc_user => "smartecap_data"
        jdbc_password => "123456"
        jdbc_driver_library => "/home/es/logstash-6.5.3/lib/ojdbc6-11.2.0.3.jar"
        jdbc_driver_class => "Java::oracle.jdbc.driver.OracleDriver"
        jdbc_paging_enabled => "true"
        jdbc_page_size => "5000000"
        lowercase_column_names => "false"
        statement => "select * from DT_STUDENTINFO"
        schedule => "* * * * *"
        type => "DT_STUDENTINFO"
        use_column_value => "true"
        tracking_column => "ID" #oracle字段默认大写
    }
}
filter {
  mutate {
    remove_field => [ "@timestamp","@version"]
  }
}
output{  
    elasticsearch{  
        hosts => "192.168.106.213:9200"
        index => "dt_studentinfo"
        document_id => "%{ID}"
    }  
} 
```
# elasticsearch 集群配置
>elasticsearch.yml文件配置示例
```roboconf
# ======================== Elasticsearch Configuration =========================
#
# NOTE: Elasticsearch comes with reasonable defaults for most settings.
#       Before you set out to tweak and tune the configuration, make sure you
#       understand what are you trying to accomplish and the consequences.
#
# The primary way of configuring a node is via this file. This template lists
# the most important settings you may want to configure for a production cluster.
#
# Please consult the documentation for further information on configuration options:
# https://www.elastic.co/guide/en/elasticsearch/reference/index.html
#
# ---------------------------------- Cluster -----------------------------------
#
# Use a descriptive name for your cluster:
#
cluster.name: logdig
#
# ------------------------------------ Node ------------------------------------
#
# Use a descriptive name for the node:
#
node.name: node3
#
# Add custom attributes to the node:
#
#node.attr.rack: r1
#
# ----------------------------------- Paths ------------------------------------
#
# Path to directory where to store the data (separate multiple locations by comma):
#
#path.data: /path/to/data
#
# Path to log files:
#
#path.logs: /path/to/logs
#
# ----------------------------------- Memory -----------------------------------
#
# Lock the memory on startup:
#
bootstrap.memory_lock: false
bootstrap.system_call_filter: false
#
# Make sure that the heap size is set to about half the memory available
# on the system and that the owner of the process is allowed to use this
# limit.
#
# Elasticsearch performs poorly when the system is swapping the memory.
#
# ---------------------------------- Network -----------------------------------
#
# Set the bind address to a specific IP (IPv4 or IPv6):
#
network.host: 192.168.106.178
#
# Set a custom port for HTTP:
#
#http.port: 9200
#
# For more information, consult the network module documentation.
#
# --------------------------------- Discovery ----------------------------------
#
# Pass an initial list of hosts to perform discovery when new node is started:
# The default list of hosts is ["127.0.0.1", "[::1]"]
#
discovery.zen.ping.unicast.hosts: ["192.168.106.xxx", "192.168.106.yyy","192.168.106.zzz"]
#
# Prevent the "split brain" by configuring the majority of nodes (total number of master-eligible nodes / 2 + 1):
#
discovery.zen.minimum_master_nodes: 2
#
# For more information, consult the zen discovery module documentation.
#
# ---------------------------------- Gateway -----------------------------------
#
# Block initial recovery after a full cluster restart until N nodes are started:
#
#gateway.recover_after_nodes: 3
#
# For more information, consult the gateway module documentation.
#
# ---------------------------------- Various -----------------------------------
#
# Require explicit names when deleting indices:
#
#action.destructive_requires_name: true
                                                                                              
```
>在这3台服务器分别启动，命令如下：
```bash
$ bin/elasticsearch -d
```
# elasticsearch使用示例【集群模式下】
```bash
$ curl 192.168.106.xxx:9200/_cat/shards
$ curl 192.168.106.xxx:9200/_cluster/stats?pretty
{
  "_nodes" : {
    "total" : 3,
    "successful" : 3,
    "failed" : 0
  },
  "cluster_name" : "logdig",
  "cluster_uuid" : "EFpju3lYSeOL3Vs5bLM81A",
  "timestamp" : 1545803198329,
  "status" : "green",
  "indices" : {
    "count" : 15,
    "shards" : {
      "total" : 36,
      "primaries" : 23,
      "replication" : 0.5652173913043478,
      "index" : {
        "shards" : {
          "min" : 2,
          "max" : 5,
          "avg" : 2.4
        },
        "primaries" : {
          "min" : 1,
          "max" : 5,
          "avg" : 1.5333333333333334
        },
        "replication" : {
          "min" : 0.0,
          "max" : 1.0,
          "avg" : 0.8666666666666667
        }
      }
    },
    "docs" : {
      "count" : 991540,
      "deleted" : 634
    },
    "store" : {
      "size_in_bytes" : 594606668
    },
    "fielddata" : {
      "memory_size_in_bytes" : 1920,
      "evictions" : 0
    },
    "query_cache" : {
      "memory_size_in_bytes" : 0,
      "total_count" : 0,
      "hit_count" : 0,
      "miss_count" : 0,
      "cache_size" : 0,
      "cache_count" : 0,
      "evictions" : 0
    },
    "completion" : {
      "size_in_bytes" : 0
    },
    "segments" : {
      "count" : 232,
      "memory_in_bytes" : 2724556,
      "terms_memory_in_bytes" : 1517004,
      "stored_fields_memory_in_bytes" : 217952,
      "term_vectors_memory_in_bytes" : 0,
      "norms_memory_in_bytes" : 22784,
      "points_memory_in_bytes" : 308688,
      "doc_values_memory_in_bytes" : 658128,
      "index_writer_memory_in_bytes" : 0,
      "version_map_memory_in_bytes" : 0,
      "fixed_bit_set_memory_in_bytes" : 147152,
      "max_unsafe_auto_id_timestamp" : 1545800638659,
      "file_sizes" : { }
    }
  },
  "nodes" : {
    "count" : {
      "total" : 3,
      "data" : 3,
      "coordinating_only" : 0,
      "master" : 3,
      "ingest" : 3
    },
    "versions" : [
      "6.5.3",
      "6.5.4"
    ],
    "os" : {
      "available_processors" : 4,
      "allocated_processors" : 4,
      "names" : [
        {
          "name" : "Linux",
          "count" : 3
        }
      ],
      "mem" : {
        "total_in_bytes" : 5866811392,
        "free_in_bytes" : 209567744,
        "used_in_bytes" : 5657243648,
        "free_percent" : 4,
        "used_percent" : 96
      }
    },
    "process" : {
      "cpu" : {
        "percent" : 18
      },
      "open_file_descriptors" : {
        "min" : 252,
        "max" : 331,
        "avg" : 301
      }
    },
    "jvm" : {
      "max_uptime_in_millis" : 3270812,
      "versions" : [
        {
          "version" : "1.8.0_191",
          "vm_name" : "Java HotSpot(TM) 64-Bit Server VM",
          "vm_version" : "25.191-b12",
          "vm_vendor" : "Oracle Corporation",
          "count" : 1
        },
        {
          "version" : "1.8.0_161",
          "vm_name" : "Java HotSpot(TM) 64-Bit Server VM",
          "vm_version" : "25.161-b12",
          "vm_vendor" : "Oracle Corporation",
          "count" : 1
        },
        {
          "version" : "1.8.0_171",
          "vm_name" : "Java HotSpot(TM) 64-Bit Server VM",
          "vm_version" : "25.171-b11",
          "vm_vendor" : "Oracle Corporation",
          "count" : 1
        }
      ],
      "mem" : {
        "heap_used_in_bytes" : 925867600,
        "heap_max_in_bytes" : 3186360320
      },
      "threads" : 115
    },
    "fs" : {
      "total_in_bytes" : 127439097856,
      "free_in_bytes" : 37985562624,
      "available_in_bytes" : 33092489216
    },
    "plugins" : [ ],
    "network_types" : {
      "transport_types" : {
        "security4" : 3
      },
      "http_types" : {
        "security4" : 3
      }
    }
  }
}
$ curl 192.168.106.213:9200/_cluster/settings?pretty
{
  "persistent" : {
    "xpack" : {
      "monitoring" : {
        "collection" : {
          "enabled" : "true"
        }
      }
    }
  },
  "transient" : { }
}
```
