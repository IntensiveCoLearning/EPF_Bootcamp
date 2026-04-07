---
timezone: UTC+8
---

# 0xstride

**GitHub ID:** fengjy73

**Telegram:** 

## Self-introduction

EPF 实习计划

## Notes

<!-- Content_START -->
# 2026-04-07
<!-- DAILY_CHECKIN_2026-04-07_START -->
### 今日阅读

主要阅读了「以太坊执行层全景解析」的前三章（总述、合并后的角色变化、区块验证），以及「Intro to execution 」中对 State Transition Function 和 Header Validation 的资源梳理，同时浏览了「 Execution Layer」的 Pre-reading 列表以建立预备知识框架。

### 串联逻辑

从"合并之后执行层到底还剩什么"这个问题切入，先理解架构层面的分工——共识层做决策、执行层做计算——再顺着 `process_execution_payload → notify_new_payload` 的调用链，自然过渡到状态转换函数（STF）的具体结构。STF 内部又分两步：先验区块头、再逐笔执行交易，形成"从宏观到微观"的递进。

### 重点研究

-   **模块化分层**：The Merge 的本质不只是共识机制切换，而是将一个单体客户端重构为通过 Engine API 解耦的两层架构。这与分布式系统中经典的关注点分离（Separation of Concerns）原则一致——共识层退化为一个排序预言机（ordering oracle），执行层则收缩为确定性状态转换引擎。这种拆分为后续的 proposer-builder separation（PBS）、执行层多样性（client diversity）以及潜在的 enshrined rollup 架构奠定了前提。
    
-   **STF 的形式化语义与安全性**：状态转换函数 `σ' = Υ(σ, T)` 在黄皮书中被定义为确定性映射。区块级别的 all-or-nothing 语义——一笔 intrinsically invalid transaction 足以否决整个区块——本质上是为了维护状态根的全局一致性。这与应用层的 revert 形成了清晰的语义分层：revert 是 EVM 内部的异常处理机制，交易本身仍然合法并消耗 Gas；而 invalid transaction 意味着区块构建者违反了协议规则，是共识层面的错误。
    
-   **区块头约束**：Gas Limit 的 1/1024 渐变机制是一种链上投票（on-chain governance by block producers）的隐式形式——出块者通过缓慢调整 Gas Limit 来对网络吞吐量达成社会共识，这与 EIP-1559 引入的算法化 Base Fee 形成了对照：前者是离散的、博弈驱动的，后者是连续的、机制设计驱动的。而 Uncle Hash 为空、Difficulty 为 0 等 PoS 后的"化石字段"，则引出更深层的工程问题——以太坊的协议演化始终受限于向后兼容，这也是 EIP-4444（历史数据过期）和 Verkle Tree 迁移面临的核心阻力。
    

### 收获

重新审视 The Merge 的架构意义：不只是"PoW → PoS"，更是一次对系统边界的重划。Engine API 的 `newPayload` / `forkchoiceUpdated` 接口本质上定义了一个狭窄的合约——共识层只需要问"这个 payload 合不合法"和"请帮我构建一个 payload"。这种最小化接口的设计思路与形式化验证的可行性直接相关：接口越窄，跨层的安全性推理就越容易。

### 💡 Insight

**执行层的确定性是整个以太坊安全模型的地基。** STF 作为纯函数，其确定性保证了任意诚实节点对同一区块会得出完全一致的状态根——这正是 Merkle 证明、轻客户端验证、乃至 L2 fraud proof 能够成立的根本前提。一旦执行层存在非确定性（如浮点运算、未定义行为），整个信任链条就会从根部断裂。EVM 的种种"笨拙"设计选择——256-bit 整数算术、无浮点、有限指令集——正是为这个确定性目标服务的。
<!-- DAILY_CHECKIN_2026-04-07_END -->
<!-- Content_END -->
