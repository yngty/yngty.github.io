---
title: mongodb 入门-CRUD
date: 2023-08-27 15:34:59
tags:
- mongodb
categories:
- mongodb
---
# 文档主键

## 文档主键 `_id`

文档主键 `_id` 是每篇文档必备的字段,具有以下特性:

- 文档主键的唯一性
- 支持所有数据类型(数组除外)
- 复合主键

## 对象主键 `ObjectId`

当我们不提供主键，`MongoDB` 自动为我们生成的默认对象主键 `ObjectId`

- 默认的文档主键
- 可以快速生成的 `12` 字节 `id`
- 包含创建时间


`ObjectId` 默认值

```shell
test> ObjectId()
ObjectId("64eeed51b64b9d7e95e6b8ea")
```
手动设置 `ObjectId` 的值
```shell
test> ObjectId("123456789011123456789011")
ObjectId("123456789011123456789011")
```

提取 `ObjectId` 的创建时间
```shell
test> ObjectId("123456789011123456789011").getTimestamp()
ISODate("1979-09-05T22:51:36.000Z")
```
<!--more-->
## 复合主键

使用文档作为文档主键

举个例子:
```shell
db.accounts.insert({
    _id: {accountNo: "001", type: "savings"},
    name: "irene",
    balance: 100
})
```
**复合主键仍然要满足文档主键的唯一性**，需要字段值和顺序完全一致才算重复。

下面是不同的 `_id`:
```
_id: {accountNo: "001", type: "savings"}
```
```
_id: { type: "savings", accountNo: "001"}
```

# 创建文档

## 创建单个文档

```
db.<collection>.insertOne(
    <document>,
    {
        writeConcern: <document>
    }
)
```
- `<collection>`： 文档集合名
- `<document>`： 准备写入数据库的文档
    - `Json` 文档格式
        ```
        {
            _id: "account1",
            name: "alice",
            balance: 100
        }
        ```
- `writeConcern`： 文档定义了本次文档创建操作的安全写级别简单来说，安全写级别用来判断一次数据库写入操作是否成功。安全写级别越高，丢失数据的风险就越低，然而写入操作的延迟也可能更高。如果不提供`writeConcern` 文档，`MongoDB` 使用默认的安全写级别

举个例子:
```shell
db.accounts.insertOne(
    {
        _id: "account1",
        name: "alice",
        balance: 100
    }
)
```

- `insertOne`、`insetMany`、`insert` 会自动创建相应集合

- `_id` 字段**不能重复**，省略创建文档中的 `_id` 字段，会自动生成 `_id`。

## 创建多个文档

```
db.<collection>.insertMany(
    [<document1>, <document2>, ...],
    {
        writeConcern: <document>,
        ordered: <boolean>
    }
)
```

- `ordered`: `MongoDB` 是否按照顺序来写入这些文档。默认为 `true`。

举个例子:
```shell
db.accounts.insertMany(
    [
        {name: "charlie", balance: 500},
        {name: "david", balance: 200}
    ]
)
```

### 错误处理

在**顺序**写入时，一旦遇到错误，操作便会退出，剩余的文档无论正确与否，都不会被写入。 

在**乱序**写入时，即使某些文档造成了错误，剩余的正确文档仍然会被写入

## 创建单个或多个文档

```
db.<collection>.insert(
    <document or array of documents>,
    {
        writeConcern: <document>,
        ordered: <boolean>
    }
)
```
`insert` 命令既可以提供单个 `document` 也可以提供 `document` 数组, 返回结果不同， 正确返回 `WriteReult` 对象

举个例子:
```shell
db.accounts.insert(
    {
        name: "george",
        balance: 1000
    }
)

```

返回结果:

```
WriteResult({ "nInserted": 1 })
// nInserted: 插入数量
```

## 三个命令的区别

- 返回结果不同
- `insertOne` 和 `insertMany` 命令不支持 `db.collection.explain()` 命令，`insert` 命令支持。


## save

```
db.<collection>.save(
    <document>,
    {
        writeConcern: <document>
    }
)
```

当 `db.collection.save()` 命令处理一个新文档的时候，会调用 `db.collection.insert()` 命令

# 读取文档

```
db.<collection>.find(<query>, <projection>)
```
- `<query>`: 查询文档
- `<projection>`: 定义了对读取结果进行的投射

## 读取全部文档

既不筛选，也不使用投射

```shell
db.accounts.find()
```

格式友好:

```shell
db.accounts.find().pretty()
```

## 筛选文档

### 匹配查询

```shell
db.accounts.find({name: "alice", balance: 100})
```
嵌入文档的查询:

```shell
db.accounts.find({"_id.type": "savings"})
```

### 比较操作符

常用的比较操作符包括:

- `$eq` 匹配字段值相等的文档
- `$ne` 匹配字段值不等的文档
    - **也会筛选出不包含查询字段的文档**
- `$gt` 匹配字段值大于查询值的文档
- `$gte` 匹配字段值大于或等于查询值的文档
- `$le` 匹配字段值小于查询值的文档
- `$lte` 匹配字段值小于或等于查询值的文档

操作命令格式:
```
{ <field>: {$<operator>: <value> }}
```

举个例子:

读取 `alice` 的文档：
```shell
db.accounts.find({ name: { $eq: "alice" }})
```
读取余额大于 `100` 的文档：
```shell
db.accounts.find({ balance: { $gt: 100 }})
```

- `$in` 匹配字段值与任一查询值相等的文档
- `$nin` 匹配字段值与任何查询值都不相等的文档
    -  **也会筛选出不包含查询字段的文档**

操作命令格式:
```
{ <field>: { $in: [<value1>, <value2>, ...] }}
```

举个例子:

读取 `alice` 和 `charlie` 的银行账户文档:

```shell
db.accounts.find({ name: { $in: ["alice", "charlie"] }})
```
### 逻辑操作符

- `$not` 匹配筛选条件不成立的文档
    - 操作命令格式:
        ```
        { <field>: { $not: {<operator-expression> } }}
        ```
    - **也会筛选出不包含查询字段的文档**
- `$and` 匹配筛选条件全部成立的文档
     - 操作命令格式:
        ```
        { $and: [{<operator-expression1>}, {<operator-expression2>}, ...]}
        ```
- `$or` 匹配筛选条件至少一个成立的文档
    - 操作命令格式:
        ```
        { $or: [{<operator-expression1>}, {<operator-expression2>}, ...]}
        ```
    - 当所有的筛选条件都是 `$eq` 操作符时，`$or` 和 `$in` 效果一样
- `$nor` 匹配多个筛选条件全部不成立的文档
    - 操作命令格式:
        ```
        { $nor: [{<operator-expression1>}, {<operator-expression2>}, ...]}
        ```
    - **也会筛选出不包含查询字段的文档**


举个例子：

读取余额不小于 `500` 的银行账户文档：
```shell
db.accounts.find(
    {balance: {$not: {$lt: 500 }}}
)
```

读取余额大于 `100` 且用户姓名排在 `fred` 之后的银行账户文档：
```shell
db.accounts.find(
    { and: {
        {balance: {$gt: 100}},
        {name: {$lt: "fred" }}
    }}
)
```

当筛选条件应用在不同字段上时，可以省略 `$and` 操作符, 上面等价于:

```shell
db.accounts.find(
    {balance: {$gt: 100}},
    {name: {$lt: "fred" }}
)
```

当筛选条件应用在同一字段上时，也可以省略 `$and` 操作符。例如：

查询余额大于 `100` 并且小于 `500` 的银行账户:
```shell
db.accounts.find(
    {balance: {$gt: 100, $lt: 500 }}
)
```

### 字段操作符

- `$exists` 
    - 查询包含字段值的文档
    - 操作命令格式:
        ```
        { field: { $exists: <boolean> }}
        ```

举个例子:
读取文档主键 `_id.type` 存在并且不等于 `checking`
```shell
db.accounts.find(
    {"_id.type": { $ne: "checking", $exists: true }}
)
```

- `$type` 
    - 查询包含字段值类型的文档
    - 操作命令格式:
        ```
        { field: { $type: <BSON type> }}
        { field: { $type: [<BSON type1>, <BSON type2>, ... ]}}
        ```
举个例子:

读取文档主键是字符串的银行账户文档
```shell
db.accounts.find(
    {_id: { $type: "string" }}
)
```

读取文档主键是对象主键或者是复合主键的银行账户文档
```shell
db.accounts.find(
    {_id: { $type: ["ObjectId", "object"] }}
)
```

也可以使用 `BSON` 类型序号作为 `$type` 操作符的参数
```shell
db.accounts.find(
    {_id: { $type: 2 }}
)
```
### 数组操作符

- `$all` 
    - 筛选数组中所有元素满足查询条件的文档
    - 操作命令格式:
        ```
        { field: { $all: [<value1>, <value2>, ... ]}}
        ```
- `$elemMatch` 
    - 筛选数组中任一一个元素满足查询条件
    - 操作命令格式:
        ```
        { field: { $elemMatch: {<query1>, <query2>, ... }}}
        ```

举个例子
```shell
db.accounts.find(
    {contact: { $elemMatch: { $gt: 100, %lt: 200 }}}
)
```
`$all` 可以 与 `$elemMatch` 结合使用:
读取包含一个在 `1000` 至 `2000` 之间和一个在 `2000` 至 `3000` 之间的联系电话的银行账户文档:
```shell
db.accounts.find(
    {contact: { $all: [
        {$elemMatch:{$gt: 1000, $lt: 2000}},
        {$elemMatch:{$gt: 2000, $lt: 3000}},
    ]}}
)
```

### 运算操作符


# 更新文档

```
db.collection.update(<query>, <update>, <options>)
```
- `<query>` 筛选条件
- `<update>` 更新文档
- `<options>` 设置参数

## 更新整篇文档   

如果 `<update>` 文档不包含任何更新操作符，`db.collection.update()` 将会使用 `<update>`文档直接替换集合中符合 `<query>` 文档筛选条件的文档

```shell
db.account.update({name:"alice" }, {name : "alice", "balance":123})
```

注意问题: 

- 文档主键 `_id` 是不可以更改的
- 只有**第一篇**符合 `<query>` 文档符合筛选条件的文档会被更新

## 更新特定字段

更新文档特定字段，需要使用到文档更新操作符，如下:

- `$set` 更新或新增字段
- `$unset` 删除字段
- `$rename` 重命名
- `$inc` 加减字段值
- `$mul` 相乘字段值
- `$min` 比较减小字段值
- `$max` 比较增大字段值
 
 ### $set

```
{ $set: { <filed1>: <value1>, ...}}
```

举个例子: 

```shell
db.accounts.update(
    {name: "jack"},
    {
        $set: 
        {
            balance: 3000,
            info: {
                dateOpened: new Date("2016-05-18T16:00:00Z"),
                branch: "branch1" 
            },
        }
    }
)
```

#### 更新或新增内嵌文档的字段

```shell
db.accounts.update(
    {name: "jack"},
    {
        $set: 
        {
            "info.dateOpened": new Date("2017-01-01T16:00:00Z")
        }
    }
)
```

#### 更新或新增数组字段

在 `$set` 中使用**数组名加下标**，如果向现有数组字段范围以外的位置添加新值，数组字段的长度会被扩大，未被赋值的数组成员将被设置为 `null`

举个例子 `jack`的  `contact` 数组中默认有 `3` 个元素: 

- 更新  `jack`的 `contact` 数组中第一个元素

    ```shell
    db.accounts.update(
        {name: "jack"},
        {
            $set: 
            {
                "contact.0": "666666"
            }
        }
    )
    ```
- 更新 `jack`的 `contact` 数组中第四元素

    在 `$set` 中使用数组名加下标 `3`， 默认只有 `3` 个，这个操作会在数组后边新增一个元素.

    ```shell
    db.accounts.update(
        {name: "jack"},
        {
            $set: 
            {
                "contact.3": "77777"
            }
        }
    )
    ```

- 更新  `jack`的 `contact` 数组中第六个元素

    在 `$set `中使用数组名加下标 `5`， 现在只有 `4` 个，这个操作会在数组后边新增两个元素，第五个元素为 `null` 值，第六个为新增值

    ```shell
    db.accounts.update(
        {name: "jack"},
        {
            $set: 
            {
                "contact.5": "99999"
            }
        }
    )
    ```

### $unset 

删除字段, 传入任何值都一样，默认设置为 `""`, 格式如下:

```shell
{ $unset: {field1: "", ...}}
```

例如删除 `jack` 的银行账户余额和开户地点:

```shell
db.accounts.update(
    {name:"jack"},
    {
        $unset:
        {
            balance: "",
            "info.branch": ""
        }
    }
)
```

- 删除内嵌文档的字段
    - 同更新内嵌文档字段方式
- 删除数组内的字段
    - 同更新内嵌文档字段方式。删除数组中元素成功时，这个元素不会被删除，只会被赋以 `null` 值，而**数组的长度不会改变**。

**如果 `$unset` 命令中的字段不存在，那么文档内容将保持不变**


### $rename

重命名文档字段

```
{ $rename: {<field1>: <newName1>, <field2>:<newName2>, ...}}
```

- 如果 `$rename` 命令中的字段不存在，那么文档内容将保持不变
- 如果新的字段名已经存在，那么原有的这个字段会被覆盖

**当 `$rename` 命令中的新字段存在时，`$rename` 命令会先 `$unset` 旧字段，然后再 `$set` 新字段**



- 重命名内嵌文档的字段
    - 普通重命名字段名
        - 同更新内嵌文档字段方式
    - 更新内嵌文档中字段的位置
        ```shell
        db.accounts.update(
            {name: "karen" }
            {
                $rename:
                {
                    "info.branch": "branch",
                    "balance": "info.balance"
                }
            }
        )
        ```

**$rename 命令中的旧字段和新字段都不可以指向数组元素**

### $inc & $mul

更新字段值, `$inc` 加减字段值(正负数)，`$mul` 相乘字段值，格式如下：

```
{ $inc: { <field1> : <value1>, ...}}

{ $mul: { <field1> : <value1>, ...}}
```

举个例子 `david` 账户的 `notYetExist` 值加  `10`:
```shell
db.accounts.update(
    {name: "david"}，
    {
        $inc:
        {
            notYetExist: 10
        }
    }
)
```
**使用注意**: 

- `$inc`、 `$mul` 只能应用在**数字**字段上
- 当更新字段不存在时
    - `$inc` 会创建字段，并且将字段值设定为命令中的**增减值**
    - `$mul` 会创建字段，但是把字段值设为 `0`

### $min & $max

比较后更新值，`$min` 比较后保留较小字段值 `$max` 比较后保留较大字段值：

```
{ $min: { <field1> : <value1>, ...}}

{ $max: { <field1> : <value1>, ...}}
```

举个例子:

会将 `info.balance` 的当前值同 `5000` 比较，保留较小的值。

```shell
db.accounts.update(
    {name: "david"}，
    {
        $min:
        {
            "info.balance": 5000
        }
    }
)
```

**使用注意**: 

- 可以在任何可以比较大小的字段上使用
- 当更新字段不存在时
    - `$min` 和 `$max` **都会创建字段**，并且将字段值设定为命令中的**更新值**。
- 被更新的字段类型和更新值类型不一致
    - `$min` 和 `$max`会按照 `BSON` 数据类型排序规则进行比较。
    > 不同 `BSON` 类型的值时，`MongoDB` 使用以下**从低到高**的比较顺序：\
    \
        MinKey (internal type)\
        Null\
        Numbers (ints, longs, doubles, decimals)\
        Symbol, String\
        Object\
        Array\
        BinData\
        ObjectId\
        Boolean\
        Date\
        Timestamp\
        Regular Expression\
        MaxKey (internal type)
        

## 更新数组操作符

- `$addToSet` 向数组中增添元素
- `$pop `从数组中移除元素
- `$pull` 从数组中有选择性的移出元素
- `$pullAll` 从数组中有选择性的移出元素
- `$push` 向数组中增添元素

**以上字段只能用到数组字段上，否则会收到错误**

### $addToSet

`$addToSet` 向数组中增添元素:

```
{ $addToSet: { <field1> : <value1>, ...}}
```
- 如果要插入的值不存在数组字段中
    - 新增字段会被添加到原文档中
- 如果要插入的值已经存在数组字段中
    - `$addToSet` 不会再添加重复值。 **使用 `$addToSet`插入数组和文档时，插入值中的字段顺序和已有值重复时，才被算着重复值被忽略**

`$addToSet` 会将数组插入被更新的数组字段中，成为内嵌数组。

```shell
db.accounts.update(
    {name: "karen"},
    {
        $addToSet: {contact: ["contact1", "contact2"]}
    }
)
```
如果想要将多个元素直接添加到数组字段中，则需要使用 `$each` 操作符

```shell
db.accounts.update(
    {name: "karen"},
    {
        $addToSet: {contact: { $each:   ["contact1", "contact2"] }}
    }
)
```

### $pop

`$pop` 从数组字段中删除元素, **只能删除数组中的第一个(-1)和最后一个元素(1)**, 删除掉数组中最后一个元素后，会留下**空数组**。

```
{ $pop: { <field>: < -1 | 1 >, ...}}
```

从 `karen` 的账户文档中删除最后一个联系方式:

```shell
db.accounts.update(
    {name: "karen"},
    { $pop: { contact: 1 }}
)
```

### $pull

`$pull` 从数组中删除符合值或者条件的元素

```
{ $pull: { <field1>: <value|condition>, ... }}
```

从 `karen`的联系方式中删除包含 `hi`字母的元素:

```shell
db.accounts.update(
    { name: "karen"},
    { $pull: { contact: { $regex: /hi/ }}}

)
```

要删除数组元素是内嵌数组，可以使用 `$elemMatch` 对内嵌数组进行筛选:

从 `karen`的联系方式中删除包含 `22222222`的内嵌数组元素:
```shell
db.accounts.update(
    { name: "karen"},
    { $pull: { contact: { $elemMatch: { $eq: "22222222" }}}}  
)
```

### $pullAll

```
{ $pullAll: { <field1>: [<value1, value2>, ...], ... }}
```

其实
```
{ $pullAll: { <field1>: [<value1, value2>] }}
```

相当于

```
{ $pull: { <field1>: {$in: [<value1, value2>] }}}
```

**如果要删除的元素是一个数组，数组的元素的值和排列顺序都必须和要被删除的数组完全一样**。

举个例子:
```shell
db.accounts.update(
    {name: "lawrence"},
    { $pullAll: {contact: [["333333", "222222"]]}}
)
```

如果删除的元素是一个文档
- `$pullAll` 命令只会删除字段和排列顺序都**完全匹配**的文档元素
- `$pull` 命令会删除匹配的文档元素，模糊度更高(**形成包含关系，或者能通过字段名找到**)。

### $push

`$push `向数组中添加元素

```
{ $push: { <field1> : <value1>, ...}}
```

`$push` 和 `addTosSet`命令相似地方:

- 如果要插入的值不存在数组字段中
    - 新增字段会被添加到原文档中
- 可以搭配 `$each`
    ```
       db.accounts.update(
            {name: "lawrence"},
            { $push: {
                newArray: {
                    $each:[2, 3, 4]
                }
            }}
        )
    ```
`$push` 比 `addTosSet`增强地方:

- 搭配 `$each` 实现更复杂的操作
    - 使用 `$position` 操作符将元素插入到数组的指定位置, `$position` 的值 `0` 表示 从数组第一个位置开始插入，`-2` 表示从数组最后一个元素往前走 `2` 个开始插入。
        ```shell
        db.accounts.update(
            {name: "lawrence"},
            { $push: {
                newArray: {
                    $each:["pos1", "pos2"],
                    $position: 0
                }
            }}
        )
        ```
    - 使用 `$sort` 排序
        ```shell
        db.accounts.update(
            {name: "lawrence"},
            { $push: {
                newArray: {
                    $each:["sort1"],
                    $sort: 1
                }
            }}
        )
        ```
    - 如果插入的元素是内嵌文档，也可以根据内嵌文档的字段值排序
        ```shell
        db.accounts.update(
            {name: "lawrence"},
            { $push: {
                newArray: {
                    $each:[{key: "sort", value: 100}, {key: "sort1", value: 200}],
                    $sort: { value: -1 } // 只排序插入的内嵌文档
                }
            }}
        )
        ```
    - 不想插入，只想对数组中的字段进行排序:
        ```shell
        db.accounts.update(
            {name: "lawrence"},
            { $push: {
                newArray: {
                    $each:[],
                    $sort: -1
                }
            }}
        )
        ```
    - 使用 `$slice` 截取部分数组

        插入 `slice1` 元素后，保留从后往前数 `8` 个元素:
        ```shell
        db.accounts.update(
            {name: "lawrence"},
            { $push: {
                newArray: {
                    $each:["slice1"],
                    $slice: -8
                }
            }}
        )
        ```
    - 不想插入，只想截取部分数组:

        保留数组中的前 `6` 个元素:
        ```shell
        db.accounts.update(
            {name: "lawrence"},
            { $push: {
                newArray: {
                    $each:[],
                    $slice: 6
                }
            }}
        )
        ```

`$position`, `$sort`, `$slice` 可以一起使用, 他们的执行顺序是 `$position > $sort >  $slice`。**写在命令中的操作顺序并不重要，并不会影响命令的执行顺序**。

```shell
db.accounts.update(
    {name: "lawrence"},
    { $push: {
        newArray: {
            $each:["push1", "push2"],
            $position: 2,
            $sort: -1,
            $slice: 5
        }
    }}
)
```

### `$` 占位符

`$` 是数组中第一个符合筛选条件的数组元素的占位符, 搭配更新操作符使用，可以对满足筛选条件的数组元素进行更新。

```
db.collection.update(
    { <array>: <query selector> },
    { <update operator> : { "<array>.$": value }}
)
```

举个例子，将 `lawrence` 中 `newArray` 中的 `pos2` 替换为 `updated`:
```shell
db.accounts.update(
    {
        name: "lawrence",
        newArray: "pos2"
    },
    {
        $set: {
            "newArray.$": "updated"
        }
    }
)
```

### `$[]` 占位符

`$[]` 指代数组中的所有元素
```
{ <update operator> : { "<array>.$[]": value }}
```

举个例子，将 `lawrence` 账户中的 `contact`数组中的第一个内嵌数组，全部替换为 `999999`:

```shell
db.accounts.update(
    {name: "lawrence"},
    {
        $set: {
            "contact.0.$[]": 999999
        }
    }
)
```
## options
 
### multi


``` 
{ multi: <boolean> }
```

默认情况下，即使筛选条件对应了多篇文档，`update` 命令仍然只会更新**一篇**文档

设置 `multi` 为 `true` 选项可以更新所有符合筛选条件的文档, 注意，`MongoDB` 只能保证**单个**文档操作的原子性，不能保证**多个**文档操作的原子性。多个文档操作时的原子性需要 `MongoDB 4.0` 版本引入的事务功能进行操作。

举个例子：
```shell
db.accounts.update(
    {},
    {
        $set: {
            currency: "USD"
        },
        { multi: true}
    }
)
```

### upsert

``` 
{ upsert: <boolean> }
```

更新或者创建文档

在默认情况下，如果 `update` 命令中的筛选条件没有匹配任何文档，则不会进行任何操作

将 `upsert` 设置为` true`， 如果 `update` 命令中的筛选条件没有匹配任何文档，则会创建新文档

举个例子:
```shell
db.accounts.update(
    {name: "maggie"},
    {
        $set: {
            balance: 700
        },
        { upsert: true}
    }
)
```

**如果筛选条件中能推断出确定的字段，新创建的文档会包含筛选条件设计的字段**

## save

```
db.<collection>.save(<document>)
```

如果 `document` 文档中包含 `_id` 字段， `save()` 命令将会调用 `db.collection.update()` 命令`(upsert: true)`, `_id` 值存在就更新，不存在就创建新文档。

举个例子:

```shell
db.accounts.save(
    { _id : "account1", name: "alice", balance: 100 }
)
```

# 删除

- 删除集合
- 删除特定文档

## 删除文档

```
db.<collection>.remove(<query>, <options>)
```
- `<query>` 筛选条件
- `<options>` 设置参数


- 在默认情况下，`remove` 命令会删除**所有**复合筛选条件的文档

- 如果只想删除复合筛选条件的**第一篇**文档，可以使用`justOne` 选项

    ```shell
    db.accounts.remove(
        {balance: { $lt: 100} },
        {justOne: true }
    )
    ```

- 删除集合内的**所有**文档

    ```shell
    db.accounts.remove({})
    ```

举个例子, 删除 `balance` 等 `50` 的文档:
```shell
db.accounts.remove(
    {balance: 50 }
)
```

## 删除集合

`remove` 只会删除集合内所有的文档，但不会删除集合

`drop` 命令可以删除整个集合，包括集合中的所有文档，以及集合的索引

```
db.<collection>.drop({ writeConcern: <document> })
```
- `writeConcern` 定义了本次集合删除操作的安全写级别

举个例子:
```shell
db.accounts.drop()
````

> 如果集合中的文档数量较多，使用 `remove` 命令删除所有的文档效率不高，这种情况下推荐，使用 `drop` 命令删除集合，然后再创建空集合并创建索引。

