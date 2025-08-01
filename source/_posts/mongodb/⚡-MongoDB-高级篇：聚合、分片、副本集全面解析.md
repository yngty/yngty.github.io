---
title: ⚡ MongoDB 高级篇：聚合、分片、副本集全面解析
date: 2025-08-01 10:06:31
tags:
- mongodb
categories:
- mongodb
- 数据库
---

# 1. 聚合（Aggregation Framework）
聚合是 `MongoDB` 中最强大的分析与数据处理工具，相当于 `SQL` 中的 `GROUP BY`、`JOIN`、`WHERE`、`ORDER BY` 等组合。

## 1.1 聚合管道的基本原理
`MongoDB` 的聚合是一个数据流管道：

```bash
输入文档  →  $match  →  $group  →  $sort  →  $project  →  输出结果
```

每个阶段都会对数据进行筛选、计算、排序或重塑。

<!--more-->

## 1.2 常用聚合操作
**1) $match 过滤数据**

```js
db.orders.aggregate([
  { $match: { status: "completed" } }
])
```
**2) $group 分组统计**

```js
db.orders.aggregate([
  { $group: { _id: "$customerId", totalSpent: { $sum: "$amount" } } }
])
```

**3) $project 重塑字段**

```js
db.orders.aggregate([
  { $project: { _id: 0, orderId: 1, amountUSD: "$amount" } }
])
```

**4) $sort 排序**

```js
db.orders.aggregate([
  { $sort: { amount: -1 } }
])
```
**5) $lookup 关联集合（类似 SQL JOIN）**

```js
db.orders.aggregate([
  {
    $lookup: {
      from: "customers",
      localField: "customerId",
      foreignField: "_id",
      as: "customerInfo"
    }
  }
])
```

## 1.3 聚合示例：统计用户总消费金额并排序
```js
db.orders.aggregate([
  { $match: { status: "completed" } },
  { $group: { _id: "$customerId", total: { $sum: "$amount" } } },
  { $sort: { total: -1 } }
])
```

# 2. 副本集（Replica Set）
## 2.1 副本集的作用
- **高可用**：主节点宕机，自动选举新的主节点。

- **读写分离**：读操作可以分担到从节点。

- **数据冗余**：多节点保存相同数据，防止单点故障。

## 2.2 副本集架构图
```sql
       +----------------+
       |   Primary      |
       |  (主节点)      |
       +-------+--------+
               |
    +----------+----------+
    |                     |
+---+---+             +---+---+
|Secondary|           |Secondary|
|(从节点) |           |(从节点) |
+---------+           +---------+
```

## 2.3 启动副本集

假设我们有三台服务器：

```bash
mongod --replSet rs0 --port 27017 --dbpath /data/node1 --bind_ip localhost
mongod --replSet rs0 --port 27018 --dbpath /data/node2 --bind_ip localhost
mongod --replSet rs0 --port 27019 --dbpath /data/node3 --bind_ip localhost
```
## 2.4 初始化副本集
```js
rs.initiate({
  _id: "rs0",
  members: [
    { _id: 0, host: "localhost:27017" },
    { _id: 1, host: "localhost:27018" },
    { _id: 2, host: "localhost:27019" }
  ]
})
```

# 3. 分片（Sharding）
## 3.1 分片的作用
- **水平扩展（`Scale-out`）**：将数据分布到多台服务器，打破单机性能瓶颈。

- **海量数据存储**：支持 `TB` 甚至 `PB` 级数据。

- **自动路由**：客户端不需要关心数据在哪个分片。

## 3.2 分片架构图
```lua
+---------+       +------------+
| Client  | <---> | mongos 路由 |
+---------+       +------------+
                      |
    ---------------------------------------
    |                 |                   |
+-------+        +-------+           +-------+
|Shard1 |        |Shard2 |           |Shard3 |
|数据分片|        |数据分片|           |数据分片|
+-------+        +-------+           +-------+
```
## 3.3 启动分片集群
```bash
# 启动配置服务器
mongod --configsvr --replSet configReplSet --dbpath /data/config --port 27019

# 启动分片服务器
mongod --shardsvr --dbpath /data/shard1 --port 27018
mongod --shardsvr --dbpath /data/shard2 --port 27020

# 启动 mongos 路由
mongos --configdb configReplSet/localhost:27019 --port 27017
```
## 3.4 添加分片
```js
sh.addShard("localhost:27018")
sh.addShard("localhost:27020")
```
## 3.5 启用集合分片
```js
sh.enableSharding("mydb")
sh.shardCollection("mydb.users", { userId: 1 })
```
# 4. 高级最佳实践
1. **聚合优化**

    - `$match` 尽量放在前面，减少后续数据处理量。

    - 使用 `$project` 只保留必要字段。

2. **副本集优化**

    - 主节点存储用 `SSD`，保证写入性能。

    - 从节点可以用于只读分析，减轻主节点压力。

3. **分片优化**

    - 选择高基数字段作为分片键（如 `userId`）。

    - 避免使用时间戳作为唯一分片键（会导致热点写入）。

4. **监控与运维**

    - 使用 `mongostat`、`mongotop` 实时监控。

    - 部署 `MongoDB Atlas` 或 `Prometheus + Grafana` 进行可视化监控。

# 5. 总结
- **聚合* 让 `MongoDB` 拥有 `SQL` 级别的数据分析能力。

- **副本集* 提供高可用和读写分离。

- **分片* 让 `MongoDB` 可以支撑超大规模数据。

如果你是中小型应用，可以只部署 **副本集**，确保高可用。
如果你是大型应用，需要水平扩展，那就必须上 **分片 + 副本集** 架构。
