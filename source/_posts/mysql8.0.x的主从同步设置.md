---
title: MySQL8.0x的主从同步设置
date: 2020-08-14 10:47:37
tags:
- MySQL
categories:
- 运维
---
手动导出主数据库sql文件导入到从数据库中保持数据和表结构的一致性。
```
mysqldump　-t　数据库名　-uroot　-p　>　xxx.sql　
```
主数据库
```
mysql>create user 'syncdata'@'localhost' identified by '1234567';
mysql>grant replication slave on *.* to 'syncdata'@'127.0.0.1';
mysql> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000001 |     1454 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
```
从数据库
```
mysql --ssl-mode=DISABLED -h172.17.0.2 -uroot -p123456 --get-server-public-key
#先获取主数据库秘钥，因为mysql修改了验证方式，不然会报错。
```
```
mysql> change master to master_host='ip', master_port=16123, master_user='syncdata',master_password='1234567',master_log_file='mysql-bin.000001';
mysql> show slave status\G; #查看从数据库状态
```
指定数据库同步
```
从服务器
mysql> change replication filter replicate_do_db=(base_name);
```