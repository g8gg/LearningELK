# 使用kv filter的技巧，并合理add_field

## 用户行为分析(量化统计)
> 碰到一个统计需求，需要分析用户行为并统计用户的行为
> * 数据行格式：
> ```
> 2015-12-21 03:18:35  [ resin-port-18... ] - [ INFO ]  [ PV ]  100018 211.138.105.184 http://api.xxx.com/user/getpushkey.do?method=xxx&foo=bar&foo1=bar1
> ```
> 需要解析时间，userid,clientIP,行为

## 思路和方案
> * 首先logstash进行grok
>  * `问题` grok解析后，实际的行为很可能是 /模块名/API名/方法名，而实际行为可能是方法名，故应该解析得到API+Method
> * 即便KV Filter之后，如何进行组装？
>  * 无论grok和kv在解析match后，都只能访问自己解析出的值，例如%{action}/%{method}，可能%{method}当作字符串保留下来，而不是值
>  * 我们的问题是要组装grok得到的值合并kv中得到的值 %{action}/%{method} => getpushkey.do/xxx *行为的标识，"method="是多余*
> * 得到的字段应该是一个组合字段，但是如果%{method}不匹配则无法组合-*有些接口不包含method*，所以应该预设一个值%{method}=""

## 实现
### logstash.conf
```bash
gosber@freedamadeMacBook-Pro  ~/logstash  cat logstash_user.conf
input {
    file {
        type => "user_action"
        path => ["/Users/gosber/logstash/pv.log/*"]
        start_position => "beginning"
    }
}

filter {
    grok {
      match => { 
        "message" => "%{TIMESTAMP_ISO8601:dt} %{GREEDYDATA:xx} %{NUMBER:userid} %{IPV4:client}(.http://)%{IPORHOST:svr}?%{URIPATH:action}%{URIPARAM:params}*"
      }
      add_field => { 
        "method" => "" 
      }
    }
    kv { 
      #target => "message"
      field_split => "&,?"
    }
    mutate {
      add_field => { 
        "action_method" => "%{action}/%{method}" 
      }
    }
}

output {
  elasticsearch { 
    hosts => ["127.0.0.1:9200"]
    index => "user_action"
    }
  stdout { codec => rubydebug }
}
```

### ElasticSearch Mappings
```
{
  "user_action":{
    "_all": {"enabled":false},
    "properties":{
      "dt":{"type":"date","store":"yes","index":"not_analyzed","format":"yyyy-MM-dd HH:mm:ss"},
      "userid":{"type":"string","store":"yes","index":"analyzed"},
      "action_method":{"type":"string","store":"yes","index":"analyzed"},
      "client":{"type":"string","store":"yes","index":"analyzed"}
     }
  }
}
```

## 大功告成~



