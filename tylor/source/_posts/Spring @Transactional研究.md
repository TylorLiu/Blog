---
title: Spring @Transactional学习
date: 2018/3/21 11:53
tags: Spring
category: blog
---
项目中出现几个关于Spring事务的异常，排查之后发现都是因为@Transactional应用不当导致的。
### 源码学习
在此分享学习@Transactional的过程，先看源码：

````java
//可标记方法和类
@Target({ElementType.METHOD, ElementType.TYPE})
//运行时生效
@Retention(RetentionPolicy.RUNTIME)
//自动继承
@Inherited
@Documented
public @interface Transactional {

	//指定TransactionManager
	@AliasFor("transactionManager")
	String value() default "";

  //指定TransactionManager
	@AliasFor("value")
	String transactionManager() default "";
	//指定事务传播设置
	Propagation propagation() default Propagation.REQUIRED;

	//指定事务隔离级别
	Isolation isolation() default Isolation.DEFAULT;

  //指定超时时间
	int timeout() default TransactionDefinition.TIMEOUT_DEFAULT;

  //标记只读事务
	boolean readOnly() default false;
    
	//指定回滚的异常类型
	Class<? extends Throwable>[] rollbackFor() default {};

	String[] rollbackForClassName() default {};
	
  //指定不会滚的异常类型
	Class<? extends Throwable>[] noRollbackFor() default {};

	String[] noRollbackForClassName() default {};

}
````
除了Propagation其他设置都比较简单，其中Isolation总共有5种，Default就是采用数据库默认的事务隔离级别，其他四种分别指定对应数据库的隔离级别，感兴趣的话可以看下[《数据库事务》]()

而Propagation可以看做使用事务的策略，下面来看下Propagation的代码：
````java
public enum Propagation {
    //REQUIRED必须有事务，也就是说当前线程有事务就用，没有就New一个
	REQUIRED(TransactionDefinition.PROPAGATION_REQUIRED),

   //支持事务，即有事务就用，没有就不用
	SUPPORTS(TransactionDefinition.PROPAGATION_SUPPORTS),

  //强制事务，必须有，没有就抛异常
	MANDATORY(TransactionDefinition.PROPAGATION_MANDATORY),

  //必须new事务
	REQUIRES_NEW(TransactionDefinition.PROPAGATION_REQUIRES_NEW),

  //非事务操作，暂停当前事务，只对分布式JtaTransactionManager有效
	NOT_SUPPORTED(TransactionDefinition.PROPAGATION_NOT_SUPPORTED),

  //不能在事务内执行，存在事务则抛异常
	NEVER(TransactionDefinition.PROPAGATION_NEVER),
    //嵌套事务(JDBC3.0 DataSourceTransactionManager下有效)
	NESTED(TransactionDefinition.PROPAGATION_NESTED);
    
	//省略
}

````
场景： 循环内部调用Transactional方法时，可以设置为Propagation.REQUIRES_NEW 并在循环体内部捕捉异常，防止单个操作失败，导致所有操作都被回滚。

### 注意事项
1. 只对public方法生效
2. 自调用无效：被内部其他方法调用时，不生效
据说AspectJ可以避免这两种情况，有待研究