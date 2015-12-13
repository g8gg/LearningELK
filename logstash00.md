* Env
```
gosber@freedamadeMBP  ~  uname -a
Darwin freedamadeMBP.lan 15.0.0 Darwin Kernel Version 15.0.0: Sat Sep 19 15:53:46 PDT 2015; root:xnu-3247.10.11~1/RELEASE_X86_64 x86_64

✘ gosber@freedamadeMBP  ~  java -version
java version "1.8.0_51"
Java(TM) SE Runtime Environment (build 1.8.0_51-b16)
Java HotSpot(TM) 64-Bit Server VM (build 25.51-b03, mixed mode)
```

* install
```
Last login: Sun Dec 13 00:13:05 on ttys003
 gosber@freedamadeMBP  ~  brew info logstash
logstash: stable 2.1.1, HEAD
Tool for managing events and logs
https://www.elastic.co/products/logstash
Not installed
From: https://github.com/Homebrew/homebrew/blob/master/Library/Formula/logstash.rb
==> Caveats
Please read the getting started guide located at:
  https://www.elastic.co/guide/en/logstash/current/getting-started-with-logstash.html
 gosber@freedamadeMBP  ~  brew install logstash
==> Downloading https://download.elastic.co/logstash/logstash/logstash-2.1.1.tar.gz
######################################################################## 100.0%
==> Caveats
Please read the getting started guide located at:
  https://www.elastic.co/guide/en/logstash/current/getting-started-with-logstash.html
==> Summary
🍺  /usr/local/Cellar/logstash/2.1.1: 9349 files, 148M, built in 2.5 minutes
```

* Hello World *(command line)*
```
 ✘ gosber@freedamadeMBP  ~  logstash -e 'input{stdin{}}output{stdout{codec=>rubydebug}}'
Hello Wrold
Settings: Default filter workers: 2
Logstash startup completed
{
       "message" => "Hello Wrold",
      "@version" => "1",
    "@timestamp" => "2015-12-13T17:48:45.058Z",
          "host" => "freedamadeMBP.lan"
}
```

* Using conf file *(logstash.conf)*
```
input{
  stdin  {

  }
}output{
  stdout  {
    codec => rubydebug    {

    }
  }  elasticsearch  {
  }
}
```
```
gosber@freedamadeMBP  ~  logstash --configtest -f logstash.conf
Configuration OK
gosber@freedamadeMBP  ~  logstash -f logstash.conf
Settings: Default filter workers: 2
Logstash startup completed
Hello World Again
{
       "message" => "Hello World Again",
      "@version" => "1",
    "@timestamp" => "2015-12-13T18:02:23.007Z",
          "host" => "freedamadeMBP.lan"
}
```

* Web
```
gosber@freedamadeMBP  ~  curl http://127.0.0.1:9200/_search\?q\=hello
{"took":63,"timed_out":false,"_shards":{"total":5,"successful":5,"failed":0},"hits":{"total":1,"max_score":1.0,"hits":[{"_index":"logstash-2015.12.13","_type":"logs","_id":"AVGcggAyJ-YMR9K6AXqf","_score":1.0,"_source":{"message":"Hello World Again","@version":"1","@timestamp":"2015-12-13T18:02:23.007Z","host":"freedamadeMBP.lan"}}]}}%
```



