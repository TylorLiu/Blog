---
title: 踩坑记5-Mybatis合集
date: 2018/5/19 16:20
tags: Mybatis
category: troubleshooting
---
从刚参加工作开始就接触了Mybatis，但是大部分时候只是照猫画虎，一直没有系统的学习过，遇到些问题就很容易卡住，搜不到解决方法，只能参照错误逐行排查来定位问题，效率十分低下。
反思之后，决定从头学习一下，顺便整理下踩过的坑，分析下原因。
[官网链接](http://www.mybatis.org/mybatis-3/index.html#)
### MyBatis是什么
它是一款持久层框架，支持自定义SQL、存储过程和结果集映射。通过XML或者注解来实现ORM映射。

### 依赖
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>x.x.x</version>
    </dependency>
 与Spring框架的集成
 
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis-spring</artifactId>
        <version>x.x.x</version>
    </dependency>
### 框架配置   
    1. 数据源(必须)、SqlSessionFactory
    2. Settings 缓存、代理、作用域等
    3. typeAliases POJO别名
    4. typeHandlers 可以自定义类型映射，比如Enum映射为Name or Order
    5. environments 
        - 不同环境不同设置(开发、测试、生产等)
        - 多数据源对应多个SqlSessionFactory
    6. 插件 实现Interceptor接口，拦截下列方法
        Executor (update, query, flushStatements, commit, rollback, getTransaction, close, isClosed)
        ParameterHandler (getParameterObject, setParameters)
        ResultSetHandler (handleResultSets, handleOutputParameters)
        StatementHandler (prepare, parameterize, batch, update, query)
    7.  Mappers XML映射文件
    
### Mapper语法
- cache – 给定命名空间的缓存配置。
- cache-ref – 其他命名空间缓存配置的引用。
- resultMap – 是最复杂也是最强大的元素，用来描述如何从数据库结果集中来加载对象。
    - 自动映射以及驼峰映射mapUnderscoreToCamelCase=true\
    - 集合嵌套association、collection等
    - 鉴别器discriminator case判断
- sql – 可被其他语句引用的可重用语句块。
- insert – 映射插入语句
- update – 映射更新语句
- delete – 映射删除语句
- select – 映射查询语句
PS： 缓存使用的时候可以参考[聊聊MyBatis缓存机制](https://tech.meituan.com/mybatis_cache.html)
不过建议使用Spring+Redis实现，更靠谱一些

### 动态 SQL
- if
- choose/when/otherwise
- bind
- trim, where, set
- foreach

### JAVA API 
- 注解
- SqlSession
- SqlBuilder

### 问题集合
1. insert操作报 无效的列：111
    -  原因: 插入空值到非空字段