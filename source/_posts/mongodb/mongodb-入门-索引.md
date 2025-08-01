---
title: mongodb 入门-索引
date: 2023-08-26 14:30:06
tags:
- mongodb
categories:
- mongodb
- 数据库
---


# 索引

- 对文档部分内容进行排序的数据结构
- 加快文档查询和文档排序的速度
- 复合键索引只能支持前缀子查询

## 索引操作

### 创建索引

```
db.<collection>.createIndex(<keys>, <options>)
```
- `<keys>` 文档指定了创建索引的字段
- `<options>` 创建索引的参数和设定索引的特性

#### 创建一个单键索引

```shell
db.accountWithIndex.createIndex({name: 1})
```
#### 创建一个复合键索引

```shell
db.accountWithIndex.createIndex({name: 1, balance: -1})
```
#### 创建一个多键索引，创建在数组字段上, 数组字段中的每一个元素都会在多键索引中创建一个键

```shell
db.accountWithIndex.createIndex({currency: 1})
```
#### 列出集合中的索引

```shell
db.accountWithIndex.getIndexes()

```
<!--more-->
### 删除索引

如果需要更改某些字段上已经创建的索引必须首先删除原有索引，再重新创建新索引，否则新索引不会包含原有字段

#### 使用索引名字删除
```shell
db.accountWithIndex.dropIndex("name_1")
```
#### 使用索引定义删除
```shell
db.accountWithIndex.dropIndex( { name: 1, balance: -1 })
```


## 索引类型

- 单键索引
- 复合键索引
- 多键索引

## 索引特性

### 唯一性

```shell
db.accountWithIndex.createIndex({balance:1},{unique:true})
```

如果已有文档中的某个字段出现了重复性，就不可以在这个字段上创建唯一性索引

如果新增的文档不包含唯一性索引字段，只有第一篇缺失该字段的文档可以被写入数据库，索引中该文档的键值被默认为null
### 稀疏性

只将包含索引字段的文档加入到索引中(即使索引键字段为null)

```shell
db.accountWithIndex.createIndex({balance:1}, { sparse:true})
```
如果同一个索引既具有唯一性，又具有稀疏性，就可以保存多篇缺失索引键值的文档了

```shell
db.accountWithIndex.createIndex({balance:1}, { sparse:true, unique:true})

```
复合键索引也可以具有稀疏性，在这种情况下，只有在缺失复合键所包含的所有字段的情况下，文档才不会加入到索引中

### 生存时间

```shell
db.accountWithIndex.createIndex({lastAccess:1},{expireAfterSeconds:20})
```

**复合键索引不具有生存时间特性**，当索引键是包含日期元素的数组字段时，数组中**最小**的的日期将被用来计算文档是否过期。数据库使用一个后台线程来监测和删除过期文档，删除操作可能有一定的延迟。

## 查询分析

### 检视索引的效果 `explain()`

- 命令操作格式:
    ```
    db.<collection>.explain().<method(...)>
    ```
- 可以使用 `explain()` 进行分析的命令包括 `aggregate()`，`count()`，`distinct()`，`find()`，`group()`，`remove()`，`update()`

- `explain()`返回结果中包含的 `winningPlan`字段表示数据库的查询方法
    - `stage`
        - `COLLSCAN` 完整查询整个集合
        - `FETCH` `IXSCAN` 通过索引查询，返回完整文档
        -` PROJECTION_COVERED`  `IXSCAN`, 直接使用索引存在的字段，无序完整文档，没有 `FETCH` 阶段

没有 `IXSCAN` 阶段，都耗费资源

举个例子，`explain` 排序:

```shell
db.accountWithIndex.explain().find().sort({name:1, balance: -1})
```
## 索引的选择

- 如何创建一个合适的索引
- 索引对数据库写入操作的影响
