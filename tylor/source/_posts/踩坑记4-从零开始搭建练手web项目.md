---
title: 踩坑记4-Java Web相关
date: 2018/3/22 11:28
tags: Spring
category: troubleshooting
---

### Filter使用不当导致服务空返回
自定义的Filter验证通过后，没有调用职责链，导致链断开。
````java
filterChain.doFilter(servletRequest, servletResponse);
````

### 自定义权限校验的filter链，结束后又重复调用主链，导致接口执行两遍，返回两次结果