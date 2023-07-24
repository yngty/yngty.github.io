---
title: tcpdump 抓包
date: 2023-07-19 13:49:02
tags:
- tools
- network
categories:
- Network
---

`tcpdump` 命令行抓包工具。

```
tcpdump [ -AbdDefhHIJKlLnNOpqStuUvxX# ] [ -B buffer_size ]
        [ -c count ] [ --count ] [ -C file_size ]
        [ -E spi@ipaddr algo:secret,...  ]
        [ -F file ] [ -G rotate_seconds ] [ -i interface ]
        [ --immediate-mode ] [ -j tstamp_type ] [ -k (metadata_arg) ]
        [ -m module ]
        [ -M secret ] [ --number ] [ --print ]
        [ -Q packet-metadata-filter ] [ -Q in|out|inout ]
        [ -r file ] [ -s snaplen ] [ -T type ] [ --version ]
        [ -V file ] [ -w file ] [ -W filecount ] [ -y datalinktype ]
        [ -z postrotate-command ] [ -Z user ]
        [ --time-stamp-precision=tstamp_precision ]
        [ --micro ] [ --nano ]
        [ expression ]
```