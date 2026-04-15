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
