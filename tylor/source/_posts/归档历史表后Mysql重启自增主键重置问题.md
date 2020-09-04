---
title: 归档历史表后Mysql重启自增主键重置问题
date: 2018/5/19 11:29
tags: Mysql
category:  blog
---
### 问题场景
    COP日初始化时，会将当前表数据归档到历史表，然后把数据从当前表删除。
    由于MySql InnoDB数据库自增列auto_increment的最大值保存在内存中，启动后auto_increment会重置为当前表的最大值+1。
    如果此时重启数据库，由于当前表无数据，重启后当前表新增的数据主键就变为1。
    再次日初始化时，由于历史表中已存在主键为1的数据，就会主键冲突。
    
### 解决方案
    1. 升级Mysql到8.0.0版本 (参考官网bug issue，未验证) 
    2. 为当前表添加触发器，插入数据时，取当前表和历史表中较大的auto_increment作为当前表的auto_increment。
        以COP为例，当前表为cop_tliquidateins，历史表为cop_thisliquidateins，主键为liquidate_ins_id，按以下方式创建触发器：
    
````
delimiter $$
drop trigger if exists trig_autoinc_liquidateins;
CREATE TRIGGER trig_autoinc_liquidateins BEFORE INSERT ON cop_tliquidateins
FOR EACH ROW
BEGIN
  declare auto_incr1 BIGINT;
  declare auto_incr2 BIGINT;
  IF (NEW.liquidate_ins_id=0) THEN
    SELECT AUTO_INCREMENT INTO auto_incr1 FROM information_schema.TABLES WHERE TABLE_SCHEMA = SCHEMA() and UPPER(TABLE_NAME) = UPPER('cop_tliquidateins');
SELECT AUTO_INCREMENT INTO auto_incr2 FROM information_schema.TABLES WHERE TABLE_SCHEMA = SCHEMA() and UPPER(TABLE_NAME) = UPPER('cop_thisliquidateins');
    IF (auto_incr2 > auto_incr1) THEN
      SET NEW.liquidate_ins_id = last_insert_id(auto_incr2);
    END IF;
  END IF;
END;
delimiter ;
````
### 参考资料
[Mysql bugs](https://bugs.mysql.com/bug.php?id=199)

[触发器方案](https://www.slicewise.net/index.php?id=82)
