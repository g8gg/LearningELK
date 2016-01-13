# GROK Filter 技巧记录

> 这里记录一下GORK碰到的问题和解决

## Case 1
> 原始数据
```
2016-01-13 12:13:17  [ resin-port-8080-1022:62745255 ] - [ INFO ]  [ PV ]   [U]120968[U] [IP]115.224.255.163[IP] [VU]http://api.xxx.com/zml/user/saveUserDevice.do?method=test&t=1[VU] [V]1.4.1[V] [M]1[M]
```
> grok filter
```
%{TIMESTAMP_ISO8601:dt} %{GREEDYDATA:xx}(?:[\[U\]]{3})%{NUMBER:uid}(?:[\[U\]]{3}) (?:[\[IP\]]{4})%{IPV4:client}(?:[\[IP\]]{4}) (?:[\[UV\]]{4})(?:[http://]{7})%{IPORHOST:svr}?%{URIPATH:action}%{URIPARAM:params}*(?:[\[UV\]]{4}) (?:[\[V\]]{3})%{DATA:version}(?:[\[V\]]{3}) (?:[\[M\]]{3})%{NUMBER:mobile}(?:[\[M\]]{3})
```

> 效果 `Named Captures Only`
```
{
  "dt": [
    [
      "2016-01-13 12:13:17"
    ]
  ],
  "xx": [
    [
      " [ resin-port-8080-1022:62745255 ] - [ INFO ]  [ PV ]   "
    ]
  ],
  "uid": [
    [
      "120968"
    ]
  ],
  "client": [
    [
      "115.224.255.163"
    ]
  ],
  "svr": [
    [
      "api.xxx.com"
    ]
  ],
  "action": [
    [
      "/zml/user/saveUserDevice.do"
    ]
  ],
  "params": [
    [
      "?method=test&t=1"
    ]
  ],
  "version": [
    [
      "1.4.1"
    ]
  ],
  "mobile": [
    [
      "1"
    ]
  ]
}
```


