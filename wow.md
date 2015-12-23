# 我勒个去
> 有些指令经常出乎我的意料，然后就悲剧了

* 误删除索引
> 本来我想删除某个(`user_action_new`)索引下的某些或者全部文档的，但是...
```bash
gosber@freedamadeMacBook-Pro  ~/logstash  curl -XGET 'http://127.0.0.1:9200/_cat/indices?v'
health status index           pri rep docs.count docs.deleted store.size pri.store.size
yellow open   user_action_new   5   1          3            0     18.1kb         18.1kb
yellow open   bank              5   1       1000            0    442.5kb        442.5kb
yellow open   user_action       5   1    2571687            0    988.6mb        988.6mb
yellow open   .kibana           1   1         10            2     80.1kb         80.1kb
yellow open   api_duration      5   1   10731922            0      2.4gb          2.4gb
yellow open   customer          5   1          3            0      9.8kb          9.8kb

gosber@freedamadeMacBook-Pro  ~/logstash  curl -XDELETE 'localhost:9200/user_action_new/' -d '
{
  "query": { "match_all": {} },
}'
"acknowledged":true}%
gosber@freedamadeMacBook-Pro  ~/logstash  curl -XGET 'http://127.0.0.1:9200/_cat/indices?v'
health status index        pri rep docs.count docs.deleted store.size pri.store.size
yellow open   bank           5   1       1000            0    442.5kb        442.5kb
yellow open   user_action    5   1    2571687            0    988.6mb        988.6mb
yellow open   .kibana        1   1         10            2       29kb           29kb
yellow open   api_duration   5   1   10731922            0      2.4gb          2.4gb
yellow open   customer       5   1          3            0      9.8kb          9.8kb
```

> `警告` 千万别...
