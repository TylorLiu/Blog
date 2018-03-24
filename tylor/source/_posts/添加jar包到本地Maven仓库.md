---
title: 添加jar包到本地Maven仓库
date: 2018/3/24 10:29
tags: maven
category: tips
---

### 添加jar包到本地Maven仓库
````
mvn install:install-file -DgroupId=com.h2database -DartifactId=h2 -Dversion=1.4.197 -Dpackaging=jar -Dfile=F:\iVMS9620\Downloads\h2-1.4.197.jar
````