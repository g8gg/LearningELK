# 讨厌的json

> 处理json格式的文件，远没有看上去那么简单，通过今天的经历学习到一些，特此记录一下。
> 感谢绕琛琳给的关键字，不然我还在绕远路呢 T_T
> 参考: [走心的链接](http://stackoverflow.com/questions/31402997/how-to-split-a-json-array-inside-an-object)

## 问题
> 无非就是包含嵌套数组怎么解析怎么mapping呗
> 具体一点吧，我们以极光聊天的记录格式为例：
> 原始数据如下：

```json
{
  "total": 3,
  "start": 0,
  "count": 3,
  "messages": [
    {
      "from_platform": "i",
      "from_name": "姚禹",
      "target_name": "猫神",
      "msg_body": {
        "text": "你去看看情况",
        "extras": {}
      },
      "from_id": "629163",
      "msg_type": "text",
      "create_time": 1451288267,
      "from_type": "user",
      "target_type": "single",
      "target_id": "629222",
      "version": 1,
      "msgid": 29039120
    },
    {
      "from_name": "王曼",
      "from_platform": "i",
      "msg_body": {
        "text": "我对该报价感兴趣",
        "extras": {
          "customMsgType": 6,
          "customMsgBody": "{\"specDesc\":\"\",\"updateDT\":\"2015-12-29 10:36:13.0\",\"storeAddress\":\"童模\",\"userId\":100448,\"contactMobile\":\"15200000012\",\"storerName\":\"李十二\",\"pictures\":[{\"url\":\"http:\\/\\/xxx.cdn.com\\/sell_pic\\/55\\/38034b1f96367a16\",\"img\":null,\"type\":5,\"sequence\":0,\"isLocal\":0,\"order\":0,\"resKey\":null,\"pictureID\":144883,\"resourceId\":0,\"imgfilePath\":null,\"title\":\"\"}],\"userBidId\":16431,\"price\":89,\"createDT\":\"2015-12-29 10:36:13.0\",\"voice\":\"\",\"pic\":\"http:\\/\\/xxx.cdnxxx.com\\/sell_pic\\/55\\/38034b1f96367a16?imageView2\\/1\\/w\\/150\\/h\\/150\\/format\\/jpg\",\"voiceLength\":\"0\",\"unit\":\"米\",\"storeAreaID\":1190,\"userPurchaseId\":27248,\"feedback\":0,\"bidDesc\":\"\",\"storeTitle\":\"咯哦哦哦\",\"supplyType\":0}"
        }
      },
      "target_name": "李十二\n",
      "from_id": "300153",
      "msg_type": "text",
      "from_type": "user",
      "create_time": 1451359215,
      "target_type": "single",
      "target_id": "622344",
      "version": 1,
      "msgid": 29190000
    }
  ]
}
```

## 解答

### 第一步
> 先做简单的吧，json的mapping挺好写的
> 把json结构格式化出来，然后稍微改动一下就出来了
* mapping.json
```json
{
  "zbim": {
    "_all": {
      "enabled": false
    },
    "properties": {
      "total": {
        "type": "integer"
      },
      "start": {
        "type": "integer"
      },
      "count": {
        "type": "integer"
      },
      "messages": {
        "properties": {
          "from_platform": {
            "type": "string"
          },
          "from_name": {
            "type": "string"
          },
          "target_name": {
            "type": "string"
          },
          "msg_body": {
            "properties": {
              "text": {
                "type": "string"
              },
              "extras": {
                "properties": {
                  "customMsgType": {
                    "type": "short"
                  },
                  "customMsgBody": {
                    "type": "string"
                  }
                }
              }
            }
          },
          "from_id": {
            "type": "string"
          },
          "msg_type": {
            "type": "string"
          },
          "create_time": {
            "type": "long"
          },
          "from_type": {
            "type": "string"
          },
          "target_type": {
            "type": "string"
          },
          "target_id": {
            "type": "string"
          },
          "version": {
            "type": "short"
          },
          "msgid": {
            "type": "long"
          }
        }
      }
    }
  }
}
```
### 第二步
> 看看conf怎么写
* my.conf
```ruby
input {
    file {
        type => "zbim"
        path => ["/zbim.log/*.json"]
        start_position => "beginning"
        codec => json {
            charset => "UTF-8"
        }
    }
}

filter {
    json {
        source => "message"
    }
    split {
        field => "messages"
    }
}

output {
    elasticsearch { 
    hosts => ["127.0.0.1:9200"]
    index => "zbim"
  }
  stdout { codec => rubydebug }
}
```

### 思考题
> 看完这里，你还不会解决json的嵌套数组问题，那么...
