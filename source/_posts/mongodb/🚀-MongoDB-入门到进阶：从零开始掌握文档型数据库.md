---
title: 🚀 MongoDB 入门到进阶：从零开始掌握文档型数据库
date: 2025-08-01 09:51:39
tags:
- mongodb
categories:
- mongodb
- 数据库
---

# 1. 什么是 MongoDB？
`MongoDB` 是一种 **面向文档的 `NoSQL` 数据库**，与传统的关系型数据库（`MySQL`、`PostgreSQL`）不同，它不使用表格和行，而是使用 **BSON**（二进制 `JSON`）存储数据。
简单来说，它是一个 **JSON 存储与查询引擎**。

## MongoDB 的核心特性：
- **文档型存储**：每条数据是一个 JSON 对象。

- **无固定模式（Schema-less）**：字段可以灵活增加或减少。

- **高扩展性**：支持分片（`Sharding`）与副本集（`Replica Set`）。

- **强大的查询语言**：支持聚合（`Aggregation`）、全文检索（`Full-Text Search`）。

- **天然适合大数据与高并发场景**。

<!--more-->

# 2. MongoDB 的基本概念
|关系型数据库   |	MongoDB | 对应概念	说明 |
| :---  | :---    | :---     |
|Database|	Database    |	数据库，存放集合的容器|
|Table   |	Collection  |	集合，存放文档|
|Row	 |  Document    |	文档，一条数据记录|
|Column  |	Field       |	字段，文档的属性|
|Index	 |  Index       |	索引，提高查询性能|



# 3. 安装与启动

**macOS（Homebrew）**

```bash
brew tap mongodb/brew
brew install mongodb-community@7.0
brew services start mongodb-community@7.0
```
**Ubuntu / Debian**
```bash
sudo apt-get install -y mongodb
sudo systemctl start mongodb
sudo systemctl enable mongodb
```
**连接**

```bash
mongosh
```

# 4. 基本操作
## 4.1 创建数据库与集合
```js
// 切换数据库（没有会自动创建）
use mydb

// 创建集合
db.createCollection("users")
```
## 4.2 插入数据
```js
db.users.insertOne({ name: "Alice", age: 25 })
db.users.insertMany([
  { name: "Bob", age: 30 },
  { name: "Charlie", age: 28 }
])
```
## 4.3 查询数据
```js
// 查询全部
db.users.find()

// 条件查询
db.users.find({ age: { $gt: 25 } })

// 指定字段
db.users.find({}, { name: 1, _id: 0 })

// 排序 & 限制
db.users.find().sort({ age: -1 }).limit(2)
```
## 4.4 更新数据
```js
db.users.updateOne(
  { name: "Alice" },
  { $set: { age: 26 } }
)
```
## 4.5 删除数据
```js
db.users.deleteOne({ name: "Charlie" })
```
# 5. 高级功能
## 5.1 索引
```js
// 创建索引
db.users.createIndex({ name: 1 })

// 查看索引
db.users.getIndexes()
```
## 5.2 聚合（Aggregation）
```js
db.users.aggregate([
  { $match: { age: { $gt: 25 } } },
  { $group: { _id: null, avgAge: { $avg: "$age" } } }
])
```
## 5.3 文本搜索

```js
db.articles.createIndex({ content: "text" })
db.articles.find({ $text: { $search: "MongoDB" } })
```

# 6. 副本集与分片
- **副本集（Replica Set）**：多台服务器存储相同数据，实现高可用与自动故障切换。

- **分片（Sharding）**：将数据水平拆分到多台服务器，适合海量数据场景。

# 7. Python 操作 MongoDB 示例

```python
from pymongo import MongoClient

client = MongoClient("mongodb://localhost:27017/")
db = client.mydb
users = db.users

# 插入
users.insert_one({"name": "David", "age": 35})

# 查询
for user in users.find({"age": {"$gt": 25}}):
    print(user)

# 更新
users.update_one({"name": "David"}, {"$set": {"age": 36}})

# 删除
users.delete_one({"name": "David"})
```
# 8. 最佳实践
1. **为高频查询字段加索引**，避免全表扫描。

2. ****控制文档大小**，避免超过 `16MB` 限制。

3. ****合理分库分集合**，提升性能。

4. ****聚合管道尽量减少** `$lookup`，避免跨集合过多关联。

5. ****使用连接池**（例如 `Python` 的 `MongoClient` 默认支持）。

# 9. 总结

`MongoDB` 以其灵活的文档存储和强大的查询能力，成为现代 `Web` 应用、日志分析、大数据处理的重要数据库之一。
如果你需要：

- 存储结构灵活的数据

- 快速开发 `MVP`

- 支持高并发与大规模数据

那么 `MongoDB` 将会是一个非常不错的选择。
