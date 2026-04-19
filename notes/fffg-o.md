---
timezone: UTC+8
---

# #fffg

**GitHub ID:** fffg-o

**Telegram:** 

## Self-introduction

EPF 实习计划

## Notes

<!-- Content_START -->
# 2026-04-19
<!-- DAILY_CHECKIN_2026-04-19_START -->
今日水一天，在忙
<!-- DAILY_CHECKIN_2026-04-19_END -->

# 2026-04-18
<!-- DAILY_CHECKIN_2026-04-18_START -->

今日学习SSZ序列化

**SSZ 概述**

-   **定位**：以太坊共识层（CL） 专用序列化 + 默克尔化方案，替代执行层（EL）的 RLP（仅节点发现协议仍用 RLP）。
    
-   **核心目标**：提升以太坊 PoS 共识的效率、安全性、可扩展性，支持复杂状态结构与高效哈希更新。
    
-   **关键特性**：确定性序列化、原生支持丰富类型、适配默克尔化、支持部分直接访问。
    

**SSZ vs RLP 核心对比**

| 维度 | RLP | SSZ |
| --- | --- | --- |
| 紧凑性 | 是 | 否 |
| 表达能力 | 弱（仅字节串 / 列表） | 强（原生支持整数、布尔、容器等） |
| 哈希友好 | 可哈希，但更新低效 | 高效哈希 / 重哈希，适配分片与无状态客户端 |
| 索引能力 | 无（访问内部数据 O (N)） | 弱，但支持部分直接访问 |
| 确定性 | 是 | 是 |
| 适用层 | 执行层 | 共识层（信标链） |

**SSZ 基本类型**

1\. 无符号整数 uintN

-   N 支持：8/16/32/64/128/256 位。
    
-   **序列化**：转小端序字节数组，长度 = N/8 字节。
    
-   **反序列化**：按小端序解析字节数组还原整数。
    

2\. 布尔值 Boolean

-   **序列化**：True→`01`，False→`00`（单字节）。
    
-   **反序列化**：`01`→True，`00`→False。
    

**SSZ 复合类型**

1.Vector（固定长度、同构集合）

-   定义：`Vector[类型, 长度]`，长度编译期确定。
    
-   **序列化**：逐个序列化元素 → 直接拼接，无长度前缀。
    
-   **反序列化**：按固定长度分段 → 逐个反序列化 → 组装。
    
-   示例：`Vector[uint64, 3](256,512,768)` → 三段 8 字节小端序拼接。
    

2\. List（可变长度、同构集合）

-   定义：`List[类型, 最大长度]`，实际长度≤最大值。
    
-   **序列化**：逐个序列化 → 拼接，内嵌长度元数据。
    
-   **反序列化**：按长度 / 偏移分段 → 还原列表。
    
-   与 Vector 区别：List 可变，序列化带长度；Vector 固定，无额外开销。
    

3\. Bitvector（固定长度比特序列）

-   定义：`Bitvector[N]`，N 为比特数，极致压缩布尔数组。
    
-   **序列化**：按字节内 LSB 优先打包；非 8 倍数则高位补 0。
    
-   优势：比 `Vector[boolean, N]` 节省约 8 倍空间。
    

4\. Bitlist（可变长度比特列表）

-   定义：`Bitlist[最大长度]`，可变长度比特集合。
    
-   **序列化**：LSB 优先打包 → 末尾加哨兵位 1 标记结束 → 补 0 凑整字节。
    
-   关键：哨兵位用于确定真实长度，长度为 8 倍数时需额外 1 字节。
    

5\. Container（结构体 / 复杂对象）

-   定义：类结构体，多字段有序组合，字段可为任意 SSZ 类型。
    
-   **序列化**：按 schema 顺序逐个序列化字段 → 拼接；变长字段用偏移量定位。
    
-   **反序列化**：按 schema 解析偏移与固定字段 → 递归还原嵌套结构。
    
-   典型应用：信标链 `IndexedAttestation`、`AttestationData` 等共识对象。
<!-- DAILY_CHECKIN_2026-04-18_END -->

# 2026-04-17
<!-- DAILY_CHECKIN_2026-04-17_START -->


今日学习CL Networking

共识客户端使用 libp2p 作为点对点协议， libp2p-noise 用于加密， discv5 用于对等发现， SSZ 用于编码，并且使用 Snappy 用于压缩。

**libp2p**是一种点对点的通信协议， 包含以下部分：

1.  传输层：支持TCP，也可以支持QUIC
    
2.  加密和识别：libp2p-noise 用于加密，并使用多地址约定，将多层寻址编码到单个路径中
    
3.  多路复用：允许多个独立的通信流在单个网络连接诶上并发运行。常用的多路复用器有两种：mplex和yamux
    
4.  消息传递：实现了Gossipsub和Req/Resp协议。
    

**libp2p 协议栈**

| 层 | 协议 | 目的 |
| --- | --- | --- |
| 🧠 应用层 | pubsub 、 gossipsub 、 ping 、自定义协议 | 运行用户自定义或内置逻辑（聊天、文件传输、发布/订阅等） |
| 🔀 多路复用层 | yamux ， mplex | 允许通过单个连接传输多个逻辑流 |
| 🔐 安全层 | noise 、 tls 、 secio （已弃用） | 对等连接进行加密和身份验证 |
| 🔌 传输层 | tcp 、 websockets 、 quic （具有多路复用和安全性）、 webrtc 、 webtransport | 处理机器之间的物理或虚拟数据传输 |
| 🌍 NAT/中继层 | relay 、 dcutr 、 autonat 、 pnet | 启用通过 NAT/防火墙或专用网络的连接 |
| 📡 发现层 | mdns 、 kademlia 、 rendezvous 、 identify | 在网络上寻找并了解同行 |

**ENR(以太坊节点记录)**

以太坊节点记录提供了一种结构化且灵活的方式来存储和共享以太坊点对点网络中的节点身份和连接详细信息。它是一种面向未来的格式，允许新节点之间更轻松地交换识别信息，并且是以太坊节点的首选网络地址格式。

它的核心组成部分包括：

1.  **签名** ：每条记录都使用身份方案（例如 secp256k1）进行签名，以确保真实性。
    
2.  **序列号** （seq）：一个 64 位无符号整数，每当记录更新时都会递增，以便对等方确定最新版本。
    
3.  **键/值对** ：记录以键值对的形式保存各种连接详细信息。
    

**discv5**

基于 UDP 协议，仅用于节点发现。它允许节点动态交换和更新 ENR，从而确保节点发现的及时性。它与 libp2p 并行运行。

**SSZ**

取代了共识层中除对等体发现协议之外所有执行层使用的 RLP 序列化。SSZ 的设计目标是确保确定性并高效地进行默克尔化。SSZ 可以被视为包含两个组件：序列化方案和默克尔化方案，后者旨在高效地处理序列化后的数据结构。
<!-- DAILY_CHECKIN_2026-04-17_END -->

# 2026-04-16
<!-- DAILY_CHECKIN_2026-04-16_START -->



今日学习CL板块overview

共识层的目的是确保全球数万个独立的节点保持相对的同步，每个节点保存的账本必须完全一致，而且必须快速达成一致。

POS

**验证者**本质上是PoS协议的参与者。他们负责提议和验证新的区块，确保区块链的完整性和安全性。

每个**时隙**为 12 秒，一个周期为 32 个时隙。每个时隙都分配有一个验证者来提议一个区块，而验证者委员会则负责验证该区块的有效性。

**委员会**是由至少 128 个验证者组成的团体，每个验证者被分配到不同的时隙以增强安全性。攻击者控制一个委员会三分之二成员的概率不到万亿分之一。

以太坊的权益证明（PoS）系统采用了一套全面的奖惩机制来激励验证者的行为并维护网络安全。

1.  测试者奖励
    
2.  测试人员处罚
    
3.  投资者面临的典型下行风险
    
4.  严厉惩罚和举报人奖励
    
5.  提案人奖励
    
6.  不活跃数据泄露惩罚
    

这一节我不太知道怎么总结，pow转向pos，pos的具体机制，并不是非常结构化的知识。

这期水了（

大学真是可恶，他把我关在教室，上了一天课，什么也没学到，却失去了时间，每天都是各种抽不开身的无意义的事，磨灭着我的心智，他没有教我怎么就业，只是将我关在这里，维持着表面的和平与稳定。

囚徒
<!-- DAILY_CHECKIN_2026-04-16_END -->

# 2026-04-15
<!-- DAILY_CHECKIN_2026-04-15_START -->




今日学习DevP2P，RLP序列化，EOF

devp2p是以太坊执行层的p2p网络协议栈，采用经典分层设计，分别为：

-   传输层（Transport Layer）：建立安全连接，加密通信，多协议复用。
    
-   发现层（Discovery Layer）：用于找到网络中的其他节点，包含discv4，discv5，DNS Discovery三种机制
    
-   应用层（Application Layer）：包括RLPx
    

PLPx只要负责以下三件事：

1.  加密通信，MAC校验
    
2.  连接管理
    
3.  多协议复用
    

**RLP序列化**

-   单字节编码：如果输入为单个字节，且其值介于 `0x00` 和 `0x7F` 之间，保持不变
    
-   短字节编码：如果字符串的长度在 1 到 55 字节之间，输出结果为字符串长度加上 `0x80` ，后面跟着字符串本身。
    
-   长字节编码：如果字符串长度超过 55 字节，字符串的长度被编码为大端字节数组，以 `0xb7` 为前缀，加上该长度数组的长度。
    
-   短列表编码：如果列表项的总编码有效载荷在 1 到 55 字节之间，列表前缀为 `0xc0` ，加上编码项的总长度
    
-   长列表编码：如果列表项的编码有效载荷总和超过 55 字节，与长字符串类似，有效载荷的长度采用大端格式进行编码，以 `0xf7` 为前缀，加上此长度数组的长度
    
-   空值：编码为0x80，空列表编码为0xc0
    

解码规则：确定数字的类型，先解码第一个字节，通过字节区别分辨数据类型长度。

**EOF**

EOF是一种可扩展的、版本化的 EVM 字节码容器格式，在部署时进行一次性验证。

EOF 的设计旨在解决原始 EVM 字节码格式的几个局限性：

-   更完善的验证：可在执行前对智能合约进行更全面的静态分析和验证。
    
-   提升执行效率：通过更清晰的代码段定义，优化 EVM 执行。
    
-   增强安全性：通过强制执行更严格的代码结构规则来减少攻击途径
    
-   面向未来：通过版本化容器为未来的 EVM 改进和功能奠定基础。
    

与传统 EVM 字节码的区别

| 特征 | 传统 EVM | EOF |
| --- | --- | --- |
| 容器结构 | 非结构化字节码 | 明确的章节 |
| 代码验证 | 有限的运行时 | 全面、执行前 |
| 跳跃机制 | 动态： JUMP/JUMPI | 包括静态跳跃 |
| 打字 | 没有任何 | 函数签名和类型 |
| 数据处理 | 与代码混合 | 单独的数据部分 |

今日彻底完成第一周的内容，初次接触底层的东西，有些算法真是晦涩难懂。如果作为了解的话，这一周我基本了解了，但如果作为面试的话，我觉得我仍有很大不足，高抽象的东西究竟怎么才算真正掌握？

总而言之，第二周要开始了。
<!-- DAILY_CHECKIN_2026-04-15_END -->

# 2026-04-14
<!-- DAILY_CHECKIN_2026-04-14_START -->





第一周还没学完....还是尽量加快速度吧

今日学习交易结构与JSON-RPC

交易的具体结构包含以下：

-   nonce
    
-   gasPrice
    
-   gasLimit
    
-   to
    
-   value
    
-   data
    
-   Signature
    

Receipts是 EVM 状态转换函数的输出产物。每个成功或失败执行的事务都会产生一个相应的Receipts，结构如下：

-   Transaction Type
    
-   Status
    
-   Gas Used
    
-   Logs
    
-   Logs Bloom
    

EIP-2718 通过“类型化信封”的概念，为交易和收据引入了一种统一且可扩展​​的格式。此扩展简化了新增交易和收据类型的流程，同时保持了与旧版交易的完全向后兼容性。

在 EIP-2718 之前，在 RLP 编码的限制下，添加新的交易类型需要使用繁琐的技术来区分它们，这导致系统设计不够稳定。EIP-2718 通过定义专用的交易类型前缀解决了这个问题。

EIP-2718 之后的交易遵循信封格式： `Typed Transaction = Transaction Type + Transaction Payload`

**JSON-RPC**

JSON-RPC 规范是一种基于 OpenRPC 的远程过程调用协议，它使用 JSON 编码。该协议允许调用远程服务器上的函数并返回结果。它是执行 API 规范的一部分，该规范提供了一组与以太坊区块链交互的方法。众所周知，它是用户使用客户端与网络交互的方式，也是共识层 (CL) 和执行层 (EL) 通过引擎 API 进行交互的方式。

以下为常见命名空间示例：

| Namespace | Description | Sensitive |
| --- | --- | --- |
| eth | The eth API allows you to interact with Ethereum. | Maybe |
| web3 | The web3 API provides utility functions for the web3 client. | No |
| net | The net API provides access to network information of the node. | No |
| txpool | The txpool API allows you to inspect the transaction pool. | No |
| debug | The debug API provides several methods to inspect the Ethereum state, including Geth-style traces. | No |
| trace | The trace API provides several methods to inspect the Ethereum state, including Parity-style traces. | No |
| admin | The admin API allows you to configure your node. | Yes |
| rpc | The rpc API provides information about the RPC server and its modules | No |

JSON-RPC使用16进制编码，与传输协议无关，http，websocket等都可以使用。
<!-- DAILY_CHECKIN_2026-04-14_END -->

# 2026-04-13
<!-- DAILY_CHECKIN_2026-04-13_START -->






今日有早八，上一天回到家，又上晚课啦。

实在是忙了一天，没时间学习，第二周的课也上了，我第一周还没看完呢....
<!-- DAILY_CHECKIN_2026-04-13_END -->

# 2026-04-12
<!-- DAILY_CHECKIN_2026-04-12_START -->







学习Data Structures 部分

-   Merkle Tree Trie ：（MTP）以太坊最核心的数据结构，在Merkle 树的基础上进行修改，使用键值对存储数据，使用于World State Trie，Transaction Trie ,Receipt Trie,Storage Trie部分。
    
-   Patricia Tries ：与 Merkle 树不同，它用于高效存储数据而不是验证。
    
    ![patricia-trie.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/fffg-o/images/2026-04-12-1775998757270-patricia-trie.png)

在这个官方的例子中，比如rubens所对应的值为4这个叶节点储存的值，通过字符的方式提高了存储效率和查找效率。

-   Transaction Trie :交易树中存储特定区块的所有交易，以键值对的形式存储，其中值包含以下部分：
    
    -   nonce ：一个跟踪交易顺序的数，每笔新交易nonce都会递增。
        
    -   maxPriorityFeePerGas：gas作为小费的最高
        
    -   gasLimit ： gas限制最高值
        
    -   maxFeePerGas： 交易中愿意支付的每单位 gas 的最高费用
        
    -   from ：发送方地址
        
    -   to： 接收方地址
        
    -   value：eth数量
        
    -   input data：输入的可选字段
        
    -   data：message调用的输入数据
        
    -   （v,r,s）：发送者的编码签名值
        
-   Receipt Trie：与交易树类似，用于验证每笔事务指令是否实际执行。
    
-   World State Trie：核心数据结构，将keccak-256 哈希的 20 字节账户地址映射到其 RLP 编码的状态，其中键值对以字节数组的形式存储在树的叶子节点中。其本身并不存储在区块链中，但其 32 字节的 keccak-256 状态根会被存储在每个区块的区块头中，前提是该区块中的所有交易都已处理完毕。
    
-   Storage Trie:其将合约的持久状态表示为 256 位存储槽索引（键）到 256 位 RLP 编码值的映射。每个这样的键值对被称为一个存储槽。
<!-- DAILY_CHECKIN_2026-04-12_END -->

# 2026-04-11
<!-- DAILY_CHECKIN_2026-04-11_START -->








今天主要学习了Block Building，周六竟然还要上一天实验课，我已心力憔悴

block building部分聚焦区块产生及之后过程，cl提出新区块，驱动el构建区块，在交易池中选择交易写入区块，执行交易，更新状态，返回payload，广播，进入下次循环。

这期水了(
<!-- DAILY_CHECKIN_2026-04-11_END -->

# 2026-04-10
<!-- DAILY_CHECKIN_2026-04-10_START -->









重新梳理了一下之前学的：

主要分为L1L2L3:

-   L1：底层，主链，比特币，以太坊，solana都处于这一层级，gas很高
    
-   L2：加速层，并包层，主要将相似的交易合并发送到L1上，加速处理速度，不考虑任何安全性，全部交由L1处理
    
-   L3：应用层，上层建筑，dapp等处于这一层级，将底层的抽象简化成简单的事务，但是失去了扩展性
    

evm分成了el ，cl 两部分：

-   el：执行层，进行底层的计算，使用message ，调用智能合约
    
-   cl：共识层，pos部分，确认区块正确性
    

底层调用流程如下：

1.  用户构造并签名交易
    
2.  通过 JSON-RPC 发送到节点
    
3.  节点验证并放入 Transaction Pool
    
4.  Validator / Builder 选择交易构建区块
    
5.  Execution Layer 执行交易（EVM）
    
6.  更新全局状态（State Transition）
    
7.  生成 Execution Payload
    
8.  发送给 Consensus Layer（Engine API）
    
9.  Consensus Layer 完成区块共识与最终确认
    

新学的部分：

-   el clients：el 逻辑的具体实现，使用不同语言，遵循同样的协议，覆盖了上文的2,3,5,6,7,8这几个步骤
    
-   evm ： 栈上虚拟机，负责处理具体的逻辑比如智能合约，完全处于栈上，状态机执行器
<!-- DAILY_CHECKIN_2026-04-10_END -->

# 2026-04-09
<!-- DAILY_CHECKIN_2026-04-09_START -->










今天主要学习了EL Architecture部分

**1\. Execution Layer 的职责**

-   负责执行交易和智能合约
    
-   维护以太坊全局状态
    
-   处理账户数据（余额、nonce、storage、代码）
    
-   计算交易执行结果
    
-   生成 execution payload
    
-   向共识层提供执行结果
    

**2\. Merge 后的整体架构**

-   以太坊由 Consensus Layer 和 Execution Layer 组成
    
-   Consensus Layer 负责区块共识和验证者机制
    
-   Execution Layer 负责交易执行和状态更新
    
-   两层通过 Engine API 进行通信
    

**3\. Execution Client 的主要模块**

-   JSON-RPC API
    
-   Transaction Pool
    
-   EVM
    
-   State & Storage
    
-   Block Processing
    
-   P2P Networking
    

**4\. JSON-RPC API**

-   提供外部访问接口
    
-   支持查询区块链数据
    
-   支持发送交易
    
-   支持查询账户和合约状态
    
-   被钱包、DApp 和开发工具使用
    

**5\. Transaction Pool**

-   存储尚未进入区块的交易
    
-   管理 pending transactions
    
-   按 gas 费用优先级排序
    
-   验证交易 nonce 和基本有效性
    
-   为区块构建提供交易来源
    

**6\. EVM**

-   以太坊虚拟机
    
-   执行智能合约字节码
    
-   运行 EVM opcode
    
-   计算 gas 消耗
    
-   执行交易逻辑并修改状态
    

**7\. State & Storage**

-   维护以太坊的 global state
    
-   包含账户、合约代码和存储数据
    
-   使用 Merkle Patricia Trie 结构
    
-   支持状态验证和加密证明
    
-   每个区块都会产生新的 state root
    

**8\. Block Processing**

-   接收新区块
    
-   验证区块结构和交易有效性
    
-   顺序执行区块中的交易
    
-   更新全局状态
    
-   生成交易回执和日志
    

**9\. P2P Networking**

-   节点之间进行网络通信
    
-   传播交易和区块数据
    
-   进行节点发现和连接管理
    
-   同步区块链数据和状态
    

**10\. Engine API**

-   Consensus Layer 与 Execution Layer 的通信接口
    
-   CL 向 EL 发送区块执行请求
    
-   EL 返回 execution payload
    
-   支持区块验证和区块构建流程
    

**11\. Execution Payload**

-   交易执行后的区块数据结构
    
-   包含交易列表
    
-   包含状态根
    
-   包含收据根
    
-   包含日志信息
    

**12\. Execution Client 实现**

-   Geth
    
-   Nethermind
    
-   Besu
    
-   Erigon
    

这些客户端实现相同协议但采用不同实现方式。
<!-- DAILY_CHECKIN_2026-04-09_END -->

# 2026-04-08
<!-- DAILY_CHECKIN_2026-04-08_START -->











今天上一天课，水一天。  
看见epf.wiki上的大段大段的英文，冷汗更是直流（  
底层的东西真是任重而道远，也就是看不懂的意思（  
昨天参加了第一次例会，主要讲了密码学的大概，并没有深入，主要讲解了一些概念，数学方面的东西真是不懂，什么域，组之类的概念听不懂啊，完全不懂离散数学，数学什么的从来没有开心过（  
提前截了图，忘了提交，想要提交时已经截止了，痛失30学分，我c，用户彻底怒了。

![微信图片_20260408175729_154_7.jpg](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/fffg-o/images/2026-04-08-1775642900108-_____20260408175729_154_7.jpg)
<!-- DAILY_CHECKIN_2026-04-08_END -->

# 2026-04-07
<!-- DAILY_CHECKIN_2026-04-07_START -->












今天主要学习了EL中的规格，核心设计哲学，以太坊并不是提供某些高级的功能，只是提供了基础的虚拟机（evm），用户可以通过智能合约实现任意数学上可以定义的应用。  
架构遵循以下原则：

1.  简单性：规范设计不需要高深的理解，普通程序员就可以实现整个协议。
    
2.  普遍性：以太坊只提供虚拟机，所有的智能合约都基于evm。
    
3.  模块化：通过封装，将复杂的系统封闭，向外提供简单的接口，而且许多组件都可以作为独立库复用.
<!-- DAILY_CHECKIN_2026-04-07_END -->

# 2026-04-06
<!-- DAILY_CHECKIN_2026-04-06_START -->













第一次参加epf计划，有点紧张，感觉大家学的都很深入，微信群里发的东西基本都看不懂(，接下来我会好好努力，追赶大家的脚步

今天主要学习了The Protocol部分，了解了以太坊史前时期的历史，从uinx出现到密码学的发展，密码朋克的出现，前人充满了反叛精神。追求自由，开源精神，只能说开源才是对的（
<!-- DAILY_CHECKIN_2026-04-06_END -->
<!-- Content_END -->
