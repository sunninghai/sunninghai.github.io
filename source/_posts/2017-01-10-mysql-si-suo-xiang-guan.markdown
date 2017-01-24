---
layout: post
title: "mysql 死锁相关"
date: 2017-01-10 10:33:18 +0800
comments: true
categories: 
---

###查询 正在执行的事务


SELECT * FROM information_schema.INNODB_TRX

###查看正在锁的事务


SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCKS; 

###查看等待锁的事务


SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCK_WAITS;

show engine innodb status ;

