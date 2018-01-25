---
title: 踩坑记2—SpringDataJPA合集
date: 2018/1/24 17:40
tags: [Spring,JPA]
category: troubleshooting
---

### lombok@Data注解 pojo -- 存在外键关联
- toString()导致StackOverFlow
- 作为DTO输出，JSON序列化报错 (属性添加@JsonIgnore解决)

### 依靠数据库unique防业务字段重复
- 示例中,由于saveDevice被事务控制，若device与已有设备冲突，saveDevice()执行完后提交事务时才会报错，此时

已调用外部接口上报了新增设备信息，本系统回滚save操作，就会导致数据不一致。

```` java
@Transactional
public void saveDevice(Device device){
  deviceDao.save(device);
  outerService.reportDeviceAdded(device);
}
````

### 非事务性方法调用@Transactional方法，偶尔会导致异常
Spring - No EntityManager with actual transaction available for current thread - cannot reliably process 'persist' call


### 外键关联对象先后save时偶现
- 错误问题：
org.springframework.orm.jpa.JpaObjectRetrievalFailureException: Unable to find com.xxx.DeviceChannel with id 12;
- 代码：
```` java
deviceChannelDao.save(channels);
deviceDao.save(device);
````
deviceChannel与device外键关联，修改后代码：
```` java
device.setDeviceChannels(deviceChannelDao.save(channels));
deviceDao.save(device);
````