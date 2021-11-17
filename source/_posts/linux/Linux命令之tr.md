---
title: Linux命令之tr
date: 2021-11-17 11:08:28
tags:
- Linux
categories:
- Linux
---

`Linux` 中 `tr` 命令用于转换或删除文件中的字符。

### 语法

```shell
$ tr [OPTION] SET1 [SET2]
```

### 选项

```shell
-c, --complerment：反选设定字符。也就是符合 SET1 的部份不做处理，不符合的剩余部份才进行转换;
-d, --delete：删除所有属于第一字符集的字符；
-s, --squeeze-repeats：把连续重复的字符以单独一个字符表示；
-t, --truncate-set1：先删除第一字符集较第二字符集多出的字符;
```

### 参数

- 字符集1(`SET1`)：指定要转换或删除的原字符集。当执行转换操作时，必须使用参数 "字符集2"指定转换的目标字符集。但执行删除操作时，不需要参数“字符集2”；
- 字符集2(`SET2`)：指定要转换成的目标字符集。

### 实例

1. 小写字母转换为大写字母:

    ```shell
    ➜ echo "HELLO WORLD" | tr 'A-Z' 'a-z'
    hello world
    ```

2. 删除字符：

    ```shell
    ➜ echo "hello 123 world 456" | tr -d '0-9'
    hello  world

    ➜ echo "hello 123 world 456" | tr -cd '0-9'
    123456
    ```
3. 压缩字符

    ```shell
    ➜ echo "hello          world" | tr -s '[:space:]'
    hello world
    
    ➜  share echo "hellooooo worldddddddddddd" | tr -s 'od' 
    hello world
    ```

### 常用的字符类

```
[:alnum:]：字母和数字
[:alpha:]：字母
[:cntrl:]：控制（非打印）字符
[:digit:]：数字
[:graph:]：图形字符
[:lower:]：小写字母
[:print:]：可打印字符
[:punct:]：标点符号
[:space:]：空白字符
[:upper:]：大写字母
[:xdigit:]：十六进制字符  
```