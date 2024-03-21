---
title: QUIC 协议初探
date: 2024-03-08 14:24:01
tags:
- QUIC
categories:
- Network
---

Package Type

- Long Header Packets
    - Version Negotiation Packet
    - Initial Packet
    - 0-RTT Packet
    - Handshake Packet
    - Retry Packet
- Short Header Packets
    - 1-RTT Packet


```
   +=====================+=================+==================+
   | Packet Type         | Encryption Keys | PN Space         |
   +=====================+=================+==================+
   | Initial             | Initial secrets | Initial          |
   +---------------------+-----------------+------------------+
   | 0-RTT Protected     | 0-RTT           | Application data |
   +---------------------+-----------------+------------------+
   | Handshake           | Handshake       | Handshake        |
   +---------------------+-----------------+------------------+
   | Retry               | N/A             | N/A              |
   +---------------------+-----------------+------------------+
   | Version Negotiation | N/A             | N/A              |
   +---------------------+-----------------+------------------+
   | Short Header        | 1-RTT           | Application data |
   +---------------------+-----------------+------------------+
```
