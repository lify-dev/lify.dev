---
title: sqlserver启用CDC
date: 2021-05-31 10:00:00
tags: [sqlserver, cdc]
categories: [sqlserver, cdc]
---



**要启动cdc必须先开启SQL Server代理**



**启动数据库捕获**

```sql
-- 开启cdc
USE [databaseName]
EXECUTE sys.sp_cdc_enable_db
GO
-- 如果开启数据库捕获报错误:15517，使用下面的方法处理
ALTER AUTHORIZATION ON DATABASE::[databaseName] TO [sa]

-- 查看是否设置成功，1代表开启cdc
SELECT is_cdc_enabled  FROM SYS.databases WHERE name='databaseName'
```



**设置跟踪表**

为当前数据库中指定的源表启用变更数据捕获。对表启用变更数据捕获时，应用于此表的每个数据操纵语言 (DML) 操作的记录都将写入事务日志中。变更数据捕获进程将从日志中检索此信息，并将其写入可通过使用一组函数访问的更改表中。

**格式：**

```sql
USE databaseName;
GO
EXEC sys.sp_cdc_enable_table
@source_schema ='dbo', --表所属的架构名
@source_name='t_student',	--  被监控的数据表 
@role_name=null, -- 是用于控制更改数据访问的数据库角色的名称。
@capture_instance=NULL, -- 是用于命名变更数据捕获对象的捕获实例的名称，这个名称在后面的存储过程和函数中需要经常用到。
@supports_net_changes=1, -- 指示是否对此捕获实例启用净更改查询支持如果此表有主键，或者有已使用 @index_name 参数进行标识的唯一索引，则此参数的默认值为 1。否则，此参数默认为 0。
@index_name=null, -- 用于唯一标识源表中的行的唯一索引的名称。index_name 为 sysname，并且可以为 NULL。如果指定，则 index_name 必须是源表的唯一有效索引。如果指定 index_name，则标识的索引列优先于任何定义的主键列，就像表的唯一行标识符一样。
@captured_column_list=null, -- 被监控的列，NULL时监控所有列
@filegroup_name=default, -- 是要用于为捕获实例创建的更改表的文件组。
@allow_partition_switch=1; -- 指示是否可以对启用了变更数据捕获的表执行 ALTER TABLE 的 SWITCH PARTITION 命令。allow_partition_switch 为 bit，默认值为 1。

-- 查询表是否启动跟踪,值为1标示已启动
SELECT is_tracked_by_cdc  FROM SYS.tables WHERE name='t_student'

```

**禁用表**

```sql
USE hellodb;
GO
EXEC sys.sp_cdc_disable_table  
    @source_schema = 'dbo',  
    @source_name = 't_student',  
    @capture_instance = 'all';  
```

