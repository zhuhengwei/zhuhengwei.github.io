---
title: mysql批量备份多表
date: 2015-07-04 14:26:45
categories: [数据库管理]
tags: [shell,mysql, 脚本,备份]
---

### 1. 批量备份 mysql 多表，每个表一个.sql ###
对于每天生成的表，需要把以前的表备份，即定义起始的表和结束的表，针对有规律的表的备份。

```bash
	#!/bin/bash
	IPAddress=8.8.8.8
	DB_pwd=mysqltest   #数据库密码；
	DB_name=mysqltest   #数据库名称；
	table_name="zhw_"  #数据表名称前缀；
	
	Start=20150112 # 每天生成的表 的开始的第一个
	End=20150610   #每天生成的表 最后需要备份的一个
	
	
	BACKUP_tmp=/usr/local/mysqltestbackup/backupsql     #备份文件临时存放目录；
	BACKUP_path=/usr/local/mysqltestbackup    #备份文件压缩打包存放目录；
	
	
	DATERange=`seq -f%.f $Start $End | date -f - "+%Y%m%d" 2>/dev/null` #日期范围
	
	for i in $DATERange
	  do
	    mysqldump -h$IPAddress  -umysqltest -p$DB_pwd $DB_name $table_name$i > $BACKUP_tmp/$table_name$i-$(date +%Y%m%d%H%M%S).sql
	  sleep 1
	done
	  sleep 10
	#将备份数据打包；
	    tar -cvzf $BACKUP_path/$DB_name-$table_name-$(date +%Y%m%d%H%M%S).tar.gz $BACKUP_tmp/*
	exit 0

```
### 2. 备份数据库里的所有的表，每个表一个.sql ###
1）首先需要从information_schema表中获得每个表的名称,保存为文件，如all_tables
```sql
	 SELECT TABLE_NAME FROM information_schema.`TABLES`
	WHERE TABLE_SCHEMA ='mysqltest' AND TABLE_TYPE ='BASE TABLE'
```
2） 执行脚本.
```bash
	#!/bin/bash
	IPAddress=8.8.8.8
	DB_pwd=mysqltest   #数据库密码；
	DB_name=mysqltest   #数据库名称；
	
	BACKUP_tmp=/usr/local/mysqltestbackup/backupsql     #备份文件临时存放目录；
	BACKUP_path=/usr/local/mysqltestbackup    #备份文件压缩打包存放目录；
	
	
	for i in `cat all_tables`
	  do
	    mysqldump -h$IPAddress  -umysqltest -p$DB_pwd $DB_name $i > $BACKUP_tmp/$i-$(date +%Y%m%d%H%M%S).sql
	  sleep 1
	done
	  sleep 10
	#将备份数据打包；
	    tar -cvzf $BACKUP_path/$DB_name-$(date +%Y%m%d%H%M%S).tar.gz $BACKUP_tmp/*
	exit 0

```
### 3. 备份整个数据库，一个.sql,不赘述。  ###
### 4. 备份多表，无规律。若把表名准备好，结合第二个脚本使用，也可以每个表单个.sql。###
```bash
	mysqldump -h$IPAddress  -umysqltest -p$DB_pwd $DB_name 表1 表2 表3.... > $BACKUP_tmp/$i-$(date +%Y%m%d%H%M%S).sql
```
