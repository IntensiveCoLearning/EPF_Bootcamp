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
# 2026-04-08
<!-- DAILY_CHECKIN_2026-04-08_START -->
### 今日阅读

继续推进「以太坊执行层全景解析」的第四、五章（go-ethereum 实现细节 + EVM 深入），并对照「Intro to execution - Resources」中 EVM high-level 和 Block building 部分的大纲来理清脉络。

### 串联逻辑

昨天停在"交易被送进 EVM 执行"这个入口，今天先从 Geth 的代码路径（Engine API → Catalyst → insertBlock → verifyHeaders → process → applyTransaction → evm.Call / evm.Create）建立端到端认识，再深入 EVM 本身的执行模型——栈、内存、Gas、指令分类。最后跳到区块构建，和昨天的验证流程形成对称。

### 重点研究

-   **Geth 实现中的 STF 分解策略**：Geth 的执行管线（`newPayload → insertBlockWithoutSetHead → verifyHeaders → Process → applyTransaction → transitionDB → evm.Call/Create`）并非对黄皮书 STF 的线性翻译，而是包含了大量的工程性分解：比如 `transitionDB` 中将 intrinsic gas 校验、nonce 检查、EIP-1559 费用逻辑等前置于 EVM 调用之前，这种分层校验的思路与设计模式中的 fail-fast 原则一致。值得注意的是，`finalize` 阶段处理的 Withdrawals 是一个跨层语义操作——它绕过了 EVM，直接修改状态，这实际上破坏了"所有状态变更都经过 EVM"的纯粹性。类似的跨层操作在 EIP-4788（beacon root ring buffer）和 EIP-4844（excess blob gas）中越来越多，执行层的"纯函数"假设正在被逾动式地侵蚀。
    
-   **EVM 作为智能合约安全性的约束框架**：栈式架构看似是一个简单的 VM 设计选择，但它对安全性的含义很深：栈深度限制（1024）和 call depth 限制直接防御了早期的 reentrancy 和 stack overflow 攻击。CALL 与 DELEGATECALL 的语义差异——前者切换存储上下文，后者保持调用者上下文——是代理合约模式和可升级合约的基础，但也是大量安全漏洞的源头。EIP-4788 合约的字节码中用 JUMPI 判断调用者是否为 system address，这是一个将权限控制下沉到字节码层的有趣案例。
    
-   **区块构建与 MEV 的博弈结构**：表面上的贪心算法（按 effective tip 排序、逐笔执行直到 Gas 用完）已经不是实际生产环境的全貌。在 MEV-Boost / PBS 架构下，构建者不再是 proposer 自己，而是专业的 block builder，它们通过 auction 竞争出块权，这个过程引入了组合拍卖理论、信息不对称博弈等学术问题。而验证与构建的不对称性（无效交易在验证中是 fatal，在构建中只是 skip）还暗示了一个深层结构：构建是在不完全信息下的决策（mempool 信息滞后），验证是在完全信息下的审计——这种信息不对称性正是 MEV 存在的结构性原因。
    
-   **Gas 定价的机制设计缺陷**：Gas 的原始设计是将计算成本映射为线性价格，但现实中的瓶颈往往是非线性的（如 state access 的 I/O 开销、内存扩展的二次方成本）。早期基于特定硬件的 benchmark 已经与当今多样化的硬件环境严重脱节，而很多新 opcode 直接继承同类价格的做法更是缺乏严谨性。这类问题已经催生了学术上对 multi-dimensional fee market 的研究（如 EIP-4844 的 blob gas 就是第一步尝试），以及 Broken Metre 等论文对 Gas mispricing 攻击向量的系统性分析。
    

### 收获

今天最重要的认知转变是：执行层的"纯函数"假设正在被协议演化本身侵蚀。Withdrawals、EIP-4788、EIP-4844 等机制都在 EVM 执行之外引入状态变更，使得 STF 不再是单纯的"输入交易 → 输出状态"映射。这对形式化验证、跨客户端一致性测试（如 Hive test suite）和未来可能的 zkEVM 等价性证明都提出了挑战。

### Insight

**区块构建的本质是在不完全信息下的组合优化问题。** 表面的贪心算法（按 tip 排序）只是基线，真实的区块构建涉及交易间的状态依赖（一笔交易的执行可能改变下一笔交易的合法性）、bundle 的原子性约束（MEV searcher 提交的交易包必须整体执行或整体丢弃）、以及 PBS 下 builder 与 proposer 的信息不对称博弈。这将一个看似简单的排序问题转化为组合拍卖与机制设计的前沿课题。
<!-- DAILY_CHECKIN_2026-04-08_END -->

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
