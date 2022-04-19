---
title: 解决 由于没有公钥，无法验证下列签名 :NO_PUBKEY
date: 2022-04-19 22:41:41
tags:
- Linux
- apt
categories:
- Linux
---

```shell
➜ sudo apt update 
命中:1 https://pro-driver-packages.uniontech.com eagle InRelease
获取:2 http://mirrors.tuna.tsinghua.edu.cn/ubuntu hirsute InRelease [269 kB]                                 
命中:3 http://packages.microsoft.com/repos/code stable InRelease                                             
命中:4 https://home-packages.chinauos.com/home plum InRelease                                                
命中:5 https://home-packages.chinauos.com/home plum/beta InRelease   
命中:6 https://home-packages.chinauos.com/printer eagle InRelease
错误:2 http://mirrors.tuna.tsinghua.edu.cn/ubuntu hirsute InRelease
  由于没有公钥，无法验证下列签名： NO_PUBKEY 871920D1991BC93C
命中:7 https://home-store-img.uniontech.com/appstore eagle InRelease
正在读取软件包列表... 完成
W: GPG 错误：http://mirrors.tuna.tsinghua.edu.cn/ubuntu hirsute InRelease: 由于没有公钥，无法验证下列签名： NO_PUBKEY 871920D1991BC93C
E: 仓库 “http://mirrors.tuna.tsinghua.edu.cn/ubuntu hirsute InRelease” 没有数字签名。
N: 无法安全地用该源进行更新，所以默认禁用该源。
N: 参见 apt-secure(8) 手册以了解仓库创建和用户配置方面的细节。
```

执行：

```shell
➜ sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 871920D1991BC93C
```