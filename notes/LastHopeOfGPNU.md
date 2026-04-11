---
timezone: UTC+8
---

# LastHopeOfGPNU

**GitHub ID:** LastHopeOfGPNU

**Telegram:** 

## Self-introduction

EPF 实习计划

## Notes

<!-- Content_START -->
# 2026-04-11
<!-- DAILY_CHECKIN_2026-04-11_START -->
# RLP

**RLP（Recursive‑Length Prefix）是以太坊执行层中核心的数据编码/序列化格式，用于把任意嵌套的二进制数据结构序列化成确定性字节流。**  
它保证不同客户端之间在交换区块、交易、状态等信息时能**按同一规则编码/解码**，从而支持一致性验证和网络互操作。

## 1\. RLP 的目的与作用

-   **标准化数据编码**：确保节点之间对数据结构的表示一致。
    
-   **节省空间**：对嵌套结构进行紧凑编码，适合链上/网络传输。
    
-   **用于执行层对象序列化**：区块、交易、账户状态等都广泛采用 RLP。
    

## 2\. RLP 能处理的项（Item）

RLP 可编码以下三类“待编码对象”：

1.  **字节串（byte array）**
    
2.  **正整数** （转换为不带前导零的大端字节串后编码）
    
3.  **嵌套列表**（列表内每个元素也必须是 RLP Item）
    

## 3\. 编码规则概要

RLP 的编码根据输入类型和长度不同分成几种情况：

### 3.1 单字节

-   如果是单个字节且在 `0x00–0x7F` 范围内，则**该字节本身就是编码结果**。
    

### 3.2 字符串

-   **短字符串（≤55 字节）**：`0x80 + 长度` 前缀 + 内容。
    
-   **长字符串（>55 字节）**：用 `0xB7 + len(len)` 前缀 + len（大端） + 内容。
    

### 3.3 列表

-   **短列表**：`0xC0 + 列表编码总长度` + 列表内容。
    
-   **长列表**：类似长字符串，用更长的长度前缀。
    

这些规则支持嵌套结构递归编码，使任意复杂的数据结构都能统一编码。

## 4\. 解码规则概要

-   RLP 解码以**首字节前缀**判断当前是单字节/短/长 string 或 list，然后递归解析出原始数据结构。
    

## 5\. 为什么 RLP 重要

-   **一致性**：任何客户端只要按 RLP 规范编码/解码，就能确保读出完全一致的数据结构。
    
-   **高效性**：相比通用格式（如 JSON），RLP 更简洁、更适合低层协议的数据交换。
    
-   **基础性**：执行层几乎所有链上对象（例如区块头、交易、账户状态）都通过 RLP 编码序列化后进行哈希及传播。
    

## 6\. 适合记住的要点

1.  **RLP 是执行层核心的序列化协议**，专门设计来编码嵌套的二进制数据结构。
    
2.  **它既不是 JSON 也不是 protobuf**，而是更简洁的“长度前缀 + 数据”格式。
    
3.  **单字节、小/长字符串与列表编码的首字节前缀范围不同**，可以通过前缀判定结构类型。
    

# DevP2P

DevP2P（Ethereum Dev Peer‑to‑Peer Protocol）是执行层节点之间的基础点对点通信栈。它规定了如何发现节点、建立加密连接以及交换区块、交易等协议消息，从而使执行客户端能够在去中心化网络中协作。

### 1\. 网络基础

-   **协议栈**：EL（Execution Layer）使用 TCP（可靠、顺序传输）传输信息，UDP（快速、无连接）用于节点发现。
    
-   **节点类型**：每个以太坊节点包含执行客户端（交易传播）和共识客户端（区块传播），各自维护独立 P2P 网络。
    

### 2\. 节点发现（Discv 协议）

-   **Discv4/Discv5**：基于 Kademlia DHT，节点通过 bootstrap 节点加入网络。
    
-   **发现流程**：
    
    1.  新节点向 bootstrap 节点发送 PING。
        
    2.  bootstrap 响应 PONG → 节点验证连接。
        
    3.  FIND-NEIGHBOURS 请求 → 获取邻居列表。
        
-   **数据结构**：
    
    -   **ENR（Ethereum Node Record）**：节点连接信息、加密密钥和元数据。
        
    -   **k-buckets**：存储节点信息，每桶最多 16 个节点，按最近活动排序。
        
-   **Discv5 改进**：
    
    -   加密通信（AES-GCM）、服务发现（Topic-based）、自适应路由。
        
    -   支持扩展 ENR，消除对时钟依赖，优化大网络可扩展性。
        

### 3\. 信息传输（RLPx 协议）

-   **协议特点**：
    
    -   TCP 基础，安全加密通信。
        
    -   支持多路复用（Multiplexing）与子协议。
        
-   **连接建立**：
    
    1.  节点发现后，用 secp256k1 临时密钥进行握手。
        
    2.  建立认证、生成 session keys（前向安全）。
        
-   **加密机制**：
    
    -   AES-128-CTR 消息加密
        
    -   HMAC-SHA-256 消息完整性校验
        
    -   ECDH（椭圆曲线 Diffie-Hellman）生成共享密钥
        
-   **消息封装**：
    
    -   帧结构：header-ciphertext + frame-ciphertext + MAC
        
    -   支持多个应用层协议并行运行
        

### 4\. 核心消息类型（Wire/子协议）

| 消息 | 功能 |
| --- | --- |
| Ping/Pong | 验证节点可达性 |
| FindNode/Neighbors | 查询并返回邻居节点 |
| ENRRequest/ENRResponse | 请求/响应节点记录 |
| WhoAreYou/Handshake | 节点认证、加密会话建立 |
| TalkReq/TalkResp | 自定义应用通信 |

### 5\. 常用 Ethereum 子协议

-   **eth**：区块和交易传播
    
-   **snap**：状态同步
    
-   **les**：轻客户端支持
    
-   **portal**：去中心化状态/区块/交易查询网络
    

**总结**：以太坊 EL 的 DevP2P 网络分为发现层（UDP、Discv4/5）和传输层（TCP、RLPx），通过加密握手、Kademlia 路由、ENR 记录实现去中心化、高安全性、可扩展的节点通信机制

# JSON-RPC

### 1\. JSON-RPC 概述

-   JSON-RPC 是基于 JSON 的远程过程调用协议，允许客户端调用远程服务器函数并获取结果。
    
-   在以太坊中，JSON-RPC 是执行层（EL）与共识层（CL）交互、用户与网络交互的主要方式。
    
-   请求格式统一：
    

```
{
  "id": 1,
  "jsonrpc": "2.0",
  "method": "<prefix_methodName>",
  "params": [...]
}
```

-   参数说明：
    
    -   `id`：请求唯一标识
        
    -   `jsonrpc`：协议版本
        
    -   `method`：调用方法
        
    -   `params`：方法参数，可为空数组
        

### 2\. 方法命名空间

-   方法由命名空间前缀 + 方法名组成，用 `_` 分隔。
    
-   常见命名空间及用途：
    
    | Namespace | 描述 | 是否敏感 |
    | --- | --- | --- |
    | eth | Ethereum 核心交互，如读取余额、发送交易 | 可能 |
    | web3 | 工具函数 | 否 |
    | net | 节点网络信息 | 否 |
    | txpool | 交易池检查 | 否 |
    | debug | 状态调试，如区块/交易原始数据 | 否 |
    | trace | 状态追踪，Parity 风格 | 否 |
    | admin | 节点配置 | 是 |
    | rpc | RPC 服务信息 | 否 |
    

### 3\. 常用 `eth` 方法

-   `eth_blockNumber`：返回最新区块号
    
-   `eth_call`：立即执行交易，不上链
    
-   `eth_chainId`：返回链 ID
    
-   `eth_estimateGas`：估算交易所需 Gas
    
-   `eth_getBalance(address, block)`：查询账户余额
    
-   `eth_getBlockByHash/hash`：获取区块信息
    
-   `eth_getCode(address, block)`：获取智能合约代码
    
-   `eth_getLogs(filter)`：获取日志
    

### 4\. debug 命名空间

-   用于原始数据访问和状态检查，可能计算量大：
    
    -   `debug_getBadBlocks`：返回最近的错误区块
        
    -   `debug_getRawBlock`：返回 RLP 编码区块
        
    -   `debug_getRawTransactions`：返回 EIP-2718 编码交易
        

### 5\. Engine API

-   Execution 客户端对共识客户端的通信接口，非用户接口
    
-   使用 JSON-RPC + JWT 认证，确保安全
    
-   核心方法：
    
    | 方法 | 参数 | 描述 |
    | --- | --- | --- |
    | engine_exchangeTransitionConfigurationV1 | 共识客户端配置 | 配置交换 |
    | engine_forkchoiceUpdatedV1* | forkchoice_state, payload attributes | 更新 forkchoice 状态，触发 payload 构建 |
    | engine_getPayloadBodiesByHashV1* | block_hash | 返回执行 payload |
    | engine_getPayloadV1* | forkchoice_state, payload attributes | 获取构建好的执行 payload |
    | debug_newPayloadV1* | tx_hash | payload 验证信息，调试用 |
    

### 6\. 数据编码与传输

-   参数统一使用 16 进制（`0x` 前缀）
    
-   JSON-RPC 可通过 HTTP、WSS、IPC 等协议传输：
    
    -   HTTP：单向请求-响应
        
    -   WSS：双向连接，可订阅事件
        
    -   IPC：本地进程通信，速度快
        

### 7\. 使用方式示例

-   curl:
    

```
curl <node-endpoint> -X POST -H "Content-Type: application/json" \
-d '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}'
```

-   JS/TS axios:
    

```
import axios from 'axios';
const response = await axios.post(node, {
  jsonrpc: '2.0',
  method: 'eth_getBalance',
  params: [address, 'latest'],
  id: 1,
});
```

-   web3 库封装：
    

```
from web3 import Web3
w3 = Web3(Web3.HTTPProvider('http://localhost:8545'))
w3.eth.get_balance('0xaddress')
```

```
import { ethers } from "ethers";
const provider = new ethers.providers.JsonRpcProvider('http://localhost:8545');
await provider.getBlockNumber();
```

**总结**：JSON-RPC 是以太坊 EL 与用户/CL 的主要接口，命名空间划分明确，参数统一使用 16 进制，支持多种传输方式。Engine API 专注于 EL-CL 通信，安全性高，常用于 forkchoice 和 payload 构建。web3 库对方法进行了封装，方便开发者调用。
<!-- DAILY_CHECKIN_2026-04-11_END -->

# 2026-04-10
<!-- DAILY_CHECKIN_2026-04-10_START -->

# EVM

**EVM（Ethereum Virtual Machine）本质上是以太坊这个“状态机”的状态转移执行器。**  
它接收交易作为输入，执行字节码，最后把以太坊从旧世界状态推进到新世界状态。

## 1\. 总体流程

交易输入 → EVM 执行 → 状态变化 → 新的 world state

## 2\. EVM 为什么要做成“虚拟机”

-   现实世界里，不同硬件和操作系统的执行环境不同。
    
-   EVM 作为虚拟机，提供统一抽象层。
    
-   开发者写出的合约会先编译成 **EVM bytecode**，再由不同客户端在不同平台上执行。
    

**意义：保证“同一份字节码，在全网不同节点上得到一致结果”。**

## 3\. EVM 里最重要的 4 个数据区域

### Stack

-   主要给 opcode 做即时计算用。
    
-   是 LIFO 结构。
    
-   最大 **1024** 个元素，每个元素 **32 字节**。
    
-   实际可直接通过 `DUP` / `SWAP` 触达的只有顶部 **16** 项。
    

### Memory

-   临时内存，只在本次执行期间有效。
    
-   是字节数组，按需扩展。
    
-   常见操作：`MSTORE`、`MLOAD`、`MSTORE8`。
    

### Calldata

-   只读输入区。
    
-   来自交易输入或 message call。
    
-   常见读取方式：`CALLDATALOAD`、`CALLDATACOPY`。
    

### Storage

-   合约持久化存储。
    
-   属于账户状态的一部分，会跨交易保留。
    
-   常见操作：`SSTORE`、`SLOAD`。
    
-   因为会影响全网复制的 world state，所以很贵。
    

## 4\. 字节码怎么跑起来

-   EVM bytecode 本质上是一串字节。
    
-   每个字节可能是 **opcode**，也可能是 opcode 的 **operand**。
    
-   `PUSH*` 这类指令会带操作数。
    
-   EVM 用 **program counter** 跟踪下一条要执行的指令。
    
-   `JUMP` / `JUMPDEST` 负责控制流。
    

**EVM 就是一个按字节码逐条解释执行的机器。**

## 5\. Gas 为什么重要

  
**如果没有 gas，EVM 的循环和计算可能无限消耗资源，造成 DoS 风险。**

所以：

-   每个 opcode 都有 gas 成本
    
-   交易执行过程中 gas 不够就会停止
    
-   gas 机制把计算资源显式定价
    

因此 EVM 常被描述为 **quasi Turing complete**，因为它理论上有灵活控制流，但实际被 gas 限制了。

# Transaction anatomy

**交易本质上是“由外部账户发起、经签名授权、提交给执行层客户端并在全网传播的指令”。**  
它的作用不是单纯转账，还包括**创建合约**和**调用合约代码**。

## 1\. 交易是什么

一笔交易由 **EOA（外部账户）** 发起，经密码学签名后，通过 **JSON-RPC** 提交给执行层客户端，再通过 **DevP2P** 广播到网络。文档强调，交易是执行层接收用户意图的标准入口。

## 2\. 一笔交易的核心字段

-   **nonce**：发送方已发交易计数，用来防重放，也会参与合约地址计算。
    
-   **gasPrice / gasLimit**：前者决定每单位 gas 愿付多少，后者限定本次执行最多可消耗多少 gas。
    
-   **to**：决定交易模式。为空表示**创建合约**；指向外部账户表示**转账**；指向合约账户表示**执行合约**。
    
-   **value**：转移的 ETH 数量。
    
-   **data / init**：合约调用时是输入数据；合约创建时是 init bytecode。
    
-   **签名** `(v, r, s)`：证明这笔交易确实由发送方授权。
    

## 3\. nonce 为什么很重要

**nonce 至少承担三件事：**

-   **防重放攻击**：同一笔交易不能被别人反复广播骗走更多资金。
    
-   **决定合约地址**：在合约创建模式下，发送方地址和 nonce 会参与新合约地址的计算。
    
-   **替换 pending 交易**：当交易卡住时，可以发一笔**相同 nonce、但更高 gas price** 的新交易去覆盖旧交易；有些钱包也利用这一点做“取消交易”。不过文档明确说，这种替换**不保证一定成功**，仍受网络和打包方行为影响。
    

## 4\. 合约创建是怎么发生的

合约创建交易的 `to` 为空，`data` 里放的不是运行时代码本体，而是 **init code**。  
**init code 只在创建时执行一次，它的返回值才会成为合约账户里真正保存的 runtime bytecode。** 这是理解部署流程的关键。

文档给了一个很直观的例子：先写一段运行时代码，让它执行 `6 * 7` 并把结果写入存储槽 `0`；然后再写 init code，把这段运行时代码复制到内存并 `RETURN` 出去，最终部署到链上。

## 5\. 合约执行是怎么发生的

在合约已经部署后，再发一笔交易：

-   `to` 指向该合约地址
    
-   `value` 可以为空
    
-   `data` 也可以为空
    

EVM 就会执行该合约的 runtime bytecode。文档示例里，这次执行后可以读到 storage slot `0 = 0x2a`，也就是十进制 **42**。

所以可以把两种模式分清楚：

| 模式 | to 字段 | data 含义 | 结果 |
| --- | --- | --- | --- |
| 创建合约 | 空 | init code | 返回 runtime code 并生成新合约 |
| 调用合约 | 合约地址 | 调用输入 | 执行已部署的 runtime code |

## 6\. Receipt 是什么

**receipt 不是交易本身，而是交易执行后的产物。**  
无论交易成功还是失败，都会对应一个 receipt，并被提交到区块的 **Receipt Trie**。

文档列出的 receipt 关键内容包括：

-   **Transaction Type**
    
-   **Status（0 或 1）**
    
-   **Gas Used**
    
-   **Logs**
    
-   **Logs Bloom**
    

其中最常用的理解是：

-   `status=1` 表示成功
    
-   `status=0` 表示失败
    
-   `logs` 是合约事件输出
    
-   `logs bloom` 是为了让应用更快筛选某地址或事件签名是否可能出现在该区块里
    

## 7\. Typed Transaction 为什么重要

后半段的重点是 **EIP-2718**。它给交易和 receipt 都引入了 **typed envelope**，核心格式是：

-   `Typed Transaction = Transaction Type + Transaction Payload`
    
-   `Typed Receipt = Transaction Type + Receipt Payload`
    

这样做的目的很明确：  
**让以太坊以后可以更容易扩展新的交易类型，同时保留对旧版 legacy 交易的兼容性。** 在 EIP-2718 之前，扩展新交易类型会比较别扭，因为都被 RLP 老格式限制住了。

识别方式：

-   首字节在 `[0x00, 0x7f]`，是 **typed**
    
-   首字节 `>= 0xc0`，是 **legacy RLP list**
    

# Data Structures

**以太坊执行层最核心的数据结构是 Merkle Patricia Trie（MPT），并围绕它组织出 Transaction Trie、Receipt Trie、World State Trie 和每个合约自己的 Storage Trie。**

## 1\. 三层概念

### 1.1 Merkle Tree

-   作用偏**完整性验证**
    
-   叶子放数据，非叶子节点放子节点哈希
    
-   最上面的根哈希就是 **Merkle Root**
    
-   只要底层数据变了，根哈希就会变，因此适合做包含性证明和篡改检测。
    

### 1.2 Patricia Trie

-   作用偏**高效键值存储与查找**
    
-   通过路径压缩，把公共前缀合并，减少空间浪费
    
-   值主要放在叶子节点，更利于紧凑存储。
    

### 1.3 Merkle Patricia Trie（MPT）

**MPT = Merkle Tree 的可验证性 + Patricia Trie 的高效检索。**  
它是以太坊执行层最关键的底层结构。MPT 里主要有三类节点：

-   **Branch node**：分叉导航，17 个槽位
    
-   **Extension node**：压缩公共路径
    
-   **Leaf node**：存具体 key-value 数据
    

另外，MPT 以 **nibble（半字节，十六进制一位）** 作为路径单位，每个节点最多分 16 个分支。

## 2\. 以太坊里有哪些核心 Trie

### 2.1 Transaction Trie

-   **每个区块各有一棵**
    
-   存这个区块里的所有交易
    
-   key 是交易在区块中的 **index**
    
-   value 是 **RLP 编码后的交易**
    

它是**区块级静态结构**：区块一旦确定，这棵 trie 就不会再变。

### 2.2 Receipt Trie

-   **每个区块各有一棵**
    
-   存每笔交易执行后的 **receipt**
    
-   key 同样是交易在区块里的 **index**
    
-   主要作用是提供**交易结果的可验证记录**
    

它很重要的一点是：节点在同步历史区块时，可以重建 receipt trie，并用 block header 里的 `receiptsRoot` 校验，不需要为了拿 receipts 而把所有历史交易重新执行一遍。

### 2.3 World State Trie

**这是执行层最核心的一棵 trie。**  
它表示“当前整条链的全局状态”：

-   key：`keccak-256(address)`
    
-   value：账户对象的 **RLP 编码状态**
    

账户对象里最关键的字段是：

-   `nonce`
    
-   `balance`
    
-   `storageRoot`
    
-   `codeHash`
    

注意：**World State Trie 本体不直接写进链里，写进每个区块头的是它的** `stateRoot`**。** 这个根哈希就是对全局状态的密码学承诺。

### 2.4 Storage Trie

-   **每个合约账户各有一棵**
    
-   用来存这个合约自己的持久化 storage
    
-   key 是 256-bit storage slot index，插入前会做 `keccak-256`
    
-   value 是对应 slot 的值
    

这棵 trie 通过账户对象里的 `storageRoot` 挂到 World State Trie 上。  
EOA 没有实际可用的 storage trie；合约的 storage 主要通过 `SSTORE` 写入、`SLOAD` 读取。

## 3\. 这四棵 Trie 的关系

| Trie | 作用 | 生命周期 |
| --- | --- | --- |
| Transaction Trie | 存区块里的交易 | 每块一棵，区块确定后不变 |
| Receipt Trie | 存区块里交易执行结果 | 每块一棵，区块确定后不变 |
| World State Trie | 存当前全局账户状态 | 持续演化 |
| Storage Trie | 存单个合约的持久化存储 | 跟随合约状态演化 |
<!-- DAILY_CHECKIN_2026-04-10_END -->

# 2026-04-09
<!-- DAILY_CHECKIN_2026-04-09_START -->


# EL Specs

**执行层的任务，本质上就是实现状态转移函数（State Transition Function, STF）。**  
也就是回答两个问题：

1.  **这个区块能不能接到链上**
    
2.  **接上后，全局状态会怎样变化**
    

## 1\. 执行层到底在做什么

  
**EL 负责“给定一个区块，按规则执行并产出新状态”。  
公式上就是：**

-   旧状态：`σ_t`
    
-   当前区块：`B`
    
-   新状态：`σ_(t+1) = Π(σ_t, B)`
    

可以把它理解为：

-   **区块级 STF**：处理整个 block
    
-   **交易级 STF**：逐笔执行 transaction
    
-   **EVM 执行**：真正跑 bytecode，修改状态
    

## 2\. 区块执行的大致流程

block 处理流程：

1.  取父区块 header
    
2.  校验当前区块 header
    
3.  检查 ommers 为空
    
4.  执行区块内所有交易
    
5.  得到 gas used、交易树根、收据树根、logs bloom、state
    
6.  检查这些结果是否和 block header 一致
    
7.  一致才把区块加入链上，否则判为 invalid block
    

## 3\. Header 校验里最值得记的点

文档列了很多 header validity 条件，最值得记的是这几类：

### 3.1 资源约束

-   `gasUsed <= gasLimit`
    
-   gas limit 不能相对父块突变太多
    
-   gas limit 不能低于最小值 5000
    

### 3.2 时间与链关系

-   当前块时间戳必须大于父块
    
-   区块号必须等于父块号 + 1
    
-   `parentHash` 必须匹配父块 header 哈希
    

### 3.3 Merge 之后的痕迹

这页明确体现了 **PoS 后执行层的“后 Merge 规则”**：

-   `difficulty = 0`
    
-   `nonce = 0x0000000000000000`
    
-   ommers hash 固定为空列表哈希
    

这说明：  
**今天 EL 还在处理 block，但它已经不再承担 PoW 挖矿语义。**

### 3.4 EIP-4844 相关字段

文档还纳入了 blob 相关 header 校验：

-   `blobGasUsed`
    
-   `excessBlobGas`
    
-   `MaxBlobGasPerBlock`
    
-   `TargetBlobGasPerBlock`
    

这表示：  
**执行层规范已经把 blob 交易带来的新资源维度纳入了状态转移规则。**

## 4\. 经济模型：这页重点讲了 EIP-1559

文档花了不少篇幅展开 base fee 公式，主要机制：

-   gas target = 父块 gas limit 的一半
    
-   如果父块 gasUsed 高于 target，base fee 上升
    
-   低于 target，base fee 下降
    
-   调整幅度受常数 `ξ = 8` 限制，避免变化过猛
    

**EIP-1559 让手续费不是纯拍卖，而是有一个可预测、按拥堵程度自动调节的 base fee。**  
而且 base fee 会被 burn，priority fee 才是给打包者的激励。

## 5\. 交易执行层面要记什么

交易级状态转移函数 `Υ(σ_t, T_index)`，流程是：

1.  先检查交易本身是否合法
    
2.  合法后进入 EVM
    
3.  EVM 根据 calldata、value、code、上下文执行
    
4.  成功则提交状态与子状态
    
5.  失败则回滚状态，gas 处理按规则执行
    

**交易失败不等于“什么都没发生”**：

-   状态可能回滚
    
-   但 gas 可能已经消耗掉
    
-   只有成功执行，logs / refund 等 substate 才会提交
    

## 6\. EVM 执行模型

EVM 执行可以抽象成几个函数：

-   `Ξ`：程序执行函数
    
-   `X`：递归执行函数，驱动整段 code 跑完
    
-   `O`：单步 opcode 执行推进
    

### 可以这样理解

-   `X` 像主循环
    
-   每一步执行一个 opcode
    
-   每一步都会消耗 gas
    
-   可能正常结束、REVERT、或者异常终止（比如 OOG）
    

**所以 EVM 不是“直接跑完一段代码”，而是“在 gas 约束下逐步解释执行”。**

# Client Architecture

**执行层客户端 = 状态转移引擎 + 网络层 + 交易池 + 对共识层暴露的 Engine API**

## 1\. 执行层客户端到底负责什么

执行层客户端不只是“跑交易”。它还要做几件事：

-   验证区块并保存本地区块链副本
    
-   通过 DevP2P 和其他 EL 客户端通信
    
-   维护 mempool
    
-   响应共识层的驱动与请求
    

所以它本质上是一个**被共识层驱动的执行系统**。

## 2\. 这套架构里最关键的几个部件

### 2.1 EVM

-   EVM 是以太坊的虚拟执行引擎
    
-   作用类似“统一指令语义”的虚拟 CPU
    
-   目的是让不同硬件、不同客户端执行同一笔交易时得到一致结果
    

**你可以把 EVM 理解成：保证全网计算结果一致的最核心抽象层。**

### 2.2 State

-   以太坊是**全局状态机**
    
-   状态里包含地址、余额、合约代码、存储，以及相关数据结构和数据库
    
-   文档明确拿它和比特币 UTXO 模型作对比：以太坊维护的是 global state，而不是仅维护 UTXO 集合。
    

### 2.3 Transactions

-   交易触发状态转移
    
-   交易先进入 mempool
    
-   再通过 EL 客户端在 P2P 网络中传播
    
-   其他节点收到后会先验证，再继续转发
    

**所以交易不是直接“上链”，而是先在网络里扩散、校验、等待打包。**

### 2.4 DevP2P

-   DevP2P 是执行层客户端之间通信的接口和网络基础
    
-   新节点靠 bootnodes 接入网络
    
-   区块、交易、状态同步都依赖这个网络栈。
    

### 2.5 JSON-RPC API

-   钱包和 DApp 与执行层交互，走的是标准 JSON-RPC
    
-   用来查询状态、发送交易等
    

这部分是**外部应用面向 EL 的公开接口**。

### 2.6 Engine API

-   Engine API 只用于 **CL 和 EL 的内部通信**
    
-   不是给钱包或 DApp 用的
    
-   主要有两类调用：
    
    -   `engine_newPayload`：校验并插入 payload
        
    -   `engine_forkchoiceUpdated`：更新 fork choice，并在需要时触发构建新区块
        
-   启动时，CL 和 EL 还会先通过 `engine_exchangeCapabilities` 交换支持的 API 版本
    

## 3\. Merge 之后，执行层的定位变了

这篇文档很关键的一点是明确说明：

**Merge 之后，执行层不再负责链上共识、区块排序和重组决策；这些职责转给了共识层。执行层现在主要承担状态转移函数（STF）。**

这点非常重要，因为它解释了为什么今天 EL 的系统边界更清晰：

-   **CL 决定哪条链是 head / safe / finalized**
    
-   **EL 负责验证 payload、执行交易、更新状态**
    

## 4\. 状态转移函数怎么理解

文档用一个简化的 Geth 风格伪代码说明 STF 的流程：

1.  先验证 header
    
2.  再按顺序执行区块里的每笔交易
    
3.  任何一笔交易出错，整个区块无效
    
4.  全部成功，得到新区块对应的新状态
    

**EL 的核心工作就是“给定父块和当前块，算出这个块是否合法，以及新状态是什么”。**

## 5\. CL 和 EL 的典型协作流程

### 节点启动

-   CL 先和 EL 做 capability exchange
    
-   然后发送 `forkchoiceUpdated`
    
-   如果 EL 还没追上链，会返回 `SYNCING`
    
-   追上后会返回 `VALID`。
    

### 验证者正常工作时

-   每个 slot，CL 都会给 EL 发 `forkchoiceUpdated`
    
-   如果轮到本验证者提议区块，CL 会附带 payload attributes，让 EL 开始 build payload
    
-   EL 返回 `payloadId`
    
-   CL 后续通过 `engine_getPayload` 取回构建好的 execution payload
    
-   若收到别人出的 beacon block，CL 会把其中 execution payload 抽出来，调用 `engine_newPayload` 让 EL 校验。
    

## 6\. 状态同步

文档提到，执行客户端要验证和构建区块，必须有足够新的 world state。  
为此会通过 DevP2P 子协议进行同步：

-   `eth/*`：同步区块头、区块体、收据
    
-   `snap/1`：做状态快照同步
    

并且客户端一般有两类同步策略：

-   **full sync**
    
-   **snap sync**
    

区别在于 snap sync 从较新的检查点启动，而不是从 genesis 一块块完整重放。
<!-- DAILY_CHECKIN_2026-04-09_END -->

# 2026-04-08
<!-- DAILY_CHECKIN_2026-04-08_START -->



# Design rationale

## 1\. 设计哲学

### 1.1 简单性

-   以太坊早期追求让“普通程序员也能理解和实现协议”。
    
-   目标之一是降低少数精英开发者对协议的垄断影响。
    
-   后来协议变复杂了，但仍希望通过**模块化**和**清晰规范**维持可理解性。
    

### 1.2 通用性

-   **Ethereum “没有预设功能”**，它提供的是一个 **Turing-complete 的 EVM**。
    
-   协议尽量提供底层能力，让开发者自己组合出合约和应用。
    

### 1.3 模块化

-   模块化的目的，是让协议更**可升级、可替换、可持续演进**。
    
-   复杂性可以存在，但最好被封装在子系统里，通过简单接口暴露出来。
    
-   这类思路叫 **encapsulated complexity（封装复杂性）**。
    

### 1.4 非歧视与自由

-   协议不主动限制特定用途。
    
-   它关注的是“对协议本身的伤害和成本”，不按应用类型做价值判断。
    
-   所以 Ethereum 更倾向用**费用机制**约束资源滥用，而不是直接封禁某类用法。这类思路可以类比到激励兼容和 Pigovian taxation。
    

### 1.5 可演进性

-   以太坊的很多细节**不是一开始就定死的**。
    
-   如果后续发现某些修改能明显改善扩展性或安全性，协议会继续迭代。
    

## 2\. 复杂性管理

**以太坊的设计反对“无法隔离的复杂”。**  
其中包括两种复杂性：

-   **Sandwich model complexity**：底层和用户接口尽量简单，把复杂度推到中间层。
    
-   **Encapsulated complexity**：复杂可以存在于子系统内部，只要外部接口清晰。
    

偏好的复杂性顺序：  
**Layer 2 > 客户端 > 协议规范**

## 3\. 关键技术取舍

### 3.1 Accounts over UTXOs

以太坊选择了**账户模型**，没有采用比特币式 UTXO。  
主要理由：

-   更适合复杂交易和智能合约
    
-   更容易实现和推理
    
-   资产更强可替代（fungibility）
    
-   对以太坊想支持的场景，例如 DEX，更合适
    

同时也包含代价：

-   为防 replay attack，需要 **nonce**
    
-   账户状态更难裁剪，历史上会累积状态负担
    

### 3.2 MPT 与 Verkle

-   以太坊状态结构长期使用 **Merkle Patricia Trie (MPT)**，因为它可验证、可生成 Merkle proof。
    
-   MPT 在大状态规模下，证明体积和效率会成为问题。
    
-   因此研究方向转向 **Verkle trees**，目的是用更小的 witness/proof 支持更强的无状态化能力。
    
-   文档中有提示提示：**Verkle 部分是活跃研究区，文档可能已经不是最新状态。** 这一点需要特别注意。
    

### 3.3 RLP 与 SSZ

-   **RLP**：早期的序列化方式，优点是简单、确定性强。
    
-   **SSZ**：Beacon Chain 使用的序列化方式，支持固定/可变长度数据，并且支持 Merkleization。
    
-   文中给出的核心理由很明确：  
    **SSZ 解决了 RLP 不利于 Merkleization、难以支持简洁轻客户端证明的问题。**  
    这又和 Ethereum 想推进的 **statelessness** 目标直接相关。
    

### 3.4 Finality

-   在 PoS 语境下，文档把 finality 描述为：区块一旦 finalized，要逆转它需要付出极高代价。文档中给出的表述是需要烧毁至少 **33% 的质押 ETH**。
    
-   对应机制是 **Casper FFG**。
    
-   它通过验证者对 checkpoint 进行投票，经过两轮投票后实现 finalized。
    

# Evolution

## 1\. Frontier：以太坊正式启动，但当时本质上还是 beta

-   Frontier 是以太坊协议的首次发布。
    
-   它在 **2015 年 7 月 30 日 03:26:13 UTC** 启动，这个时间就是以太坊创世区块时间戳。
    
-   当时初始 gas limit 只有 **5000**，目的是让矿工和用户能先把客户端跑起来，降低启动阶段的风险。
    
-   后来 gas limit 在 **Frontier thawing fork** 中提高到 **3,141,592**。
    
-   Frontier 还有一个很早期的机制叫 **canary contracts**：它们只输出 0 或 1；如果客户端发现多个 canary contract 都切到 1，就会停止挖矿并提示用户升级客户端。
    
-   这个机制的目的，是避免矿工长期阻碍链升级。
    

## 2\. Homestead：从 beta 走向更稳定的平台

**Homestead 是第二个主要版本，发布时间是 2016 年 3 月 14 日。** 文档把它定位为：以太坊从测试/试验性质，走向更成熟稳定平台的节点。

### 这一阶段的几个关键变化

-   **EIP-2** 做了多项修正，包括：提高通过交易创建合约的 gas 成本、修复高 `s-value` 签名导致的交易可塑性问题、避免 gas 不足时留下空合约、并调整难度算法。
    
-   其中一个非常具体的改动是：**通过交易创建合约的 gas 成本从 21,000 提高到 53,000**。
    
-   **EIP-7** 引入了 `DELEGATECALL`，它和 `CALLCODE` 类似，但会把父调用环境中的 sender 和 value 传递下去，因此更适合代理/可升级合约这类模式。
    
-   **EIP-8** 则强调网络协议的前向兼容性，要求客户端在 devp2p / RLPx 相关协议上对未来升级更宽容，从而减少未来升级时的兼容问题。
    

## 3\. The Merge：共识机制切换到 PoS

**The Merge 是这页历史线中最重要的现代节点。** 文档写明：以太坊在 **2022 年 9 月 15 日** 激活 **EIP-3675**，把共识机制升级为 **Proof-of-Stake**。

### Merge 之后，架构理解要变

-   Merge 之前，PoW 共识逻辑和执行逻辑在同一逻辑层里。
    
-   Merge 之后，PoW 被弃用，PoS 共识由单独的一层承担，这一层有自己的逻辑和独立的 P2P 网络，也就是 **Beacon Chain**。
    
-   文档还提到，**Beacon Chain 从 2020 年 12 月 1 日起就已经在运行并达成共识**；在经过一段稳定运行后，才成为以太坊正式的共识提供者。
<!-- DAILY_CHECKIN_2026-04-08_END -->

# 2026-04-07
<!-- DAILY_CHECKIN_2026-04-07_START -->




# Architecture

**以太坊当前的协议架构，核心是“双层结构”**：

-   **Execution Layer（EL，执行层）**
    
-   **Consensus Layer（CL，共识层）**
    

### 1\. 执行层（EL）负责什么

-   负责**交易执行**和**用户交互**
    
-   可以理解为：以太坊这台“全球计算机”真正运行程序的地方
    

### 2\. 共识层（CL）负责什么

-   提供**Proof-of-Stake（PoS，权益证明）**共识机制
    
-   作用是确保所有节点都跟随同一条链头（tip），共同维护**执行层的规范主链（canonical chain）**
    

### 3\. 工程实现上怎么落地

-   EL 和 CL **不是一个单体系统**
    
-   实际部署时，它们通常由**各自独立的客户端**实现
    
-   两层之间通过 **API** 连接协同工作
    

### 4\. 网络层面的特点

-   EL 和 CL **各自有自己的 P2P 网络**
    
-   两边处理的数据类型并不相同
<!-- DAILY_CHECKIN_2026-04-07_END -->

# 2026-04-06
<!-- DAILY_CHECKIN_2026-04-06_START -->





# Prehistory

以太坊并不是凭空出现的，它继承了早期互联网、Unix/FOSS、密码学与密码朋克、以及比特币时代数字货币实验的思想脉络。

### 1\. 以太坊的思想来源

-   早期互联网带来了**开放协作、跨边界连接**的想象。
    
-   Unix、GNU/Linux 和 FOSS 文化强调**开放、模块化、自由协作**，这直接影响了以太坊的设计与开发文化。
    
-   非对称密码学的发展，让个人也能掌握数字世界中的身份、签名与资产控制能力。
    

### 2\. 密码朋克脉络

-   密码朋克主张用技术保障个人自由、隐私、自主权，反对过度中心化控制。
    
-   以太坊承接了这种思路，目标不只是做一种货币，而是构建**无需许可、全球可用的数字经济基础设施**。这是从资料中可以直接推出来的主线
    

### 3\. 为什么会有以太坊

-   在比特币之前，已经有 Digicash、E-Gold、B-Money、Bit Gold 等数字货币尝试。
    
-   比特币证明了去中心化货币可以成立，但它在**通用应用承载能力**上有限；围绕“在比特币上开发更复杂应用”的尝试并不理想，这推动了以太坊出现
<!-- DAILY_CHECKIN_2026-04-06_END -->
<!-- Content_END -->
