---
title: wsimport和xjc命令
date: 2018/3/12 15:29
tags: [WebService,XML]
category: tips
---

-encoding utf-8 指定编码集

### wsimport 生成Webservice客户端
````
wsimport -s D:\temp\s -p com.map -verbose http://ws.webxml.com.cn/WebServices/MobileCodeWS.asmx?wsdl 
````
用法详见：wsimport -help

### xjc 根据xsd生成Java Bean

用法详见: xjc -help