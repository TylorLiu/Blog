---
title: 踩坑记4-从零开始搭建练手web项目
date: 2018/3/22 11:28
tags: Spring
category: troubleshooting
---

虽然经历过不少web项目，却没有独自从头开发过，不失为一个遗憾，于是决定汇集一身所学，打造我的个人项目，过程必然是坎坷的，就以此篇记录下我踩过的坑坑洼洼。
### Filter使用不当导致服务空返回
自定义的Filter验证通过后，没有调用职责链，导致链断开。
````java
filterChain.doFilter(servletRequest, servletResponse);
````

### 