---
layout: post
title: "mysql注意"
date: 2017-01-03 19:25:28 +0800
comments: true
categories: mysql
---

1. group by 会自动排序. group by 字段 如果字段没有索引,explain会出现 Using filesort,在对性能要求比较高的时候，可以不自动排序 group by 字段 order by null;
2. 尽量避免 or 查询，因为如果条件中字段没有索引，会导致用不到任何索引，全表扫描;
3. 尽可能字段不要not null,不要认为null 不需要空间，其需要额外的空间;
4. 涉及到ip地址的存储用unsigned int 不要用varchar
5. 尽量少计算、join、排序，有些可以移到应用程序中去做;
6. 尽量用join代替自查询;
7. union all 替换union, 特别是我们可以确认不会出现重复数据的时候;
8. 避免类型转换，如果类型不一致，在传入参数时进行转换，不然可能会导致不走索引;
9. 关联表时关联字段数据类型长度要一致，不然也可能会导致关联表时索引未使用;
10. 尽量早的过滤数据，特别是在关联表的时候；