# 初窥ELK全貌
**按照我现在的学习进度，虽然使用Python可以访问并使用ES**
**但是还远远不能满足需求，所以本文只记录使用REST API的方式**

> 继续上一篇的内容，还有一些问题没有解决呢！难道你不会有疑问吗？
 1. ES服务中的Document是可以Schema free的，但是聚合函数是需要数值类型的→Mapping
 2. Logstash如何知道如何放入正确的索引和Mapping呢？
 3. GROK是什么？
 4. 报表？
 
 **从流程和思路谈起**
 * 首先，ES中应该创建的是索引，对吧？
 * 其次，有了索引，我们应该描述我们即将要存入的文档。
 * 文档作为_source，肯定是存入并被索引的，如何描述文档中我们后期需要使用的对象呢？
 * 有了索引，对象，如何让logstash解析到正确的结果，并触发mapping呢？
 * 最后，如何验证数据，并且开始制作报表呢？
 
**带着这些疑问，我开始痛苦的思考和学习，终于解决了这些问题**

* 1.准备工作
```bash
gosber@localhost  ~  elasticsearch --version
Version: 2.1.0, Build: 72cd1f1/2015-11-18T22:40:03Z, JVM: 1.8.0_51
gosber@localhost  ~  logstash --version
logstash 2.1.1
gosber@localhost  ~  kibana --version
4.3.0
```
* 2.创建索引
```bash
gosber@freedamadeMacBook-Pro  ~/logstash  curl -XPUT 'localhost:9200/api_duration'
{"acknowledged":true}%
```
* 3.创建mapping，注意！我这里索引和文档中对象是重名的，就是这么任性！
```bash
gosber@localhost  ~/logstash  cat splog_mappings.json
{
  "api_duration":{
    "_all": {"enabled":true},
    "properties":{
      "datetime":{"type":"date","store":"yes","index":"not_analyzed","format":"yyyy-MM-dd HH:mm:ss"},
      "api":{"type":"string","store":"yes","index":"analyzed"},
      "duration":{"type":"long","store":"yes","index":"not_analyzed"}
    }
  }
}
```
* 4.检查验收索引和mapping
```bash
gosber@freedamadeMacBook-Pro  ~/logstash  curl -XGET 'localhost:9200/api_duration/?pretty'
{
  "api_duration" : {
    "aliases" : { },
    "mappings" : {
      "api_duration" : {
        "_all" : {
          "enabled" : true
        },
        "properties" : {
          "@timestamp" : {
            "type" : "date",
            "format" : "strict_date_optional_time||epoch_millis"
          },
          "@version" : {
            "type" : "string"
          },
          "api" : {
            "type" : "string",
            "store" : true
          },
          "datetime" : {
            "type" : "date",
            "store" : true,
            "format" : "yyyy-MM-dd HH:mm:ss"
          },
          "duration" : {
            "type" : "long",
            "store" : true
          },
          "host" : {
            "type" : "string"
          },
          "message" : {
            "type" : "string"
          },
          "path" : {
            "type" : "string"
          },
          "tags" : {
            "type" : "string"
          },
          "type" : {
            "type" : "string"
          },
          "xx" : {
            "type" : "string"
          }
        }
      }
    },
    "settings" : {
      "index" : {
        "creation_date" : "1450425286213",
        "number_of_shards" : "5",
        "number_of_replicas" : "1",
        "uuid" : "6JEMjYpsSzKCua53vuKy7A",
        "version" : {
          "created" : "2010099"
        }
      }
    },
    "warmers" : { }
  }
}
```
* 5.logstash的两个问题，解析文本，输出到ES服务
```bash
 gosber@localhost  ~/logstash  cat logstash.conf
input {
    file {
        type => "api_duration"
        path => ["/Users/gosber/logstash/sp.log/*"]
        start_position => "beginning"
    }
}
filter {
    grok {
      match => { "message" => "%{TIMESTAMP_ISO8601:datetime} %{GREEDYDATA:xx} %{URI:api} %{NUMBER:duration}"}
    }
}
output {
   elasticsearch {
       hosts => ["127.0.0.1:9200"]
       index => "api_duration"
   }
   stdout { codec => rubydebug }
}%

# 可以看出来，grok插件进行过滤，并进行文本解析，提取出我们需要的datetime,api和duration字段
```
> grok的验证测试，可以访问在线工具 → [Grok Debug](http://grokdebug.herokuapp.com/)

> 注意我们指定了/Users/gosber/logstash/sp.log/目录，并且尝试从头开始扫描此文件夹下的所有文件

> 然后利用grok filter来提取数据

> 最后指定输出到ES服务，同时在终端屏幕上打出所有输出信息，方便查证是否有解析错误

> ES服务不要忘记指定索引哦

* 6.如果够幸运，没有出错的话，我们就可以验证索引是否成功被更新咯
```bash
gosber@localhost  ~/logstash  curl 'localhost:9200/_cat/indices?v'
health status index        pri rep docs.count docs.deleted store.size pri.store.size
yellow open   bank           5   1       1000            0    442.5kb        442.5kb
yellow open   .kibana        1   1          3            1     15.6kb         15.6kb
yellow open   api_duration   5   1   10731922            0      2.4gb          2.4gb
yellow open   customer       5   1          3            0      9.8kb          9.8kb
```
> 10731922多条记录呢！检查下对不对！
```bash
gosber@localhost  ~/logstash/sp.log  wc -l sp.log*
 1279177 sp.log.2015-12-10
 1312641 sp.log.2015-12-11
 1198706 sp.log.2015-12-12
 1324841 sp.log.2015-12-13
 1377651 sp.log.2015-12-14
 1417124 sp.log.2015-12-15
 1375684 sp.log.2015-12-16
 1446098 sp.log.2015-12-17
 10731922 total
```
> Lucky!
* 7.开始使用kibana制作报表吧，这简直毫无难度啊...



 
