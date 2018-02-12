---
title: SpringBoot下解决webservice与Restful接口路径冲突
date: 2018/2/9 10:02
tags: [cxf,webservice]
category: troubleshooting
---

先来解释下前因后果吧，虽然是一个Long Long story。从前，这个web项目的框架结构是Spring+Struts2+myBatis，对外接口用Apache CXF实现，既有webservice又有restful。
由于cxf.path默认值为/services，所有这些ws和rest接口全都在services路径下。后来由于struts2频繁暴露安全漏洞以及前后端技术的换代需要，项目的框架升级到Spring (Boot+MVC+JPA)。
重构时做了业务拆分，webservice接口不再保留，其他rest接口改为Spring @RestController实现，路径保持不变。
再后来业务又有变动，webservice接口又要原封不动的添加进来，作为优秀的代码搬运工，首先想到的是把CXF集成到SpringBoot中来，然后把CXF ws server端代码迁移过来即可。
做法如下：

Google It，找到这篇[官方指南](http://cxf.apache.org/docs/springboot.html)，按照指南引入Spring Boot CXF JAX-WS Starter，配置一个javax.xml.ws.Endpoint，再参考[官方Demo](https://github.com/apache/cxf/tree/master/distribution/src/main/release/samples/jaxws_spring_boot)在原实现上加几个注解，最后跑一下测试用例，结果OK，感觉差不多完事儿了。

结果万万没想到，更新接口文档时，顺便在Swagger上试了下Rest接口，竟然返回404！看了下body中的错误信息:
````
<html><body>No service was found.</body></html>
````
接口是services路径下的，八成是被拦截丢给CXF处理了。
接下来进入调试模式，根据控制台打印的信息：
````
[http-nio-80-exec-7] WARN  o.a.c.t.servlet.ServletController- Can't find the the request for http://XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX's Observer 
````
我在ServletController中相应位置打了个断点，然后再触发请求，进入断点后，查看线程执行的方法链，追溯到线程的起点，按调用顺序，逐步查看代码，必要时打断点调试。

![93L48H.jpg](https://s1.ax1x.com/2018/02/09/93L48H.jpg)

粗略的正常处理流程：
NIO监听到web请求->分配线程处理->Http处理器解析->CoyoteAdapter映射分发request->FilterChain过滤->调用相应的servlet分发到具体的业务代码->处理业务逻辑->返回

问题基本上是出在CoyoteAdapter映射请求的过程中，从CoyoteAdapter.service()开始断点排查，发现最终在Mapper.internalMapWrapper方法中完成request path与上下文信息的映射。

![93XNhF.jpg](https://s1.ax1x.com/2018/02/09/93XNhF.jpg)

最后的方法代码如下：

````java
   /**
     * Wrapper mapping.
     */
    private final void internalMapWrapper(ContextVersion contextVersion,
                                          CharChunk path,
                                          MappingData mappingData)
        throws Exception {

        int pathOffset = path.getOffset();
        int pathEnd = path.getEnd();
        int servletPath = pathOffset;
        boolean noServletPath = false;

        int length = contextVersion.path.length();
        if (length != (pathEnd - pathOffset)) {
            servletPath = pathOffset + length;
        } else {
            noServletPath = true;
            path.append('/');
            pathOffset = path.getOffset();
            pathEnd = path.getEnd();
            servletPath = pathOffset+length;
        }

        path.setOffset(servletPath);

        // Rule 1 -- Exact Match 精确匹配
        Wrapper[] exactWrappers = contextVersion.exactWrappers;
        internalMapExactWrapper(exactWrappers, path, mappingData);

        // Rule 2 -- Prefix Match 前缀匹配
        boolean checkJspWelcomeFiles = false;
        Wrapper[] wildcardWrappers = contextVersion.wildcardWrappers;
        if (mappingData.wrapper == null) {
            internalMapWildcardWrapper(wildcardWrappers, contextVersion.nesting,
                                       path, mappingData);
            if (mappingData.wrapper != null && mappingData.jspWildCard) {
                char[] buf = path.getBuffer();
                if (buf[pathEnd - 1] == '/') {
                    /*
                     * Path ending in '/' was mapped to JSP servlet based on
                     * wildcard match (e.g., as specified in url-pattern of a
                     * jsp-property-group.
                     * Force the context's welcome files, which are interpreted
                     * as JSP files (since they match the url-pattern), to be
                     * considered. See Bugzilla 27664.
                     */
                    mappingData.wrapper = null;
                    checkJspWelcomeFiles = true;
                } else {
                    // See Bugzilla 27704
                    mappingData.wrapperPath.setChars(buf, path.getStart(),
                                                     path.getLength());
                    mappingData.pathInfo.recycle();
                }
            }
        }

        if(mappingData.wrapper == null && noServletPath) {
            // The path is empty, redirect to "/"
            mappingData.redirectPath.setChars
                (path.getBuffer(), pathOffset, pathEnd-pathOffset);
            path.setEnd(pathEnd - 1);
            return;
        }

        // Rule 3 -- Extension Match 后缀匹配
        Wrapper[] extensionWrappers = contextVersion.extensionWrappers;
        if (mappingData.wrapper == null && !checkJspWelcomeFiles) {
            internalMapExtensionWrapper(extensionWrappers, path, mappingData,
                    true);
        }

        // Rule 4 -- Welcome resources processing for servlets 静态文件匹配
        if (mappingData.wrapper == null) {
            boolean checkWelcomeFiles = checkJspWelcomeFiles;
            if (!checkWelcomeFiles) {
                char[] buf = path.getBuffer();
                checkWelcomeFiles = (buf[pathEnd - 1] == '/');
            }
            if (checkWelcomeFiles) {
                for (int i = 0; (i < contextVersion.welcomeResources.length)
                         && (mappingData.wrapper == null); i++) {
                    path.setOffset(pathOffset);
                    path.setEnd(pathEnd);
                    path.append(contextVersion.welcomeResources[i], 0,
                            contextVersion.welcomeResources[i].length());
                    path.setOffset(servletPath);

                    // Rule 4a -- Welcome resources processing for exact macth
                    internalMapExactWrapper(exactWrappers, path, mappingData);

                    // Rule 4b -- Welcome resources processing for prefix match
                    if (mappingData.wrapper == null) {
                        internalMapWildcardWrapper
                            (wildcardWrappers, contextVersion.nesting,
                             path, mappingData);
                    }

                    // Rule 4c -- Welcome resources processing
                    //            for physical folder
                    if (mappingData.wrapper == null
                        && contextVersion.resources != null) {
                        Object file = null;
                        String pathStr = path.toString();
                        try {
                            file = contextVersion.resources.lookup(pathStr);
                        } catch(NamingException nex) {
                            // Swallow not found, since this is normal
                        }
                        if (file != null && !(file instanceof DirContext) ) {
                            internalMapExtensionWrapper(extensionWrappers, path,
                                                        mappingData, true);
                            if (mappingData.wrapper == null
                                && contextVersion.defaultWrapper != null) {
                                mappingData.wrapper =
                                    contextVersion.defaultWrapper.object;
                                mappingData.requestPath.setChars
                                    (path.getBuffer(), path.getStart(),
                                     path.getLength());
                                mappingData.wrapperPath.setChars
                                    (path.getBuffer(), path.getStart(),
                                     path.getLength());
                                mappingData.requestPath.setString(pathStr);
                                mappingData.wrapperPath.setString(pathStr);
                            }
                        }
                    }
                }

                path.setOffset(servletPath);
                path.setEnd(pathEnd);
            }

        }

        /* welcome file processing - take 2
         * Now that we have looked for welcome files with a physical
         * backing, now look for an extension mapping listed
         * but may not have a physical backing to it. This is for
         * the case of index.jsf, index.do, etc.
         * A watered down version of rule 4
         */
        if (mappingData.wrapper == null) {
            boolean checkWelcomeFiles = checkJspWelcomeFiles;
            if (!checkWelcomeFiles) {
                char[] buf = path.getBuffer();
                checkWelcomeFiles = (buf[pathEnd - 1] == '/');
            }
            if (checkWelcomeFiles) {
                for (int i = 0; (i < contextVersion.welcomeResources.length)
                         && (mappingData.wrapper == null); i++) {
                    path.setOffset(pathOffset);
                    path.setEnd(pathEnd);
                    path.append(contextVersion.welcomeResources[i], 0,
                                contextVersion.welcomeResources[i].length());
                    path.setOffset(servletPath);
                    internalMapExtensionWrapper(extensionWrappers, path,
                                                mappingData, false);
                }

                path.setOffset(servletPath);
                path.setEnd(pathEnd);
            }
        }


        // Rule 7 -- Default servlet 
        if (mappingData.wrapper == null && !checkJspWelcomeFiles) {
            if (contextVersion.defaultWrapper != null) {
                mappingData.wrapper = contextVersion.defaultWrapper.object;
                mappingData.requestPath.setChars
                    (path.getBuffer(), path.getStart(), path.getLength());
                mappingData.wrapperPath.setChars
                    (path.getBuffer(), path.getStart(), path.getLength());
            }
            // Redirection to a folder
            char[] buf = path.getBuffer();
            if (contextVersion.resources != null && buf[pathEnd -1 ] != '/') {
                Object file = null;
                String pathStr = path.toString();
                try {
                    file = contextVersion.resources.lookup(pathStr);
                } catch(NamingException nex) {
                    // Swallow, since someone else handles the 404
                }
                if (file != null && file instanceof DirContext) {
                    // Note: this mutates the path: do not do any processing
                    // after this (since we set the redirectPath, there
                    // shouldn't be any)
                    path.setOffset(pathOffset);
                    path.append('/');
                    mappingData.redirectPath.setChars
                        (path.getBuffer(), path.getStart(), path.getLength());
                } else {
                    mappingData.requestPath.setString(pathStr);
                    mappingData.wrapperPath.setString(pathStr);
                }
            }
        }

        path.setOffset(pathOffset);
        path.setEnd(pathEnd);

    }

````
上面的几种Wrapper都是根据web.xml或ServletRegisterBean中设置的Url Pattern产生的，用来根据请求路径映射相应的Servlet。对应规则如下：
- path.endsWith("/*") : wildcardWrapper 通配符匹配
- path.startsWith("*.")  : extensionWrapper 扩展名匹配
- path.equals("/") : defaultWrapper 默认Wrapper
- 其他url Pattern : exactWrapper 精确匹配
详见Mapper.addWrapper()方法。
而实际的路径比对过程中，匹配规则的优先顺序是：
exactWrapper>wildcardWrapper>extensionWrapper> welcomeResources(静态资源)>defaultWrapper。
而此时的context中的只有defaultWrapper和值为/services的wildcardWrapper：

[![93XttU.md.jpg](https://s1.ax1x.com/2018/02/09/93XttU.md.jpg)](https://imgchr.com/i/93XttU)

根据path的值/context/services/xxxxServices/xxx/xxxx, 请求就被映射到了CXFServlet。

要保证接口路径不变的同时，解决路径冲突，比较好的办法就是从urlPattern着手，如果能将cxfServlet的匹配范围缩小，将rest接口排除在外就OK了。
在CXF合入SpringBoot的指南中有这样一句：

````
Use "cxf.path" property to customize a CXFServlet URL pattern
````
通过cxf.path自定义CXFServlet的匹配路径,在项目中搜索cxf.path,定位到mvn仓库的文件：

org/apache/cxf/cxf-spring-boot-autoconfigure/3.1.12/cxf-spring-boot-autoconfigure-3.1.12.jar!/META-INF/spring-configuration-metadata.json
内容：
````
  {
      "name": "cxf.path",
      "type": "java.lang.String",
      "description": "Path that serves as the base URI for the services.",
      "sourceType": "org.apache.cxf.spring.boot.autoconfigure.CxfProperties",
      "defaultValue": "/services"
    }
````
找到CxfProperties类，查找path的引用，找到CxfAutoConfiguration：

````java
@Configuration
@ConditionalOnWebApplication
@ConditionalOnClass({ SpringBus.class, CXFServlet.class })
@EnableConfigurationProperties(CxfProperties.class)
@AutoConfigureAfter(EmbeddedServletContainerAutoConfiguration.class)
public class CxfAutoConfiguration {

    @Autowired
    private CxfProperties properties;

    @Bean
    @ConditionalOnMissingBean(name = "cxfServletRegistration")
    public ServletRegistrationBean cxfServletRegistration() {
        String path = this.properties.getPath();
        String urlMapping = path.endsWith("/") ? path + "*" : path + "/*";
        ServletRegistrationBean registration = new ServletRegistrationBean(
                new CXFServlet(), urlMapping);
        CxfProperties.Servlet servletProperties = this.properties.getServlet();
        registration.setLoadOnStartup(servletProperties.getLoadOnStartup());
        for (Map.Entry<String, String> entry : servletProperties.getInit().entrySet()) {
            registration.addInitParameter(entry.getKey(), entry.getValue());
        }
        return registration;
    }
    
    //......省略其余方法
}
````
我们可以看到CXF在c.path路径下注册了一个wildcardWrapper,所有该路径下的请求都会被拦截。
考虑到有ConditionalOnMissingBean的存在，可以注入一个cxfServletRegistration来覆盖cxf的默认实现，以下是我的代码：

````java
@Configuration
public class WebServiceConfig {

    private static  final String SERVICE_PATH = "SysConfigService";

    @Autowired
    private CxfProperties properties;

    @Bean(name = "cxfServletRegistration")
    public ServletRegistrationBean cxfServletRegistration() {
        String path = this.properties.getPath();

        String urlMapping = path.endsWith("/") ? path + SERVICE_PATH : path + "/"+SERVICE_PATH;
        String urlMappingWsdl = path.endsWith("/") ? path + SERVICE_PATH+"?wsdl" : path + "/"+SERVICE_PATH+"?wsdl";

        ServletRegistrationBean registration = new ServletRegistrationBean(
            new CXFServlet(), urlMapping,urlMappingWsdl);
        CxfProperties.Servlet servletProperties = this.properties.getServlet();
        registration.setLoadOnStartup(servletProperties.getLoadOnStartup());
        for (Map.Entry<String, String> entry : servletProperties.getInit().entrySet()) {
            registration.addInitParameter(entry.getKey(), entry.getValue());
        }
        return registration;
    }

    @Autowired
    private Bus bus;

    @Autowired
    ICmsService cmsService;

    @Bean
    public Endpoint endpoint() {
        EndpointImpl endpoint = new EndpointImpl(bus, new SysConfigService(cmsService));
        endpoint.publish("/"+SERVICE_PATH);
        return endpoint;
    }
}
````
但是，发布的webservice地址变成了/services/SysConfigService/SysConfigService?wsdl，把Endpoint地址修改成endpoint.publish(")之后，地址恢复正常。

然后，我们重写cxfServletRegistration的初衷是在不改变path的前提下修改cxfServlet的url Pattern。
显然，这个行不通，Endpoint发布的url地址并非由path决定，而是跟cxfServletRegistration的urlMapping有关。因此，这种方式虽然勉强解决了目前的问题，但仍有难以避免的缺陷:
- 目前的方式相当于在services/SysConfigService路径下发布addr为""的Endpoint，且路径固定，无法添加新的webservice
- 如果采取cxf.path =/services/SysConfigService的方式，则可以在SysConfigService的路径下扩展其它ws接口

调试了几遍Endpoint.publish的过程，没有发现与cxfServletRegistration有明显的关联，暂时放弃这方面的探索。
由于时间还算充足，没必要苟且妥协，我又研究了下SpringWs，发现对于每一个Webservice接口都可以通过Wsdl11Definition.setLocationUri("/services/SysConfigService")指定接口的地址，而不是必须置于某个指定路径下。
所以只需要在ServletRegistrationBean加入每个webservice的准确路径的urlMapping,即可避免拦截到Rest接口的请求。
````java
@Bean
public ServletRegistrationBean messageDispatcherServlet(ApplicationContext applicationContext) {
    MessageDispatcherServlet servlet = new MessageDispatcherServlet();
    servlet.setApplicationContext(applicationContext);
    servlet.setTransformWsdlLocations(true);
    return new ServletRegistrationBean(servlet, "/services/SysConfigService/SysConfigService.wsdl", "/services/SysConfigService");
}
````
CXF Webservice切换到SpringWs的步骤改天再写。