---
title: TLS1.3(四) Handshake Protocol
tags:
- SSL/TLS
---

# `4. Handshake Protocol`

握手协议用于协商连接的安全参数。把握手消息传递给 `TLS` 记录层，`TLS` 记录层把它们封装到一个或多个 `TLSPlaintext` 或 `TLSCiphertext` 中，然后按照当前活动连接状态的规定进行处理和传输。

```c
   enum {
          client_hello(1),
          server_hello(2),
          new_session_ticket(4),
          end_of_early_data(5),
          encrypted_extensions(8),
          certificate(11),
          certificate_request(13),
          certificate_verify(15),
          finished(20),
          key_update(24),
          message_hash(254),
          (255)
      } HandshakeType;

      struct {
          HandshakeType msg_type;    /* handshake type */
          uint24 length;             /* remaining bytes in message */
          select (Handshake.msg_type) {
              case client_hello:          ClientHello;
              case server_hello:          ServerHello;
              case end_of_early_data:     EndOfEarlyData;
              case encrypted_extensions:  EncryptedExtensions;
              case certificate_request:   CertificateRequest;
              case certificate:           Certificate;
              case certificate_verify:    CertificateVerify;
              case finished:              Finished;
              case new_session_ticket:    NewSessionTicket;
              case key_update:            KeyUpdate;
          };
      } Handshake;
```

协议消息必须以[Section 4.4.1](https://datatracker.ietf.org/doc/html/rfc8446#section-4.4.1)中定义的顺序发送，这也已经在第2章[[Section 2](https://datatracker.ietf.org/doc/html/rfc8446#section-2)]的图中展示了。对端如果收到了不按顺序发送的握手消息必须使用一个"unexpected_message"警报来中止握手。
新的握手消息类型已经由IANA指定并在第11章中描述。

## `4.1.  Key Exchange Messages`

密钥交换消息用于确定客户端和服务器的安全能力，并建立共享的秘密，包括用于保护握手其余部分和数据的流量密钥。

### `4.1.1.  Cryptographic Negotiation`

在 `TLS` 中，密码协商的过程是由客户端在其 `ClientHello` 中提供以下四组选项：

- 一个密码套件列表，指示客户端支持的 `AEAD` 算法/ `HKDF` 哈希对。
- 一个 `"supported_groups"`（第4.2.7节）扩展，指示客户端支持的`（EC）DHE`组，以及一个包含这些组中一些或全部的`（EC）DHE`共享的 `"key_share"`（第4.2.8节）扩展。
- 一个 `signature_algorithms`（第4.2.3节）扩展，指示客户端可以接受的签名算法。还可以添加一个 `"signature_algorithms_cert"`（第4.2.3节）扩展，以指示与证书相关的签名算法。
- 一个 `"pre_shared_key"`（第4.2.11节）扩展，其中包含客户端已知的对称密钥标识的列表，以及一个指示可以与 `PSK` 一起使用的密钥交换模式的 `"psk_key_exchange_modes"`（第4.2.9节）扩展。


如果服务器未选择预共享密钥（`PSK`），那么这些选项的前三个是完全正交的：服务器独立选择密码套件、`(EC)DHE组` 和密钥共享以建立密钥，以及签名算法(`signature algorithm`)/证书对(`certificate pair`)用于对客户端进行身份验证。如果接收到的支持的组（`"supported_groups"`）与服务器支持的组之间没有重叠，则服务器必须通过（`"handshake_failure"`）或（`"inadequate_security"`警报中止握手。

如果服务器选择预共享密钥（`PSK`），则必须从客户端的
`"psk_key_exchange_modes"` 扩展所指示的集合中选择密钥建立模式（目前有 `仅PSK`或带 `(EC)DHE`）。请注意，如果可以在不使用（`EC）DHE` 的情况下使用 `PSK`，则在 `"supported_groups"` 参数中的不重叠不一定是致命的错误，就像之前讨论过的非 `PSK` 场景一样。

如果服务器选择了一个`（EC）DHE` 组，而客户端在初始的 `ClientHello` 中没有提供兼容的 `"key_share"` 扩展，那么服务器必须以 `"HelloRetryRequest"`（[Section 4.1.4](https://datatracker.ietf.org/doc/html/rfc8446#section-4.1.4)）消息作出响应。

如果服务器成功地选择了参数且没有发送 `HelloRetryRequest`，表明 `ServerHello`中所选的参数如下：
  - 如果正在使用 `PSK`，则服务器将发送一个指示所选密钥的 `"pre_shared_key"` 扩展
  - 当使用`（EC）DHE` 时，服务器还将提供一个 `"key_share"` 扩展。如果未使用 `PSK`，则始终使用`（EC）DHE`和基于证书的身份验证。
  - 当通过证书进行身份验证时，服务器将发送 `Certificate`（[Section 4.3.2](https://datatracker.ietf.org/doc/html/rfc8446#section-4.4.2)）和 `CertificateVerify`（[Section 4.3.3](https://datatracker.ietf.org/doc/html/rfc8446#section-4.4.3)）消息。在由本文档定义的`TLS 1.3`中，始终使用 `PSK` 或证书之一，但不能同时使用。未来的文档可能会定义如何将它们一起使用。

如果服务器无法协商出一个支持的参数集（即客户端和服务器参数之间没有重叠），则必须通过 `"handshake_failure"` 或 `"insufficient_security"` 致命警报（[Section 6](https://datatracker.ietf.org/doc/html/rfc8446#section-6)）中止握手。

###  `ClientHello`

当客户端首次连接到服务器时，必须将 `ClientHello` 作为其第一个 `TLS` 消息发送。当服务器通过 `HelloRetryRequest` 响应其 `ClientHello` 时，客户端还将发送一个 `ClientHello` 。在这种情况下，客户端必须发送相同的 `ClientHello` ，除非有以下修改：
- 如果 `HelloRetryRequest` 带有一个 `"key_share"` 扩展，则用一个包含来自指定组的单个 `KeyShareEntry` 的列表替换共享列表。
- 如果存在 `"early_data"` 扩展([Section 4.2.10](https://datatracker.ietf.org/doc/html/rfc8446#section-4.2.10)）, 则删除该扩展。在 `HelloRetryRequest` 之后，不允许使用 `"early_data"`。
- 如果在 `HelloRetryRequest` 中提供了 `"cookie"` 扩展，则包括该扩展。
- 如果存在 `"pre_shared_key"` 扩展，则通过重新计算 `"obfuscated_ticket_age"` 和绑定器值来更新它，并（可选地）移除与服务器指定的密码套件不兼容的任何 `PSK`。
- 选择性增加、删除或更改`"padding"` 扩展的长度 [RFC 7685](https://datatracker.ietf.org/doc/html/rfc7685) 。
- 将来定义的其他 `HelloRetryRequest` 扩展允许的修改。

由于 `TLS 1.3` 禁止重新协商，如果服务器已经协商了 `TLS 1.3` 并在任何其他时间收到 `ClientHello`，它必须使用 `"unexpected_message"` 警报终止连接。

如果服务器使用先前版本的 `TLS` 建立了 `TLS` 连接并在重新协商中收到 `TLS 1.3` 的 `ClientHello`，它必须保留先前的协议版本。特别是，它不得协商 `TLS 1.3`。

#### `ClientHello` 消息结构:
```c
uint16 ProtocolVersion;
opaque Random[32];

uint8 CipherSuite[2];    /* Cryptographic suite selector */

struct {
    ProtocolVersion legacy_version = 0x0303;    /* TLS v1.2 */
    Random random;
    opaque legacy_session_id<0..32>;
    CipherSuite cipher_suites<2..2^16-2>;
    opaque legacy_compression_methods<1..2^8-1>;
    Extension extensions<8..2^16-1>;
} ClientHello;
```

- `legacy_version`：在以前版本的TLS里，这个字段用于版本协商和表示client支持的最高版本号。经验表明很多server没有适当地实现版本协商，导致“版本容忍”，会使server拒绝本来可接受的版本号高于支持的ClientHello。在TLS 1.3中，client在"supported_versions" 扩展(4.2.1节)中表明版本偏好，且legacy_version字段必须设置为TLS1.2的版本号0x0303。TLS 1.3 ClientHello的legacy_version为0x0303，supported_versions扩展值为最好版本0x0304（关于后向兼容的细节见附录D）

- `random`： 由一个安全随机数生成器产生的32字节随机数，更多信息见附录C。

- `legacy_session_id`： 之前的TLS版本支持"会话恢复"特性，在TLS 1.3中此特性与预共享密钥合并了(见2.2节)。client需要将此字段设置为TLS 1.3之前版本的server缓存的 session ID。在兼容模式下(见附录D.4)此字段必须非空，所以一个不能提供TLS 1.3之前版本会话的client必须产生一个32字节的新值。这个值不必是随机的但应该是不可预测的以避免实现上固定为一个具体的值(也被称为僵化)。否则，它必须被设置为一个0长度向量(例如，一个0字节长度字段)。

- `cipher_suites`： client支持的对称密码族选项列表，具体有记录保护算法（包括密钥长度）和与HKDF一起使用的hash算法，这些算法以client偏好降序排列。值定义在附录B.4。如果列表中包含server不认识，不支持，或不想使用的密码族，server必须忽略这些密码族并且正常处理其余的密码族。如果client试图确定一个PSK，应该通告至少一个密码族以表明PSK关联hash算法。

- `legacy_compression_methods`：`TLS 1.3` 之前的版本支持使用在此字段中发送的受支持压缩方法列表进行压缩。对于每个`TLS 1.3` 的 `ClientHello`，此向量必须只包含`1`字节并设置为`0`，对应于TLS早期版本中的 `"null"` 压缩方法。 如果接收到 `TLS 1.3` 的 `ClientHello`，而此字段中包含任何其他值，服务器必须通过发送 `"illegal_parameter"` 警报中止握手。请注意，`TLS 1.3` 服务器可能会接收到包含其他压缩方法的 `TLS 1.2` 或先前的 `ClientHello`（如果协商了先前版本），服务器必须遵循相应先前版本 `TLS` 的程序。服务器必须忽略未识别的扩展。

- `extension`：客户端通过在扩展字段中发送数据来向服务器请求扩展功能。实际的 `"Extension"` 格式在 [第4.2节](https://datatracker.ietf.org/doc/html/rfc8446#section-4.2) 中定义。在 `TLS 1.3` 中，使用某些扩展是强制性的，因为功能已经移入扩展，以保持 `ClientHello` 与先前版本的 `TLS` 的兼容性。服务器必须忽略未识别的扩展。

`TLS` 的所有版本都允许在 `compression_methods` 字段之后选择性地添加一个扩展字段。`TLS 1.3` 的 `ClientHello` 消息始终包含扩展（至少包含 `"supported_versions"`，否则将被解释为 `TLS 1.2` 的 `ClientHello` 消息）。然而，`TLS 1.3` 服务器可能会收到来自先前版本的 `TLS` 的 `ClientHello` 消息，其中没有扩展字段。可以通过确定在`ClientHello` 末尾的 `compression_methods` 字段之后是否有字节来检测是否存在扩展。请注意，这种检测可选数据的方法与`TLS` 通常的具有可变长度字段的方法不同，但它用于与在定义扩展之前的 `TLS` 兼容。`TLS 1.3` 服务器将需要首先执行此检查，仅在 `"supported_versions"` 扩展存在时尝试协商 `TLS 1.3`。如果要协商 `1.3` 之前的 `TLS` 版本，服务器必须检查消息是否在 `legacy_compression_methods` 之后不包含数据，或者是否包含没有后续数据的有效扩展块。如果不是，则必须通过使用 `"decode_error"` 警报中止握手。

如果客户端通过扩展请求额外的功能，而服务器未提供这些功能，客户端可以选择中止握手。

在发送 `ClientHello` 消息后，客户端等待 `ServerHello` 或 `HelloRetryRequest` 消息。如果正在使用早期数据，客户端可以在等待下一个握手消息的同时传输早期应用数据（[Section 2.3](https://datatracker.ietf.org/doc/html/rfc8446#section-2.3)）。


### `4.1.3. Server Hello`

如果服务器能够根据 `ClientHello` 协商出一组可接受的握手参数，它将以此消息作为响应 `ClientHello` 消息，以继续握手过程。

#### `Server Hello`消息结构

```c
struct {
    ProtocolVersion legacy_version = 0x0303;    /* TLS v1.2 */
    Random random;
    opaque legacy_session_id_echo<0..32>;
    CipherSuite cipher_suite;
    uint8 legacy_compression_method = 0;
    Extension extensions<6..2^16-1>;
} ServerHello;
```

### `Hello Retry Request`

## `Extensions`

许多`TLS`消息包含标签-长度-值（`tag-length-value`）编码的扩展结构

```c
struct {
    ExtensionType extension_type;
    opaque extension_data<0..2^16-1>;
} Extension;

enum {
    server_name(0),                             /* RFC 6066 */
    max_fragment_length(1),                     /* RFC 6066 */
    status_request(5),                          /* RFC 6066 */
    supported_groups(10),                       /* RFC 8422, 7919 */
    signature_algorithms(13),                   /* RFC 8446 */
    use_srtp(14),                               /* RFC 5764 */
    heartbeat(15),                              /* RFC 6520 */
    application_layer_protocol_negotiation(16), /* RFC 7301 */
    signed_certificate_timestamp(18),           /* RFC 6962 */
    client_certificate_type(19),                /* RFC 7250 */
    server_certificate_type(20),                /* RFC 7250 */
    padding(21),                                /* RFC 7685 */
    pre_shared_key(41),                         /* RFC 8446 */
    early_data(42),                             /* RFC 8446 */
    supported_versions(43),                     /* RFC 8446 */
    cookie(44),                                 /* RFC 8446 */
    psk_key_exchange_modes(45),                 /* RFC 8446 */
    certificate_authorities(47),                /* RFC 8446 */
    oid_filters(48),                            /* RFC 8446 */
    post_handshake_auth(49),                    /* RFC 8446 */
    signature_algorithms_cert(50),              /* RFC 8446 */
    key_share(51),                              /* RFC 8446 */
    (65535)
} ExtensionType;
```

```
+--------------------------------------------------+-------------+
   | Extension                                        |     TLS 1.3 |
   +--------------------------------------------------+-------------+
   | server_name [RFC6066]                            |      CH, EE |
   |                                                  |             |
   | max_fragment_length [RFC6066]                    |      CH, EE |
   |                                                  |             |
   | status_request [RFC6066]                         |  CH, CR, CT |
   |                                                  |             |
   | supported_groups [RFC7919]                       |      CH, EE |
   |                                                  |             |
   | signature_algorithms (RFC 8446)                  |      CH, CR |
   |                                                  |             |
   | use_srtp [RFC5764]                               |      CH, EE |
   |                                                  |             |
   | heartbeat [RFC6520]                              |      CH, EE |
   |                                                  |             |
   | application_layer_protocol_negotiation [RFC7301] |      CH, EE |
   |                                                  |             |
   | signed_certificate_timestamp [RFC6962]           |  CH, CR, CT |
   |                                                  |             |
   | client_certificate_type [RFC7250]                |      CH, EE |
   |                                                  |             |
   | server_certificate_type [RFC7250]                |      CH, EE |
   |                                                  |             |
   | padding [RFC7685]                                |          CH |
   |                                                  |             |
   | key_share (RFC 8446)                             | CH, SH, HRR |
   |                                                  |             |
   | pre_shared_key (RFC 8446)                        |      CH, SH |
   |                                                  |             |
   | psk_key_exchange_modes (RFC 8446)                |          CH |
   |                                                  |             |
   | early_data (RFC 8446)                            | CH, EE, NST |
   |                                                  |             |
   | cookie (RFC 8446)                                |     CH, HRR |
   |                                                  |             |
   | supported_versions (RFC 8446)                    | CH, SH, HRR |
   |                                                  |             |
   | certificate_authorities (RFC 8446)               |      CH, CR |
   |                                                  |             |
   | oid_filters (RFC 8446)                           |          CR |
   |                                                  |             |
   | post_handshake_auth (RFC 8446)                   |          CH |
   |                                                  |             |
   | signature_algorithms_cert (RFC 8446)             |      CH, CR |
   +--------------------------------------------------+-------------+
```
## `4.3 Server Parameters`

服务器的下两条消息，`EncryptedExtensions` 和 `CertificateRequest`，包含了决定握手的其余部分的来自服务器的信息。这些消息是使用从 `server_handshake_traffic_secret` 派生的密钥进行加密的。

### `4.3.1. Encrypted Extensions`

在所有的握手过程中，服务器必须在发送ServerHello消息后立即发送EncryptedExtensions消息。这是第一个使用从server_handshake_traffic_secret派生的密钥加密的消息。

EncryptedExtensions消息包含可以被保护的扩展，即那些不需要用于建立加密上下文但与个别证书无关的扩展。客户端必须检查EncryptedExtensions以查看是否存在任何被禁止的扩展，如果发现任何禁止的扩展，客户端必须通过发送"illegal_parameter"警报中止握手。

#### 消息结构：

```c
struct {
    Extension extensions<0..2^16-1>;
} EncryptedExtensions;
```
- `extensions`: 一组扩展。有关更多信息，请参见[第4.2节](https://datatracker.ietf.org/doc/html/rfc8446#section-4.2)中的表格

### `4.3.2. Certificate Request`

服务器在使用证书进行身份验证时，可以选择性地请求客户端提供证书。如果发送此消息，必须在EncryptedExtensions之后发送。

## `4.4. Authentication Messages`

正如在第2节中讨论的那样，TLS通常使用一组共同的消息进行身份验证、密钥确认和握手完整性：Certificate、CertificateVerify和Finished。（PSK binders也执行密钥确认，方式类似。）这三个消息总是作为握手流的最后消息发送。Certificate和CertificateVerify消息仅在特定情况下发送，如下所定义。Finished消息始终作为认证块的一部分发送。这些消息使用从[sender]_handshake_traffic_secret派生的密钥进行加密。

## `4.5. End of Early Data`

如果服务器在EncryptedExtensions中发送了一个"early_data"扩展，客户端在收到服务器的Finished消息后必须发送一个EndOfEarlyData消息。如果服务器在EncryptedExtensions中未发送"early_data"扩展，则客户端不得发送EndOfEarlyData消息。该消息表示所有的0-RTT application_data消息（如果有）已经传输，并且接下来的记录受到握手流量密钥的保护。服务器不得发送此消息，而接收到此消息的客户端必须通过发送"unexpected_message"警报终止连接。此消息使用从client_early_traffic_secret派生的密钥进行加密。

## `4.6. Post-Handshake Messages`

`TLS` 还允许在主握手之后发送其他消息。这些消息使用握手内容类型，并且在适当的应用流量密钥下加密。

### `4.6.1. New Session Ticket Message`
### `4.6.2. Post-Handshake Authentication`
### `4.6.3. Key and Initialization Vector Update`

