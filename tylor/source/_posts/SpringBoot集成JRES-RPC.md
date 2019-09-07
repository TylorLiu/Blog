---
title: SpringBoot集成JRES-RPC 实现T2服务
date: 2018/11/1 10:03
tags: 
category: 
---

#### 项目背景
1. 需要使用Java对接第三方平台,提供给C程序调用
2. 保持项目架构简单、容易部署

#### 过程
1. 新建一个SpringBoot项目
2. 参考[JRES Server框架说明](http://rdc.hundsun.com/product/download/jresServer.htm)，单独引入RPC组件，主要是引入以下依赖：
````
<dependency>
    <groupId>com.hundsun.jresplus</groupId>
    <artifactId>jresplus-rpc-api</artifactId>
    <version>1.2.2</version>
</dependency>
<dependency>
    <groupId>com.hundsun.jresplus</groupId>
    <artifactId>jresplus-rpc-core</artifactId>
    <version>1.2.2</version>
</dependency>
<dependency>
    <groupId>com.hundsun.jresplus</groupId>
    <artifactId>jresplus-rpc-spi-t2def</artifactId>
    <version>1.2.2</version>
</dependency>
<dependency>
    <groupId>com.hundsun.jresplus</groupId>
    <artifactId>jresplus-rpc-spi-spring4</artifactId>
    <version>1.2.2</version>
</dependency>
<dependency>
    <groupId>com.hundsun.jresplus</groupId>
    <artifactId>jresplus-common-core</artifactId>
    <version>1.2.0</version>
</dependency>
````
3. 在启动类中引入JRES-RPC的Bean配置并设置配置文件路径
[![iWrDl6.md.png](https://s1.ax1x.com/2018/11/01/iWrDl6.md.png)](https://imgchr.com/i/iWrDl6)
4. 配置好server.properties和ares-app-config.xml文件，并放到resources目录下
5. 开发remoting接口，并使用HSAdmin工具测试 （可以咨询研发中心客服苏响）

#### 遇到的一些问题
1. 引入SpringBoot的热部署工具spring-boot-devtools之后，参数为JavaBean的T2接口报argument type miss match。
    调试后发现，热部署工具使用RestartedClassLoader加载类，而T2接口的参数Bean是由其它的类加载器加载，所以虽然是同一个类型，但是Class对象不一致，反射调用method报错。
    移除spring-boot-devtools后成功解决。
2. 项目打成Jar包运行之后 发现T2Server没有启动，也没有报错。
    几经排查，对比Application类Main启动的方式下的日志,怀疑XML配置的JavaBean CepContextLoader没有加载，因此把配置文件从jresplus-rpc-spi-spring4中拿出来放到项目路径下，将其中的CepContextLoader改为@Bean加载。
````java
@SpringBootApplication
//注释掉jresplus-cep-beans.xml里的CepContextLoader和SpringPluginFramework配置
@ImportResource({"classpath:config/jresplus-cep-beans.xml","classpath:config/remoting-main-beans.xml"})
@Slf4j
public class CopApplication {

    public static void main(String[] args) {
        System.setProperty("JresConfigLocation","classpath:server.properties");
        SpringApplication.run(CopApplication.class, args);
    }

    @Bean("jres.framework")
    public SpringPluginFramework springPluginFramework(){
        return new SpringPluginFramework();
    }

    @Bean
    CepContextLoader CepContextLoader(SpringPluginFramework springPluginFramework){
        CepContextLoader cepContextLoader = new CepContextLoader();
        cepContextLoader.setPropPath("classpath*:ares-app-config.xml");
        cepContextLoader.setFramework(springPluginFramework);
        return  cepContextLoader;
    }
}
````
重新打包后，测试可用。