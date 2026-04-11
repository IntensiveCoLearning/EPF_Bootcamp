---
timezone: UTC+8
---

# wLynna

**GitHub ID:** wLynna

**Telegram:** 

## Self-introduction

EPF 实习计划

## Notes

<!-- Content_START -->
# 2026-04-11
<!-- DAILY_CHECKIN_2026-04-11_START -->
04/11

还得再休一天，睡觉更重要

明天一定
<!-- DAILY_CHECKIN_2026-04-11_END -->

# 2026-04-10
<!-- DAILY_CHECKIN_2026-04-10_START -->

04/10

今天折腾网络，休息下

周末补
<!-- DAILY_CHECKIN_2026-04-10_END -->

# 2026-04-08
<!-- DAILY_CHECKIN_2026-04-08_START -->


04/08

### 🧭 Ethereum Protocol Design Philosophy

* * *

## 一、核心设计理念（Core Design Philosophy）

Ethereum 的设计不是围绕某一个具体应用，而是围绕一组长期有效的原则展开。这些原则决定了以太坊为何成为一个开放、可扩展的通用平台。

Ethereum is not designed for a specific application, but around a set of long-term principles that shape it into an open and extensible platform.

* * *

### 1\. 简单性（Simplicity）

以太坊在设计之初就强调简单性，希望普通开发者也能够理解协议，从而减少对少数精英开发者的依赖。

Ethereum aims to keep the protocol as simple as possible so that average developers can understand it, reducing reliance on a small group of experts.

👉 核心理解：  
简单意味着更容易被理解，从而更有利于去中心化。

Simplicity enables accessibility, which supports decentralization.

* * *

### 2\. 通用性（Universality）

以太坊并不内置具体功能，而是提供一个图灵完备的虚拟机（EVM），允许开发者构建任意类型的应用。

Ethereum does not define specific features but provides a Turing-complete virtual machine (EVM) to support arbitrary applications.

👉 核心理解：  
以太坊不是一个应用，而是一个平台。

Ethereum is not an application, but a platform.

* * *

### 3\. 模块化（Modularity）

以太坊采用模块化设计，使不同部分可以独立演进和升级，而不影响整体系统运行。

Ethereum is designed in a modular way so that components can evolve independently without breaking the system.

👉 核心理解：  
模块化使系统更具长期适应能力。

Modularity enables long-term adaptability.

* * *

### 4\. 非歧视性（Non-discriminant）

以太坊不限制用户的使用方式，不对应用类型进行判断或干预，而是通过 gas 机制来调节资源使用。

Ethereum does not restrict use cases and does not judge applications. Resource usage is regulated through gas fees.

👉 核心理解：  
协议保持中立，只约束资源消耗。

The protocol remains neutral and only regulates resource usage.

* * *

### 5\. 灵活性（Agility）

以太坊的设计允许持续改进，通过 EIP（Ethereum Improvement Proposal）机制不断演化。

Ethereum is designed to evolve over time through mechanisms like Ethereum Improvement Proposals (EIPs).

👉 核心理解：  
协议不是静态的，而是不断发展的。

The protocol is not fixed but continuously evolving.

* * *

## 二、设计原则（Design Principles）

除了核心理念之外，以太坊还遵循一系列具体设计原则，以应对系统复杂性和长期发展问题。

* * *

### 1\. 管理复杂性（Managing Complexity）

以太坊的目标之一是尽量降低复杂性，同时保证系统功能完整。

Ethereum aims to minimize complexity while maintaining full functionality.

* * *

（1）夹心模型（Sandwich Model）

系统的底层（核心协议）和用户接口应尽量简单，而复杂性被放置在中间层。

The lowest layer and user-facing interface should be simple, while complexity is pushed into middle layers.

👉 核心理解：  
用户看到简单，复杂被隐藏。

Keep the edges simple, push complexity inward.

* * *

（2）封装复杂性（Encapsulated Complexity）

系统内部可以复杂，但对外必须提供清晰简单的接口。

Internal complexity is acceptable if it is properly encapsulated behind simple interfaces.

👉 核心理解：  
复杂可以存在，但必须被良好封装。

Complexity should be hidden behind abstraction.

* * *

### 2\. 自由（Freedom）

用户可以自由使用以太坊，协议不应限制用途或偏好某类应用。

Users should be free to use Ethereum without restrictions or preferences.

👉 核心理解：  
协议不做价值判断。

The protocol should not enforce value judgments.

* * *

### 3\. 泛化（Generalization）

以太坊更倾向于提供底层原语，而不是直接实现具体功能。

Ethereum focuses on providing low-level primitives instead of high-level features.

👉 核心理解：  
提供“积木”，而不是“成品”。

Provide building blocks, not finished products.

* * *

### 4\. 无功能原则（We Have No Features）

以太坊尽量避免将高层功能直接写入协议，而是交由智能合约实现。

Ethereum avoids embedding high-level features into the protocol, leaving them to smart contracts.

👉 核心理解：  
功能属于应用层，而不是协议层。

Features belong to the application layer, not the protocol.

* * *

## 三、区块链层设计（Blockchain-Level Design）

在具体实现上，以太坊在数据模型、结构和网络方面也体现了这些哲学。

* * *

### 1\. 账户模型（Account Model）

以太坊采用账户模型，而不是比特币的 UTXO 模型，使系统更简单和灵活。

Ethereum uses an account-based model instead of Bitcoin’s UTXO model, making it simpler and more flexible.

* * *

### 2\. 状态结构（Merkle Patricia Trie）

以太坊使用 MPT 来存储状态，使数据具有可验证性。

Ethereum uses a Merkle Patricia Trie to store state, ensuring cryptographic verifiability.

👉 核心理解：  
每一个状态变化都可以被验证。

State is verifiable through cryptographic proofs.

* * *

### 3\. Verkle Tree（未来优化）

Verkle tree 被提出作为替代方案，以提高效率并支持无状态客户端。

Verkle trees are proposed to improve efficiency and support statelessness.

* * *

### 4\. 数据序列化（RLP & SSZ）

RLP 用于早期数据编码，而 SSZ 在以太坊 2.0 中提供更高效的结构化序列化方式。

RLP is used for encoding, while SSZ improves efficiency and structure in Ethereum 2.0.

* * *

### 5\. 最终性（Finality）

通过 Casper FFG 和 LMD-GHOST，共识机制可以确保区块最终不可回滚。

Finality ensures that blocks cannot be reverted once confirmed, using Casper FFG and LMD-GHOST.

* * *

### 6\. 网络机制（Networking）

以太坊网络结合 DHT（节点发现）和 Gossip（数据传播）来实现高效通信。

Ethereum uses DHT for peer discovery and gossip protocols for data propagation.

👉 核心理解：  
DHT 用来“找节点”，Gossip 用来“传数据”。

* * *

## 四、总结（Summary）

以太坊的设计可以归纳为以下核心特征：

Ethereum’s design can be summarized as:

> **Simple, General, Modular, and Permissionless**

简单、通用、模块化、无许可。

* * *
<!-- DAILY_CHECKIN_2026-04-08_END -->

# 2026-04-07
<!-- DAILY_CHECKIN_2026-04-07_START -->



04/07

## 🧩 核心结构（Core Structure）

以太坊被拆分为两层：  
**执行层（Execution Layer, EL）**  
**共识层（Consensus Layer, CL）**

👉 分离：  
**执行（what happens） vs 共识（what is agreed）**

* * *

## 🧮 执行层（Execution Layer, EL）

所有交易真正发生的地方  
👉 执行逻辑 + 数据变化  
👉 Global Computer

~ **EVM（Ethereum Virtual Machine）**  
执行智能合约的“虚拟计算机”

~ **State（状态）**  
存储所有链上数据（余额、合约）

~ **Transactions（交易池 / mempool）**  
等待被打包的交易

~ **p2p network**  
传播交易与区块数据

* * *

## 🤝 共识层（Consensus Layer, CL）

决定哪条链是“正确的”  
👉 验证 + 选择 + 最终确认  
👉 Global Agreement

~ **Validators（验证者）**  
负责出块和投票（质押ETH）

~ **PoS（Proof of Stake）**  
基于质押的共识机制

~ **Fork Choice（LMD-GHOST）**  
决定当前主链是哪条

~ **Finality（Casper FFG）**  
确定区块不可回滚

~ **RANDAO**  
生成随机数（用于选择验证者）

~ **p2p network**  
传播区块与投票

* * *

## 🔌 层间连接（Layer Communication）

**Engine API（核心桥梁）**

~ 共识层 → 执行层  
“执行这个区块”

~ 执行层 → 共识层  
“执行结果是这样”

👉 Execution ↔ Consensus

* * *

## 🌐 对外接口（Interfaces）

~ **JSON-RPC API**  
用户 / dApp → 执行层  
（发送交易、调用合约）

~ **Beacon API**  
验证者 → 共识层  
（参与共识）

* * *

## 🧠 三层极简模型（Mental Model）

| 层 | 作用 | 关键词 |
| --- | --- | --- |
| 🧮 Execution | 做事情 | 执行 |
| 🤝 Consensus | 定规则 | 共识 |
| 🔌 Interface | 连接外部 | 入口 |

* * *

## ⚡ 一句话总结

**Ethereum = Execution + Consensus + Communication**

以太坊 = 执行 + 共识 + 连接
<!-- DAILY_CHECKIN_2026-04-07_END -->

# 2026-04-06
<!-- DAILY_CHECKIN_2026-04-06_START -->




2026/04/06

今天学习两部分

首先搞明白 如何学习，在哪里打卡

其次快速学习 以太坊 史前历史

## 《Prehistory of Ethereum》

**Ethereum is not an isolated invention — it is the natural outcome of decades of open-source culture, cryptography, and cypherpunk ideology.  
以太坊不是一个孤立的发明，而是几十年开源文化、密码学发展与赛博朋克思想的自然结果.**

### 五条核心演进路径

1.  Internet（互联网）
    
2.  Unix & Open Source（开源文化）
    
3.  Cryptography（密码学）
    
4.  Cypherpunk Movement（赛博朋克）
    
5.  Digital Currency Experiments（数字货币尝试）
    

👉 最终汇聚 → Ethereum

### Ethereum = 技术 × 文化 × 思想 × 实验 的交汇点

1️⃣技术层 Technology ：Internet Unix Cryptography，提供“能力”  
2️⃣ 文化层（Culture）： Open Source、Hacker Culture；提供“协作方式”  
3️⃣ 思想层（Ideology）：Cypherpunk，Freedom / Privacy；提供“价值观”  
4️⃣ 实验层（Experiments）：DigiCash，RSA / PGP；提供“试错路径”

### Key Insights：

Ethereum is an evolution, not an invention.  
以太坊是演化出来的，不是凭空创造的。

Cryptography made trust optional.  
密码学让“信任”变成可选项。

Open systems beat closed systems in the long run.  
开放系统最终胜过封闭系统。
<!-- DAILY_CHECKIN_2026-04-06_END -->
<!-- Content_END -->
