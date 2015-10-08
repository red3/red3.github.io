---
layout: post
title:  "iOS数据持久化的思考"
date:   2015-06-24 22:19:36
---
### 缘由

之前做佳缘一对一iOS App的时候，需要解决网络数据做离线缓存，现在红娘经纪人项目考虑用数据库作为缓存解决方案。后来觉得有必要思考下数据持久化该采取的方案。

### 说到数据持久化，到底有哪些方案可以采取

- NSUserDefault
- NSKeyedArchive
- Write To File (NSString, NSArray, NSData, NSDictionary)
- 数据库
当数据有本地存取的需求的时候，如何保证数据在本地的合理安排？

### 应该采取哪种方案

### 业界是怎么做的

-[YTKKeyValueStore](https://github.com/yuantiku/YTKKeyValueStore)

[唐巧](www.devtang.com)开源的基于sqlite的Key-Value式的存储，猿题库的项目采用该方案。

-[SQLiteManager4iOS](https://github.com/casatwy/SQLiteManager4iOS)

[casatwy](http://casatwy.com/)基于SQLiteMananger这个repo做的迁移方案，解决版本迁移问题。

