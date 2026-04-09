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
