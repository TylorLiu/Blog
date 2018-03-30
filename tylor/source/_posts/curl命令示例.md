---
title: curl命令示例
date: 2018/3/21 16:15
tags: 命令
category: tips
---
### 示例： 
````
curl -v  -H Content-Type:application/json -H appKey:2DHYBU6 -X POST --data '[{ "indexCode": "00001"}]' http://10.41.10.21/start
````

- -v 展示详细信息
- -H 添加header
- -X POST 指定为Post请求
- --data HTTP POST方式传送数据


### windows修改命令行字符编码

````
chcp 65001
````
然后右键属性，字体修改为
【Lucida Console】