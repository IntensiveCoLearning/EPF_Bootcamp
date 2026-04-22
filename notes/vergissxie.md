---
timezone: UTC+8
---

# vergissxie

**GitHub ID:** vergissxie

**Telegram:** 

## Self-introduction

EPF 实习计划

## Notes

<!-- Content_START -->
# 2026-04-22
<!-- DAILY_CHECKIN_2026-04-22_START -->
# FCR（Fast Confirmation Rule）

来源

[FCR](https://epf.wiki/#/wiki/research/FCR)

## 学习定位

-   这页处理的是“最终性太慢，但用户又需要更快确认信号”这个现实问题。
    
-   它不是 finality 的替代，而是 finality 之前的一个更快、更安全的确认层。
    

## 一句话总结

-   FCR 想在良好网络同步条件下，尽可能快地判断一个区块是否已经“事实上不会离开 canonical chain”，从而提供比 finalization 更快的安全确认信号。
    

## 为什么需要它

-   现在最强的确认规则是 finalization。
    
-   但 finalization 对很多应用来说太慢：
    
    -   日常支付体验差
        
    -   CEX 充值到账要等很久
        
    -   钱包常用的“进块即算确认”其实不够安全
        
-   所以现实世界里，很多系统都在用 heuristic。
    
-   FCR 的目标是提供一个比 heuristic 更正式、更安全的快速确认规则。
    

## 核心思想

-   FCR 不是说“这个块已经 final”。
    
-   它说的是：在同步网络和诚实多数假设下，这个块已经足够稳，不会再被 honest canonical chain 排除。
    

## 它依赖什么假设

-   网络同步性足够好。
    
-   恶意 stake 占比没有超过配置阈值 `β`。
    
-   也就是说，它更偏“同步条件下的快确认”，而不是异步条件下的最终性保证。
    

## 和 Gasper 的关系

-   Ethereum 现在的 PoS 共识可粗略分成：
    
    -   `LMD-GHOST`：决定当前 head
        
    -   `FFG / Casper`：提供最终性
        
-   FCR 是建立在这些现有结构之上的补充判断层，用来更快推进“确认”。
    

## 图示理解

research-fcr-gasper.png

-   这张图适合提醒自己：
    
    -   fork choice 决定当前大家跟哪条链；
        
    -   finality 决定哪些检查点不可逆；
        
    -   FCR 试图填补两者之间的 UX 空档。
        

## 它真正提供了什么性质

-   `Safety`：已确认区块不会在 honest view 中被回滚。
    
-   `Monotonicity`：确认不会来回倒退，除非触发 reset-to-finalized 之类的假设失效处理。
    

## 为什么这页和 PBS 也有关系

-   builder / proposer / 交易用户都很关心“这个块到底稳不稳”。
    
-   如果能更快更安全地知道某块已相当稳固：
    
    -   builder 风险评估会不同；
        
    -   用户体验会更好；
        
    -   一些快速交互型场景会更现实。
        

## 和 SSF 的关系

-   长期看，大家希望走向 `Single Slot Finality`。
    
-   但在 SSF 真正落地前，FCR 可以被理解为“填补空档期”的快确认研究。
    

## 和已有笔记的关系

-   [Preconfirmations](10 - Web3/Ethereum 协议/EPF/Research 与 Development/Research/Preconfirmations)
    
-   [PBS](10 - Web3/Ethereum 协议/EPF/Research 与 Development/Research/PBS)
    
-   [Roadmap overview](10 - Web3/Ethereum 协议/EPF/Research 与 Development/Research/Roadmap overview)
    
-   [Overview](10 - Web3/Ethereum 协议/EPF/Consensus Layer \(CL\)/Overview)
    

## 我的理解

-   FCR 可以看作“在 finalization 之前，给现实世界一个更靠谱的 yes / no 信号”。
    
-   它的价值不在于替代最终性，而在于把“太保守”和“太草率”之间的空档补起来。
<!-- DAILY_CHECKIN_2026-04-22_END -->

# 2026-04-21
<!-- DAILY_CHECKIN_2026-04-21_START -->

休息+1
<!-- DAILY_CHECKIN_2026-04-21_END -->

# 2026-04-20
<!-- DAILY_CHECKIN_2026-04-20_START -->


# Beacon API（Beacon 节点接口）

来源

[Beacon Chain Spec and APIs](https://epf.wiki/#/wiki/CL/beacon-api)

## 学习定位

-   **所属模块**：Consensus Layer 节点对外 API。
    
-   **前置概念**：Beacon State、validator、slot、epoch、block。
    
-   **后续关联**：[CL Specs](CL Specs)、[Client architecture](10 - Web3/Ethereum 协议/EPF/Consensus Layer \(CL\)/进阶部分/Client architecture)、validator client。
    
-   **本文解决的问题**：外部程序如何读取 CL 节点数据、提交 validator 相关操作、和 Beacon Node 交互？
    

## 一句话总结

Beacon API 是访问 Consensus Layer 节点的 HTTP API 标准，提供读取 beacon chain 状态、区块、validator 信息、finality 信息以及 validator client 所需数据的接口。

## 核心心智模型

如果 `JSON-RPC` 是 EL 给钱包和 DApp 的主要入口，那么 Beacon API 就是 CL 节点暴露共识层状态和 validator workflow 的入口。

它面向的使用者包括：

-   validator client。
    
-   staking 服务。
    
-   区块浏览器和索引器。
    
-   研究工具、监控系统、协议开发者。
    

## 术语解释

-   **Beacon Node（信标节点）**：运行 CL 客户端的节点，维护 beacon chain 状态。
    
-   **Validator Client（验证者客户端）**：管理 validator key，并通过 Beacon API 获取职责、提交签名结果。
    
-   **REST API**：Beacon API 常见的 HTTP 接口形式。
    
-   **Container**：API 中返回的结构化数据对象，通常和 CL specs 数据结构对应。
    

## API 关注的对象

Beacon API 通常围绕这些对象：

-   **Blocks**：slot 中提议的 beacon block。
    
-   **States**：某个 slot / block root 对应的 beacon state。
    
-   **Validators**：validator registry、余额、状态。
    
-   **Committees**：某个 epoch/slot 的 committee assignment。
    
-   **Attestations**：validators 对链头和 checkpoint 的投票。
    
-   **Finality checkpoints**：justified / finalized checkpoint。
    
-   **Node info**：节点同步状态、peer 信息、版本信息。
    

## Containers（数据容器）

原文强调 containers，是因为 Beacon API 返回的对象不是随意 JSON，而是和 consensus specs 中的 SSZ 数据结构有对应关系。

理解 Beacon API 时，不要只把它当成“查接口”，而要把它看成 specs 数据结构的外部视图：

-   API 字段来自 CL 对象。
    
-   字段含义应回到 specs 理解。
    
-   不同客户端需要在 API 行为上保持兼容。
    

## 和 EL JSON-RPC 的区别

-   **Beacon API**：面向 CL 状态，如 validators、attestations、finality、beacon blocks。
    
-   **EL JSON-RPC**：面向执行层状态，如账户、交易、合约、logs、receipts。
    
-   **Engine API**：不是给普通外部用户用的，而是 EL 与 CL 内部通信接口。
    

## 易混淆点

-   Beacon API 不是 Engine API。
    
-   Beacon API 也不是以太坊执行层 JSON-RPC。
    
-   Validator client 通常通过 Beacon API 和 Beacon Node 协作，但 validator key 不应该随便暴露给 Beacon Node。
    

## 和已有笔记的关系

-   [Overview](Overview)：Beacon API 暴露的是 Overview 中讲到的 CL 状态对象。
    
-   [CL Specs](CL Specs)：API 数据结构和 specs 中的对象强相关。
    
-   [Client architecture](10 - Web3/Ethereum 协议/EPF/Consensus Layer \(CL\)/进阶部分/Client architecture)：Beacon API 属于 CL client 对外提供服务的一部分。
    

## 我的理解

Beacon API 是把共识层内部状态“打开一扇窗”。对学习 CL 来说，它很适合作为 specs 和实际客户端之间的桥：读 specs 知道对象是什么，读 API 知道外部如何访问这些对象。

## 复习问题

-   Beacon API 和 EL JSON-RPC 分别面向哪些数据？
    
-   Validator client 为什么需要 Beacon API？
    
-   Beacon API 返回的数据和 SSZ/specs 有什么关系？
    
-   Beacon API 和 Engine API 的使用者有什么区别？
<!-- DAILY_CHECKIN_2026-04-20_END -->

# 2026-04-19
<!-- DAILY_CHECKIN_2026-04-19_START -->



# Client architecture（CL 客户端架构）

来源

[Consensus Layer architecture](https://epf.wiki/#/wiki/CL/cl-architecture)

## 学习定位

-   **所属模块**：Consensus Layer 客户端内部设计。
    
-   **前置概念**：[Overview](Overview)、fork choice、attestation、state transition、networking。
    
-   **后续关联**：[CL Clients](CL Clients)、[CL Networking](CL Networking)、[Beacon API](Beacon API)。
    
-   **本文解决的问题**：CL client 内部如何组织 block tree、fork choice、state transition、network 和 validator duties？
    

## 一句话总结

CL 客户端负责维护 Beacon Chain 的共识视图：接收网络消息、验证区块和 attestations、维护 block tree、执行状态转换、运行 fork choice，并通过 Beacon API 服务 validator client 和外部查询。

## 核心心智模型

CL client 像一个“实时共识状态机”：

-   网络层不断输入 blocks、attestations、sync committee messages。
    
-   状态转换验证这些输入是否符合 specs。
    
-   fork choice 在 block tree 中选择 head。
    
-   API 层把当前视图暴露给 validator client 和外部工具。
    

## 术语解释

-   **Block Tree（区块树）**：CL 看到的候选区块结构，可能存在分叉。
    
-   **Fork Choice（分叉选择）**：根据投票权重和规则选择当前 head。
    
-   **LMD-GHOST**：CL 使用的 fork choice 核心思想之一。
    
-   **State Transition（状态转换）**：按 specs 将旧状态和 block 输入转换为新状态。
    
-   **Beacon State**：CL 当前维护的核心状态对象。
    
-   **Validator Duties（验证者职责）**：validator 在某个 slot/epoch 的 proposer、attester 等任务。
    

## Fork-choice Mechanism（分叉选择机制）

Fork choice 的目标是从 block tree 中选出当前 head。

cl-architecture-blocktree.svg

### 图解：Block Tree

-   **树不是链**：节点可能同时看到多个候选分支。
    
-   **投票会改变权重**：attestations 会影响 fork choice 的选择。
    
-   **head 是当前选择**：head 不是永远固定，会随新区块和投票更新。
    

## Reorg（重组）

当新的投票或区块让另一个分支权重更高时，客户端可能发生 reorg。

cl-architecture-reorg-0.svg

### 图解：Reorg 前

-   客户端当前认为某个分支是 head。
    
-   另一个分支可能已经存在，但权重不足。
    

cl-architecture-reorg-1.svg

### 图解：Reorg 后

-   新投票让另一个分支成为更优 head。
    
-   客户端切换 head，这就是 reorg。
    
-   finality 越深，reorg 成本越高。
    

## Architecture（整体架构）

CL client 通常由网络、存储、状态转换、fork choice、API、validator 相关模块组成。

cl-architecture-cl.png

### 图解：CL Client 内部模块

-   **Network**：接收和广播 blocks、attestations。
    
-   **State Transition**：验证 block 并更新 Beacon State。
    
-   **Fork Choice**：维护 block tree 并选择 head。
    
-   **Storage**：保存 blocks、states、metadata。
    
-   **Beacon API**：对外提供查询和 validator workflow 接口。
    

## Network（网络视角）

CL client 需要在 P2P 网络中传播共识消息。

cl-architecture-network.png

### 图解：网络如何驱动客户端

-   网络输入包括 blocks、attestations、sync messages。
    
-   客户端先验证消息，再更新本地状态。
    
-   有效消息会继续 gossip，帮助全网收敛。
    

## State Transitions（状态转换）

状态转换是 CL client 的规则核心。给定旧 state 和 block，客户端必须得到唯一的新 state。

状态转换通常包括：

-   slot processing。
    
-   block processing。
    
-   epoch processing。
    
-   rewards / penalties。
    
-   validator registry updates。
    

## Validity Conditions（有效性条件）

客户端不能只接收消息，还要判断消息是否有效：

-   block proposer 是否正确。
    
-   签名是否有效。
    
-   slot/epoch 是否匹配。
    
-   state root 是否一致。
    
-   attestation 是否符合当前 fork choice 和 committee assignment。
    

## 易混淆点

-   **Fork choice 不等于 finality**：fork choice 选择当前 head，finality 提供更强经济确定性。
    
-   **Network 收到不代表接受**：消息必须经过验证才能影响状态。
    
-   **CL client 和 validator client 不同**：CL client 维护链状态，validator client 管理签名职责。
    

## 和已有笔记的关系

-   [Overview](Overview)：这里把 Overview 的概念落到客户端内部模块。
    
-   [CL Networking](CL Networking)：网络层是 CL client 的输入通道。
    
-   [CL Specs](CL Specs)：状态转换和有效性判断都来自 specs。
    
-   [Beacon API](Beacon API)：API 是 CL client 对外暴露状态和职责的接口。
    

## 我的理解

CL client 的难点在于它同时是网络节点、状态机和投票解释器。它必须在网络延迟和分叉存在的情况下，持续维护一个“当前最合理的链头”，并保证每一步状态转换都符合 specs。

## 复习问题

-   Fork choice 和 finality 有什么区别？
    
-   CL client 为什么需要维护 block tree？
    
-   网络消息进入 CL client 后要经历哪些验证？
    
-   Validator client 和 CL client 的职责边界是什么？
<!-- DAILY_CHECKIN_2026-04-19_END -->

# 2026-04-18
<!-- DAILY_CHECKIN_2026-04-18_START -->




# Client architecture（CL 客户端架构）

来源

[Consensus Layer architecture](https://epf.wiki/#/wiki/CL/cl-architecture)

## 学习定位

-   **所属模块**：Consensus Layer 客户端内部设计。
    
-   **前置概念**：[Overview](Overview)、fork choice、attestation、state transition、networking。
    
-   **后续关联**：[CL Clients](CL Clients)、[CL Networking](CL Networking)、[Beacon API](Beacon API)。
    
-   **本文解决的问题**：CL client 内部如何组织 block tree、fork choice、state transition、network 和 validator duties？
    

## 一句话总结

CL 客户端负责维护 Beacon Chain 的共识视图：接收网络消息、验证区块和 attestations、维护 block tree、执行状态转换、运行 fork choice，并通过 Beacon API 服务 validator client 和外部查询。

## 核心心智模型

CL client 像一个“实时共识状态机”：

-   网络层不断输入 blocks、attestations、sync committee messages。
    
-   状态转换验证这些输入是否符合 specs。
    
-   fork choice 在 block tree 中选择 head。
    
-   API 层把当前视图暴露给 validator client 和外部工具。
    

## 术语解释

-   **Block Tree（区块树）**：CL 看到的候选区块结构，可能存在分叉。
    
-   **Fork Choice（分叉选择）**：根据投票权重和规则选择当前 head。
    
-   **LMD-GHOST**：CL 使用的 fork choice 核心思想之一。
    
-   **State Transition（状态转换）**：按 specs 将旧状态和 block 输入转换为新状态。
    
-   **Beacon State**：CL 当前维护的核心状态对象。
    
-   **Validator Duties（验证者职责）**：validator 在某个 slot/epoch 的 proposer、attester 等任务。
    

## Fork-choice Mechanism（分叉选择机制）

Fork choice 的目标是从 block tree 中选出当前 head。

cl-architecture-blocktree.svg

### 图解：Block Tree

-   **树不是链**：节点可能同时看到多个候选分支。
    
-   **投票会改变权重**：attestations 会影响 fork choice 的选择。
    
-   **head 是当前选择**：head 不是永远固定，会随新区块和投票更新。
    

## Reorg（重组）

当新的投票或区块让另一个分支权重更高时，客户端可能发生 reorg。

cl-architecture-reorg-0.svg

### 图解：Reorg 前

-   客户端当前认为某个分支是 head。
    
-   另一个分支可能已经存在，但权重不足。
    

cl-architecture-reorg-1.svg

### 图解：Reorg 后

-   新投票让另一个分支成为更优 head。
    
-   客户端切换 head，这就是 reorg。
    
-   finality 越深，reorg 成本越高。
    

## Architecture（整体架构）

CL client 通常由网络、存储、状态转换、fork choice、API、validator 相关模块组成。

cl-architecture-cl.png

### 图解：CL Client 内部模块

-   **Network**：接收和广播 blocks、attestations。
    
-   **State Transition**：验证 block 并更新 Beacon State。
    
-   **Fork Choice**：维护 block tree 并选择 head。
    
-   **Storage**：保存 blocks、states、metadata。
    
-   **Beacon API**：对外提供查询和 validator workflow 接口。
    

## Network（网络视角）

CL client 需要在 P2P 网络中传播共识消息。

cl-architecture-network.png

### 图解：网络如何驱动客户端

-   网络输入包括 blocks、attestations、sync messages。
    
-   客户端先验证消息，再更新本地状态。
    
-   有效消息会继续 gossip，帮助全网收敛。
    

## State Transitions（状态转换）

状态转换是 CL client 的规则核心。给定旧 state 和 block，客户端必须得到唯一的新 state。

状态转换通常包括：

-   slot processing。
    
-   block processing。
    
-   epoch processing。
    
-   rewards / penalties。
    
-   validator registry updates。
    

## Validity Conditions（有效性条件）

客户端不能只接收消息，还要判断消息是否有效：

-   block proposer 是否正确。
    
-   签名是否有效。
    
-   slot/epoch 是否匹配。
    
-   state root 是否一致。
    
-   attestation 是否符合当前 fork choice 和 committee assignment。
    

## 易混淆点

-   **Fork choice 不等于 finality**：fork choice 选择当前 head，finality 提供更强经济确定性。
    
-   **Network 收到不代表接受**：消息必须经过验证才能影响状态。
    
-   **CL client 和 validator client 不同**：CL client 维护链状态，validator client 管理签名职责。
    

## 和已有笔记的关系

-   [Overview](Overview)：这里把 Overview 的概念落到客户端内部模块。
    
-   [CL Networking](CL Networking)：网络层是 CL client 的输入通道。
    
-   [CL Specs](CL Specs)：状态转换和有效性判断都来自 specs。
    
-   [Beacon API](Beacon API)：API 是 CL client 对外暴露状态和职责的接口。
    

## 我的理解

CL client 的难点在于它同时是网络节点、状态机和投票解释器。它必须在网络延迟和分叉存在的情况下，持续维护一个“当前最合理的链头”，并保证每一步状态转换都符合 specs。

## 复习问题

-   Fork choice 和 finality 有什么区别？
    
-   CL client 为什么需要维护 block tree？
    
-   网络消息进入 CL client 后要经历哪些验证？
    
-   Validator client 和 CL client 的职责边界是什么？
<!-- DAILY_CHECKIN_2026-04-18_END -->

# 2026-04-17
<!-- DAILY_CHECKIN_2026-04-17_START -->





# Weak Subjectivity（弱主观性）

来源

[Syncing in Weak Subjectivity](https://epf.wiki/#/wiki/CL/syncing)

## 学习定位

-   **所属模块**：Consensus Layer 安全同步。
    
-   **前置概念**：Proof of Stake、finality、validator set、checkpoint。
    
-   **后续关联**：[Overview](Overview)、[Client architecture](10 - Web3/Ethereum 协议/EPF/Consensus Layer \(CL\)/进阶部分/Client architecture)、light client。
    
-   **本文解决的问题**：新节点或离线很久的节点，为什么不能只靠“最长链/最重链”无条件同步？
    

## 一句话总结

Weak Subjectivity（弱主观性）是 PoS 链同步中的安全前提：节点如果离线太久或刚加入网络，需要从可信来源获得一个近期 checkpoint，才能安全地区分真实链和长程攻击链。

## 核心心智模型

PoW 的安全性更多依赖历史工作量，而 PoS 的安全性依赖当前和近期 validator set 的经济约束。如果攻击者拿到很久以前的 validator keys，可能构造一条“看起来合理”的旧历史分叉。

Weak Subjectivity 的意思是：节点需要一点点来自社会层/可信来源的近期信息，才能安全进入网络。

## 术语解释

-   **Weak Subjectivity（弱主观性）**：PoS 节点同步时需要可信近期 checkpoint 的性质。
    
-   **Checkpoint（检查点）**：通常是某个 finalized epoch 的状态锚点。
    
-   **Weak Subjectivity Period（弱主观期）**：节点可以安全离线而不需要额外可信信息的时间窗口。
    
-   **Long-range Attack（长程攻击）**：攻击者用旧 validator keys 构造历史分叉的攻击。
    
-   **Syncing（同步）**：节点追赶当前链状态的过程。
    

## 为什么 PoS 需要 Weak Subjectivity

在 PoS 中，validator 退出后，其旧密钥可能被泄漏或出售。攻击者可以用这些旧密钥签一条从很久以前开始的替代链。

如果一个节点完全离线很久，再回来时只看“签名是否有效”，就可能无法区分：

-   真实主链。
    
-   攻击者用旧 validator set 构造的历史链。
    

所以节点需要一个近期可信 checkpoint，作为同步锚点。

## Syncing in Weak Subjectivity（弱主观同步）

安全同步的直觉流程：

1.  从可信来源获得近期 finalized checkpoint。
    
2.  从该 checkpoint 开始验证后续链。
    
3.  检查 validator set、finality 和状态转换是否连续有效。
    
4.  同步到当前 head。
    

这个 checkpoint 不一定来自单一中心化来源，可以来自多个客户端团队、区块浏览器、社区公告、自己之前保存的节点状态等。

## 易混淆点

-   **Weak Subjectivity 不是中心化共识**：它只是新节点入网或长期离线后需要一个近期锚点。
    
-   **finalized 不代表永远无需信任来源**：如果你离线超过弱主观期，仍需要可信 checkpoint。
    
-   **同步快慢不是核心问题**：核心问题是同步到哪条链才安全。
    

## 和已有笔记的关系

-   [Overview](Overview)：finality 和 validator lifecycle 是理解弱主观性的基础。
    
-   [Client architecture](10 - Web3/Ethereum 协议/EPF/Consensus Layer \(CL\)/进阶部分/Client architecture)：CL client 的 sync 逻辑必须考虑弱主观 checkpoint。
    
-   [Merkleization](10 - Web3/Ethereum 协议/EPF/Consensus Layer \(CL\)/基础部分/SSZ Serialization/Merkleization)：checkpoint/root 可信后，proof 才有安全意义。
    

## 我的理解

Weak Subjectivity 是 PoS 安全模型里很诚实的一点：协议不假装“完全不需要外部信息”。它承认新节点需要一个近期社会共识锚点，但只要进入正确链后，后续验证仍然由密码学、状态转换和经济惩罚来保证。

## 复习问题

-   为什么 PoS 比 PoW 更需要 weak subjectivity checkpoint？
    
-   长程攻击利用了什么历史条件？
    
-   Weak Subjectivity Period 的意义是什么？
    
-   一个节点离线很久后，应该如何安全恢复同步？
<!-- DAILY_CHECKIN_2026-04-17_END -->

# 2026-04-16
<!-- DAILY_CHECKIN_2026-04-16_START -->






### Gas Accounting

Intrinsic Gas Calculation

Intrinsic Gas是交易开始执行前所需的最低Gas成本。  
这笔费用会在交易一开头，直接从你设置的 `gasLimit` 中扣除。只有扣完这笔“入场费”后，剩下的 Gas 才会喂给 EVM 去执行代码。如果你的 `gasLimit` 连这笔入场费都不够，交易会直接报错 `Out of Gas`，连 EVM 的门都进不去。

以下是Intrinsic Gas的5个组成部分：

1.  G\_{transaction}（基础费）
    

-   **费用**：**21000 Gas**
    
-   **含义**：这是基础手续费。它用于支付以太坊网络处理一笔普通转账的最基础的密码学签名验证和账户状态修改的成本。
    

1.  `CALLDATA` 成本（数据存储费）
    

-   **费用**：**非零字节 16 Gas / 零字节 4 Gas**
    
-   **公式对应**：\\sum\_{i \\in \\{T\_{inputData}\\}} \\begin{cases} G\_{txdatazero} & \\text{if } i = 0 \\\\ G\_{txdatanonzero} & \\text{otherwise} \\end{cases}
    
-   **含义**：如果你在交易里附带了留言，或者你要调用智能合约（这就需要传递参数数据 `inputData`），系统会按字节对这些数据收“托运费”。
    
    -   数据里的内容只要不是 0（非零字节），每个字节收 **16 Gas**。
        
    -   数据里的内容如果是 0（零字节），每个字节只收 **4 Gas**。
        
-   **💡 为什么 0 比较便宜？** 因为对底层硬盘的 KV 数据库和网络带宽来说，大量的 0 在压缩和传输时更容易，占用的物理资源更少。（注：文本中提到，这套 16/4 的计费模型最初是为 PoW 时代优化的，在现在的 PoS 时代，这部分定价未来可能会有进一步优化的空间）。  
    

3.  G\_{txcreate}（合约部署费）
    

-   **费用**：**32000 Gas**
    
-   **公式对应**：  
    \\begin{cases} G\_{txcreate} & \\text{if } T\_{to} = \\emptyset \\\\ 0 & \\text{otherwise} \\end{cases}
    
-   **含义**：如果你的交易没有接收方（`To` 地址为空，也就是 T\_{to} = \\emptyset），以太坊就知道你这不是在转账，而是在**部署一个全新的智能合约**。
    

4.  G\_{initCodeWordCost} (Init Code Cost)
    

-   **公式对应**：  
    G\_{initCodeWordCost} \\times \\text{words}(length) (向上取整到 32 字节的倍数)
    
-   **含义**：这是上海升级（Shanghai Specification）后引入的新规。如果你在部署合约（结合上一条），你不仅要交 32000 的G\_{txcreate}，还要为你上传的初始化代码 `initializationCode`按体积交钱。  
    计费方式是**以 32 个字节（一个 Word）为一段**。哪怕你只多出一个字节，也会按一整段 32 字节来收费。
    

5.  Access List 成本
    

-   **公式对应**：  
    \\sum\_{j=0}^{length-1} (G\_{accesslistaddress} + length(keys) \\times G\_{accessliststorage})
    
-   **含义**：这是 EIP-2930 引入的功能。相当于你提前给系统列了一个“访问清单（Access List）”。系统把这些数据提前从硬盘加载到内存里，后续 EVM 执行时会便宜很多。但天下没有免费的午餐，你提前列这个清单时，必须在入场费里预先支付：每个地址大约收 2400 Gas，每个抽屉（Key）收 1900 Gas。
    

Effective Gas Fee

核心公式（已扣除预付费用）：  
effectiveGasFee≡effectiveGasPrice×T\_{gasLimit}

这是决定你这笔交易**最终按多少钱一单位 Gas 来算账**的逻辑。系统必须兼容老版本的交易（Type 0/1）和 EIP-1559 后的新交易（Type 2/3）。

-   **Type 0 或 1：**
    
    -   `gasPrice` 就是实际单价 p。
        
    -   小费 f 就是你的出价减去系统当前的基础费：f = T\_{gasPrice} - H\_{baseFeePerGas}。  
        
-   **Type 2 或 3：**
    
    -   **小费计算 (f)：** 取决于你设置的“最大小费”和“最大总费用减去基础费”中**较小的一个**。公式极其严谨：  
        f \\equiv \\min(T\_{maxPriorityFeePerGas}, T\_{maxFeePerGas} - H\_{baseFeePerGas}) _这意味着：系统会优先保证够付 Base Fee，剩下的余额才拿来给矿工发小费。_
        
    -   **实际单价 (p)：** 实际单价等于你给的实际小费加上系统当前的基础费。  
        p \\equiv f + H\_{baseFeePerGas}
        

Blob Gas Math

如果你的交易是坎昆升级后的 `Type 3`（携带了 Blob 数据），系统会用一套完全独立的逻辑来计算你的Gas费：

-   **Total Blob Gas：**  
    每个 Blob 固定消耗 **131,072**（即 2^{17}）个 Blob Gas。总耗费就是 Blob 数量乘以这个基数：totalBlobGas \\equiv 2^{17} \\times length(T\_{blobVersionedHashes})
    
-   **Blob Gas Price：** 如果网络没满载，价格就是 **1 Wei**。如果累积超载（`excessBlobGas` 增加），价格就会按照指数级狂飙：  
    blobGasPrice \\approx 1 \\times e^{excessBlobGas / 3338477}
    
-   **Blob Gas Fee：**  
    blobGasFee≡totalBlobGas×blobGasPrice
    

Max Gas Fee

在真正让你执行交易前，以太坊的节点必须做一个最坏打算的检查：**你的钱包余额够不够付可能产生的最大费用？** 这就是 `maxGasFee`。

-   **Type 0/1:** 你的最大开销就是 `GasLimit` 乘上你指定的固定 `GasPrice`。
    
-   **Type 2:** 你的最大开销是 `GasLimit` 乘上你设置的 `MaxFeePerGas`。
    
-   **Type 3 (Blob):** 这是最贵的组合。你的最大开销是**“常规交易的最大开销” 加上 “Blob 数据的最大开销”**。
    

maxGasFee \\equiv \\begin{cases} T\_{gasLimit} \\times T\_{gasPrice}, & \\text{if } T\_{type} = 0 \\lor 1 \\\\ T\_{gasLimit} \\times T\_{maxFeePerGas}, & \\text{if } T\_{type} = 2 \\\\ (T\_{gasLimit} \\times T\_{maxFeePerGas}) + maxBlobFee, & \\text{if } T\_{type} = 3 \\end{cases}

-   _注意：这只是为了查你钱包的可用余额（验资），不代表最后一定会扣这么多钱。_  
    

Up-Front Cost

一旦验资通过，交易准备进入 EVM 虚拟机执行。在执行第一行代码之前，系统会无情地从你的账户里**预先扣除一笔钱 (v\_0)**。

这笔预扣款的公式非常简单明了，它等于你的“常规交易执行预算”加上“Blob 货运费账单”：

v\_0 \\equiv (effectiveGasPrice \\times T\_{gasLimit}) + blobGasFee  
  

  
Client的整体架构

697

这张架构图详细展示了以太坊在合并（The Merge）之后的执行层客户端（Execution Layer Client）的内部构造，以及它与共识层（Consensus Layer）、P2P 网络和外部用户之间的交互逻辑。

以下是对各个组件、架构流程及核心业务流程的严谨解析：

### 一、 核心组件解析 (Components)

1\. 外部接口与网关

-   **User/web3 & JSON-RPC API:**
    
    -   **定义：** 节点的外部访问接口。
        
    -   **作用：** 允许 DApp 前端、钱包（如 MetaMask）或开发者通过 HTTP/WebSocket 发送 JSON-RPC 请求，例如查询余额、读取合约状态或发起新的交易。  
        
-   **Engine API:**
    
    -   **定义：** 执行层（EL）与共识层（CL）之间的标准通信接口，通常基于经过 JWT 身份验证的 RPC。
        
    -   **作用：** 它是整个架构的中枢，负责接收来自共识层的区块指令（如 `New Payload` 执行新区块，`Fork Choice Updated` 更新分叉选择），并将执行层的交易池数据（通过 `Sync Initiation & Block Builders Pipeline`）传递给区块构建器。  
        

2\. 网络通信层 (DevP2P)

-   这是以太坊底层的点对点网络协议栈，负责与其他执行层客户端进行通信（Gossips）。
    
-   **Discovery (发现模块):** 基于 UDP 和 IP 协议，负责在全网广播并寻找其他活跃的以太坊节点。
    
-   **Transport (传输模块):** 基于 TCP 和 IP 协议，并封装了 RLPx（一种加密和认证传输协议），负责节点间实际的数据流传输（如区块下载、交易广播）。  
    

3\. 核心处理引擎

-   **Transactions (mempool / 交易池):**
    
    -   **作用：** 内存中的暂存区，用于存放已经过初步验证但尚未被打包进区块的待处理交易。它通过 DevP2P 接收全网广播的交易，同时也准备将高价值交易提供给区块构建过程。  
        
-   **EVM (以太坊虚拟机):**
    
    -   **作用：** 系统的“CPU”与状态转换引擎。负责解析智能合约字节码、执行运算、扣除 Gas，并执行核心逻辑。它接收来自 Engine API 的 `Payload Validation & Insertion Pipeline` 指令，对区块内的交易进行最终处理（Final Processing）。  
        
-   **State & Sync (状态与同步机制):**
    
    -   **State:** 持久化存储以太坊世界状态的数据库（通常基于 MPT/Verkle 树结构底层使用 LevelDB/RocksDB），记录所有账户的余额、Nonce 及合约数据。
        
    -   **Sync:** 负责网络落后时的状态追赶。通过 DevP2P 下载历史区块（Download Blocks），并交由 EVM 进行历史验证（Validate Blocks），最终更新到本地 State。  
        

### 二、 架构级通信流程 (Architectural Flow)

架构图清晰地划分了执行层客户端的三个主要数据流向：

1.  **横向交互（端到端通信）：**
    
    客户端通过左侧的 `DevP2P` 接口，持续不断地与其他执行层客户端节点进行 P2P Gossip 广播。网络层接收到远端交易后，存入本地 Mempool；接收到远端区块后，触发 Sync 模块。
    
2.  **纵向交互（层级通信）：**
    
    客户端通过顶部的 `Engine API` 严格听从共识层的指挥。共识层负责决定“哪条链是主链”（LMD-GHOST），而执行层只负责“无脑”执行共识层递交过来的数据包（Payload），并将执行结果反馈给共识层。
    
3.  **外部交互（客户端服务）：**
    
    通过右侧的 `JSON-RPC`，向外部世界提供读写以太坊状态的能力。
    

### 三、 具体业务流程 (Business Flows)

结合架构图，我们可以梳理出以太坊节点日常运作的三大核心业务流程：

业务流程一：用户发起交易 (Transaction Submission)

1.  **接收与验证：** 用户通过钱包签署一笔交易，经由 `JSON-RPC API` 提交给客户端。
    
2.  **入池：** 客户端初步校验交易格式、签名有效性及发起者余额。校验通过后，交易进入 `Transactions (mempool)`。
    
3.  **广播：** 执行层客户端通过左侧的 `DevP2P` 模块，将这笔交易 Gossip 广播给与其相连的其他对等节点，使该交易迅速蔓延至全网交易池。
    

业务流程二：接收并执行新区块 (Block Execution & Payload Validation)

这是合并后最关键的流程，严格受共识层驱动。

1.  **接收指令：** 共识层节点在 P2P 网络中监听到一个新区块，提取出其中的交易列表（Payload），通过 `Engine API` 调用 `New Payload` 发送给执行层。
    
2.  **流转至 EVM：** Payload 沿着 `Payload Validation & Insertion Pipeline` 进入 `EVM`。
    
3.  **状态计算：** EVM 从 `mempool` 提取或直接基于 Payload 内的交易，对照当前的 `State` 依次执行。进行签名核对、Gas 扣除和余额转移。
    
4.  **状态更新：** 执行完毕后，EVM 将产生一个新的状态根。执行层将结果（VALID/INVALID）返回给共识层。
    
5.  **分叉确认：** 随后，共识层经过网络投票达成共识后，通过 `Engine API` 发送 `Fork Choice Updated` 指令，通知执行层：“这个区块已确认，将其设为链的最新头部”。
    

业务流程三：节点数据同步 (Node Synchronization)

当一个新节点刚启动，或节点因断网落后于主网时触发：

1.  **触发同步：** `Sync` 模块向 `DevP2P` 发出请求，开始向其他对等节点请求缺失的区块数据（对应图中的 `Download Blocks` 虚线）。
    
2.  **数据回传：** 下载到的区块数据被送入内部流水线。
    
3.  **历史验证：** `EVM` 根据下载的区块，从创世区块或特定快照点开始，重放历史交易（对应图中的 `Validate Blocks` 虚线）。
    
4.  **状态落盘：** 每重放完一个有效区块，更新并持久化本地 `State`，直至追平当前共识层指示的最新头部。
<!-- DAILY_CHECKIN_2026-04-16_END -->

# 2026-04-14
<!-- DAILY_CHECKIN_2026-04-14_START -->







### Block Execution Process

以太坊处理区块的总流程：先查头 -> 置空变量 -> 接通共识层拿数据 -> 挨个执行交易 -> 发放提款工资 -> 更新全局状态。 并且它详细规定了每次启动虚拟机时，必须给虚拟机喂什么样的“全局上下文参数”。

1.  Initialization
    

把区块头检查放在最前面，是为了“防白嫖计算力”。如果区块头有错，直接扔给共识层一个“无效负载（Invalid Payload）”的报错。这样就不需要启动极其耗费 CPU 的 EVM 虚拟机了。

-   **Initialize** `blobGasUsed` **to 0**
    
-   **set Gas (**`gasAvailable`**) to H\_{gasLimit}**
    
-   **Initialize additional execution components**，包含收据（Receipts）、提款（Withdrawals）和日志（Logs）的空篮子
    

2.  连接共识层 (EIP-4788)
    

这是坎昆（Dencun/Duncan）升级中引入的一个极其重要的新特性。

以前，执行层（EVM，管智能合约的）和共识层（信标链，管质押和投票的）是物理隔离的，EVM 就像个瞎子，根本不知道信标链上发生了什么。

-   **系统级交易 (System Transaction)：** 协议设计了一个非常巧妙的后门。在每次执行普通用户的交易之前，以太坊会**强行塞入一笔由“系统地址”发起的“系统交易”**。  
    
-   **目的：** 调用一个特殊的**信标区块根合约 (BEACON ROOTS ADDRESS)**。它把共识层刚刚产生的状态根（H\_{parentBeaconBlockRoot}）像传纸条一样，强行写入执行层的智能合约存储里。  
    
-   **现实意义：** 使得Lido 这样的流动性衍生品（质押池）、跨链桥或者 EigenLayer（再质押协议），可以直接在智能合约里读取这个数据，从而极其安全、无需信任地知道共识层上谁质押了多少钱、谁被罚没了。
    

3.  处理用户交易 (Process Transactions)
    

-   Recovering the transaction sender's address using the signature components Tv,Tr,TsTv​,Tr​,Ts​.
    
-   Verifying intrinsic transaction validity.
    
-   Calculating the effective gas price.
    
-   Initializing the execution environment.
    
-   **Executing the decoded transaction** within the virtual machine, including validation against the current state, gas calculations, and applying state changes upon success.
    

4.  处理验证者提款 (EIP-4895)
    

这是上海（Shapella）升级引入的功能。处理完交易后，以太坊还要给验证者发工资。

-   系统会读取信标链传过来的**提款列表 (Withdrawals)**。
    
-   **汇率转换：** 因为共识层计算金额的单位是 Gwei，而执行层（EVM）的单位是 Wei（1 Gwei = 10^9 Wei），所以在这里系统会自动做乘法转换。
    
-   **凭空打钱：** 系统直接把钱“印”到验证者的执行层钱包地址里。注意，**提款不是交易，它不需要消耗 Gas**，它是系统底层的直接状态修改。
    
-   **清理垃圾：** 把刚才操作中产生余额为 0 的空账户统统删掉，保持数据库整洁。
    

Environment initialization

**1\. 身份与位置信息：**

-   I\_{caller} / I\_{origin}：谁在调我的合约？最初发起这笔交易的钱包是谁？
    
-   I\_{chainId}：我现在是在哪条链上？（防止测试网的交易在主网被重放）。
    

**2\. 区块与时间信息：**

-   I\_{number}：现在是第几个区块？
    
-   I\_{time}：现在是几点？（Unix 时间戳，很多 DeFi 合约用它来算利息或判断锁仓到期没）。
    
-   I\_{blockHashes}：前 256 个区块的哈希是什么？
    

**3\. 经济与 Gas 信息：**

-   I\_{baseFeePerGas} / I\_{gasPrice}：现在的过路费底价和实际价格是多少？
    
-   I\_{gaslimit}：这笔交易还剩多少 Gas 额度可以给我挥霍？
    
-   I\_{coinbase}：这个区块的小费最后要打给哪个矿工/验证者？
    

**4\. 坎昆升级的新数据：**

-   I\_{prevRandao}：共识层传来的随机数（常被用来做轻度随机场景）。
    
-   I\_{excessBlobGas} / I\_{blobVersionedHashes}：当前市场的 Blob 拥堵情况，以及这笔交易挂载了哪些 Blob 数据指纹。
<!-- DAILY_CHECKIN_2026-04-14_END -->

# 2026-04-13
<!-- DAILY_CHECKIN_2026-04-13_START -->








休息一天
<!-- DAILY_CHECKIN_2026-04-13_END -->

# 2026-04-12
<!-- DAILY_CHECKIN_2026-04-12_START -->









Transaction Parameters and Bounded Natural Numbers

1.  Dynamics of Gas Price Block to Block
    

the price of base gas within the scope of a single block.png

-   **阶梯状的线性分布(Step-like linear progression)**：函数呈现出阶梯状的线性级数，这与以太坊底层只用整数有关；在中点（即 15,000 目标值）附近的阶梯最宽，这说明在区块容量在50%左右时基础费比较稳定，这也是EIP-1559“降低波动性”的完美体现。
    
-   **最大涨幅的物理极限(Maximum unward change~12.5%)**：这是机制设定的硬性涨停板。对应前面的公式里的常量 \\xi = 8，也就是 \\frac{1}{8}。如果上一个区块的 Base Fee 是 100，下一个区块最多只能涨到 112.5。
    
-   **最大跌幅的机制下限(Maximum downward change~10%)**：这比上面的最大涨幅不同是因为以太坊的区块Gas消耗量不可能绝对为0，系统设置了最低下限是5,000units。
    
-   **目标缓冲区的灵敏触发器(The 1% increase near target)**：精确命中目标值（或者略微超过一点点，比如用了 15,000 到 17,000）会导致 1% 的微小增长。这说明了该函数在目标值附近设计了“弹性”。这种设计让系统在目标值附近拥有了极好的“弹性缓冲”，既不会乱涨价，又保持了轻微的通胀抑制力。
    

2.  Long-term Effects on Gas Limit and Fee
    

the gas limit of different block number.png

-   **基础费爆炸 (Base Fee Sensitivity)**：因为每次涨 12.5% 是指数级的（复利），基础费会涨得极其夸张。在持续满载的情况下，仅仅 200 个区块（大概 40 分钟），发一笔交易的底价可能就会高达 1 个 ETH。不到 2000 个区块，价格就会触及数学上的极限。
    
-   **Gas 上限的无限生长 (Unbounded Gas Limit Growth)**：不同于基础费，Gas 上限（Gas Limit）在理论上是没有天花板的，它可以一直往上调以适应需求。
    
-   **市场动态与新平衡 (Market Dynamics and Equilibrium)**：在现实中，基础费不可能真的涨到无穷大。因为当价格涨到一定程度时，用户就付不起了，需求自然会锐减。同时，随着 Gas Limit 被节点逐渐调高，它的“目标值（一半）”也会跟着水涨船高。最终，高昂的价格和变大的容量会相互抵消，让系统达到一个新的动态平衡点。
    

3.  Different \\xi and \\rho in EIP-1559
    

\\rho 和 \\xi 配合在一起，就像是汽车的“减震弹簧”和“底盘高度”，它们共同决定了以太坊面对交易流量冲击时的柔韧度。

-   调整 \\xi（基础费最大变动分母 / 波动阻尼器）
    
    -   现状：目前 \\xi = 8（意味着最大涨跌幅是 \\frac{1}{8}，即 12.5%）。
        
    -   如果调大 \\xi（比如调成 16）：阶梯会变得更宽（broader step widths）。这意味着费用的调整会变得非常**平缓和迟钝**，最大涨幅可能只有 6%。
        
    -   如果调小 \\xi（比如调成 4）：阶梯会变窄，曲线的斜率会陡然增加。系统会变得极其**敏感和剧烈**，稍微一拥堵，价格就会以 25% 的速度疯涨。
        
-   调整 \\rho（弹性乘数 / 拥堵触发线）
    
    -   现状：目前 \\rho = 2（意味着目标 Gas 是上限的 \\frac{1}{2}，即 50% 的位置）。
        
    -   影响拐点 (Shifts the inflection point)：如果你把 \\rho 改成 4，那么目标值就变成了上限的 25%。这意味着，原本区块装到一半才算拥堵，现在只要装了四分之一，系统就会判定为拥堵并开始涨价。
        

4.  Dynamics of Blob Gas Price
    

这是EIP-4844(Proto-Danksharding)的Blob Gas定价模型。

-   **Blob的物理容量**：Blob是以太坊给 L2（如 Arbitrum, Optimism）开辟了专门的数据通道。  
    它的Target：每个区块 3 个 Blob（约 384 KB）；它的Max：每个区块 6 个 Blob（约 768 KB）。只要 L2 们上传的数据量不超过 3 个 Blob，Blob 的底价就保持在绝对的地板价：1 Wei（几乎免费）。
    
-   **延迟计费机制**：如果某个区块塞了 6 个 Blob（超载了 3 个），系统不会立刻涨价。相反，系统内部有一个记账本（Accumulated Excess Gas，累积超额 Gas）。它会把这超出来的 3 个 Blob 记在账上。只要这个“累积账本”里的数字没有达到触发阈值，价格就依然是 1 Wei。
    
-   **指数级爆发**：当 L2 的需求持续爆满，连续很多个区块都塞满 6 个 Blob，那个“超额记账本”里的数字就会越滚越大。Blob 基础费的计算公式是基于自然常数 e 的指数函数： Blob\\\_Base\\\_Fee = 1 \\times e^{\\frac{Excess\\\_Blob\\\_Gas}{Update\\\_Fraction}}，当 `Excess_Blob_Gas` 累积突破阈值后，价格会呈现恐怖的指数级暴涨（Exponential increase）。这是一种“防 DOS 攻击”的监管措施，强行把那些低价值的数据挤出市场。
<!-- DAILY_CHECKIN_2026-04-12_END -->

# 2026-04-11
<!-- DAILY_CHECKIN_2026-04-11_START -->










### 区块头验证 (Header Validation) 公式里都在检查什么？

一系列极其严苛的规则，包括：

-   消耗的 Gas 绝不能超上限，且 Gas Limit 每块调整不能超过 1/1024。
    
-   时间戳必须严格向前，高度递增。
    
-   包含上海升级的提款根，以及坎昆升级的 Blob 限制（如 Blob Gas 必须是 131072 的整数倍，总量不超过 6 个 Blob 等）。
    

### 什么是 Unbounded block limits (无界区块限制)？

交易参数（如手续费上限）在底层被死死限制在 256 位 ($\\mathbb{N}\_{256}$) 内以防止 EVM 内存溢出。但 **区块 Gas 上限** 和 **基础费** 在纯粹的数学协议上被定义为**无界自然数 ($\\mathbb{N}$)**，这意味着以太坊理论吞吐量和防 DOS 攻击的涨价惩罚都没有天花板。

### 为什么以太坊底层极其排斥小数，只用自然数 (整数) 计算？

为了**绝对的确定性**。不同 CPU 架构处理浮点数（实数）会产生精度舍入误差，哪怕 `0.0000001` 的偏差也会导致网络立刻硬分叉崩溃。整数计算能保证全球节点状态绝对一致，且便于进行安全代码的数学形式化验证。

### EIP-1559 底层数学模型模拟揭示了什么？

-   **阶梯化与底线：** 价格变动由于向下取整呈现阶梯状，涨停板为 12.5%（因为常量 $\\xi=8$），但由于区块不可能消耗 0 Gas，跌停板约为 10%。
    

-   **10万区块极限施压：** 持续满载会导致 Base Fee 在几百个区块内达到天文数字；市场会通过调高 Gas 上限和用户退出来寻找新平衡。
    
-   **修改物理定律：** 未来如果修改常量 $\\xi$ (波动阻尼器) 或 $\\rho$ (弹性乘数)，可以彻底改变基础费曲线的陡峭程度和触发涨价的拥堵阈值。
<!-- DAILY_CHECKIN_2026-04-11_END -->

# 2026-04-10
<!-- DAILY_CHECKIN_2026-04-10_START -->











## EL Spec(规范)

最新的规范：[EELS python spec](https://ethereum.github.io/execution-specs/).

状态转移函数：the state transition function (STF)

去中心化网络传输机制：**Gossip 协议（八卦/流言协议）** 和 \*\*Mempool（内存池）。\*\*可以让所有节点的池子里“共享”所有的交易，从而使Validator打包gas较高的tx

State Collapse Function：

以太坊是如何从零开始“推导”出当前状态的：

1.  底层存储坍溃：先把某个智能合约里所有的细碎变量（Account Storage），通过哈希计算，压缩成一个单独的指纹，叫 `StorageRoot`。
    
2.  Account Collapse：把这个人的 `Nonce`（交易次数）、`Balance`（余额）、刚才算出的 `StorageRoot`，以及智能合约代码的哈希 `CodeHash`，这四样东西绑在一起，再算一次哈希。这就是账户状态 (Account State)。
    
3.  World State Collapse：把全网几千万个这样的账户，放进一棵巨大的树（默克尔树）里。
    
4.  最终Collapse成根节点：把这棵树一层一层往上算哈希，最终在树顶端算出一个唯一的、32 字节的哈希值。这就叫 World State ID（或者 State Root，状态树根）。
    

由于Hash的单向性，无法通过Root去计算所有账户的状态，但是可以通过Root去验证特定的交易，这就是Merkle Proof的由来。  
全节点是通过key-value数据库进行数据存储的，类似于查字典一样一项一项向下进行查询。这又让我复习里一下数据库的类型

之前为一直不知道Merkle树为什么是16叉的，原来这是和数据的查询有关，就比如可以使用Keccak-256算法对需要查询的地址或者特定字符串进行哈希计算，就得到了一个16位的地址，EVM可以根据这个地址进行逐项查询。当然，如果路径可以进行压缩，就会产生一种“特殊节点”

## Block Header Validation

以下是关于区块头验证的必要条件的解释：

在数学上，它定义了一个函数 $V(H)$（Validity of Header）。图中那一串长长的 $\\land$ 符号代表\*\*“逻辑与（AND）”\*\*，意思是：一个新区块想要被以太坊网络合法接收，它必须同时、绝对地满足以下所有的条件（57a 到 57t）。只要有一条不符合，这个区块就会被直接抛弃。

为了让你更容易理解，我将这些晦涩的数学公式按功能进行了分类拆解。这里的 $H$ 代表当前处理的新区块，而 $\\mathcal{P}(H)$ 代表它的父亲（上一个区块）。

1.  1\. 传统 Gas 的物理学限制 (57a - 57d)
    

这一组公式在管辖以太坊计算资源的消耗界限：

-   (57a) 绝不超载：$H\_{gasUsed} \\le H\_{gasLimit}$。区块里所有交易消耗的 Gas 总和，绝对不能超过该区块设定的 Gas 上限。
    
-   (57b & 57c) 渐进式调整上限：这两个公式定义了区块 Gas 上限的“弹性”。当前区块的 Gas Limit 不能由验证者随便填，它必须在父区块的 Gas Limit 基础上，变动幅度不能超过 $\\frac{1}{1024}$。这保证了网络容量只会平滑放大或缩小，不会因为某人的恶意提议而瞬间翻倍导致网络崩溃。
    
-   (57d) 兜底底线：Gas Limit 无论怎么降，都不能低于 5000（这是早期设定的一个极低阈值，确保至少能跑点最基本的操作）。
    

1.  2\. 时间之箭与族谱证明 (57e - 57i)
    

这一组确保区块链是一条时间向前、连贯不断的“链”：

-   (57e) 时间必须向前：$H\_{timeStamp} > P(H)\_{H\_{timeStamp}}$。当前区块的时间戳必须严格大于父区块，不能时光倒流。
    
-   (57f) 高度递增：当前区块的祖先数量（即区块高度 Number）必须精确等于父区块高度加 1。
    
-   (57g) 额外数据防滥用：区块头里的 `ExtraData`（供验证者涂鸦或写口号的地方）长度严格限制在 32 字节以内，防止有人往里面塞垃圾文件导致状态爆炸。
    
-   (57i) 认祖归宗：当前区块头记录的 `parentHash`，必须是对父区块头进行 RLP 编码后再做 Keccak 哈希运算得到的结果。这就是区块链“链”在一起的钢铁锁扣。
    

1.  3\. EIP-1559 经济模型 (57h)
    

-   (57h) 算法定价，不可篡改：$H\_{baseFeePerGas} = F(H)$。当前区块的基础费（Base Fee）必须严格等于算法 $F(H)$ 计算出的结果。验证者不能私自抬高基础费来多烧用户的钱，算法说了算。
    

1.  4\. 合并 (The Merge) 后的“历史遗留物”清理 (57j - 57l)
    

这三个公式极其有趣。以太坊从 PoW（挖矿）转为 PoS（质押）后，某些字段没用了，但为了保持区块头数据结构的向后兼容性，只能把它们“封印”为固定的零值：

-   (57j) 告别叔块：`ommersHash` 必须等于一个空列表的哈希。PoS 机制下再也没有“叔块”这个概念了。
    
-   (57k) 告别算力难度：`difficulty` 必须等于 0。不需要显卡算数学题了，难度这个概念作废。
    
-   (57l) 告别挖矿随机数：`nonce` 必须等于全零（`0x0000000000000000`）。
    

1.  5\. 上海升级与坎昆升级 (Blobs) 时代 (57n - 57t)
    

这是图片中最下方、也是目前以太坊最新的验证规则：

-   (57n) 提款凭证：`withdrawalHash` 不能为 nil。上海升级后，区块头必须包含验证者提取质押以太坊的收据树根。
    
-   (57o, 57p, 57q) Blob 的物理约束：这是坎昆升级的核心。区块必须带有 `blobGasUsed` 字段；而且一个区块里消耗的 Blob Gas 绝对不能超过 786,432（这正好对应 6 个 Blob 数据包的最大硬上限）；并且使用的 Gas 必须是单块 Blob 基准参数（$2^{17} = 131,072$）的整数倍。
    
-   (57r, 57s, 57t) Blob 版的 EIP-1559：公式 (57s) 是计算“超额 Blob Gas”（Excess Blob Gas）的逻辑。它规定了一个目标值（Target Blob Gas）为 393,216（相当于 3 个 Blob）。如果父区块携带的 Blob 数量小于 3 个，超额为 0；如果大于 3 个，多出来的部分就会累加进 Excess 变量中。这个变量直接决定了下一个区块 Blob 存储的动态费用。
    

### the Ethereum economic model(EIP-1559)

1\. Targeted Gas Limit for Reduced Volatility (设定目标 Gas 限制以降低波动性)

-   文本原意：通过将目标 Gas 设定为最大 Gas 限制的一半，以太坊旨在减少满载区块可能带来的剧烈波动，从而确保一个更可预测的交易处理环境。
    
-   深度解析：这就是我们之前提到的 1500万（目标）/ 3000万（上限） 机制。在以前，只要交易稍微多一点，区块满了，Gas 价格就会像火箭一样毫无规律地暴涨。现在，因为有了这 50% 的“缓冲带（弹性空间）”，大部分时间区块都不会被打满，这就像是给高速公路增加了一条应急车道，极大地吸收了流量洪峰，让 Gas 费用的波动变得平滑。
    

1.  2\. Prevention of Unnecessary Delays (防止不必要的延迟)
    

-   文本原意：该模型试图通过优化交易处理时间来消除用户遭受的不必要延迟，从而提升网络上的整体用户体验。
    
-   深度解析：在旧的固定区块（1500万上限）时代，如果你遇到了热门 NFT 发售，你的交易可能因为排不上队而在内存池里卡上好几个小时。而在 EIP-1559 下，由于区块可以瞬间膨胀到 3000 万，网络可以“加班”把积压的交易迅速处理掉。虽然随后的基础费会上涨，但至少只要你愿意付当前的公开标价，你的交易就能立刻上链，不用干等。
    

1.  3\. Stabilizing Block Reward Issuance (稳定区块奖励发行与系统安全)
    

-   文本原意：区块奖励的发行有助于增强系统的稳定性，为参与者提供一个更可预测的经济环境。
    
-   深度解析：这条其实暗含了一个非常深的安全博弈论。在 EIP-1559 之前，矿工的收入（区块奖励 + 交易手续费）波动极其巨大。有时手续费甚至比区块奖励高出十几倍。这种巨大的利益诱惑会导致矿工互相攻击（比如试图回滚别人的区块来抢夺那些高昂的手续费，这就是所谓的“重组攻击”）。通过销毁基础费，验证者现在的收入主要变成了\*\*“固定的出块奖励 + 微薄的小费”\*\*。收入稳定了，验证者作恶的动机就小了，整个共识网络的安全基石就更稳固了。
    

1.  4\. Predictable Base Fee Adjustments (可预测的基础费调整)
    

-   文本原意：EIP-1559 引入了一种可预测的基础费变动机制，这对钱包应用特别有利。这种可预测性有助于提前准确估算交易成本，简化了交易创建过程。
    
-   深度解析：这彻底终结了“盲拍时代”。因为基础费是由协议通过纯数学公式算出来的（每个区块最多涨降 12.5%），所以像 MetaMask 这样的钱包，只需看看上一个区块的数据，就能绝对精准地算出下一个区块的最低标价。用户再也不用为了防卡死而瞎填一个离谱的高价了。
    

1.  5\. Base Fee Burn and Priority Fee (基础费销毁与优先费机制)
    

-   文本原意：在这个模型下，矿工（验证者）有权保留优先费（小费）作为激励，而基础费则被销毁，有效地将其从流通中移除。这种方法是以太坊对抗通货膨胀的对策，通过随着时间推移减少总供应量，促进一个更健康的经济环境。
    
-   深度解析：这就是 “超声波货币 (Ultra Sound Money)” 的核心引擎。
    
    -   优先费（给验证者）：是为了在网络完全拥堵时，提供一个排序机制（谁给的小费多先打包谁）。
        
    -   基础费（被销毁）：强行切断了验证者自己发垃圾交易抬高底价的可能。燃烧掉的 ETH 让所有持有以太坊的人隐性受益（相当于一种全网的股份回购），在网络高负载时实现真正的通缩。
<!-- DAILY_CHECKIN_2026-04-10_END -->

# 2026-04-09
<!-- DAILY_CHECKIN_2026-04-09_START -->













今日欠一天吧
<!-- DAILY_CHECKIN_2026-04-09_END -->

# 2026-04-08
<!-- DAILY_CHECKIN_2026-04-08_START -->














### 1\. 核心数学模型：状态转换函数 (State Transition Function)

整个执行层（EL）存在的唯一目的，就是执行下面这个极其优雅的公式：

$$\\sigma\_{t+1} \\equiv \\Pi(\\sigma\_t, B)$$

-   $\\sigma\_t$：**当前状态**（比如，此时此刻所有人的账户余额和智能合约数据）。
    
-   $B$：**当前区块**（包含了一大堆待处理的交易）。
    
-   $\\Pi$：**状态转换函数**（也就是以太坊虚拟机的底层逻辑）。
    
-   $\\sigma\_{t+1}$：**新状态**（执行完这些交易后，更新后的账户余额和数据）。
    

文档特别指出，规范中的“状态”并不是存在某个特定硬盘里的静态文件，而是通过**状态坍缩函数 (World State Collapse Function)** 动态计算出来的默克尔树根哈希（State Root）。

### 2\. 区块头验证与经济模型 (Header Validation & Economic Model)

在执行任何交易前，节点必须先验证区块头是否合法。文档列举了诸如 Gas 限制、时间戳、基础费等一系列数学不等式（例如 $H\_{gasUsed} \\le H\_{gasLimit}$）。

其中最关键的是对 **EIP-1559 基础费 (Base Fee)** 和 **EIP-4844 Blob 费用** 的详细剖析：

-   **EIP-1559 动态调峰**：以太坊设定的“目标 Gas”（即理想状态下的区块拥挤度）是区块 Gas 上限的一半（比如上限是 3000万，目标就是 1500万）。如果上一个区块消耗的 Gas 超过目标值，基础费就会上涨（最多涨 12.5%）；如果低于目标值，基础费就会下降。
    
-   **为何使用自然数 ($N$)**：文档解释了以太坊底层全都是用离散的自然数（整型）进行计算的，完全避免了浮点数（小数），这是为了保证无论在什么硬件上计算，结果都能达到绝对的一致性（防止分叉）。
    

为了让您直观理解 EIP-1559 是如何根据区块拥挤程度自动调节 Gas 费用的，我为您生成了一个交互式的基础费计算器：

Show me the visualization

### 3\. 交易的生命周期 (Transaction Execution)

文档详细拆解了一笔交易在执行层是如何被抽丝剥茧处理的：

1.  **内在有效性检查**：验证签名是否正确、账户里有没有足够的钱付 Gas 费、Nonce（交易序号）是否匹配等。
    
2.  **预扣费 (Upfront Cost)**：在执行前，先从发送者账户里扣除“最大可能的 Gas 费”。
    
3.  **消息标准化**：不管是部署新合约（`T_{to} = 0`）还是调用已有合约/转账（`T_{to} = Address`），都会被转换成统一的 `Message` 对象。
    
4.  **子状态初始化**：创建用于记录日志 (Logs)、自毁合约 (Self-destruct)、以及热点账户访问列表 (Access Lists) 的缓存空间。
    

### 4\. EVM 执行核心细节 (Machine State & $\\Xi$)

这是黄皮书中最硬核的部分。以太坊虚拟机 (EVM) 是一个**基于栈 (Stack-based)** 的计算机。文档定义了机器状态 $\\mu$：

-   **程序计数器 ($PC$)**：指示当前执行到哪一行代码。
    
-   **内存 (Memory)**：临时存储，按需扩展，扩展时要额外扣除 Gas。
    
-   **栈 (Stack)**：深度最大 1024，进行加减乘除和逻辑运算的地方。
    

执行循环 $O$ 会不断读取操作码（Opcode），推入/弹出栈数据，扣除对应的 Gas，直到遇到 `STOP`、`RETURN`（正常停机）或者发生 Gas 耗尽等错误（异常停机，状态直接回滚）。

### 5\. 区块整体有效性 (Block Holistic Validity)

单笔交易执行没问题还不算完。当区块里的**所有**交易都按顺序执行完毕后，执行层必须进行最后的“对账”：

-   所有交易的收据（Receipts）重新打包计算出来的**收据树根 (Receipts Root)**，必须和矿工/验证者提供的区块头里的哈希值一模一样。
    
-   所有账户余额更新后计算出的**状态树根 (State Root)**，也必须和区块头里的一模一样。
    
-   只要有一个比特对不上，整个区块就会被判定为非法并被丢弃。
    

* * *
<!-- DAILY_CHECKIN_2026-04-08_END -->

# 2026-04-07
<!-- DAILY_CHECKIN_2026-04-07_START -->















* * *

## 1\. 史前史 (Prehistory)

_记录了以太坊从 2013 年底的一个概念到 2015 年 7 月主网上线之前的关键历程。_

-   **起源背景**: 比特币脚本缺乏图灵完备性，在比特币上构建复杂应用（如染色币）十分受限。这催生了构建一个**内置通用编程语言的底层区块链**的需求。
    
-   **三大基石文档**:
    
    -   **白皮书 (Whitepaper)**: Vitalik Buterin 提出“智能合约”与“状态”概念，奠定图灵完备的基础。
        
    -   **黄皮书 (Yellow Paper)**: Gavin Wood 编写，提供了严格的数学形式化规范，定义了以太坊虚拟机 (EVM)、Gas 机制和默克尔帕特里夏树 (MPT)。
        
    -   **紫皮书 (Mauve Paper)**: 早期关于 PoS 和分片的远景规划。
        
-   **PoC 阶段 (Proof of Concepts)**: 经历了从 PoC 1 到 PoC 9 的迭代。通过多客户端并行开发（C++, Go, Python）确保协议的稳健性。
    
-   **Olympic 测试网**: 主网上线前的最后一次极限压力测试，鼓励黑客攻击以寻找漏洞。
    
-   **Frontier 创世**: 2015 年 7 月，以太坊主网正式诞生，史前史结束。
    

* * *

## 2\. 架构 (Architecture)

_以太坊合并（The Merge）之后解耦的节点架构设计。_

-   **执行层 (Execution Layer)**: 系统的“大脑”，负责状态转换和计算。
    
    -   **JSON-RPC API**: 供用户、钱包和 Web3 DApp 调用的接口。
        
    -   **Txs (Mempool)**: 交易内存池，暂存待打包的交易。
        
    -   **EVM**: 以太坊虚拟机，执行智能合约代码。
        
    -   **State Data**: 保存全网所有的账户余额和合约状态。
        
-   **共识层 (Consensus Layer / Beacon Chain)**: 系统的“心脏”，基于 PoS（权益证明）机制确保全网对状态达成一致。
    
    -   **Validators (验证者)**: 质押 ETH 负责提议和验证区块的节点。
        
    -   **PoS 共识引擎**: 包含 **LMD-GHOST**（决定主链的分叉选择规则）和 **Casper-FFG**（区块最终敲定机制）。
        
    -   **Blobs**: 坎昆升级引入，为 Layer 2 提供廉价的临时数据存储。
        
-   **Engine API**: 执行层与共识层之间的内部通信桥梁。
    

* * *

## 3\. 设计原理 (Design rationale)

_解释以太坊底层架构为什么这样设计的哲学指导思想。_

-   **核心原则**:
    
    -   **简单性 (Simplicity)**: 宁可牺牲部分效率也要保持协议简单，便于多客户端的独立实现。
        
    -   **通用性 (Universality)**: 不内置特定功能（如多签），只提供图灵完备的 EVM 供开发者自由发挥。
        
    -   **模块化 (Modularity)** & **敏捷性 (Agility)**: 方便未来升级替换（如 PoW 换成 PoS），拥抱技术迭代。
        
-   **关键技术抉择**:
    
    -   **采用账户模型 (Account Model)**: 放弃比特币的 UTXO，因为账户模型更适合存储和更新智能合约的“状态”。
        
    -   **256位 EVM**: 为高效处理 256 位密码学哈希运算（如 Keccak-256）而专门设计。
        
    -   **Gas 机制**: 为了解决图灵完备带来的“停机问题”（死循环），通过消耗预付费 Gas 来限制计算资源被恶意耗尽。
        
    -   **MPT 状态树**: 允许轻节点通过密码学证明快速验证数据，而无需下载全节点数据。
        

* * *

## 4\. 演进 (Evolution)

_以太坊主网自 2015 年上线以来的重大升级历程（硬分叉编年史）。_

-   **早期定型 (2015 - 2016)**:
    
    -   **Frontier**: 创世版本。
        
    -   **The DAO 分叉**: 应对黑客攻击进行的状态回滚，导致了以太坊与以太坊经典 (ETC) 的分家。
        
-   **能力扩展 (2017 - 2019)**:
    
    -   **拜占庭 (Byzantium) / 伊斯坦布尔 (Istanbul)**: 引入 zk-SNARKs 支持，调整出块奖励，为 Layer 2 扩容做密码学层面的准备。
        
-   **经济变革与共识转型 (2020 - 2023)**:
    
    -   **信标链创世 (Beacon Chain)**: PoS 共识层启动。
        
    -   **伦敦升级 (London, EIP-1559)**: 引入基础费销毁机制 (Burn)，改变 Gas 费市场，赋予 ETH 通缩属性。
        
    -   **合并 (The Merge / Paris)**: 执行层与共识层完美对接，彻底告别 PoW 矿机时代。
        
    -   **上海升级 (Shapella)**: 开放 PoS 节点质押提款，闭环经济模型。
        
-   **Rollup 扩容时代 (2024 及以后)**:
    
    -   **坎昆升级 (Dencun, EIP-4844)**: 引入 Blobs，大幅降低 Layer 2 交易手续费，正式确立以 Rollup 为中心的扩容路线。
        

* * *
<!-- DAILY_CHECKIN_2026-04-07_END -->

# 2026-04-06
<!-- DAILY_CHECKIN_2026-04-06_START -->
















### 📖 第一部分：核心概念与原理字典

1\. 普通 EOA (Externally Owned Account)

-   **概念**：外部拥有账户，也就是你平时用的传统 MetaMask 钱包。
    
-   **原理**：它是由一对“公钥-私钥”通过密码学运算直接生成的地址。**它的内部没有任何代码（Code 为空）**。
    
-   **特征与局限**：
    
    -   绝对的私钥控制：谁掌握私钥，谁就拥有资产。
        
    -   **必须自己付 Gas**：发起任何交易，该地址里必须有原生的 ETH 作为起步费。
        
    -   **逻辑锁死**：只能做最简单的“A 转账给 B”，无法实现复杂的逻辑（如：批量转账、多重签名、每日限额、社交恢复）。  
        

2\. UserOperation (简称 UserOp)

-   **概念**：EIP-4337 账户抽象体系中引入的一种**“伪交易”**数据结构。
    
-   **原理**：用户不再直接构造以太坊底层原生交易，而是把自己的“意图”（比如我想去 Uniswap 交易、我的最高 Gas 出价是多少、我的签名）打包成一个 UserOp 结构体。
    
-   **特征**：它不会被广播到以太坊的公共内存池，而是进入一个专门为账户抽象准备的**替代内存池 (Alt-Mempool)**。  
    

3\. Bundler (捆绑者)

-   **概念**：EIP-4337 网络中的“快递集散中心”与“垫资人”。它是一个跑在链下的节点服务器，在链上拥有一个装满 ETH 的普通 EOA 账户。
    
-   **原理**：
    
    1.  从 Alt-Mempool 抓取用户的 UserOp。
        
    2.  在本地极其严格地**模拟执行**，确保这笔操作有钱报销。
        
    3.  把几十个互不相干的 UserOp “捆绑”成一个巨大的数组。
        
    4.  用自己的 EOA 账户，垫付一笔巨大的 ETH Gas 费，将这个超级交易发送到主网。
        
-   **盈利模式**：赚取用户支付的手续费差价和批量打包带来的 Gas 规模经济红利。  
    

4\. 普通 Proxy / Relayer (传统代付代理)

-   **概念**：Web2 思维的补丁，早期的“免 Gas”方案提供商。
    
-   **原理**：用户用私钥对一段信息签名，发送给一个中心化的服务器。服务器用自己的以太坊账户垫付 Gas，把用户的操作发上链。
    
-   **致命缺陷**：
    
    -   **诸侯割据**：每个 dApp 都得自己写一套验证逻辑，无法通用。
        
    -   **中心化单点故障**：服务器一停，免 Gas 交易直接瘫痪。
        
    -   **易被白嫖 (Griefing)**：由于缺乏像 Bundler 那样的标准化底层模拟机制，代理很容易垫付了 Gas 费却因为链上执行失败而血本无归。
        

5\. EIP-4337 (以太坊账户抽象核心提案)

-   **概念**：以太坊试图将 EOA 转换为“智能合约钱包”的宏大基础设施。
    
-   **原理**：它**没有修改以太坊的底层共识**，而是纯粹在应用层（智能合约层面）搭建了一套系统。核心引入了 **EntryPoint（入口点全局合约）**。
    
-   **运作机制**：Bundler 把捆绑好的包裹交给 EntryPoint，由 EntryPoint 负责去验证每个用户的签名、执行智能合约钱包的逻辑、并强制从用户余额中扣款补偿给 Bundler。
    
-   **最大痛点**：**必须换钱包**。老用户必须把资产从原来的 EOA 迁移到全新的智能合约地址里，迁移成本极高。  
    

6\. EIP-7702 (下一代账户抽象桥梁)

-   **概念**：Vitalik 亲自推进的神级提案，旨在解决 4337 的历史遗留（换钱包）问题。
    
-   **原理（机甲附体）**：引入了一种新型的底层交易格式。用户依然使用旧的 EOA 钱包，但可以在交易中附加一个签名，**授权 EVM 在执行这笔交易的短暂瞬间，将一段智能合约代码“临时注入”到自己的 EOA 地址上**。
    
-   **特征**：
    
    -   交易执行时，EOA 临时变身为强大的智能合约（可享受 4337 的所有功能，如代付 Gas）。
        
    -   交易结束后，代码瞬间蒸发，EOA 恢复原样，私钥依然拥有最高控制权。
        

* * *

### 🧠 第二部分：演进脉络与深度总结

这场关于以太坊账户体系的进化史，本质上是为了解决一个核心矛盾：**“极客追求的底层绝对安全（EOA）” 与 “大众渴望的 Web2 级丝滑体验（免 Gas、自动化）” 之间的冲突。**

阶段一：缝缝补补的“草莽时代” (普通 Proxy)

为了让小白不用买 ETH 就能玩转 Web3，开发者弄出了中心化的代付代理（Relayer）。这就像是雇了一个私人跑腿，体验虽然上去了，但系统变得脆弱、昂贵，且代理商常常因为被黑客“白嫖” Gas 费而破产。

阶段二：建立大一统的“高速公路” (EIP-4337 与 Bundler)

为了消灭割据，以太坊社区花了数年时间打造了 EIP-4337。

它通过 **Bundler (去中心化的垫资打包者)** 和 **EntryPoint (统一的链上清算中心)**，彻底规范了免 Gas 交易和复杂账户逻辑的标准。它不再是跑腿，而是“集装箱海运”。

**但它犯了一个战略错误：** 步子迈得太大，要求所有人放弃陪伴多年的老 EOA 钱包，遭到老玩家的强烈抵触。

阶段三：教科书级别的“架构妥协” (EIP-7702 登场)

EIP-7702 是对 4337 的终极补完。

它极其巧妙地在 EVM 底层打了一个 Patch，让老的 EOA 钱包可以通过“临时机甲附体”的方式，直接对接 EIP-4337 已经建好的那套 Bundler/EntryPoint 高速公路。

**结局完美**：用户不用换钱包，项目方不用改基础设施，Web3 终于可以同时拥有“绝对的私钥掌控权”和“无缝的代付与批量体验”。

* * *
<!-- DAILY_CHECKIN_2026-04-06_END -->
<!-- Content_END -->
