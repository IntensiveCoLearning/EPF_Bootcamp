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
# 2026-04-10
<!-- DAILY_CHECKIN_2026-04-10_START -->
### 今日阅读

主要阅读了「09 Consensus Layer」中的 Overview 概览与 Client architecture 客户端架构两篇，同时对照「03 CL Deep dive」中 Gasper 共识机制的讲座大纲与互动问题，梳理从拜占庭容错到 Gasper 组合协议的理论脉络。

### 串联逻辑

从执行层转入共识层，首先要回答一个问题：共识层的"共识"到底在解决什么？从 BFT 的经典定义出发——在不可靠基础设施上构建可靠分布式系统——然后理解 PoS 并非共识协议本身，而是 Sybil 抵抗机制，再过渡到以太坊实际的共识协议 Gasper。Gasper 可粗略表示为 $\\text{Gasper} = \\text{LMD GHOST} \\oplus \\text{Casper FFG}$ ，其中 $\\oplus$ 并非数学意义上的"直和"，而是两个子协议在同一验证者集合上的工程耦合。架构层面则关注合并后的双层客户端结构（Beacon Node + Validator Client）以及 CL 与 EL 之间通过 Engine API 的交互语义。

### 重点研究

-   **Gasper 的组合语义与 CAP 定理的工程权衡**：Gasper 将 LMD GHOST 的 liveness 保证与 Casper FFG 的 safety 保证"螺栓式"（bolt together）拼接。然而这种组合并非形式化证明下的结果，而是一种工程上的折中。根据 CAP 定理，在网络分区（Partition）条件下，系统无法同时满足一致性（Consistency）与可用性（Availability）；以太坊在此处显式选择了 liveness 优先——分区期间两侧各自出块但无法 finalize，极端情况下可能分裂为两条不可调和的链。这一设计选择直接影响跨链桥的安全模型：依赖 finality 的桥（如基于 Casper FFG checkpoint 的轻客户端桥）在分区期间会停摆，而依赖 confirmation depth 的桥则面临回滚风险。从学术角度看，Gasper 论文（Neu et al., 2021）后续被指出存在 liveness 攻击向量，催生了 view-merge 等修补方案——这说明"两个各自安全的子协议组合后不一定仍然安全"，是协议组合理论（protocol composition）中的经典难题。
    
-   **Fork choice 作为客户端本地逻辑的深层含义**：fork choice rule 的变更不需要硬分叉，原因在于它属于客户端侧逻辑，不改变状态转换函数 $f(S, B) = S'$ 。这意味着 fork choice 的安全性分析不能假设所有节点运行相同版本——异构 fork choice 视图（heterogeneous fork choice views）下的网络行为是一个开放的研究问题。以 Proposer Boost 为例：它给及时到达的区块额外赋予 $w\_{\\text{boost}}$ 权重以防 balancing attack，但也引入了新的时序敏感性——若 proposer 的区块延迟到达，boost 可能被错误地授予或未授予，对地理位置偏远的验证者产生系统性不利。
    
-   **Reorg 的频率分布与 MEV 的激励扭曲**：短距离 reorg（1–2 个区块）在正常网络延迟下属于预期行为，但 MEV 的存在使得 proposer 有动机主动发起 ex-post reorg——在观察到下一个区块的内容后，回滚并替换为包含更多 MEV 的区块。这不再是网络层的偶发现象，而是经济激励驱动的策略性行为。学术上，此类问题被建模为"时间强盗攻击"（time-bandit attack），其可行性取决于 reorg 的深度 $d$ 与 finality 的距离：当 $d$ 小于 finalization lag 时，攻击者可以在不触发 slashing 的条件下重写历史。EIP-3675 后的单 slot finality（SSF）研究正是试图从根本上消除这一攻击面——若每个 slot 都能 finalize，则 reorg 在协议语义上不再可能。
    
-   **验证者集合规模与委员会安全性的概率边界**：当前以太坊约有 $N \\approx 10^6$ 个活跃验证者，每个 epoch（32 slots）将验证者均匀分配到各 slot。每个 slot 的委员会规模至少为 $n = 128$ 。这个数字来自严格的概率分析：设攻击者控制全网 $\\beta < 1/3$ 比例的质押量，则在大小为 $n$ 的委员会中攻击者占据 $\\geq 2n/3$ 席位的概率可由超几何分布的尾部界给出，当 $n = 128$ 且 $\\beta = 1/3$ 时，该概率 $p < 10^{-12}$ 。但这一分析假设了理想均匀随机抽样。RANDAO 的可操纵性——proposer 可以选择不揭示 RANDAO reveal 以影响下一轮洗牌——构成了实际的攻击面。设 proposer 通过 $k$ 次连续出块机会进行"随机数磨削"（RANDAO grinding），其可操纵的熵量级约为 $O(k)$ bits。Single Secret Leader Election（SSLE）的研究旨在消除此攻击面，但目前尚无效率足够高的方案进入生产。
    

### 收获

Gasper 的组合语义存在已知的理论缺陷（如 bouncing attack），fork choice 的修补是持续的工程过程，而 MEV 对 proposer 激励的扭曲正在从根本上改变共识层的博弈结构。共识层的安全性并非静态属性，而是攻击者策略空间与协议防御机制之间持续演化的动态均衡。

### 💡 Insight

**子协议各自满足局部最优性质，不保证组合后的系统满足全局最优。** LMD GHOST 在纯 liveness 场景下表现良好，Casper FFG 在纯 safety 场景下具有严格的 accountable safety 证明（若 finality 被违反，可以识别并惩罚 $\\geq 1/3$ 的质押量），但两者组合后的交互效应——如 unrealized justification 导致的 reorg 攻击——需要额外的"胶水逻辑"（proposer boost、fork choice filter 修改等）来修补。这与形式化验证领域中"组合验证"（compositional verification）的困难性直接对应：对子系统 $A$ 和 $B$ 分别成立的性质 $\\varphi\_A$ 和 $\\varphi\_B$ ，不能直接推出 $A \\| B \\models \\varphi\_A \\wedge \\varphi\_B$ 。
<!-- DAILY_CHECKIN_2026-04-10_END -->

# 2026-04-09
<!-- DAILY_CHECKIN_2026-04-09_START -->

### 今日阅读

完成「以太坊执行层全景解析」剩余章节（P2P 网络、Snap Sync、JSON-RPC、总结），阅读「Running Ethereum - Node workshop」的节点运行 Workshop 指引以了解实操层面的准备工作，并回顾「Intro to execution - Resources」的 p2p high-level 和 JSON-RPC 部分做整体收束。

### 串联逻辑

前两天关注的是**单个节点内部**发生了什么（验证、构建、EVM），今天把视角拉到**节点之间**——数据怎么流动、新节点怎么加入网络。devp2p 是底层通道，eth 子协议解决"历史数据获取 + 交易广播"，snap 子协议解决"状态快速同步"。最后以 JSON-RPC 作为执行层对外的统一接口收尾，完成从内到外、从计算到通信的全景。

### 重点研究

-   **交易传播中的随机图论与抵抗审查性**：√n 优化的理论基础是 gossip protocol 在随机图上的扩散速率——在 random regular graph 上，向 O(√n) 个邻居转发即可在 O(log n) 轮内实现全网覆盖。但现实网络并非理想随机图：ISP 级别的审查、地理聚类、eclipse attack 都可能破坏拓扑的随机性假设。这也是为什么 devp2p 的 peer discovery 基于 Kademlia DHT，但又做了以太坊特定的修改（如 node ID 绑定 secp256k1 公钥）来增强 Sybil 抵抗。当前学术界对 P2P 层审查抵抗性的研究（如 Dandelion++ 协议对交易源 IP 的保护）也与此直接相关。另外，加密 mempool 的难点在于它与区块构建的张力——如果交易内容对 builder 不可见，如何进行有效排序？这导向了 threshold encryption、commit-reveal scheme 等密码学原语的应用研究。
    
-   **Snap Sync 与状态膨胀的结构性矛盾**：Snap Sync 的 Healing 问题本质上可以建模为一个 moving target problem：状态树的变化速率 ΔS/Δt 与下载速率的比值决定了 Healing 是否收敛。而以太坊状态的持续膨胀（当前约 300+ GB trie nodes）正在使这个问题恶化。这也是 Verkle Tree、state expiry、Portal Network 等方案的深层动机——它们不只是为了减小证明大小，更是为了让新节点的 bootstrapping 保持可行性。从系统设计角度看，这是一个 liveness 问题：如果新节点无法在合理时间内同步完成，网络的去中心化程度就会实质性降低。
    
-   **同步的安全模型：从弱主观性到形式化信任谱系**：Snap Sync 的安全性建立在三层信任之上：① 弱主观性检查点（信任某个 checkpoint provider）；② Merkle proof 对每批下载数据的完整性验证；③ 经济多数假设（质押量足够大使得全网作恶不可行）。其中①是最弱的环节——如果 checkpoint provider 被攻陷，新节点可以被引导到一条假链上。这也是为什么以太坊社区强调"从多个独立源获取 checkpoint"。从更广泛的角度，这构成了一个信任谱系：Full Sync → Snap Sync → Light Client → Ultra-Light Client，每一级都用额外的信任假设换取同步效率。这与学术上对 bootstrapping trust 的研究（如 Buterin 的 weak subjectivity paper）直接对应。
    
-   **加密 Mempool 的开放问题**：现有方案（如 Cosmos 链上的尝试）必须泄露 Gas 量、发送者地址等元数据才能让构建者工作，但这些元数据本身就是强信息泄露（Gas 量可推断交易复杂度，发送者地址更是直接暴露身份）。这本质上是一个 privacy-utility trade-off：在当前的账户模型下，nonce 顺序性约束使得完全加密几乎不可能，因为 builder 必须知道交易的先后顺序才能确保 nonce 合法性。可能的突破方向包括转向 UTXO 模型的部分语义、或者基于 delay encryption 的新原语。
    

### 收获

今天把视角从单节点拉到网络层面后，发现执行层的很多设计约束不是来自计算逻辑，而是来自网络的物理性质：带宽有限、延迟不可消除、拓扑不可信。节点 Workshop 的存在本身也说明了一个问题：如果运行一个全节点的门槛持续升高（状态膨胀、硬件要求、同步时间），那么去中心化就不再是一个二元状态，而是一个随硬件成本和网络条件连续变化的度量。

### Insight

**以太坊的去中心化不是一个静态属性，而是一个受状态增长率约束的动态量。** 去中心化的实质是"任意人都能在合理成本内独立验证链的状态"。这意味着它受到三个变量的共同约束：状态大小的增长速率、消费者硬件的进步速率、以及同步协议的效率。当状态膨胀快于硬件进步和协议优化时，全节点数量就会自然下降，去中心化程度随之降低，Verkle Tree、EIP-4444、Portal Network 本质上都是在试图改变这个不等式的方向。
<!-- DAILY_CHECKIN_2026-04-09_END -->

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
