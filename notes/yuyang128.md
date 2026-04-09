---
timezone: UTC+8
---

# yuyang128

**GitHub ID:** yuyang128

**Telegram:** 

## Self-introduction

EPF 实习计划

## Notes

<!-- Content_START -->
# 2026-04-09
<!-- DAILY_CHECKIN_2026-04-09_START -->
## 1\. 执行层（EL）概述

在以太坊合并（The Merge）之后，以太坊节点被拆分为两个主要部分：

-   **共识层 (Consensus Layer, CL)：** 负责权益证明（PoS）共识、区块选择和分叉选择规则（例如 Prysm, Lighthouse）。
    
-   **执行层 (Execution Layer, EL)：** 负责执行事务、维护世界状态（World State）和处理智能合约（例如 Geth, Nethermind, Besu）。
    

两者通过 **Engine API** 进行通信。

* * *

## 2\. 核心架构组件

执行层的架构可以拆解为以下几个关键模块：

### A. 以太坊虚拟机 (EVM)

EVM 是执行交易逻辑的运行时环境。

-   **基于栈的架构：** 大多数操作是在一个 1024 层的栈上完成的，字长为 256 位。
    
-   **内存 (Memory)：** 一个易失性的字节数组，在合约调用期间存在，按字（32字节）扩展，成本随使用量呈指数增长。
    
-   **存储 (Storage)：** 持久化存储，映射 `uint256 -> uint256`。这是最昂贵的资源（Gas 消耗最高）。
    
-   **Gas 机制：** 防止图灵完备系统中的死循环，确保计算资源得到补偿。
    

### B. 世界状态 (World State)

以太坊的状态是所有账户（EOA 和合约）的集合。

-   **账户模型：**
    
    -   **Nonce：** 交易计数器。
        
    -   **Balance：** 余额。
        
    -   **StorageRoot：** 该账户存储树的根哈希。
        
    -   **CodeHash：** 合约代码的哈希。
        

**数据结构：** 目前主要使用 **梅克尔-帕特里夏树 (Merkle Patricia Trie, MPT)**。

### C. 交易 (Transactions)

交易是改变世界状态的唯一触发方式。

-   **类型：** 包括 Legacy (Type 0), EIP-2930 (Type 1), EIP-1559 (Type 2) 以及 EIP-4844 (Blob 交易)。
    
-   **生命周期：**
    
    1.  **Mempool (内存池)：** 待处理交易的临时存放处。
        
    2.  **验证：** 检查签名、Nonce、Gas 费用是否足够。
        
    3.  **执行：** 更改状态并生成收据（Receipts）。
        

* * *

## 3\. 执行层状态转换函数

执行层的核心逻辑可以用一个数学公式来概括：

$$\\sigma\_{t+1} = \\Upsilon(\\sigma\_t, B)$$

其中：

-   $\\sigma\_t$：当前状态。
    
-   $B$：包含一系列交易的新区块。
    
-   $\\Upsilon$：以太坊状态转换函数（处理交易、分配奖励等）。
    
-   $\\sigma\_{t+1}$：执行后的新状态。
    

* * *

## 4\. 关键接口与通信

### Engine API (JSON-RPC)

这是 CL 和 EL 之间的“桥梁”。主要调用包括：

-   `engine_newPayloadV*`：CL 发送新区块给 EL 进行执行和验证。
    
-   `engine_forkchoiceUpdatedV*`：CL 通知 EL 当前的链头（Head）和终局化（Finalized）区块。
    
-   `engine_getPayloadV*`：CL 请求 EL 构建一个新的区块（用于提议者角色）。
    

### 网络层 (DevP2P)

EL 通过 `devp2p` 协议与其他对等节点发现并交换数据：

-   **RLPx：** 加密传输层。
    
-   **eth/68 (及更高版本)：** 用于广播交易、同步区块快照（Snap Sync）。
    

* * *

## 5\. 执行层客户端内部结构

一个典型的执行层客户端（如 Geth）通常包含以下子系统：

| 组件 | 描述 |
| P2P Stack | 管理节点发现（Discovery）和加密通信。 |
| Transaction Pool | 管理未打包交易的排序、替换和驱逐。 |
| State Manager | 读写底层数据库（LevelDB/Pebble），计算 Trie 哈希。 |
| EVM Engine | 解释执行操作码（Opcodes）。 |
| JSON-RPC Server | 响应用户和 DApp 的查询请求。 |
| Consensus Engine | 在 PoS 下，这部分已弱化为验证区块头和处理提款逻辑。 |
<!-- DAILY_CHECKIN_2026-04-09_END -->

# 2026-04-07
<!-- DAILY_CHECKIN_2026-04-07_START -->

-   **1\. 执行层与共识层（The Separation）：**
    
    -   **执行层 (EL - Execution Layer)：** 负责处理交易、执行智能合约、管理状态。
        
        -   **核心：** 以太坊虚拟机 (EVM)。
            
        -   **主流客户端：** Geth, Besu, Reth (Rust 开发，性能极高)。
            
    -   **共识层 (CL - Consensus Layer)：** 负责权益证明 (PoS) 共识、验证者管理。
        
        -   **核心：** 信标链 (Beacon Chain)。
            
        -   **主流客户端：** Prysm, Lighthouse, Teku。
            
-   **2\. 关键组件与接口：**
    
    -   **Engine API：** 连接 EL 和 CL 的桥梁。CL 告诉 EL 哪些交易需要打包，EL 执行后返回结果。
        
    -   **网络层（Networking）：**
        
        -   EL 使用 `devp2p` 协议。
            
        -   CL 使用 `libp2p` 协议。
            
    -   **状态转换（State Transition）：** 理解交易是如何改变全球状态（State Tree）的，这是安全分析的核心。
        
-   **3\. 数据结构（Data Structures）：**
    
    -   **梅克尔帕特里夏树 (Merkle Patricia Trie)：** 以太坊存储账户、交易和收据的高效数据结构。
        
    -   **RLP 序列化：** 以太坊内部数据传输和存储的标准编码格式。
<!-- DAILY_CHECKIN_2026-04-07_END -->
<!-- Content_END -->
