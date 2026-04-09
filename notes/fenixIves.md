---
timezone: UTC+8
---

# fenixIves

**GitHub ID:** fenixIves

**Telegram:** 

## Self-introduction

EPF 实习计划

## Notes

<!-- Content_START -->
# 2026-04-10
<!-- DAILY_CHECKIN_2026-04-10_START -->
# 以太坊执行层（EL）架构 核心笔记

## 一、执行层核心定位与整体架构

### 1\. 核心职责（合并后The Merge）

以太坊合并后，原共识相关职责完全移交共识层（CL），执行层（EL）客户端核心职能聚焦为：

-   区块链数据合法性校验，维护链数据本地副本
    
-   基于DevP2P协议实现P2P网络通信，完成节点间区块、交易的 gossip 传播
    
-   维护交易池（mempool），校验交易合法性并全网广播
    
-   响应共识层指令，执行交易、完成以太坊全局状态转换
    
-   对外提供标准化JSON-RPC API，对内通过专属Engine API与共识层通信
    

### 2\. 整体架构层级

**驱动链路**：共识层（CL）→ Engine API → 执行层核心（执行引擎）→ DevP2P网络层

-   执行引擎是EL的核心，完全由CL通过Engine API驱动
    
-   DevP2P为EL提供网络接入能力，通过合法启动节点完成网络初始化
    
-   架构设计遵循**职责分离原则**：CL负责共识与分叉选择，EL负责交易验证与执行
    

## 二、EL核心基础组件

### 1\. EVM（以太坊虚拟机）

-   核心定位：确定性虚拟执行引擎，解决底层硬件异构导致的执行结果不一致问题，保障全节点计算结果达成共识
    
-   设计理念：**三明治复杂度模型**——外层（EVM字节码）保持简洁，复杂度集中在中间层（Solidity高级语言→EVM字节码的编译器）
    
-   核心作用：执行智能合约逻辑，是触发以太坊状态转换的执行载体
    

### 2\. State（全局状态）

-   核心定位：以太坊是基于全局状态的状态机，这是与比特币UTXO模型的核心差异
    
-   状态内容：账户地址与余额、合约代码与存储数据、当前链状态与网络状态
    
-   底层结构：通过Merkle-Patricia Tries（MPT）存储，保障状态的可验证性与防篡改性
    

### 3\. Transactions（交易）

-   唯一作用：作为输入触发以太坊的状态转换
    
-   核心流程：交易进入mempool→节点校验合法性→EVM内执行→校验通过则完成全局状态变更
    
-   关键规则：区块内任意一笔交易执行失败，整个区块将被判定为无效
    

### 4\. DevP2P网络层

-   核心定位：EL客户端之间P2P通信的标准接口层
    
-   核心功能：实现交易与区块的全网传播；节点间链数据同步；接收数据后先校验合法性再广播，防范垃圾交易与无效状态转换
    

### 5\. JSON-RPC API

-   核心定位：外部应用（钱包、DApp）与EL交互的标准化接口
    
-   核心能力：支持外部查询以太坊链上状态、发送钱包签名后的交易；交易由EL完成校验后，通过DevP2P全网广播
    

## 三、Engine API：CL与EL通信的核心

### 1\. 基础规则

-   专属权限：仅用于CL与EL的内部通信，基于带JWT认证的JSON-RPC over HTTP，不对外暴露
    
-   认证说明：JWT仅用于校验发送方为合法CL客户端，不加密通信流量
    
-   版本兼容：支持V1/V2/V3三个版本，节点启动时先完成能力协商
    

### 2\. 前置流程：能力交换

-   方法：`engine_exchangeCapabilities`
    
-   执行时机：节点启动时，常规业务运行前
    
-   核心作用：CL与EL互相交换支持的Engine API方法与版本，协商通用协议版本，保障兼容性，兼顾新特性落地与向后兼容
    

### 3\. 核心端点与功能

（1）New Payload (V1/V2/V3)

-   核心作用：执行负载（Execution Payload）的验证与链上插入
    
-   触发时机：CL收到网络中新的信标区块，提取其中的执行负载后调用
    
-   校验流程：
    

1.  校验负载头中的父区块哈希存在，且与本地链的预期父块匹配
    
2.  验证额外执行承诺（如Cancun升级后的相关数据）
    
3.  在EVM中执行负载内的所有交易，完成状态更新
    

-   核心返回状态：
    

-   `VALID`：完全执行且所有校验通过
    
-   `INVALID`：负载或其祖先区块校验失败
    
-   `SYNCING`：EL仍在链同步中，缺失相关区块
    
-   `ACCEPTED`：基础校验通过，完整执行待完成（浅状态客户端常见）
    

（2）Fork Choice Updated (V1/V2/V3)

-   核心作用：管理EL链状态同步，触发区块构建
    
-   触发时机：CL按LMD-GHOST算法更新分叉选择规则，或验证者被分配出块权限时
    
-   执行流程：
    

1.  EL更新本地规范链头（canonical head）
    
2.  若CL传入负载属性，EL立即启动区块构建流程
    
3.  返回处理状态，若启动区块构建则同步返回`payloadId`
    

-   返回状态：与New Payload一致，`VALID`表示分叉选择更新处理成功，EL链状态已同步至最新
    

（3）辅助端点：`engine_getPayload`

-   核心作用：CL通过Fork Choice Updated返回的`payloadId`，从EL获取已构建完成的执行负载，用于信标区块打包
    

## 四、核心工作流程

### 1\. 节点启动全流程

1.  CL调用`engine_exchangeCapabilities`，与EL协商支持的Engine API版本
    
2.  CL发送初始`engine_forkchoiceUpdated`调用（无负载属性），告知EL当前链的分叉选择状态
    
3.  EL若未完成链同步，返回`SYNCING`状态；同步完成后返回`VALID`状态，节点进入正常运行模式
    

### 2\. 验证者常规运行流程（每个Slot）

1.  CL调用`engine_forkchoiceUpdated`，更新EL的本地链状态
    
2.  出块场景：若该验证者被分配本轮出块权限，CL在调用中传入负载属性，触发EL构建区块；EL返回`payloadId`，CL后续通过`engine_getPayload`获取完整执行负载
    
3.  区块校验场景：若收到其他验证者提出的信标区块，CL提取执行负载，调用`engine_newPayload`，由EL完成负载全量校验
    

### 3\. 核心执行逻辑：状态转换函数（STF）

STF是EL的核心功能，是`New Payload`的底层实现，负责完成区块校验与状态更新。

-   入参：父区块、当前待校验区块、父区块对应的状态数据库（StateDB）
    
-   返回值：交易执行后的新状态DB；执行失败则返回错误，不更新原有状态
    
-   执行步骤：
    

1.  **区块头校验**：验证父哈希合法性、区块号连续性、gas limit合规性（单块调整幅度不超1/1024）、EIP-1559基础费更新正确性等
    
2.  **交易逐笔执行**：在EVM中按顺序执行区块内每笔交易，传入区块头上下文、交易数据、当前状态
    
3.  **状态提交**：所有交易执行成功后，更新状态DB，生成新的状态根
    

## 五、节点同步机制

### 1\. 同步基础规则

-   触发源：由CL的LMD-GHOST分叉选择规则驱动，通过Engine API的Fork Choice Updated端点通知EL执行同步
    
-   核心动作：从对等节点下载远程区块，在EVM中完成区块全量校验
    
-   同步状态：通过`VALID/INVALID/SYNCING/ACCEPTED`标识EL当前的同步进度
    

### 2\. 两种核心同步策略对比

| 特性 | Full Sync（全量同步） | Snap Sync（快照同步） |
| 核心逻辑 | 从创世块开始，逐块重放所有历史交易，一步步重建状态树 | 以最近终局块为锚点，直接下载状态树叶子节点+默克尔证明，本地重建完整状态 |
| 核心流程 | 1. 下载创世到链头的所有区块头/体2. 逐块在EVM中执行所有交易3. 校验本地状态根与链头状态根匹配 | 1. 选定终局锚点块，获取其状态根2. 下载状态树叶子（账户、存储槽）、合约字节码3. 批量校验默克尔证明，写入快照DB4. 修复阶段：补全缺失数据，保障状态完整一致5. 基于锚点状态，执行后续区块至链头 |
| 安全性 | 最高，全量校验所有历史状态转换 | 锚点块基于弱主观性校验，后续区块全量校验 |
| 性能 | 主网同步需数天，占用极高CPU、磁盘、网络资源 | 主网同步缩短至数小时，仅修复阶段硬件压力较大 |
| 兼容性 | EIP-4444全面实施后，将不再支持从创世块的全量同步 | EIP-4444后成为主流同步方案，基于检查点同步 |

## 六、EL其他核心组件

### 1\. 交易池（Transaction Pools）

核心作用：存储待打包的合法交易，为区块构建提供交易来源，通过DevP2P实现全网交易传播。分为两大类型：

-   **传统交易池（Legacy Pools）**：基于价格排序的堆/优先级队列，按有效小费、gas费上限双维度排序，池饱和时按堆大小驱逐低优先级交易
    
-   **Blob交易池（Blob Pools）**：为EIP-4844 Blob交易设计，基于对数函数的驱逐队列，通过优先级堆管理交易驱逐逻辑
    

### 2\. 内置共识引擎（Ethone）

-   定位：EL自带的轻量共识引擎，仅保留完整共识引擎约一半的功能，兼容PoS、PoA（Clique）、PoW（Ethash）三种共识机制
    
-   核心作用：处理合并前（TTD前）的历史区块共识校验，兼容不同共识机制的区块处理规则
    
-   合并后规则：PoS合并后的区块，难度固定为0、叔块数必须为0、nonce固定为0，相关共识逻辑全部移交CL处理
    

### 3\. 存储后端

EL需要持久化两类核心数据：区块链历史数据（远古数据库）、当前状态与近期状态（MPT结构），主流存储后端如下：

-   **LevelDB**：早期主流方案，基于LSM树的嵌入式KV数据库，现已因停止维护被多数客户端弃用
    
-   **Pebble**：LevelDB替代方案，Geth当前默认后端，优化LSM树架构，支持多活跃内存表，降低写停顿，更适配以太坊写密集、低延迟的业务需求
    
-   **MDBX**：其他EL客户端广泛采用的高性能存储方案，优化读写性能与数据一致性
    

## 七、主流EL客户端：Geth核心实现

Geth是以太坊最主流的执行层客户端，其核心模块与EL架构完全对齐：

1.  **交易执行**：交易先进入mempool，完成签名、nonce、gas费校验后，被打包进区块，在EVM中执行后更新账户余额、合约存储等状态
    
2.  **区块处理**：按顺序执行区块内所有交易，执行完成后提交最终状态，存储状态根哈希保障链一致性
    
3.  **网络通信**：基于DevP2P协议实现节点间区块、交易的P2P传播，接收数据先完成合法性校验再转发
    
4.  **EVM执行**：内置标准EVM，处理所有智能合约逻辑，保障执行的确定性
    
5.  **状态同步**：原生支持Full Sync与Snap Sync两种同步模式
<!-- DAILY_CHECKIN_2026-04-10_END -->

# 2026-04-09
<!-- DAILY_CHECKIN_2026-04-09_START -->

# 以太坊执行层规范 (EL Specs) 学习笔记

> 基于 [epf.wiki/#/wiki/EL/el-specs](https://epf.wiki/#/wiki/EL/el-specs) 整理  
> 参考来源：EELS 官方博客、ethereum/execution-specs、OpenZeppelin 技术解析

* * *

## 目录

1.  [规范概览](#1-%E8%A7%84%E8%8C%83%E6%A6%82%E8%A7%88)
    
2.  [规范文档体系](#2-%E8%A7%84%E8%8C%83%E6%96%87%E6%A1%A3%E4%BD%93%E7%B3%BB)
    
3.  [EELS：可执行规范](#3-eels%E5%8F%AF%E6%89%A7%E8%A1%8C%E8%A7%84%E8%8C%83)
    
4.  [EELS 架构与模块结构](#4-eels-%E6%9E%B6%E6%9E%84%E4%B8%8E%E6%A8%A1%E5%9D%97%E7%BB%93%E6%9E%84)
    
5.  [核心功能深入：自下而上](#5-%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BD%E6%B7%B1%E5%85%A5%E8%87%AA%E4%B8%8B%E8%80%8C%E4%B8%8A)
    
6.  [EIP 流程与规范演进](#6-eip-%E6%B5%81%E7%A8%8B%E4%B8%8E%E8%A7%84%E8%8C%83%E6%BC%94%E8%BF%9B)
    
7.  [关键 API 规范](#7-%E5%85%B3%E9%94%AE-api-%E8%A7%84%E8%8C%83)
    
8.  [测试框架](#8-%E6%B5%8B%E8%AF%95%E6%A1%86%E6%9E%B6)
    
9.  [重要概念速查](#9-%E9%87%8D%E8%A6%81%E6%A6%82%E5%BF%B5%E9%80%9F%E6%9F%A5)
    

* * *

## 1\. 规范概览

以太坊执行层（Execution Layer，EL）的规范并非一份单一文档，而是由多个相互配合的规范共同构成：

| 规范 | 作用 | 仓库 |
| --- | --- | --- |
| EELS（Python 可执行规范） | 核心协议参考实现，每个 Fork 快照 | ethereum/execution-specs |
| 黄皮书（Yellow Paper） | 原始数学形式化规范（已逐步被 EELS 取代） | ethereum/yellowpaper |
| EIP（以太坊改进提案） | 描述协议变更的标准，仅记录 diff | ethereum/EIPs |
| Execution APIs（JSON-RPC） | 执行客户端对外暴露的接口规范 | ethereum/execution-apis |
| Engine API | EL 与 CL 之间的内部通信接口 | ethereum/execution-apis/engine |
| execution-spec-tests | 用于验证客户端合规性的测试框架 | ethereum/execution-spec-tests |

* * *

## 2\. 规范文档体系

### 2.1 历史演进：从黄皮书到 EELS

```
黄皮书（2014）
  └── 数学符号表达，密集难读
      ↓ 逐渐难以维护（post-Merge 后未同步更新）
EELS（2021~2023 公开）
  └── Python 实现，可读性优先
  └── 每个 Fork 独立快照
  └── 支持 EIP 原型验证
```

-   **黄皮书的局限**：使用晦涩的数学符号（如 $\\sigma$、$\\mu$ 等），The Merge 后逐渐不再与主网同步，对新贡献者极不友好
    
-   **EELS 的目标**：成为"精神上的黄皮书继承者"，面向程序员，可直接运行和测试
    

### 2.2 EIP 与 EELS 的关系

```
EIP（描述变更）  →  EELS（实现变更）  →  生产客户端（Geth/Nethermind/...）
       ↓
  只记录 diff                完整协议快照              混合多个 Fork 在同一代码库
```

**EIP 的不足**：只记录修改内容，无法展示协议全貌；而 EELS 为每个 Fork 提供完整的协议快照，大幅降低理解难度。

* * *

## 3\. EELS：可执行规范

### 3.1 什么是 EELS

**EELS**（Ethereum Execution Layer Specification）是以太坊执行层的 **Python 参考实现**，核心特性：

-   🎯 **可读性优先**：代码风格偏向表达协议意图，而非追求运行性能
    
-   📸 **Fork 快照**：为每个硬分叉提供完整的独立代码快照，并提供 Fork 间的 diff 视图
    
-   🧪 **可执行**：能够填充和运行状态测试，跟踪主网状态
    
-   🔧 **EIP 原型平台**：EIP 作者可在 EELS 中快速原型化其提案
    

### 3.2 EELS 的局限

EELS **不实现**以下功能：

-   P2P 网络（需依赖生产客户端同步区块）
    
-   JSON-RPC API（由 `ethereum/execution-apis` 单独维护）
    
-   高性能优化（仅为参考，不用于生产环境）
    

### 3.3 EELS 的价值

> "EELS 是 EIP 作者进行原型设计的首选，也是理解以太坊工作方式的最佳参考。"

对不同受众的意义：

-   **智能合约开发者**：快速查阅特定 EVM 指令在某 Fork 的精确行为
    
-   **客户端开发者**：查看 Fork 间的精确 diff，理解需实现的变更
    
-   **研究者/EIP 作者**：在正式规范之前先在 EELS 中验证方案可行性
    

* * *

## 4\. EELS 架构与模块结构

### 4.1 两大模块分类

```
EELS 快照
├── EVM 模块（vm/）
│   ├── 操作码（opcodes）实现
│   ├── 预编译合约（precompiles）
│   ├── Gas 计算
│   ├── 栈、内存管理
│   └── 解释器（interpreter）：执行 EVM 消息的入口
│
└── 区块链执行模块（blockchain execution）
    ├── 区块（blocks）与交易（transactions）数据结构
    ├── 状态（state）与状态树（trie）
    ├── 区块验证与处理逻辑
    └── Fork 管理（fork.py）
```

### 4.2 Prague Fork 文件结构（当前主网）

```
src/ethereum/prague/
├── blocks.py          # 区块结构定义
├── bloom.py           # Bloom 过滤器
├── exceptions.py      # 异常类型
├── fork_types.py      # Fork 相关类型
├── fork.py            # 主入口：区块处理、状态转换
├── requests.py        # EL→CL 请求（EIP-7685）
├── state.py           # 世界状态管理
├── transactions.py    # 交易类型定义
├── trie.py            # Merkle Patricia Trie
├── utils/             # 工具函数
└── vm/
    ├── eoa_delegation.py     # EIP-7702 EOA 委托
    ├── exceptions.py
    ├── gas.py                # Gas 计算
    ├── instructions/         # 全部操作码实现
    ├── interpreter.py        # EVM 解释器（核心入口）
    ├── memory.py             # 内存管理
    ├── precompiled_contracts/ # 预编译合约
    ├── runtime.py
    └── stack.py              # 栈操作
```

> **注意**：不同 Fork 的文件结构可能有所不同——某些 EIP 会新增或移除特定文件。

* * *

## 5\. 核心功能深入：自下而上

以下按从底层到顶层的顺序梳理执行层的核心流程：

```
EVM 消息（Message）
    ↓
用户交易（User Transactions）
    ↓
系统交易（System Transactions）
    ↓
区块处理（Block Processing）
    ↓
状态转换（State Transition）
```

### 5.1 EVM 消息（Message）

**入口函数**：`process_message_call(message: Message) -> MessageCallOutput`

```python
def process_message_call(message: Message) -> MessageCallOutput
```

`Message` 的关键字段：

| 字段 | 说明 |
| --- | --- |
| caller | 消息发送者（即 Solidity 的 msg.sender） |
| target | 目标地址（空 = 合约创建） |
| current_target | 实际执行目标（创建时与 target 不同） |
| code | 待执行的字节码 |
| gas | Gas 限额 |
| value | 转移的 ETH 数量（wei） |

**执行分支**：

-   `target` 为空 → **合约创建**：执行 init code，设置新合约 nonce=1，生成 runtime code
    
-   `target` 有地址 → **消息调用**：转移 value，循环执行字节码，一次一条 opcode
    

**返回值** `MessageCallOutput`：

-   剩余 Gas、Gas 退款、事件日志、返回数据
    

### 5.2 系统交易（System Transactions）

> **引入时间**：Cancun Fork（2024），始于 EIP-4788

系统交易的特殊性：

-   由节点**自动创建并执行**，不来自外部用户
    
-   `tx.origin` 和 `msg.sender` 均为 `SYSTEM_ADDRESS = 0xfff...ffe`
    
-   目标是**系统合约**（System Contract）：有状态的普通合约，但系统地址有特殊写权限
    
-   **不计入区块 Gas 限额**，不执行 EIP-1559 销毁语义
    

**两种变体**：

| 变体 | 说明 | 使用场景 |
| --- | --- | --- |
| checked（检查型） | 若目标无代码或执行失败则区块无效 | EIP-7002（提款请求）、EIP-7251（合并请求） |
| unchecked（非检查型） | 不执行任何检查 | EIP-4788（Beacon 根）、EIP-2935（历史存储） |

**当前系统合约**（Prague Fork）：

-   `BeaconRoots`（EIP-4788）：存储最近 8191 个 Beacon 区块根，供 EVM 查询
    
-   `HistoryStorage`（EIP-2935）：存储区块哈希历史
    
-   `WithdrawalRequest`（EIP-7002）：允许验证者从 EL 侧触发提款
    
-   `ConsolidationRequest`（EIP-7251）：允许验证者合并质押余额
    

### 5.3 用户交易（User Transactions）

用户交易由外部账户签名后广播到网络。目前支持的交易类型：

| 类型 ID | 名称 | 说明 |
| --- | --- | --- |
| Legacy | LegacyTransaction | 原始格式（无类型前缀） |
| 0x01 | AccessListTransaction | EIP-2930，支持访问列表 |
| 0x02 | FeeMarketTransaction | EIP-1559，引入 baseFee + tip |
| 0x03 | BlobTransaction | EIP-4844，携带 blob 数据 |
| 0x04 | SetCodeTransaction | EIP-7702，EOA 临时委托智能合约代码 |

**交易处理流程**：

1.  验证签名与 nonce
    
2.  预扣 Gas（`gas_limit * max_fee_per_gas`）
    
3.  执行消息调用（`process_message_call`）
    
4.  退还未使用的 Gas
    
5.  支付优先费（tip）给区块提议者
    
6.  销毁 base fee（EIP-1559）
    

### 5.4 区块处理（Block Processing）

区块处理在 `fork.py` 中的 `apply_body` 函数实现：

```
apply_body(block_env, block_body):
  1. 执行系统交易（BeaconRoots、HistoryStorage 等）
  2. 逐笔处理用户交易
  3. 处理提款（withdrawals）
  4. 处理 EL→CL 请求（requests）
  5. 验证 Gas 用量、收据根、日志 Bloom 等
  6. 更新状态根
```

### 5.5 状态转换（State Transition）

**顶层函数**：`state_transition(chain: BlockChain, block: Block) -> None`

```python
def state_transition(chain: BlockChain, block: Block) -> None:
    # 1. 验证区块头（时间戳、Gas 上限、父区块哈希等）
    # 2. 调用 apply_body 执行区块内容
    # 3. 验证最终状态根
    # 4. 将区块追加到链
```

这是执行层的最高层接口，接受来自共识层（通过 Engine API）的新区块，执行并更新链状态。

* * *

## 6\. EIP 流程与规范演进

### 6.1 EIP 的生命周期

```
Idea（想法）
  → Draft（草稿）：在 ethereum/EIPs 提 PR
  → Review（审核）：技术讨论、安全分析
  → Last Call（最后征询）：公开意见期
  → Final（最终）：被接受，等待 Fork 纳入
  → 纳入 Fork：在 EELS 中实现 → 编写测试 → 客户端实现
```

### 6.2 EIP 的分类

| 类别 | 说明 | 例子 |
| --- | --- | --- |
| Core EIP | 协议核心变更，需要硬分叉 | EIP-1559、EIP-4844 |
| ERC | 应用层标准（Token 标准等） | ERC-20、ERC-721 |
| Networking EIP | P2P 网络相关 | EIP-2364 |
| Interface EIP | ABI、JSON-RPC 等接口 | EIP-1474 |
| Meta EIP | 流程、治理相关 | EIP-1 |

### 6.3 Fork 激活机制演进

| 时期 | 激活方式 | 说明 |
| --- | --- | --- |
| Frontier ~ Bellatrix | 区块高度（block number） | 到达指定区块号后激活 |
| Paris（The Merge） | 总难度（Total Difficulty） | TD 到达 58750000000000000000000 |
| Shanghai 之后 | 时间戳（timestamp） | 更精确，适配 PoS slot 机制 |

* * *

## 7\. 关键 API 规范

### 7.1 JSON-RPC API

执行客户端对外暴露的标准接口，规范位于 `ethereum/execution-apis`：

-   所有执行客户端必须实现的统一接口
    
-   使用 OpenRPC 规范编写（YAML 格式）
    
-   主要命名空间：`eth_`、`net_`、`web3_`
    

常用方法示例：

```
eth_getBlockByHash      # 按哈希获取区块
eth_getTransactionByHash # 获取交易详情
eth_call                # 执行只读调用
eth_estimateGas         # 估算 Gas 用量
eth_sendRawTransaction  # 广播已签名交易
```

### 7.2 Engine API

EL（执行层）与 CL（共识层）之间的**内部通信接口**，是 The Merge 引入的关键设计：

| 方法 | 方向 | 说明 |
| --- | --- | --- |
| engine_forkchoiceUpdated | CL → EL | 通知 EL 当前最优链头（head/safe/finalized） |
| engine_newPayload | CL → EL | 提交新执行载荷（一个区块的交易列表）请求验证 |
| engine_getPayload | CL → EL | 请求 EL 构建新区块（出块时） |
| engine_exchangeCapabilities | 双向 | 协商双方支持的 Engine API 版本 |

**典型流程**：

```
CL 收到新 Beacon Block
  → engine_newPayload（发送执行载荷给 EL 验证）
  → EL 返回 VALID / INVALID / SYNCING
  → engine_forkchoiceUpdated（更新链头）
```

* * *

## 8\. 测试框架

### 8.1 execution-spec-tests

`ethereum/execution-spec-tests` 是执行层规范的**合规性测试框架**：

-   测试用例用 **Python 编写**（而非手动编写 JSON fixtures）
    
-   通过 `t8n` 工具生成 JSON fixtures，可被任意执行客户端消费
    
-   主要关注**近期和即将到来的**规范变更
    
-   与 `ethereum/tests`（传统测试套件）互补，不互相取代
    

### 8.2 测试流程

```
Python 测试代码 (tests/**/*.py)
  ↓ 由 t8n 工具执行
JSON Fixtures（状态测试、区块测试）
  ↓ 被各执行客户端消费
验证客户端实现是否符合 EELS 规范
```

支持的 `t8n` 工具：

-   `evm t8n`（go-ethereum/Geth）
    
-   其他执行客户端提供的 t8n 工具
    

* * *

## 9\. 重要概念速查

| 概念 | 解释 |
| --- | --- |
| EELS | Ethereum Execution Layer Specification，Python 参考实现 |
| Fork 快照 | EELS 为每个硬分叉维护一份完整的独立代码，互不干扰 |
| 系统交易 | 节点自动创建执行的交易，用于协议级系统合约调用 |
| 系统合约 | 由系统地址（0xfff...ffe）管理的特殊链上合约 |
| EIP-7702 | 允许 EOA 临时委托智能合约代码（账户抽象的关键一步） |
| EIP-4844 Blob | 携带 blob 数据的新交易类型，用于降低 L2 数据成本 |
| Engine API | EL 与 CL 之间的内部 JSON-RPC 接口，The Merge 后引入 |
| t8n 工具 | 状态转换工具，用于生成测试 fixtures |
| EL→CL 请求 | EIP-7685 引入的跨层请求机制（如 EIP-7002 提款请求） |
| Precompile | 预编译合约，用原生代码（而非 EVM 字节码）实现的内置功能 |

* * *

## 延伸资源

-   📦 **EELS 仓库**：[github.com/ethereum/execution-specs](https://github.com/ethereum/execution-specs)
    
-   🌐 **EELS 渲染文档**：[ethereum.github.io/execution-specs](https://ethereum.github.io/execution-specs)
    
-   🔍 **Fork 间 Diff**：[ethereum.github.io/execution-specs/diffs](https://ethereum.github.io/execution-specs/diffs/index.html)
    
-   📋 **Execution APIs**：[github.com/ethereum/execution-apis](https://github.com/ethereum/execution-apis)
    
-   🧪 **Spec Tests**：[github.com/ethereum/execution-spec-tests](https://github.com/ethereum/execution-spec-tests)
    
-   📰 **EELS 官方博客**：[blog.ethereum.org/2023/08/29/eel-spec](https://blog.ethereum.org/2023/08/29/eel-spec)
    

* * *

_整理时间：2026年4月 | 基于 epf.wiki EL/el-specs 页面内容_
<!-- DAILY_CHECKIN_2026-04-09_END -->

# 2026-04-08
<!-- DAILY_CHECKIN_2026-04-08_START -->


# **Ethereum Protocol**

# 一、前史：密码朋克与比特币 (Prehistory)

**以太坊的诞生不是凭空而来，而是几十年密码学、隐私运动与数字货币实验积累的产物。**

**1.1 密码朋克运动**

**1980年代，David Chaum 开创了匿名数字现金与假名声誉系统的研究。1992年，Eric Hughes、Timothy C. May 和 John Gilmore 在旧金山湾区创立了密码朋克（Cypherpunks）小组，核心信念是：**

-   隐私是开放社会不可或缺的权利
    
-   强加密技术是个人自由的工具，不应受到政府管控
    
-   去中心化系统是对抗审查与监控的解药
    

| 核心宣言"Privacy is necessary for an open society in the electronic age." — Eric Hughes, A Cypherpunk's Manifesto (1993) |

**1.2 数字货币的先驱技术**

**在比特币出现前，密码朋克们已做出一系列关键技术尝试：**

| 年份 | 项目/人物 | 贡献 |
| 1991 | Hashcash (Adam Back) | 工作量证明（PoW）的前身，用于反垃圾邮件，后被比特币直接采用 |
| 1998 | B-money (Wei Dai) | 提出匿名、分布式电子现金系统，引入 PoW 作为铸币机制，在密码朋克邮件列表中发布 |
| 1998 | Bit Gold (Nick Szabo) | 设计去中心化数字货币，结合了智能合约思想与 PoW 共识算法，但未能解决双花问题 |
| 2004 | RPOW (Hal Finney) | 可重用工作量证明，首次实现类似UTXO 的一次性密码代币，但验证仍依赖中心服务器 |

  
  

**1.3 比特币的诞生与局限**

**2008年，Satoshi Nakamoto 发布比特币白皮书，2009年创世区块上线。比特币综合了 PoW、P2P 网络和密码学，第一次真正解决了去中心化双花问题。但比特币被设计为"极简计算器"——只支持货币转账，脚本功能非常有限，无法支撑复杂的去中心化应用。**  
  

**Vitalik Buterin 在 2011 年接触比特币，意识到需要一个更通用的平台——不仅仅是货币，而是一台"全球计算机"，能运行任意程序（智能合约）。这正是以太坊诞生的思想根源。**

# 二、以太坊协议架构 (Architecture)

**以太坊是一个分层的、双客户端架构系统。The Merge（2022年）之后，以太坊明确分为执行层（EL）和共识层（CL），通过 Engine API 协同工作。**  
  

## 2.1 三层嵌套模型

| 核心概念物理节点（Node）→ 运行以太坊客户端软件的真实计算机以太坊网络（Network）→ 全球所有节点组成的 P2P 网络以太坊虚拟机（EVM）→ 安装在节点上的可信计算平台，执行智能合约 |

**每个节点都是完整副本，这与客户端-服务器模型根本不同。所有参与以太坊的计算机都拥有完整的区块链状态，共同维护网络的去中心化与安全。**

## 2.2 执行层（Execution Layer）

**执行层负责处理交易和运行智能合约，是"计算"发生的地方。**

-   **核心：**EVM（以太坊虚拟机）
    

-   解释和执行智能合约字节码
    
-   维护账户余额与合约状态
    
-   通过 **Gas 机制** 防止滥用
    

-   **账户：**账户类型：EOA（外部账户，用户私钥控制）和 CA（合约账户，由代码控制）
    
-   **状态转换：**state\_transition 函数：接收新区块 → 验证有效性 → 更新状态树
    
-   **数据结构：**交易 → 区块 → 状态根（Merkle Patricia Trie）
    

主流执行客户端：Geth（Go）、Nethermind（C#）、Besu（Java）、Erigon（Go/重构）、Reth（Rust）

## 2.3 共识层（Consensus Layer / Beacon Chain）

**共识层负责验证者管理、区块最终确定性，是"谁有权写链"的决策机构。**

-   验证者需质押 32 ETH 参与出块与投票
    
-   使用 Gasper 协议（Casper FFG + LMD-GHOST）实现最终确定性
    
-   通过 libP2P 网络传播 Beacon 区块和 Attestation
    
-   支持同步委员会（Sync Committees）以服务轻客户端
    

主流共识客户端：Lighthouse（Rust）、Prysm（Go）、Teku（Java）、Nimbus（Nim）、Lodestar（TypeScript）

  
  

**2.4 Engine API：两层协同**

**执行层与共识层通过本地 Engine API（JSON-RPC）通信：**

-   共识层发起：engine\_forkchoiceUpdated，通知 EL 当前最优链头
    
-   共识层发起：engine\_newPayload，传递新执行载荷（交易列表）
    
-   执行层返回：执行结果与状态根，由 CL 纳入 Beacon 区块
    

  
  

| ⚠️重要：客户端多样性客户端多样性是网络韧性的关键。若任何单一客户端占比超过 66%，一旦出现严重 Bug，将威胁整个网络的最终确定性。多客户端架构是以太坊的核心安全设计。 |

  
  

**三、设计理念 (Design Rationale)**

**以太坊的每个设计决策背后都有深刻的权衡与哲学，下面梳理核心原则与关键技术选择的理由。**

**3.1 五大核心设计原则**  

| 简洁性 Simplicity | 协议尽量简单，即使以牺牲部分存储或时间效率为代价。普通程序员应能理解和实现规范。复杂度越低，审计越容易，安全漏洞越少，治理的合法性也越强。 |
| 通用性 Universality | 以太坊提供图灵完备的 EVM，不内置特定应用逻辑。任何可编程的事情都可在以太坊上实现，协议本身不偏袒任何特定用例。 |
| 模块化 Modularity | 协议各部分尽量解耦，可独立升级。Patricia Trie、RLP、执行层/共识层均设计为可独立使用的组件。创新优先推至 L2 或客户端层，而非污染 L1 规范。 |
| 敏捷性 Agility | 协议应愿意根据新发现进行修改，以太坊通过硬分叉机制持续演进。低层协议保持长期稳定，高层协议则灵活迭代。 |
| 无歧视性 Non-discrimination | 协议不主动限制或阻止特定类别的使用，监管机制应直接针对具体危害，而非反对特定应用场景。以太坊是中立基础设施。 |

  
  

**3.2 账户模型 vs UTXO 模型**

**以太坊选择账户模型而非比特币的 UTXO 模型，主要理由：**

-   简洁性：账户模型对程序员更直观，UTXO 方案在支持复杂 DApp 时极为繁琐
    
-   轻客户端友好：轻节点可通过状态树路径直接访问账户数据，UTXO 引用会随每笔交易变化
    
-   状态存储效率：账户模型下同一账户的多次交易不产生冗余 UTXO，存储更高效
    

  
  

**3.3 EVM 设计理念**

**EVM 的设计目标：**

-   简洁：尽量少的操作码和数据类型，无类型系统（以有符号/无符号 opcode 替代）
    
-   完全确定性：规范不留任何歧义，Gas 计量精确到每一步计算
    
-   紧凑性：字节码尽量小（避免 C 程序默认 4000 字节基础开销）
    
-   安全的 Gas 模型：操作码成本模型使 VM 不可被经济攻击
    
-   专用优化：原生支持 20 字节地址、32 字节大数运算、自定义密码学
    

  
  

**3.4 Merkle Patricia Trie（MPT）**

**以太坊的世界状态存储在 MPT 中，设计权衡：**

-   使用 sha3(key) 作为安全树键：防止攻击者构造深度 64 层的不利链条进行 DDoS
    
-   区分终止与非终止节点：增强通用性，使 MPT 实现可被其他协议复用
    
-   不区分空值与不存在：以简洁性换取少量通用性
    

  
  

**3.5 长期稳定性原则**

| 核心思路低层协议应构建得十年内无需修改，所有必要创新都在更高层（客户端实现或 L2 协议）发生。当复杂度不可避免时，优先顺序为：L2 协议 > 客户端实现 > 协议规范。 |

  
  

**四、以太坊发展历史 (History)**

**以太坊从 2013 年一份白皮书，成长为全球最重要的智能合约平台，经历了多个关键里程碑。**

  
  

**4.1 创世与早期（2013–2015）**

-   2013 年底：Vitalik Buterin 发布以太坊白皮书，提出"可编程区块链"概念
    
-   2014 年 1 月：在迈阿密比特币大会上正式宣布，Gavin Wood 加入并编写黄皮书（EVM 技术规范）
    
-   2014 年 7–8 月：42 天公开预售，募得约 31,591 BTC（约 1,800 万美元），分发 6,000 万 ETH
    
-   2014 年 6 月：以太坊基金会在瑞士楚格注册成立
    
-   2015 年 7 月：Frontier 上线，以太坊主网正式启动（仅面向技术用户，Gas 上限 5,000）
    

  
  

**4.2 主要升级时间线**

  
  

| 时间 | 升级名称 | 核心内容 |
| 2015.07 | Frontier | 主网上线，面向开发者，PoW 挖矿，Gas 上限 5,000 |
| 2016.03 | Homestead | 第一个计划性硬分叉；移除中心化Canary 合约；引入 Mist 钱包；Solidity 新特性 |
| 2016.07 | DAO Fork | 应对 DAO 黑客事件（~5,000 万美元 ETH 被盗）；争议性硬分叉回滚交易；催生 Ethereum Classic |
| 2017.10 | Byzantium | Metropolis 第一阶段；区块奖励从 5 ETH → 3 ETH；支持 zk-SNARKs 密码学原语；引入 Difficulty Bomb |
| 2019.02 | Constantinople | Metropolis 第二阶段；区块奖励从 3 ETH → 2 ETH；多项 EVM 效率优化 |
| 2019.12 | Istanbul | 优化 EVM Gas 成本；为 Layer 2（zk-rollup）奠定基础 |
| 2020.12 | Beacon Chain | PoS 共识层上线（独立运行），验证者开始质押 32 ETH，但尚未与主网合并 |
| 2021.08 | London / EIP-1559 | 革命性 Gas 费用改革：引入基础费用销毁机制（ETH 通缩）；改善交易费可预期性 |
| 2022.09 | The Merge（巴黎） | 执行层与共识层合并；PoW → PoS 转换；能耗降低约 99.95%；以太坊历史上最重大升级 |
| 2023.04 | Shapella | Shanghai（EL）+ Capella（CL）；首次允许验证者提取质押 ETH；完成 PoS 完整闭环 |
| 2024.03 | Dencun | 引入 EIP-4844「Proto-Danksharding」；Blob 交易大幅降低 L2 Gas 费（降幅 80-90%） |
| 2025.05 | Pectra | Prague + Electra；EIP-7702 账户抽象（EOA 获得智能合约能力）；验证者上限升至 2,048 ETH；扩展Blob 吞吐量 |

  
  

**4.3 重大历史事件**

**DAO 事件（2016）**

**DAO（去中心化自治组织）于 2016 年成为以太坊最大众筹项目，28 天内募集约 1.5 亿美元。6 月，攻击者利用重入漏洞提取约 5,000 万美元 ETH。社区就是否回滚交易产生严重分歧，最终投票通过硬分叉，分裂为ETH（回滚）和 ETC（不回滚）两条链。这一事件引发了关于区块链不可变性与社区治理的深刻讨论。**

  
  

**The Merge（2022）**

**The Merge 是以太坊史上最复杂、最受瞩目的升级。在不中断主网运行的情况下，将 PoW 执行层与 PoS 共识层（Beacon Chain，2020 年已独立运行）无缝合并。这一【在飞行中换引擎】的壮举几乎消除了以太坊的能源消耗，同时为进一步的分片和扩容升级奠定了基础。**

  
  

**4.4 以太坊路线图展望**

**Vitalik 提出的长期路线图以形象命名，围绕"可扩展、安全、去中心化"三角展开：**

  
  

-   The Merge ✅：完成 PoS 过渡
    
-   The Surge：数据分片（Danksharding），将 L2 TPS 提升到万级
    
-   The Scourge：抗 MEV 机制，保护用户和去中心化
    
-   The Verge：Verkle Trees 替换 MPT，使无状态客户端成为可能
    
-   The Purge：历史数据清理（EIP-4444），降低节点运营门槛
    
-   The Splurge：其他零散改进，包括账户抽象、隐私增强等  
    

| 2025+ 展望Vitalik 在 2025 年提出「简化 L1」愿景：未来 5 年内，将以太坊共识关键代码的复杂度降至接近比特币的水平，以提升安全性和去中心化。Beam Chain（新共识层设计）和 RISC-V EVM 替换方案是这一方向的重要探索。 |

  

本笔记基于 [_epf.wiki_](http://epf.wiki) _Protocol Wiki_ 内容整理，涵盖 _prehistory / architecture / design-rationale / history_ 四个模块
<!-- DAILY_CHECKIN_2026-04-08_END -->
<!-- Content_END -->
