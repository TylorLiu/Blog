---
title: 踩坑记3—CXF ws服务端改用SpringWs实现
date: 2018/2/10 10:43
tags: WebService
category: troubleshooting
---

出于一些项目上的原因,此前将原有的CXF Webservice服务端改用SpringWs实现，中间遇到些琐碎的小问题，造成了一些困扰，在这里回顾总结一下。

实现Spring Ws的方式请参考[官方指南](https://spring.io/guides/gs/producing-web-service/)。
指南上的步骤相当于在白纸上作画，只需要定义一个符合标准的xsd，后续注入相关的Bean就可以了。但是对于需要保持原调用不变的我来说，就像是戴着镣铐跳舞了。
改写的大致步骤如下：
1. 从原实现的wsdl中截取接口定义，放到xsd文件中；
2. mvn xjc生成相应javaBean
3. 写一下自己的Endpoint、WebServiceConfig
运行之后，服务的.wsdl就可以访问了。
但是呢，SoapUi根据这个.wsdl生成的用例必然会报错。
开启调试模式，看一下报错信息，了解下SpringWs的请求处理机制：
![98hiuD.png](https://s1.ax1x.com/2018/02/10/98hiuD.png)
在EndpointAdapter这里调试，发现Request对应的Bean没有被标记为XmlElement，就被拒绝掉了。
手动在bean上面加上注解之后，继续调试，在SuffixBasedMessagesProvider.isMessageElement()返回false，又被拒了。
````java
@Override
protected boolean isMessageElement(Element element) {
  if (super.isMessageElement(element)) {
    String elementName = getElementName(element);
    Assert.hasText(elementName, "Element has no name");
    return elementName.endsWith(getRequestSuffix()) || elementName.endsWith(getResponseSuffix()) ||
        elementName.endsWith(getFaultSuffix());
  }
  else {
    return false;
  }
}
````
从代码上看出是因为请求没带默认后缀Request。但是又不能强制原调用方修改请求格式，所以只能在服务端把Request后缀匹配去掉。
看了下DefaultWsdl11Definition的方法，有个setRequestSuffix可以自定义后缀，但是比较尴尬的是：
````java
Assert.hasText(requestSuffix, "'requestSuffix' must not be empty");
````
所以，如果使用DefaultWsdl11Definition的话，Request后缀必须有，而CXF是根据service中的方法名定义Request的，一般没有统一的后缀。
因此，需要自己实现一个Wsdl11Definition，替换掉关于后缀的几个方法。
````java
private final SuffixBasedMessagesProvider messagesProvider = new SuffixBasedMessagesProvider();

private final SuffixBasedPortTypesProvider portTypesProvider = new SuffixBasedPortTypesProvider();
````
另外，关于xsd生成的javaBean没有xmlElement系列的注解的问题，最好根据demo中的格式改写下xsd。

除了这两个问题，如果原接口方法中还带有前缀，比如：
````xml
 <soapenv:Body>
      <ws:addDevice>
         <!--Zero or more repetitions:-->
         <ws:indexCodes>?</ws:indexCodes>
         <!--Optional:-->
         <ws:vagIndexCode>?</ws:vagIndexCode>
      </ws:addDevice>
   </soapenv:Body>
````
还需要在对应的EndPoint加上 @Namespace(prefix = "web",uri = NAMESPACE_URI)注解

### 总结

从CXF Webservice迁移到SpringWs需要注意：
- 自定义Wsdl11Definition，接收无后缀的Request
- xsd格式改成标准的，以生成可用的带有@XmlRootElement的Bean
- 注意加上相应的方法前缀prefix