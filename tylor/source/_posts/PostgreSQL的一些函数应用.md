---
title: PostgreSQL的一些函数应用
date: 2018/3/22 16:15
tags:  PostgreSQL
category: tips
---

### String

- replace
替换更新
````sql
 update table_1 set column_1 = replace(column_1,"原值","新值");
````

- string_agg
string聚合
````sql
select string_agg(tablename, ',') from pg_tables where SCHEMANAME='public'
````
- || 字符串拼接