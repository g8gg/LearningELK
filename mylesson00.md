
# 实战解决问题

## 问题描述和解决步骤

1. 有一个应用程序结果的log，记录了每个接口的运行耗时，格式如下：
> ```
2015-12-11 00:13:10  [ resin-port-18057-105:26190014 ] - [ INFO ]  [url] http://api.foobar.com/contacts/findUserContact.do 30\
2015-12-11 00:13:10  [ resin-port-18057-50:26190035 ] - [ INFO ]  [url] http://api.foobar.com/sys/config.do 12\
2015-12-11 00:13:10  [ resin-port-18057-48:26190040 ] - [ INFO ]  [url] http://api.foobar.com/user/updateDeviceStoken.do 21\
> ```

2. 要求，根据url和日期，分析出接口消耗时间(duration，下文用oo)

3. 首先是要在logstash里面正确的解析，并且送入到**ES**，所以我学习了logstash，并且送到了屏幕和本地的elasticsearch服务
> ```
gosber@localhost  ~/logstash  cat logstash.conf
input {
    file {
        type => logfile
        path => ["/Users/gosber/logstash/sp.log"]
        start_position => "beginning"
    }
}
filter {
    grok {
      match => { "message" => "%{TIMESTAMP_ISO8601:dt} %{GREEDYDATA:xx} %{URI} %{INT:oo}"}
    }
}
output {
   elasticsearch { hosts => ["127.0.0.1:9200"]}
   stdout { codec => rubydebug }
}
> ```


4. 查看索引
5. 查询数据




## 相关心得：List很重要

> 今天排查问题的时候，欲仙欲死，因为太不熟悉ELK，而ELK Stack书上写的太简单直接（根本不是为新手准备的T_T）
> 好吧，哥也不是吃素的，在这个过程当中，甚至搞定了grok，搞定了python的例子，还鼓捣出乱七八糟各种新手必备工具
> 记录一下吧，各种血泪史

* List All Indexes
```
gosber@localhost  ~/logstash  curl 'localhost:9200/_cat/indices?v'
health status index               pri rep docs.count docs.deleted store.size pri.store.size
yellow open   logstash-2015.12.13   5   1          4            0     12.7kb         12.7kb
yellow open   test-index            5   1          1            0      4.2kb          4.2kb
yellow open   logstash-2015.12.14   5   1       2106            0    696.6kb        696.6kb
```

* List All 
```
 gosber@localhost  ~/logstash  curl 'localhost:9200/_cat/'
=^.^=
/_cat/allocation
/_cat/shards
/_cat/shards/{index}
/_cat/master
/_cat/nodes
/_cat/indices
/_cat/indices/{index}
/_cat/segments
/_cat/segments/{index}
/_cat/count
/_cat/count/{index}
/_cat/recovery
/_cat/recovery/{index}
/_cat/health
/_cat/pending_tasks
/_cat/aliases
/_cat/aliases/{alias}
/_cat/thread_pool
/_cat/plugins
/_cat/fielddata
/_cat/fielddata/{fields}
/_cat/nodeattrs
/_cat/repositories
/_cat/snapshots/{repository}
```
* 查询语法也很重要
```
 gosber@localhost  ~/logstash  curl -XPOST 'localhost:9200/logstash-2015.12.14/_search?q=oo:64'
{
  "took":6,
  "timed_out":false,
  "_shards":{
    "total":5,
    "successful":5,
    "failed":0
  },
  "hits":{
    "total":4,
    "max_score":6.395898,
    "hits":[
      {
        "_index":"logstash-2015.12.14",
        "_type":"logfile",
        "_id":"AVGfkBjdeiDqXpB3yqZ0",
        "_score":6.395898,
        "_source":{
          "message":"2015-12-11 00:12:02  [ resin-port-18057-105:26122017 ] - [ INFO ]  [url] http://api.foobar.com/message/readMessageDetail.do 64\\",
          "@version":"1",
          "@timestamp":"2015-12-14T08:16:38.844Z",
          "host":"localhost",
          "path":"/Users/gosber/logstash/sp.log",
          "type":"logfile",
          "dt":"2015-12-11 00:12:02",
          "xx":" [ resin-port-18057-105:26122017 ] - [ INFO ]  [url]",
          "oo":"64"
        }
      },
      {
        "_index":"logstash-2015.12.14",
        "_type":"logfile",
        "_id":"AVGfkB00eiDqXpB3yqjH",
        "_score":6.315666,
        "_source":{
          "message":"2015-12-11 00:12:42  [ resin-port-18057-8196:26162552 ] - [ INFO ]  [url] http://api.foobar.com/message/sendMessage.do 64\\",
          "@version":"1",
          "@timestamp":"2015-12-14T08:16:40.057Z",
          "host":"localhost",
          "path":"/Users/gosber/logstash/sp.log",
          "type":"logfile",
          "dt":"2015-12-11 00:12:42",
          "xx":" [ resin-port-18057-8196:26162552 ] - [ INFO ]  [url]",
          "oo":"64"
        }
      },
      {
        "_index":"logstash-2015.12.14",
        "_type":"logfile",
        "_id":"AVGfkBULeiDqXpB3yqSs",
        "_score":6.295814,
        "_source":{
          "message":"2015-12-11 00:12:42  [ resin-port-18057-8196:26162552 ] - [ INFO ]  [url] http://api.foobar.com/message/sendMessage.do 64\\",
          "@version":"1",
          "@timestamp":"2015-12-14T08:16:38.042Z",
          "host":"localhost",
          "path":"/Users/gosber/logstash/sp.log",
          "type":"logfile",
          "dt":"2015-12-11 00:12:42",
          "xx":" [ resin-port-18057-8196:26162552 ] - [ INFO ]  [url]",
          "oo":"64"
        }
      },
      {
        "_index":"logstash-2015.12.14",
        "_type":"logfile",
        "_id":"AVGfkBAbeiDqXpB3yqJi",
        "_score":6.2857385,
        "_source":{
          "message":"2015-12-11 00:12:02  [ resin-port-18057-105:26122017 ] - [ INFO ]  [url] http://api.foobar.com/message/readMessageDetail.do 64\\",
          "@version":"1",
          "@timestamp":"2015-12-14T08:16:35.681Z",
          "host":"localhost",
          "path":"/Users/gosber/logstash/sp.log",
          "type":"logfile",
          "dt":"2015-12-11 00:12:02",
          "xx":" [ resin-port-18057-105:26122017 ] - [ INFO ]  [url]",
          "oo":"64"
        }
      }
    ]
  }
}
```
> 好吧，结果没有那么美丽，我格式化了一下JSON，使用的是在线工具[jsonformatter](https://jsonformatter.curiousconcept.com/)

