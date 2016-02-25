---
title: mysql随机更新表里的数据
date: 2015-07-10 20:55:45
categories: [数据库管理]
tags: [mysql,随机]
---

### mysql 随机更新数据库里的几条数据 ### 
有些情况，比如下图表中数据，除了id不一样，其余都一样。如下图：
![](http://img.blog.csdn.net/20150710205210086)

出于某些原因，我们不能得到其id，但是想**更新其中的某些数据,不是全部**，因为数据都是一样的，所以更新其中的哪一条都是一样的。 
那么要更新其中的比如3条数据怎么办呢？
<!-- more -->
**sql语句如下：**
```sql
	update test_table set type = "aa"
	order by rand() limit 3;
```

运行完如上语句，表中数据如下：
![](http://img.blog.csdn.net/20150710205332571)