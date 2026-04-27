---
timezone: UTC+8
---

# BareerahBenjamin

**GitHub ID:** BareerahBenjamin

**Telegram:** 

## Self-introduction

EPF 实习计划

## Notes

<!-- Content_START -->
# 2026-04-27
<!-- DAILY_CHECKIN_2026-04-27_START -->
# 研发协作（Coordination(PM)）

## 1\. 以太坊核心开发的协作背景

-   以太坊坚持客户端多样性与去中心化，所以核心开发不是由一个团队集中完成，而是分散在许多团队。
    
-   粗略估计，有 200+ 开发者、研究员、测试人员及其他贡献者，分布在大约 20 个团队中。
    
-   这些团队来自不同组织，但都围绕以太坊协议的演进开展工作。
    
    > 关键词：**client diversity**、去中心化、多团队协作
    

* * *

## 2\. 为什么需要“协调（Coordination）”

-   为了维护和推进涉及以太坊协议的各种改动（升级、改进提案等），这些分散的参与者必须能达成共识并高效协作。
    
-   协调工作的基础是以太坊“自由开源”的特性，以及遵循类 Unix 哲学的设计与开发方式（小而清晰的组件、组合使用等）。
    
-   即使有开源和良好设计理念支撑，大规模协作仍需要专门的协调机制与工具来保证信息同步和决策效率。
    
    > 可以把 Coordination 理解为：在多团队、多角色环境下，确保大家“看到同一份信息、在同一节奏上”工作的那部分工作。
    

* * *

## 3\. 以太坊 PM 仓库的角色

-   `ethereum/pm` GitHub 仓库主要用于以太坊项目管理（project management）。
    
-   它的核心用途是：
    
    -   规划开发者会议（安排议程、时间、议题收集）
        
    -   存档会议记录与录音（方便会后查阅、追踪决策过程）
        
        > 这个仓库可以理解为“核心开发会议的公共日程与档案库”。
        

* * *

## 4\. 会议类型与具体用途

在 `ethereum/pm` 上主要协调的会议包括：

-   主要用途：
    
    -   执行层（Execution Layer）与共识层（Consensus Layer）的 AllCoreDevs 会议。
        
    -   这些会议是核心开发者就协议升级、EIP 等问题进行讨论与决策的重要场合。
        
-   次要/扩展用途：
    
    -   就特定协议议题开设的 Breakout Rooms（分组讨论），如 EOF（EVM Object Format）、账户抽象等主题。
        
    -   通过 Google Calendar 跟踪即将到来的协议会议，方便开发者订阅和查看。
        
        > 可以记成三个关键点：AllCoreDevs 主会、主题 Breakout 分会、用日历统一时间和提醒。
<!-- DAILY_CHECKIN_2026-04-27_END -->

# 2026-04-26
<!-- DAILY_CHECKIN_2026-04-26_START -->

# 预确认（**Preconfirmations**）

## 1\. 总体概念：什么是 Based Preconfirmations？

-   Based preconfirmations（简称 preconfs）是以太坊上一种新机制，让用户在交易真正上链之前，就从 L1 proposer 那里拿到一个“有保证的提前确认承诺”。
    
-   它依托链上基础设施、proposer 责任机制（可被惩罚 / slashing）和灵活的承诺获取流程，让用户获得“快且可靠”的执行体验，显著降低等待区块确认的延迟。
    
-   与普通“交易进 mempool 等待打包”不同，preconf 是一个带责任和经济担保的承诺：如果 proposer 不履约，会被按约定惩罚（slashing 或罚没抵押）。
    

你可以把 preconf 理解为：**以太坊出块者提前给你的“合约保证书”，承诺在某个时间窗口内把你的交易按约定执行，否则他赔钱。**

* * *

## 2\. Preconf Promise 的链上构造

## 2.1 两个基础组件

Preconfirmation 承诺（preconf promise）依赖两个关键的链上基础设施组件：

1.  **Proposer Slashing（提议者可被惩罚）**
    
    -   一些 proposer 自愿“opt in”进入额外的 slashing 条件，即：
        
        -   除了正常共识规则下的惩罚外，如果违反了自己做出的 preconf 承诺，也会被额外 slash。
            
    -   这个设计借鉴了 EigenLayer 的思路：通过“再质押（restaking）”附加更多条件，使验证者对额外服务（这里是 preconf）负责任。
        
2.  **Proposer Forced Inclusions（强制包含能力）**
    
    -   为了确保交易在链上确实被打包执行，proposer 需要有一个 **“强制包含特定交易”** 的能力。
        
    -   在 PBS（proposer–builder 分离）环境下，proposer 通常不自己造块，而是从 builder 那里拿区块，这会让“自己强制打交易”变得不经济或不现实。
        
    -   解决方案：引入 **inclusion lists（包含列表）** 等机制，让 proposer 能告诉 builder：
        
        -   这些特定交易必须被包含，否则我不会接受你的区块。
            

## 2.2 “Preconfer”的角色

-   当某个 Beacon Chain 验证者选择成为“preconfer”（预确认者）时，等于同意遵守与 preconf 承诺相关的两类 slashing 条件。
    
-   作为回报：
    
    -   preconfer 可以对用户签发 preconf promises，并通过“小费（tips）”获得额外收入。
        
-   多个 preconfer 之间的优先级，按它们在一个 epoch 中 slot 的先后顺序来排：
    
    -   slot 越早的 preconfer，优先级越高。
        

## 2.3 承诺的效力范围

-   一笔拿到 preconf promise 的交易，不仅可以在该 preconfer 自己的 slot 中被包含，还可以被 **更早的 proposer** 包含执行（只要他们愿意）。
    
-   预确认者（preconfer）的主要义务是：
    
    -   在自己当班的 slot 中，必须 honor 所有他签过 promise 的交易。
        
    -   为此，他会使用 inclusion list 确保这些交易被 builder 放进区块里。
        

* * *

## 3\. 两类故障与 slashing：Liveness vs Safety

文中把与 promise 相关的错误分成两类，每一类都可能导致 slashing：

1.  **Liveness Fault（活性故障）**
    
    -   情况：preconfer 因为“错过 slot”（比如宕机或没出块），导致没能把承诺过的交易包含上链。
        
    -   核心是“没做到预期的活性”，而不是违反逻辑承诺（通常责任会区分可归责、不完全可归责）。
        
2.  **Safety Fault（安全故障）**
    
    -   情况：preconfer 并没有错过自己的 slot，却在链上包含了与自己之前 promise 相冲突的交易。
        
    -   例如：
        
        -   承诺“不会在该笔交易之前加入某类 MEV 抢跑交易”，结果却加了。
            
        -   承诺“在某状态 root 下执行交易”，却在另一个状态下执行。
            
    -   这类属于直接违背承诺内容，通常会被更严厉 slashing。
        

## 3.1 执行队列优先级

-   为了保证 preconf 交易优先执行，系统会对 **没有 preconf promise 的交易** 建立一个单独的执行队列。
    
-   规则是：
    
    -   所有带 preconf 的交易必须先执行；
        
    -   没承诺的交易排在后面。
        
-   这保证了已经“付费买承诺”的用户不会被普通交易抢在前面。
    

* * *

## 4\. Promise 的类型：从强保证到弱保证

-   Preconfer 不只提供一种类型的 promise，而是一个“光谱”：从最强到较弱多种形式。
    
-   可能的 promise 类型包括：
    
    -   **强状态保证型**：
        
        -   承诺在特定的 post-execution state root 下达成某个结果，比如“执行完后余额至少多少”、“某合约状态达到某条件”。
            
    -   **简单包含型**：
        
        -   仅承诺“会包含这笔交易”，不保证它在区块内的具体顺序，或和其他交易的相对关系。
            
-   这种灵活性使 preconfer 可以针对不同用户需求提供不同价格 / 复杂度的服务：
    
    -   用户只想快速上链，就用简单包含型；
        
    -   复杂 DeFi 操作、跨链原子交易则选择强保证型。
        

* * *

## 5\. Preconfs 的关键要素（系统设计维度）

文中列了一系列设计要点，每一项都是构建一个可用 preconf 系统需要考虑的方面。

## 5.1 Endpoints（接入方式）

-   Preconfers 可以通过：
    
    -   直接 API（例如 HTTPS endpoint）；
        
    -   去中心化 p2p 网络。
        
-   前者响应更快，后者去中心化程度更高，需要在两者之间权衡延迟和可获得性。
    

## 5.2 Latency（延迟）

-   目标是做到非常快的确认时间：
    
    -   通过直接通讯通道，理想情况下可以做到大约 100ms 级别的 preconfirmation。
        
-   这让 preconf 的 UX 更接近 Web2 的“秒级确认”。
    

## 5.3 Bootstrapping（启动与覆盖率）

-   要让 preconf 真正好用，需要有足够多的 L1 验证者 opt in 成为 preconfer：
    
    -   这样在 proposer lookahead 窗口中，用户几乎总能找到某个可用 preconfer。
        
-   否则用户经常找不到愿意提供承诺的 proposer，UX 会很差。
    

## 5.4 Liveness Fallback（活性备份）

-   为提高可靠性，用户可以同时从多个 preconfer 那里拿 promise：
    
    -   即使其中一个 preconfer 因为错过 slot 导致 Liveness Fault，其他 preconfer 仍然可以履约。
        
-   这是典型的“多路线冗余”设计，用更多 promise 换更高成功率。
    

## 5.5 Parallelization（承诺类型的并行）

-   系统允许多种 promise 类型并行存在：
    
    -   从“严格的状态后置保证”到“模糊的 intent-based promises（意图型承诺）”。
        
-   这样可以支持：
    
    -   简单用户（只需包含）
        
    -   复杂 DeFi 合约（需要状态条件）
        
    -   跨链、跨 rollup 原子操作等高级场景
        

## 5.6 Replay Protection（重放保护）

-   必须有机制防止同一承诺或交易在不该发生的场景被重复执行（重放攻击）。
    
-   这通常涉及 nonce、链 ID、域分离（domain separation）等安全技术，来确保 promise 只能在指定上下文中生效。
    

## 5.7 Single Secret Leader Election（SSLE）

-   SSLE 允许在 lookahead 窗口内，preconfer 可以：
    
    -   确认自己是未来的某个 slot 的“秘密领导者（preconfer）”；
        
    -   在不提前公开身份的情况下，向可信方证明这一点。
        
-   这样可以：
    
    -   减少提前暴露身份带来的攻击风险（例如 DDoS）；
        
    -   仍然允许用户获得必要的保证。
        

## 5.8 Delegated Preconf（委托预确认）

-   某些 proposer 可能带宽或算力有限，难以直接处理大量 preconf 请求。
    
-   他们可以把自己的 preconf 职责“委托”给第三方：
    
    -   第三方代表他们处理 promise 流程，但最终责任仍绑定在 proposer 的抵押上。
        
-   这样可以让资源有限的小验证者也能参与 preconf，而不被迫退出这块市场。
    

## 5.9 Fair Exchange（公平交换）

-   用户与 preconfer 之间有一个经典问题：
    
    -   用户怕“付了 tip 但没承诺”。
        
    -   preconfer 怕“给了承诺但用户不付 tip”。
        
-   可能的解决路径包括：
    
    -   公共流式公开 promise 信息，提高透明度；
        
    -   由可信 relay 做中介，确保两边的公平交换；
        
    -   使用密码学的公平交换协议（例如 commit–reveal + 原子交换结构），在协议层确保双方要么都完成，要么都不完成。
        

## 5.10 Tip Pricing 与 Negative Tips（小费定价与负小费）

-   Tip 定价要考虑：
    
    -   这笔交易对 proposer/ builder 的 MEV 机会有什么影响。
        
    -   如果一笔交易会减少 MEV 空间，就需要用户支付更高 tip。
        
-   反过来，有时存在 **Negative Tips（负小费）**：
    
    -   某些交易本身制造了 MEV 机会（比如改变 DEX 价格，创造套利空间）。
        
    -   在这种情况下，preconfer 甚至可以接受“负小费”，因为执行这笔交易本身就是对他们有利的。
        

* * *

## 6\. Preconfs 获取流程：从用户请求到链上执行

文中给了一个时序图，这里用文字顺一下完整流程。

1.  **用户识别下一个 preconfer**
    
    -   依据 proposer lookahead 窗口，用户或智能合约找到下一个愿意提供 preconf 的 proposer（已经抵押、声明可提供服务）。
        
2.  **发送 promise 请求**
    
    -   用户通过该 preconfer 提供的渠道（API / p2p）发送请求：
        
        -   包含交易详情和期望的条件（比如时间、顺序、状态约束）。
            
3.  **preconfer 评估请求**
    
    -   考虑因素包括：
        
        -   当前网络状况、slot 安排；
            
        -   用户给的 preconf tip；
            
        -   交易可能带来的 MEV 机会或风险。
            
    -   综合判断要不要接受该请求。
        
4.  **签发 preconf promise**
    
    -   若接受请求，preconfer 会生成并签名一个 preconf promise：
        
        -   明确：在某个 slot 内，会包含并执行这笔交易，并遵守一定条件。
            
    -   然后把该 promise 返回给用户。
        
5.  **用户支付 preconf tip**
    
    -   用户按约定向 preconfer 支付小费，可以是直接转账或通过某种 escrow。
        
    -   某些方案中，小费会暂时托管，待 promise 履约后才释放，增加用户安全感。
        
6.  **包含与执行**
    
    -   在指定 slot，preconfer 在其提议的区块中包含该交易，依 promise 条件执行。
        
    -   区块上链后，用户与 preconfer 都可以在链上验证 promise 是否被履行。
        
7.  **补充机制**
    
    -   Fallback：用户可以预先向多个 preconfer 请求 promise，防止某个 missed slot。
        
    -   Dispute Resolution：若用户认为承诺未被履行，可触发争议解决 / slashing 机制。
        

* * *

# **基于以太坊的预确认排序（Based Sequencing with Preconfirmations）**

## 1\. 背景与目标：为什么要 Based Sequencing + Preconfs？

-   以太坊的长远愿景是 “United Chains of Ethereum”：多个 L2 / rollup 像一个联邦，各链间转资产、跨应用像“跨州”一样自然、便宜、安全。
    
-   今天，以太坊给 rollup 提供两件事：
    
    -   结算（settlement）：在 L1 上对 L2 状态进行争议裁决、最终结算。
        
    -   数据可用性（DA）：把 L2 的交易数据上链，保证以后任何人都能拿到，防止 sequencer 作恶藏数据。
        
-   但仅靠这两点，还不足以让 rollup 之间“无缝协作”：
    
    -   各个 rollup 往往有**各自的 sequencer**，是中心化或半中心化的，排序权不共享。
        
    -   用户在不同 L2 之间转资产、套利、做 MEV、跨链 DeFi，经常要等长时间、承受高不确定性。
        
-   文章提出：以太坊可以再向上提供第三种服务——**Ethereum sequencing**，即把“排序权本身”变成以太坊输出给 L2 的公共服务。
    
    -   这样 L2 排序可以直接继承以太坊的经济安全性、去中心化和中立性。
        

* * *

## 2\. 几种 Sequencing 模式对比

## 2.1 去中心化 Sequencing

-   多个节点共同参与排序，通常需要一个额外的共识层（BFT、PoS 共识等）。
    
-   好处：
    
    -   抗审查能力强，不容易被一个 sequencer 垄断。
        
    -   可以在 L2 层就实现较强安全性。
        
-   问题：
    
    -   共识开销大，延迟可能较高。
        
    -   复杂的协议设计，需要解决 leader 选举、投票、恢复等细节。
        
    -   仍然有 MEV 分配、串谋、抢跑等问题。
        

## 2.2 Shared Sequencing

-   多个 rollup 共享一个独立的 sequencing 层，比如：
    
    -   一个“shared Sequencer 网络”为多个 L2 排序，或者通过 rollup aggregator / deposit sharing / shared execution 之类方式联动。
        
-   优点：
    
    -   提供一个跨 rollup 的公共排序点，有利于跨链原子性、MEV 协调和价格发现。
        
-   难点：
    
    -   MEV 分配：不同 rollup 的 MEV 如何公平分享，很复杂。
        
    -   安全与信任：Shared Sequencer 本身成了一个新的“基础设施中心”，需要非常高的信任与治理。
        
    -   标准化问题：不同 rollup 有不同规则、执行环境和延迟要求，要统一很难。
        

## 2.3 Based Sequencing（基于以太坊）

-   “Based” 的意思是：**直接基于 Ethereum L1 的 proposer / validator 集合来做 sequencing**。
    
-   把排序权默认“跟着出块权走”：谁是某个 slot 的 Beacon proposer，谁就可以为相关的 rollup 做 sequencing。
    
-   优点：
    
    -   直接利用已有的 PoS 安全性，不用搭额外的共识网络。
        
    -   抗审查和去中心化程度与以太坊本身一致。
        
    -   自然适配 MEV-Boost / PBS 生态，因为本来就依赖 proposer–builder–relay 这套结构。
        
-   但仅有 Based Sequencing 还不够：
    
    -   用户还希望**快速确认（preconfirmation）**：
        
        -   在区块“真正上链之前”就收到一个强承诺“你的交易会被包含并以某种顺序执行，否则我会损失保证金”。
            

* * *

## 3\. Preconfirmation 是什么？它解决什么问题？

-   传统上，用户只能等：
    
    -   交易进 mempool → 被某个 proposer / builder 打包进区块 → 区块被链上接受 → 等一定确认数或 finality。
        
-   在 rollup / L2 世界里，从用户视角看：
    
    -   L2 sequencer 可以立刻告诉你 “ok，你的交易排在这个位置，我保证不会反悔”，这就是 preconfirmation 的体验。
        
-   在基于 L1 的世界里，如果我们能让**未来的 L1 proposer 对某笔 L2 tx 做出类似承诺**，那么：
    
    -   用户就能在非常短时间内拿到一个“强确认信号”，而这信号由 proposer 抵押的质押金背书。
        
-   Preconfirmation 的关键点：
    
    -   是一个“有保证的承诺”（promise + collateral），不是简单的口头说说。
        
    -   违反承诺会有经济惩罚（slashing 或没收抵押），因此用户可以信任这个承诺有真金白银托底。
        

* * *

## 4\. Look-ahead + Proposer 抵押：Based Preconfs 的基本结构

## 4.1 Lookahead：提前知道谁会出块

-   在以太坊 PoS 中，未来一段时间（例如一个 epoch）的 proposer 顺序是已知的，这就是“look-ahead period”。
    
-   对于每个即将出现的 slot，我们知道：
    
    -   哪个验证者会是 proposer。
        
    -   对应的 slot 大约会在什么时间到来。
        

## 4.2 Proposer 抵押 + Opt-in

-   这些未来的 proposers 可以选择“opt in”：参加一个 **based sequencing with preconfs** 协议。
    
-   Opt-in 的方式：
    
    -   他们向某个系统（可能是合约或 off-chain 协议）锁定一定的 collateral，承诺在对应 slot 中可以为 rollup 提供 preconfirmation 服务。
        
    -   这笔 collateral 就是违约时会被 slash 的本金。
        

## 4.3 用户如何拿到 preconf 承诺？

-   用户想在某个时间附近得到一个 L2 交易的确认：
    
    -   查看 look-ahead 范围内哪些 proposer 已经 opt-in，愿意提供 preconf。
        
    -   给其中一个 proposer 发请求：附上交易内容（或摘要）、时间要求、费用（tip/fee）。
        
-   proposer 接到请求后：
    
    -   评估风险（比如这笔交易对状态的影响、可能引发的 MEV 冲突等）。
        
    -   如果接受，就签发一个 **preconfirmation promise**：
        
        -   明确“在某个 slot 内、按照某种顺序、在某个 rollup 上，将包含并执行这笔交易”的承诺。
            
        -   并把该承诺绑定到自己的抵押上。
            

* * *

## 5\. 执行路径：Preconf 如何与 MEV-Boost / PBS 结合？

## 5.1 当前 PBS / MEV-Boost 流程回顾

-   在 PBS（Proposer–Builder Separation）+ MEV-Boost 模式里：
    
    -   Builder 构建候选区块（考虑 MEV，排列交易）。
        
    -   Relay 中继这些区块给 proposer。
        
    -   Proposer 只需要选择价值最高的区块并签名广播即可。
        

## 5.2 引入 Preconfs 后的流程

1.  用户 → proposer：用户发起 preconf 请求，proposer 同意并签发 promise。
    
2.  proposer → relay：proposer 将这个 promise（包括要包含的交易及执行条件）发给 MEV-Boost relay。
    
3.  relay → builders：relay 把 preconf 信息转发给所有 builders，告诉他们“某个 slot 的区块中必须包含这些 preconfirmed 交易，并满足某些约束（例如顺序、gas 限制等）”。
    
4.  builders 构建区块：
    
    -   在考虑 MEV 的同时，必须满足所有 preconf 约束。
        
    -   否则构建出的区块即使价值高，也可能不会被 proposer 接受，因为 proposer 违约会被罚钱。
        
5.  builder → proposer：builder 把构建好的区块发回 proposer。
    
6.  proposer 发布区块：
    
    -   proposer 选择一个（或多个）满足 preconf 条件的候选区块之一签名并 broadcast。
        
    -   区块被链上接受后，preconfirmed 交易按承诺执行。
        

-   这样，通过 PBS 这条链路，**preconf 承诺被传导到了实际构建区块的 builder 层**，确保从协议路径上强制执行。
    

* * *

## 6\. 设计中的细节和变体

文章里讨论了很多可能的设计变体和细节，你可以重点记住以下维度：

1.  **承诺强度（guarantee strength）**
    
    -   弱保证：只承诺“包含”某笔交易，但不保证相对其他交易的顺序。
        
    -   强保证：承诺该交易在区块中的位置（比如“top of block”）、不会被前置 MEV 交易抢跑。
        
    -   保证越强，对 builder 的约束越大，可能导致可提取 MEV 减少。
        
2.  **时序与窗口**
    
    -   有的设计允许只对“下一个 slot” preconf。
        
    -   有的设计允许对“未来多个 slot”提前承诺（利用更长 look-ahead）。
        
    -   窗口越长，灵活性越大，但状态预测更难。
        
3.  **多 preconf 的组合**
    
    -   多个用户、多个 dApp 的 preconf 交织在一起，builder 必须同时满足全部约束，复杂度会上升。
        
    -   需要规则来解决冲突，比如优先级、定价或拒单（proposer 拒绝给某些请求承诺）。
        
4.  **对 L2 的映射**
    
    -   同一个 L1 slot 中，proposer 可能为多个 rollup 做 sequencing。
        
    -   不同 rollup 可能有不同的执行环境、gas 模型和状态依赖，如何统一处理很难。
        

* * *

## 7\. 风险、挑战与开放问题

## 7.1 区块价值与激励

-   Preconf 给 builder 和 proposer 加了约束：
    
    -   一部分原本可以通过自由排序获得的 MEV，现在被 preconf 锁死。
        
    -   这可能降低一个 slot 的最大区块价值，使 validator 收入下降。
        
-   需要设计合理的 fee / tip 机制，补偿验证者因 preconf 带来的机会成本。
    

## 7.2 中心化风险

-   实现高性能、低延迟的 preconf 基础设施并不容易：
    
    -   需要高带宽、低延迟、强工程能力，对 DoS 有抵抗力。
        
    -   这会偏向大型专业化实体，而小节点难以参与。
        
-   结果可能是：
    
    -   少数大节点集中处理大部分 preconf 请求，形成新的中心化点。
        

## 7.3 安全性与活性（Safety vs Liveness）

-   如果 proposer 因为网络问题、软件 bug、监管压力等原因没能履约：
    
    -   是 slashing（安全问题，视为作恶）还是只罚一点保证金（活性问题）？
        
    -   如何区分可归责的违约和不可归责的失败？
        
-   如果 slashing 过于严苛：
    
    -   验证者会害怕参与 preconf，减少 opt-in。
        
-   如果太宽松：
    
    -   Preconf 的承诺变成“软承诺”，用户无法真正信任。
        

## 7.4 监管与法律问题

-   Preconf 很像“带对价的合同承诺”：
    
    -   在某些司法辖区，可能被视为某种合约或金融服务，牵涉监管。
        
    -   验证者和 infra 提供方可能面临合规压力，进一步推动集中化。
        

* * *

## 8\. 总结：基于以太坊的 Preconf Sequencing 想要实现什么？

-   让以太坊不只是结算 + DA 的“最低层”，而是成为所有 L2 的**排序+快速确认基础设施**。
    
-   用户可以在多数情况下：
    
    -   快速拿到一个由 L1 proposer 抵押背书的强承诺。
        
    -   在跨 rollup 交易、DeFi 操作中显著降低不确定性。
        
-   同时，保持：
    
    -   与以太坊 PoS 同等级别的安全性与去中心化。
        
    -   尽可能兼容 PBS / MEV-Boost 现有生态。
        
-   但要真正落地，还需要在：激励设计、preconf 形式化、安全/活性权衡、中心化控制、合规问题上做大量研究和工程实践。
    

* * *

# **轻客户端（Light Clients）**

## 1\. 轻客户端的动机与基本目标

-   现在大多数以太坊用户是通过“执行层客户端（EL）提供的 RPC”接入网络，比如钱包连到某个 RPC 节点，读取余额、发交易等。
    
-   运行一个完整节点要维护当前状态（state），需要大量存储（上百 GB）、带宽和计算，这对普通用户和轻量设备来说负担太重。
    
-   因此，大部分钱包默认直接连第三方 RPC 服务商（Infura、Alchemy 等），几乎**不自己验证数据**，完全信任对方返回的信息，这引入了中心化与信任风险。
    
-   “Light client（轻客户端）”的核心想法是：让用户在不运行完整节点的情况下，也能**无信任（或最小信任）地验证网络数据**。
    
-   “轻客户端”是一个总称，具体实现设计很多，处于不同阶段：有的已经生产可用，有的还在研究/开发中。
    

* * *

## 2\. 轻客户端主要路线一览

文中列出了几类轻客户端 / 相关路线：

-   基于 RPC 的验证：通过共识层的信标根（Beacon root）来验证执行层 RPC 返回的数据。
    
-   Stateless clients（无状态客户端）：依靠从网络传播的“见证（witnesses）”来验证数据，而不用本地存全状态。
    
-   LES 协议：Geth 早期的轻节点模式，通过专门的轻协议向全节点请求数据。
    
-   Portal Network：一种 overlay 网络，利用概率性机制保证数据完整性和可用性。
    

下面逐个展开。

* * *

## 3\. RPC proxy light client：给 RPC 再加一层“真伪验证”

## 3.1 基本思路

-   这种轻客户端并不直接充当网络中的节点，而是作为钱包 / 应用 和 RPC 服务商之间的 **代理层 / 中间件**。
    
-   工作方式：
    
    -   对外表现为一个标准 RPC 接口（应用连它，就像连普通 RPC 一样）。
        
    -   它再向真实的 RPC 提供商请求数据。
        
    -   同时，从一个独立的 Beacon 节点获取信标链根（Beacon root）或共识相关证明。
        
    -   用这些证明来核实 RPC 提供的执行层数据是否与共识层一致、是否伪造或篡改。
        
-   好处是：
    
    -   用户不再**完全**信任单一 RPC。
        
    -   除非 RPC 和 Beacon 节点 **串谋并以同样方式撒谎**，否则难以骗过轻客户端。
        
-   但这种方案 **仍然需要**某个 RPC 提供商，用户自己并没有“变成一个参与 P2P 的节点”。
    

## 3.2 与 P2P 节点模式的区别

-   真正的 P2P 节点通过以太坊协议与其他节点通信：
    
    -   可以从 peers 获取当前链 tip、请求历史区块等。
        
    -   想查询余额，必须自己下载必要的区块 / 状态并验证，然后从状态里查出结果，没有“直接 RPC 拿余额”这种接口。
        
-   如果轻客户端走 P2P + 自己验证的方式，其行为就越来越接近一个“正常节点”，而不仅是 RPC 代理。
    

## 3.3 已有实现示例

-   Helios：\[a16z 推出的一个“RPC 验证型轻客户端”，用户在本地运行 Helios 作为钱包 / 应用的 RPC 代理。
    
-   Kevlar：另一个类似方向的实现，专注验证第三方 RPC 数据。
    
-   这些客户端通常自带默认连接到公共 Beacon 节点的配置，从而最小化“RPC 和 Beacon 节点同时撒谎”的概率。
    
-   社区中有人尝试在 Helios 中直接实现 CL 的 P2P 协议（libp2p），这样就可以完全通过 P2P 获取共识信息，而不是依赖第三方 Beacon API。
    

* * *

## 4\. Stateless Clients：无状态客户端的方向

-   Stateless client 的核心理念：
    
    -   客户端自己不保存完整状态（accounts、storage 等），只保存少量必要信息。
        
    -   当需要验证某个区块 / 交易时，从 P2P 网络接收该部分状态的“见证（witness）”。
        
-   Witness 通常包括：
    
    -   描述相关状态（账户、合约存储等）的 Merkle 证明。
        
    -   这样客户端只要持有最新的状态根，就能利用见证验证数据是否正确。
        
-   好处：
    
    -   不需要数百 GB 的本地存储。
        
    -   只在查询 / 验证时下载与之相关的小块状态。
        
-   缺点 / 挑战：
    
    -   Witness 的生成与传播有额外开销，需要全节点或专门节点提供。
        
    -   协议层需要更复杂的支持和标准化（目前仍在研究与演进中）。
        

* * *

## 5\. Portal Network：覆盖网络 + 概率性保证

-   Portal Network 是一个专门为轻客户端设计的 **覆盖网络（overlay network）**。
    
-   它的目标是：
    
    -   在不要求每个节点存全部数据的情况下，整体网络依然能够“概率性地保证”数据完整性与可用性。
        
-   基本特点：
    
    -   将不同类型的数据（header、state、历史数据等）分片存储在众多轻节点上。
        
    -   使用分布式哈希表（DHT）、内容寻址等技术，使节点可以在网络中找到自己需要的数据。
        
    -   利用冗余和采样，概率性确保数据未被篡改且不易丢失。
        
-   对轻客户端的意义：
    
    -   不必须直接连某个特定 RPC 或巨型全节点，而是通过 Portal Network 与大量轻节点协作获取数据。
        
    -   避免单点中心化，同时降低每个节点的存储负担。
        

* * *

## 6\. LES（Light Ethereum Subprotocol）：Geth 早期轻节点模式

## 6.1 工作模式

-   LES 是 Geth 早期设计的一种“轻节点协议”模式，通过在 P2P 上订阅 les 子协议来运行节点。
    
-   启动时配置为 light 模式：
    
    -   该节点**不会下载整个链**。
        
    -   它会向其他支持 les 的全节点请求最新区块、必要的状态数据等。
        
-   LES 提供了面向轻节点的特定请求 / 应答协议，使轻节点不必像全节点那样维护本地完整状态。
    

## 6.2 现实中的问题

-   全节点要额外配置才能“提供 LES 数据”，这不是默认行为。
    
-   结果：
    
    -   网络中实际愿意/配置好支持 LES 的节点很少。
        
    -   轻节点经常找不到足够的 LES 提供者，导致可靠性差。
        
-   因此，尽管 LES 模式在 Geth 中存在，但在现实中“轻节点模式不太好用”，阻碍了它的广泛部署。
    

* * *

# 账户抽象（EIP-7702）

## 1\. EIP-7702 是什么？

-   EIP-7702 的标题是 “Set EOA code”，是一个面向 **账户抽象（Account Abstraction）** 的提案，目标是增强以太坊账户的可编程性和灵活性。
    
-   它希望改善用户体验和安全性，让“账户如何验证交易”这件事变成可编程逻辑，而不是死板地绑定在传统 EOA 模型上。
    

换句话说：EIP-7702 想让“普通地址”也能像智能合约钱包一样变得更聪明，但又尽量保持兼容、简单。

* * *

## 2\. 账户抽象背景：为什么需要 7702？

-   传统以太坊有两种账户：
    
    -   EOA（Externally Owned Account，外部账户）：由私钥控制，验证规则固定（ECDSA 签名），不能自定义验证逻辑。
        
    -   合约账户：由代码控制，可以写各种自定义逻辑，但用户体验上常常比较重，部署和使用成本更高。
        
-   账户抽象（AA）的目标是模糊这两者的界限，让“账户验证交易”的逻辑变成可编程：
    
    -   多签 / 社交恢复
        
    -   Gas 代付 / 无 gas 交易
        
    -   批量交易（bundling）
        
    -   自定义权限控制等
        
-   在这个大背景下，EIP-7702 是朝 AA 方向迈出的另一步：
    
    -   用一个新的交易类型 + “设置 EOA code” 的机制，让 EOA 可以选择性地“变身”为合约逻辑控制的账户。
        

* * *

## 3\. 核心机制：Set EOA Code + Type 4 交易

## 3.1 新交易类型：Type 4

-   EIP-7702 定义了一种新的交易类型（Type 4），用来以“更安全、对用户更友好”的方式，为地址提供可编程钱包功能。
    
-   Type 4 交易的主作用：
    
    -   允许地址所有者签名一个“授权”，把自己这个地址设置为由某个现有智能合约的代码来表示（或模仿）。
        
-   从结果上看：
    
    -   此后，该地址在链上表现得像是运行那段合约代码一样，可以拥有自定义验证、批处理、代付等逻辑。
        

## 3.2 “Set EOA Code”：地址 → 代码化

-   EIP-7702 允许用户将自己的地址映射为某个已经存在的智能合约的代码：
    
    -   不是简单地“把代码复制到该地址”，而是让该地址在执行时“代表 / mimic”指定的智能合约逻辑。
        
-   这样，用户可以通过一次授权，把原本的 EOA 地址升级为 **可编程钱包**：
    
    -   不需要迁移资产到一个新合约地址（避免 UX 麻烦）。
        
    -   保持原来的地址不变，只改变它的“行为逻辑”。
        

* * *

## 4\. 能带来哪些功能和体验提升？

EIP-7702 的文案里直接提到，它解锁了一系列此前在传统账户模型中做起来不方便或不可能的用例。

## 4.1 可编程钱包（Programmable Wallets）

-   用户可以选择“opt in”到各种钱包逻辑：
    
    -   比如：一个现成的智能合约钱包模板（多签、Guardian、限额控制等）。
        
-   地址本身通过 7702 和这段合约绑定，后续所有交易验证走这段合约逻辑，而不是传统 ECDSA 单签。
    

## 4.2 交易打包（Transaction Bundling）

-   合约逻辑可以支持“一次签名触发多步操作”，比如：
    
    -   在一个逻辑里完成 Approve、Swap、Stake、转账等多步操作。
        
-   对用户来说：
    
    -   不需要一笔笔点确认，多笔操作可以在规则允许下统一由合约处理。
        

## 4.3 无 Gas 交易（Gasless Transacting）

-   通过合约逻辑可以实现：
    
    -   Gas 由第三方支付（sponsor / paymaster），用户只需要完成授权。
        
-   结合 EIP-7702：
    
    -   用户地址变成由合约控制，合约可以内置“谁帮我付 gas、在什么条件下”等规则。
        
-   对新用户来说：
    
    -   更像 Web2 的体验，不需要先准备 ETH 当手续费。
        

## 4.4 自定义资产访问与恢复方案

-   合约可定义复杂的资产访问控制：
    
    -   多签 + 时间锁
        
    -   社交恢复（例如好友 / 设备作为恢复人）
        
    -   分层权限（小额日常消费 vs 大额转账需更多签名）
        
-   EIP-7702 提供的是一个框架：
    
    -   用户通过 Type 4 交易将自己的地址挂接到这些合约逻辑，实现更丰富的安全 / 恢复策略。
        

* * *

## 5\. 为什么说它是“朝账户抽象迈出的一步”？

-   官方描述里明确把 7702 放在“Account Abstraction 路线”的语境里：
    
    -   它通过“抽象账户逻辑”来改善用户体验和安全。
        
-   抽象的意思是：
    
    -   把“交易验证规则”从底层协议硬编码中拉出来，变成开发者和钱包可以自定义的合约逻辑。
        
-   对用户和开发者来说：
    
    -   不必等到一个完全“原生 AA 的以太坊 2.0”，就可以在现有架构上获得大量 AA 的好处。
        

* * *

## 6\. 小结：你可以怎么在脑子里记 EIP-7702？

可以用一句话记：

> **EIP-7702 = 给 EOA 加上一层“可换逻辑”的外壳，通过 Type 4 交易把地址关联到合约代码，让普通地址变成可编程钱包。**

核心关键词：

-   新交易类型：Type 4。
    
-   Set EOA code：让 EOA 地址“模仿”已有合约的逻辑。
    
-   解锁 programmable wallets：多签、社交恢复、gasless、打包交易、自定义权限等。
    
-   账户抽象的一步：将交易验证逻辑从固定 ECDSA 签名规则抽象成合约逻辑。
<!-- DAILY_CHECKIN_2026-04-26_END -->

# 2026-04-24
<!-- DAILY_CHECKIN_2026-04-24_START -->


# eODS（Enshrined Operator-delegator Separation）

## 一、背景与问题：为什么要 eODS？

-   当下以太坊质押已经自然形成两类角色：
    
    -   Delegators：只出资金、不跑节点的质押者，承担被 slash 的风险。
        
    -   Operators：运行验证节点，投入声誉和自有资金，同样会被 slash。
        
-   这两者之间存在典型的“委托—代理（Principal–Agent）问题”：代理人（验证者 / Operator）的行为会直接影响委托人（Delegator）的资金安全。若验证者作恶或运维不好，委托人的质押也会一起被罚没。
    
-   由于质押发行奖励巨大，各种中间层（LSP、池子等）会形成多层委托关系链，加剧了激励不一致和集中化的风险。
    

* * *

## 二、什么是 eODS？

-   eODS 全称为 “Enshrined Operator-delegator Separation”，是一个在协议层进一步拆分 **Validator 角色** 的设计空间，与 ePBS、Execution Tickets 一起构成 The Scourge 路线中的质押经济闭环。
    
-   现有“拆分 Validator”升级大致对应三种分离：
    
    -   ePBS：提议者–打包者分离（proposer-builder）
        
    -   ET：验证者–提议者分离（validator-proposer）
        
    -   eODS：运营者–委托者分离（operator-delegator）
        
-   目标是：在协议层显式建模“运营者”和“资金提供者”，让协议能更好地看见并管理质押系统中的风险与激励，而不是完全依赖链下结构。
    

* * *

## 三、两层质押（Two-tier staking）与 SSF

-   在 Single-Slot Finality（SSF）的研究中，BLS 聚合能力限制意味着：不可能让所有参与者每个 slot 都签名。
    
-   因此引入两层质押范式：
    
    -   高复杂度层（Heavy tier）：
        
        -   约 1 万个参与者，每个 slot 都参与。
            
        -   提供重型节点服务（Gasper / FFG），高收益但 slash 风险高。
            
    -   低复杂度层（Light tier）：
        
        -   只偶尔被抽中参与。
            
        -   提供轻节点服务，硬件和技术门槛低，可免 slash 或选择性地短期承担 slash 风险。
            
-   奖励与风险：Heavy 提供最终性安全，承担 slash；Light 更偏向“信号与约束”服务，风险低、回报也较低。
    

* * *

## 四、Delegators 在 eODS 下的新角色

-   Vitalik 提出关键问题：“从协议视角看，为什么要有 delegators？”——如果不让 delegators 做任何有意义的事，理论上可以砍掉中间层，直接在协议层控制总质押量和奖励，也能达到相似安全性。
    
-   在 eODS 设计下，Delegators 应承担“有意义的角色”，主要有两条路：
    
    1.  运营者集合的**策展者（curation）**：
        
        -   基于费用、可靠性等标准选择不同 Operators。
            
    2.  轻服务的提供者或参与者：
        
        -   提供对 **审查阻力** 装置的输入（例如 Inclusion Lists, Multiplicity Gadgets）。
            
        -   对链头（head）的看法进行签名，作为区分于 Gasper 操作员的另一类信号。
            
-   激励方式：
    
    -   Delegators 在 eODS 中不直接提供 FFG 的经济安全（不参与 Finality 的 slashable stake），但通过暴露审查、不一致等问题，为协议提供“观察和约束”价值，并由协议重新分配发行奖励进行补偿。
        

* * *

## 五、1D 与 2D eODS：从简单分离到 “彩虹质押”

## 1\. 一维 eODS（1D-eODS）

-   单纯在协议中标明“运营者”和“委托者”的区分：
    
    -   Operator：slashable
        
    -   Delegator：slashable（或在“封顶罚没”方案中变为 non-slashable）
        
-   如果通过“封顶惩罚”只惩罚 Operator 自身的抵押，Delegator 变成 non-slashable：
    
    -   Delegators 资产不再被 slash。
        
    -   但会导致：有些人想把资产委托给愿意被 slash 的 “两层运营者”；有些人又只想委托轻服务。
        
    -   同时在最低发行（MVI）语境下，这种做法经济效果接近于把质押收益压到很低，让质押变成“显式利他行为”。
        

## 2\. 二维 eODS（2D-eODS / Rainbow staking）

-   引入 **重服务（Heavy）** 与 **轻服务（Light）** 的区分后，Validator 角色在两个维度上被拆分：
    
    -   纵向：Heavy vs. Light（服务复杂度与安全强度）
        
    -   横向：Operator vs. Delegator（运行 vs. 出资）
        
-   形成四类组合（“彩虹质押”）：
    
    -   Light Operator / Light Delegator（通常 non-slashable）
        
    -   Heavy Operator / Heavy Delegator（slashable）
        
-   经济含义：
    
    -   Light 服务：
        
        -   用 stake 作为 Sybil 控制和权重，主要服务于审查阻力、轻量信号等。
            
        -   奖励来自将部分发行“重定向”到轻服务（类似现在同步委员会的奖励方式）。
            
        -   可以通过市场化的轻服务运营者 + 可即时再委托，形成竞争、提高成本效率。
            
    -   Heavy 服务：
        
        -   stake 提供强经济安全，参与 FFG 等最重型 AVS，一旦发生安全故障（如冲突最终性），全部 stake 清零。
            
        -   通过更强的协议原语（LSM、DVT、快速再委托等）支撑安全和去中心化。
            

* * *

## 六、重服务 vs. 轻服务：机制设计对比

-   文档中给出了一张表格，对 Heavy / Light 两类服务进行机制层面对比：
    
    -   服务原型：
        
        -   Heavy：Gasper / FFG
            
        -   Light：审查阻力装置（IL、MG 等）
            
    -   奖励动态：
        
        -   Heavy：大多情况下“相关性”好时拿奖励（大家都正常在线、同步）；在发生故障时，反相关也有价值。
            
        -   Light：通常是“反相关”提供价值——在存在审查或分歧时，提供不同信号才有奖励空间。
            
    -   Slash 风险：
        
        -   Heavy：运营者与 delegator 都 slashable。
            
        -   Light：要么无 slash，要么只 slash 运营者。
            
    -   角色能力与门槛：
        
        -   Heavy Operator：需运行全节点，资本规模大，硬件成本高。
            
        -   Light Operator：小型节点即可，硬件要求低，资本量不敏感。
            
        -   Delegators：
            
            -   Heavy：提供经济安全，典型的“锁仓 + 承担 slash”。
                
            -   Light：把自己权重委托给表现好的轻运营者，以增加审查阻力和偏好熵。
                
    -   单独质押者（solo staker）：
        
        -   Heavy 侧：更易通过 LSP / DVT 的方式参与。
            
        -   Light 侧：门槛低，可以广泛吸收长尾参与者。
            

* * *

## 七、eODS 与再质押、solo staker

-   eODS 的 2D 模型可以被视作对再质押（restaking）的部分“入协议化（enshrinement）”：
    
    -   把协议内各种 AVS（包括 Gasper）视作“可被质押参与”的服务，奖励来自新增发行；
        
    -   ETH 持有人可以直接作为运营者参与，也可以作为 delegator 把权重委托给运营者。
        
-   对 solo staker 的价值：
    
    -   Gasper 是最重型 AVS，获得最多发行，容易吸引大规模托管质押、LST 等中介结构。
        
    -   为防止单一 LST 占据大部分 ETH 供应，协议需要 MVI 等机制调节 Gasper 的经济权重。
        
    -   solo staker 因为不能基于自有抵押发行有信誉的 LST，资本效率低，在目前 MVI 压力下处境较差。
        
-   eODS 视角下，solo staker 的两个核心价值主张：
    
    1.  **增强网络韧性**：
        
        -   在大运营者下线、发生故障时，solo staker 作为 fallback 参与区块推进。
            
    2.  **偏好熵的生成者**：
        
        -   通过轻服务参与，表达更分散、多样化的交易偏好与审查标准，为协议提供高“偏好熵”。
            
        -   这类贡献可以通过多元奖励（如 multiplicity gadgets）转化为收入。
            

* * *

## 八、为什么要做这种分离？

-   减少对社会层与“道德约束”的依赖：
    
    -   目前在抑制质押集中化、大型 LST 垄断方面，很大程度依赖社区共识与软约束。
        
    -   eODS 试图在协议层内建模和约束这些结构，降低“全靠社会层兜底”的风险。
        
-   形式上“照抄现有市场结构”：
    
    -   eODS 本质上是在协议内正式承认现在已经存在的两类角色（delegators / operators），并给予更明确的职责与激励接口。
        
-   为 delegators 提供更强的共识参与形式：
    
    -   强化池内投票工具和治理机制，而不是仅依赖治理代币投票。
        
    -   通过协议层 enshrine delegations，让 delegators 既能作为 heavy 资本提供者，也能以 light 方式参与审查阻力与链头信号。
        

* * *

## 九、技术方向：减少 BLS 签名与与其他 R&D 的协同

-   在 SSF 场景中，单 slot 可处理的 BLS 签名上限约在 10 万到 180 万之间，因此需要减少“每 slot 必须签名的验证者数量”。
    
-   eODS 通过：
    
    -   把 “重服务” 验证者数量控制在 < 1 万；
        
    -   同时让更多参与者通过轻服务方式参与，从而在保证 SSF 的前提下降低聚合负担。
        
-   与执行票据（Execution Tickets）的协同：
    
    -   ET 将执行与共识分离，帮助实现 MVI，让重运营者更关注安全和稳定，而不用过度参与 “timing games”。
        
    -   eODS 在此基础上再细分 Operator / Delegator 的 stake 类型（包括借助 EIP-7251 的 MAX\_EFFECTIVE\_BALANCE 提升，在同一消息中区分不同角色的份额）。
        
-   与 IL / Multiplicity Gadgets 协同：
    
    -   Light operators 负责生成 Inclusion Lists / Multiplicity 集合，对执行负载施加审查阻力约束。
        
    -   Heavy operators 只负责以 Gasper 逻辑验证和最终确认符合规则的 payload，实现“生产者”与“执行约束者”的角色分离。
        

* * *

## 十、路线与实现思路（Road ahead）

-   垂直维度（Heavy–Light）：
    
    -   通过提高 MAX\_EFFECTIVE\_BALANCE，并引入余额阈值（例如 2048 ETH），根据余额决定验证者属于哪一复杂度层级。
        
-   水平维度（Operator–Delegator）：
    
    -   按文中 “层次分离” 的方案，进一步在协议状态中建模两种 stake 类型及其关系。
        
-   仍需进一步研究的方向：
    
    -   Heavy / Light 不同类型服务的 MVI 目标。
        
    -   eODS 在协议中的具体实现细节、接口与 AVS 插拔式扩展。
        
    -   到底哪些项目需要“入协议”（enshrine），哪些保留给用户层、应用层去创新。
        

* * *

# PeerDAS

## 基本概念

-   **PeerDAS** 全称 Peer Data Availability Sampling，是在 EIP-7594 中提出的一种以太坊网络层协议，用来更高效地分发和验证数据可用性。
    
-   主要服务对象是 L2（rollup 等）的数据 blob，目标是在不压垮节点的前提下，让这些数据长期“可拿得到、可验证”。
    

## 在以太坊路线图中的位置

-   以太坊需要通过扩容（Scaling）提高吞吐量、降低交易成本，同时不能牺牲去中心化和安全性。
    
-   PeerDAS 是 Danksharding 的关键组件，属于“Surge”阶段的一部分，配合 rollup 路线来提升 L2 的性价比。
    
-   EIP-4844（Proto-Danksharding）引入了更便宜的 **blobs**，后续在真正的 Danksharding 中，这些 blobs 会被转化为 **data columns**，由 PeerDAS 在网络中进行分片分发和采样。
    

## 分阶段路线（PeerDAS Roadmap）

-   Stage 0（EIP-4844）：已有 blob 子网分发，但没有 DAS，每个相关节点要下载全部 blob 数据。
    
-   Stage 1：1D PeerDAS，只做“水平扩展”，引入列（column）子网和采样，提高数据分发效率。
    
-   Stage 2：2D PeerDAS，增加“纵向扩展”，列变为二维结构，使用更轻量的 cell 级采样和重构，实现完整的 Danksharding。
    

## 数据分片与保管（Data Partitioning & Custody）

-   一个 blob 会被拆成很多更小的单位 **columns**，这些列是数据采样的“最小颗粒”。
    
-   网络会把节点分配到不同的 **custody groups（保管组）**，每组负责一批指定的列，分配是通过公开可验证的确定性函数完成（输入例如节点 ID），保证透明可复现。
    
-   每个节点要达到一个最小的“数据保管阈值”，以保证基础的数据可用性；存更多列、保存全部列的节点被称为 **super-nodes**，提供额外冗余与容错能力。
    

## 编码与分发（Encoding & Distribution）

-   blob 会用 **Reed–Solomon 纠删码** 编码：把原始数据拆成多列，并增加冗余的 parity 列，即使有部分列丢失也能恢复。
    
-   编码后的各列通过 gossip 协议在网络中扩散：出块者先把列发给一部分节点，这些节点再继续转发；节点会订阅和自己 custody group 对应的子网，减少无关流量。
    
-   如果某个节点在 gossip 中没收到某列，可以通过 request/response 协议主动向其他节点拉取缺失列。
    

## 数据可用性采样（DAS）

-   节点不会下载整个 blob，而是随机向其他节点请求部分列，用统计方法估计“整体数据是否可用”。
    
-   当采样成功率足够高时，可以以很高的概率认为整份数据是可用的；如果发现缺失列，则可进一步发起定向请求尝试恢复数据。
    

## 加密承诺与验证（KZG Commitments）

-   PeerDAS 使用 **KZG 承诺** 来保证数据完整性：出块者对完整数据做承诺，后续每个被采样的列都要能和该承诺验证匹配。
    
-   这样节点在仅仅下载少量列时，也能确认这些列确实来自同一份原始数据，防止数据被篡改或伪造。
    

## 重构与冗余管理（Reconstruction & Redundancy）

-   节点会持续对自己保管的列做采样，如果某个节点拿到了超过 50% 的列，就可以用 Reed–Solomon 解码重构整个 blob。
    
-   一旦重构完成，这个节点会把恢复的列重新分发到网络中，提升整体数据冗余度，抵抗子网故障或临时数据缺失。
    

## 验证者协议与分叉选择规则

-   验证者在共识过程中会根据数据可用性调整 fork-choice 规则：对于新块，主要根据 gossip 子网收到的列来判断；对于较老的块，则根据之前 DAS 的结果判断数据是否仍然可用。
    
-   验证者只对“数据可验证且可获取”的区块投票，从而降低临时数据隐藏攻击的风险，保障链的安全性和完整性。
    

## 总体设计要点

-   PeerDAS 结合了：确定性的 custody 分配、概率型的数据可用性采样、纠删码冗余、KZG 加密承诺等机制。
    
-   目标是在去中心化网络中，实现可扩展、容错、且安全的数据可用性层，即便在对抗性环境下也能保证数据可验证、可恢复。
    

* * *

# FCR

## **FCR 是什么**

-   Fast Confirmation Rule（FCR）是以太坊的一种算法，用来判断一个区块在好的网络条件下是否“永远不会离开规范链（canonical chain）”。
    
-   它只输出两个结果：区块“已确认”或“未确认”，目标是在最终确定性（finalization）之前给出**快速且安全**的确认信号。
    

## **为什么需要 FCR**

-   目前协议里的确认规则只有 Gasper 中的 FFG Finalization，优点是非常安全、可以在异步网络下工作，但确认太慢：最好约 13 分钟，平均约 16 分钟。
    
-   最终确定的区块几乎不会被回滚，但对很多场景（买咖啡、CEX 入金等）体验太差；很多钱包把“进块”当“确认”是不安全的。
    
-   现有靠“区块深度”或“justified 状态”的启发式算法都不能满足严格安全性要求，因此需要一个有形式化安全保证的快速确认规则。
    

## **FCR 提供的保证与性质**

-   FCR 假设网络**同步**（诚实验证者在一个 slot 内可以互相收到对方的证明），并假设每轮委员会中恶意质押比例不超过 β（大约 20–25%，可配置）。
    
-   在这些假设下，FCR 的最佳确认时间可以做到 1–2 个 slot（约 12 秒级），远快于 FFG 的十几分钟。
    
-   关键性质：
    
    -   安全性：一旦被 FCR 确认，区块在诚实视角下不会被重组（reorg）。
        
    -   单调性：确认不会“往回退”，除非触发“reset-to-finalized”来表示假设失效（例如网络不再同步）。
        

## **与 Gasper 的关系（背景）**

-   以太坊 PoS 共识由 Gasper 定义：时间单位 slot 为 12 秒，一个 epoch 由 32 个 slot 组成，每个 epoch 把验证者分成 32 个委员会。
    
-   Gasper 包含两个子协议：
    
    -   LMD-GHOST：分叉选择算法，用来选择规范链头。
        
    -   FFG-Casper：在 LMD-GHOST 选出的链上做 checkpoint 最终确定。
        
-   出块流程：每个 slot 随机选一个 proposer 在规范链头上出块，其他验证者对其进行 attest，之后 fork choice 决定新的链头。
    

## **FCR 算法大致流程**

-   输入：前一次已确认的区块（`store.confirmed_block`）；目标是沿当前规范链往前“推进确认点”。
    
-   核心函数是 `get_latest_confirmed`，分两个阶段：
    
    -   假设检查：确认网络同步性和 β 等假设仍然成立。
        
    -   确认推进：在规范链上查找最新的、可以被视为“已确认”的后代区块。
        
-   算法使用一个 `isOneConfirmed` 判定：
    
    -   从之前已确认区块开始，对规范链的后缀逐块检查 `isOneConfirmed` 是否为真。
        
    -   若为真，说明该块获得了足够的 LMD-GHOST 支持，可以战胜任何可能的兄弟分支（考虑 proposer boost、对手预算 β、双签、空 slot 折扣等因素）。
        
    -   一旦某块不再满足这一条件，算法就停止向前确认。
        
-   复杂度主要来源于要严密处理各种边界情况，确保在各种极端场景下仍保持安全性。
    

## **实际影响与应用场景**

-   改善钱包 UX：钱包可以基于 FCR 而不是简单“进块即确认”，给用户更可信的“已确认”标记。
    
-   减少 CEX 风险：交易所可以用 FCR 降低充值交易被回滚的概率，在不等最终确定的前提下提高体验。
    
-   在 PBS 等场景下，也可以用 FCR 作为更快速、安全的确认信号，帮助相关协议做决策。
    
-   在 Single Slot Finality（SSF）真正上线之前，FCR 填补了以太坊确认模型中“太慢 vs 太不安全”之间的空缺。
<!-- DAILY_CHECKIN_2026-04-24_END -->

# 2026-04-23
<!-- DAILY_CHECKIN_2026-04-23_START -->



# PBS

网址：[epf](https://epf.wiki/#/wiki/research/PBS/pbs)

* * *

## 1\. PBS 是什么

-   传统以太坊：同一个验证者既“构建”区块，又“提议/广播”区块。
    
-   **PBS（Proposer-Builder Separation）**：把“构建区块”（builder）和“提议区块”（proposer）拆开，由专门的 block builder 负责打包交易并竞价，proposer 只选最赚钱的区块并广播，且看不到区块内容。
    
-   目前 PBS 主流实现是协议外的 MEV-Boost（依赖中介 relay），还没有直接写进以太坊协议本身。
    

* * *

## 2\. 为什么需要 PBS

-   降低成为验证者的算力/带宽门槛，有利于更多普通节点参与，从而提升去中心化。
    
-   网络向“模块化”演化：把区块构建的不同环节拆开，各个角色各司其职，提高整体效率与灵活性。
    
-   通过专业化 builder 提升区块利用率（更高 MEV、更好 gas 利用），同时验证者只需做轻量工作。
    

* * *

## 3\. 与共识层的关系

-   共识层的基本结构：
    
    -   每个 slot 12 秒，每个 epoch 32 个 slot。
        
    -   每个 slot 由 RANDAO 随机选出一个验证者提议区块，委员会验证并最终完成 finality。
        
-   PBS 在这个流程中：
    
    -   保留“每个 slot 有一个 proposer”的结构。
        
    -   但 proposer 不再亲自从 mempool 选交易，而是从多个 builder 提供的区块中选一个最优区块。
        

* * *

## 4\. 各角色职责

-   Searcher
    
    -   在公共 mempool 中寻找 MEV 机会（抢跑、三明治、套利等），把一串有顺序的 tx 组装成 bundle，并附带对 builder 的出价。
        
-   Builder
    
    -   收集交易和 searcher 的 bundle，根据 gas、MEV 等排序，构建“区块 body”，对 proposer 出价。
        
    -   不直接上链，而是把区块发给 relay。
        
-   Relay
    
    -   接收多个 builder 的区块，做基本校验后，挑出出价最高的区块发给 proposer/验证者签名。
        
    -   实际上是 builder 与 proposer 之间的中介层，当前少数 relay 控制了绝大部分 MEV-Boost 流量，引发中心化和审查担忧。
        
-   Validator / Proposer
    
    -   在 PBS 下，验证者承担 proposer 角色：从 relay 那里收到候选区块，选报酬最高的并签名广播。
        
    -   依旧负责维护网络安全与共识。外部 PBS 失败时，可以退回传统“自己打包交易”的模式。
        

* * *

## 5\. 当前状态与问题

-   当前主网：PBS 主要通过 MEV-Boost 以协议外方式运行，依赖少数 relay 和 builder，带来：
    
    -   中心化风险：极少数 relay 处理了绝大多数区块。
        
    -   审查风险：relay 可以被监管要求过滤某些地址（如 OFAC 列表）。
        
    -   信任问题：validator 要相信 relay 不偷 MEV，builder 要相信 relay 不作恶。
        
-   第三方依赖：
    
    -   如 bloXroute 事件中，外部 BDN 只传 block 不传 blob，引发 missed slots，暴露对外部服务的操作性依赖问题。
        

* * *

## 6\. 安全和审查相关担忧

-   多角色引入更多攻击面：relay、escrow、builder 故障都可能导致 missed blocks（性能受影响），虽不直接破坏共识安全，但会影响用户和验证者收益。
    
-   审查抵抗被削弱：
    
    -   builder/relay 集中时，理论上可以合谋排除某些交易，只是目前更像“延迟确认”而非完全阻止提交。
        
    -   社区在研究匿名 block proposal、强制包含某些交易的承诺等机制，提高抗审查性。
        

* * *

## 7\. 研究方向与方案

主要三条线：

1.  **ePBS（Enshrined PBS）**
    
    -   把 PBS 逻辑直接写进以太坊共识层，减轻对外部 relay/软件的依赖，降低中心化和单点故障风险。
        
    -   研究包括 TBHL（Two-Block HeadLock）、乐观中继（optimistic relaying）等设计。
        
2.  **PEPC（Protocol-Enforced Proposer Commitments）**
    
    -   把“proposer 对外包任务的承诺”写进协议：
        
        -   proposer 可以通过 EVM 形式注册各种承诺（比如：必须包含某些 tx、必须按照某种规则构建区块等）。
            
        -   区块只有满足已注册承诺才被视为有效，从“乐观相信”转为“违反就无效区块”的悲观强制模式。
            
    -   支持从“完全外包构建”到“只强制包含关键交易”等多种外包合约形式，并可与 EigenLayer 等机制互补。
        
3.  **EIP-7547 Inclusion Lists**
    
    -   允许 proposer 指定一组必须在后续块中尽快被包含的交易，否则后续区块视为无效，从协议层提升抗审查性。
        
    -   存在激励兼容性、数据暴露等问题，社区提出 forward inclusion lists、multiple inclusion lists 等变体在优化设计。
<!-- DAILY_CHECKIN_2026-04-23_END -->

# 2026-04-22
<!-- DAILY_CHECKIN_2026-04-22_START -->




# 最大可提取价值

## MEV 基本概念

-   **MEV 全称**：Maximal Extractable Value，以前叫 Miner Extractable Value。
    
-   含义：区块生产者通过「有策略地排序、包含或排除交易」在一个区块里，能从标准区块奖励 + Gas 费用之外额外提取的最大价值。
    
-   主要发生场景：以太坊上的 DeFi 协议中。
    
-   典型策略：前置交易（front‑running）、三明治攻击（sandwiching）、后置交易（back‑running）等。
    

* * *

## MEV 的影响与问题

-   为大型资金池或专业参与者带来不公平优势（信息和执行优势）。
    
-   可能导致对某些交易或用户的审查（censorship）。
    
-   DeFi 用户在交易时可能遭遇更大滑点（slippage 升高）。
    

* * *

## PBS 与 MEV 的关系

-   Proposer‑Builder Separation（PBS，提议者‑构建者分离）会改变 MEV 的分配方式。
    
-   在 PBS 模型中：
    
    -   区块构建者（builder）负责决定交易的排序与包含策略。
        
    -   区块提议者（proposer，现为验证者）从不同 builder 之间的竞争中获得收益。
        
-   结果：
    
    -   MEV 在 proposer 与 builder 之间重新分配。
        
    -   更激烈和透明的竞争有机会带来更高效率和更公平的 MEV 分布。
        

* * *

## MEV 的演变历史

-   PoW 时代称为「Miner Extractable Value」，因为矿工控制区块交易顺序。
    
-   以太坊合并（The Merge）后：
    
    -   共识从 PoW 转为 PoS，矿工角色消失，由验证者负责共识。
        
    -   术语改为「Maximal Extractable Value」，以适配新的角色结构。
        
-   早期 MEV 主要通过公共 mempool 抢 Gas 竞争：
    
    -   被称为 Priority Gas Auction（PGA）时代，参与者通过不断提高 Gas 价格来抢先打包交易，环境非常混乱。
        
    -   相关经典论文：Flashboys 2.0。
        
-   Flashbots 的出现：
    
    -   作为一个开放的研发计划，试图改进 MEV 工具的公共知识与可访问性，降低「黑箱」程度。
        

* * *

## 合并后：验证者、MEV‑boost 与 PBS 实现

-   合并之后：
    
    -   传统意义上的矿工不再存在，但「构建者 + 提议者」功能由验证者参与的体系来承担。
        
-   为应对合并后的新结构，Flashbots 联合客户端团队与以太坊基金会开发了 **mev‑boost**。
    
-   mev‑boost：
    
    -   是一种「协议外」（out‑of‑protocol）的 PBS 实现，不直接写进主协议逻辑里。
        
    -   让验证者可以从多个区块构建者那里竞价选择区块，从而提取 MEV。
        

* * *

# 提案者与构建者分离 **Proposer‑Builder Separation**

## PBS 是什么

-   PBS 全称：**Proposer‑Builder Separation**，提案者与构建者分离。
    
-   传统模式：同一个验证者既负责打包交易（建块），又负责广播区块。
    
-   PBS 模式：
    
    -   Builder 负责收集、排序交易并构建区块主体（block body）。
        
    -   Proposer 只在每个 slot 中，从多个 builder 提供的区块中挑选「最赚钱的一块」，付费后广播，但看不到区块具体内容（只见 header / 承诺）。
        

* * *

## PBS 为什么重要

-   降低成为验证者的计算门槛，家用机 / 轻节点更容易参与质押，提高去中心化程度。
    
-   让不同角色各做擅长的事：builder 专注高性能建块和 MEV，proposer 专注共识与安全。
    
-   符合以太坊向「模块化」演进的总体路线（PoS + 分层架构）。
    

* * *

## 在共识层中的工作方式

-   共识基础：
    
    -   每个 epoch 有 32 个 slot，每个 slot 12 秒。
        
    -   通过 RANDAO 随机选出一个验证者担任该 slot 的 proposer。
        
-   正常流程：
    
    -   proposer 提议区块，委员会中的验证者对其进行 attestation，最终达到 finality。
        
-   PBS 在里面做的事：
    
    -   把「建块」和「提议」两项职责隔离，简化 proposer 的工作，只做「选择 +签名」。
        

* * *

## 各角色职责

-   Searcher（搜寻者）
    
    -   在公共 mempool 里找 MEV 机会（套利 / 前置 / 三明治等）。
        
    -   构造一个「交易序列 bundle」，里面是执行某 MEV 策略的一组交易 + 对 builder 的出价。
        
-   Builder（构建者）
    
    -   收集 mempool 交易和 searcher 的 bundles。
        
    -   验证、排序、打包成区块主体，尽量最大化 MEV + gas 收益。
        
    -   把区块和出价提交给 relay，而不是直接上链。
        
-   Relay（中继）
    
    -   接收多个 builders 的区块，验证合法性，选出出价最高的那一个。
        
    -   把「最高出价 + 区块头承诺」发给 proposer。
        
-   Validator / Proposer（验证者 / 提议者）
    
    -   从 relay 收到的候选区块中选择收益最高的一个，签名并广播。
        
    -   仍负责网络安全与共识，只是把建块外包出去。
        

* * *

## 当前状态：MEV‑Boost 与中心化风险

-   目前主网没有「协议内」PBS，链上仍是验证者自己建块 + 提议。
    
-   实际上 PBS 主要通过 **MEV‑Boost + 中继网络** 以「协议外」方式存在。
    
-   问题：
    
    -   极少数 relay 处理了绝大部分 MEV‑Boost 区块，带来中心化和审查担忧。
        
    -   OFAC 制裁名单相关交易曾被部分中继/构建者审查，证明审查风险是真实存在的。
        
    -   使用外部中继（如 bloXroute）的客户端曾导致大量 missed slots，显示第三方依赖的脆弱性。
        

* * *

## 主要风险与安全问题

-   Relay 相关风险：
    
    -   去中心化受损：少数 relay 控制 90% 左右的区块来源。
        
    -   审查风险：relay 可被监管强迫拒绝某些交易。
        
    -   信任问题：validator 信任 relay 按约定发布完整区块，builder 信任 relay 不「偷 MEV」。
        
-   第三方依赖：
    
    -   链上共识依赖链下服务（relay/BDN），形成单点故障、运维依赖。
        
    -   实例：3 月因 bloXroute blob 传播问题 + Lighthouse 行为假设不匹配，出现一波 slot miss。
        
-   安全向量：
    
    -   角色变多 ⇒ 攻击面增大（故障中继、escrow 问题等）。
        
    -   好处是：如果外部 builder 挂掉，客户端仍可 fallback 回本地建块，链整体安全性不至于立刻崩溃。
        
-   审查阻力减弱：
    
    -   大型 builder / relay 集中，理论上可合谋延迟或排除特定交易。
        
    -   现状：即使大部分参与者选择审查，仍无法完全禁止交易上链，只能拖慢确认时间。
        
    -   研究方向：匿名区块提议、强制包含承诺（inclusion commitments）等。
        

* * *

## 研究方向与改进方案

## ePBS（协议内 PBS）

-   目标：把 PBS 直接写进以太坊共识层，从根本上减少对外部 relay/软件的依赖。
    
-   潜在好处：
    
    -   降低中心化和第三方信任问题，更符合去中心化与抗审查的价值观。
        
    -   减少外挂软件复杂度与兼容性成本（避免再出现类似「Low‑Carb Crusader」类事件）。
        
-   关键思路涉及：TBHL（Two‑Block HeadLock）、乐观中继（optimistic relaying）等机制。
    

## PEPC（Protocol‑Enforced Proposer Commitments）

-   工程目标：给 proposer 一个通用框架，可以对「外包建块任务」做**可执行、可强制的承诺**。
    
-   特点：
    
    -   承诺逻辑用 EVM 表达，灵活支持：整块外包、强制包含某些交易、rollup 证明检查等多种模式。
        
    -   把「承诺是否被满足」变成协议级条件，没满足就视为区块无效，比 MEV‑Boost 这类外协议方案安全性更高。
        
    -   可与 EigenLayer 等机制互补，从「乐观假设别人不会作恶」转向「违约就直接无效」。
        

## EIP‑7547 Inclusion Lists（包含列表）

-   目的：提高抗审查能力。
    
-   机制大致是：
    
    -   proposer 可以公布一份「必须在后续区块中被尽快打包的交易列表」。
        
    -   builder 若想让自己构建的区块被选中，就必须保证这些交易被包含。
        
-   挑战：存在激励不兼容、信息泄露等问题，正在通过「前向包含列表（forward lists）」「多列表」等方案迭代设计。
<!-- DAILY_CHECKIN_2026-04-22_END -->

# 2026-04-21
<!-- DAILY_CHECKIN_2026-04-21_START -->





## 以太坊协议路线图

网址：[epf](https://epf.wiki/#/wiki/research/roadmap)

## 1\. 总体

-   哲学：持续迭代升级，为显著收益接受有限协议风险。
    
-   协议 R&D 完全公开，任何人都可以学习、讨论和参与贡献。
    
-   当前常用参考：Vitalik 2023‑12 路线图图表，把路线分为多个 “urge”。
    

* * *

## 2\. 六大方向概览（“The _urge_s”）

-   The Merge：从 PoW 到 PoS、改进共识层。
    
-   The Surge：扩容与数据可用性（Rollup + Danksharding）。
    
-   The Scourge：处理 MEV、审查风险、质押集中化问题。
    
-   The Verge：Verkle 树、简洁证明、轻客户端与无状态化。
    
-   The Purge：历史/状态过期，降低节点存储负担。
    
-   The Splurge：账户抽象、UX 改进、形式化验证、量子安全等“杂项增强”。
    

* * *

## 3\. The Merge（共识层 PoS）

已完成：

-   Beacon Chain 上线（EIP‑2982）：引入 PoS 验证者机制。
    
-   Execution Layer 与 Beacon Chain 合并：以太坊主网转为 PoS，终止 PoW。
    
-   提款功能（EIP‑4895）：质押 ETH 可提款（Shanghai/Capella 升级）。
    

进行中 / 研究中：

-   单槽终局性（Single Slot Finality，SSF）。
    
-   单秘密出块者选举（SSLE）等改进提议。
    
-   提高最大验证者数量。
    
-   共识层使用量子安全签名方案的探索。
    

* * *

## 4\. The Surge（扩容与数据分片）

目标：

-   通过 Rollup + 数据可用性扩容，提高 TPS，降低 L2 成本。
    
-   实现完整 Danksharding 和数据可用性抽样（DAS）。
    

已完成：

-   Proto‑danksharding（EIP‑4844）：
    
    -   引入 blob 数据，用于存 rollup 数据且只短期保留。
        
    -   显著降低 Rollup 费用，为完整 Danksharding 铺路。
        

未完成 / 研究方向：

-   完整 Danksharding 设计与实现。
    
-   数据可用性抽样（DAS）。
    
-   移除 rollup “辅助轮”（如更强安全假设下的依赖）。
    
-   量子安全承诺方案等。
    

* * *

## 5\. The Scourge（MEV、审查与去中心化）

问题背景：

-   MEV 放大了审查、中心化和复杂激励问题。
    
-   MEV / PBS 设计直接影响质押去中心化和网络公平性。
    

当前实践：

-   MEV‑Boost 已在生产环境广泛使用，作为临时 PBS 解决方案。
    

主要研究方向（未定稿）：

-   ePBS（enshrined PBS）：协议原生 PBS 设计。
    
-   MEV‑Burn：尝试把 MEV 收入回收给协议 / 抵消通胀。
    
-   Inclusion lists：保护被审查交易获得打包机会。
    
-   分布式区块构建（去中心化 builder）。
    
-   更好的激励设计、防止寡头化和串谋。
    

* * *

## 6\. The Verge（Verkle、轻客户端与无状态化）

目标：

-   让轻客户端和无状态验证成为常态。
    
-   大幅降低验证账本状态所需数据量。
    

关键技术方向：

-   Verkle 树：
    
    -   用向量承诺替代 Merkle Patricia Trie。
        
    -   支持更小的证明，方便轻客户端。
        
-   与 DAS、历史/状态过期等配合，实现更“无状态”的节点形态。
    

预期效果：

-   轻节点更容易运行，客户端种类更多样。
    
-   全节点同步和验证成本降低。
    

* * *

## 7\. The Purge（历史与状态清理）

核心目标：

-   控制协议复杂度和存储膨胀。
    
-   降低运行全节点的硬件门槛。
    

关键点：

-   History Expiry：历史数据定期过期，只保留近期必要数据。
    
-   State Expiry：长期不活跃状态逐步“冷冻”或迁移，减轻活跃状态负担。
    

预期效果：

-   活跃状态规模控制在几十 GB 量级。
    
-   节点可以更快同步，普通硬件即可运行全节点。
    

* * *

## 8\. The Splurge（UX、安全与未来准备）

范围很杂，但围绕“提升体验 & 长期安全”。

代表性方向：

-   账户抽象（AA）：
    
    -   EIP‑4337 已在生产使用（入口合约 + Bundler 模式）。
        
    -   支持社交恢复、付费代付（sponsored tx）、批处理操作等。
        
-   量子安全：
    
    -   为后量子时代做签名、承诺方案研究。
        
-   形式化验证与工具链：
    
    -   对核心协议和客户端实现进行形式化建模与验证。
        
    -   提升实现正确性，减少共识漏洞风险。
        

* * *

# 区块链可扩展性

## 1\. 什么是区块链可扩展性

-   可扩展性 = 在不显著增加节点负担的前提下，处理更多交易的能力。
    
-   区块链的核心任务：处理用户交易、维护全网状态，随着用户量和交易需求增加，系统要能顶住负载。
    
-   目标表达：能在**不提高节点门槛**的前提下提升 TPS。
    

* * *

## 2\. Scalability Limits：有哪些“硬约”

-   两个关键参数：
    
    -   Block latency：出块和确认所需时间（以太坊规范是每 slot 12 秒，但现实会因为漏块稍有偏差）。
        
    -   Block size：单个区块能容纳的数据 / 计算量（以太坊用 gas\_limit 来限制）。
        
-   EIP‑1559：通过 base fee + gas\_target 模型动态调整区块大小，硬上限 30M gas，目标 15M gas；限制了单块可包含交易数和复杂度。
    
-   TPS 由出块间隔和单块容量共同决定，高性能、大区块、低延迟的链往往意味着节点要求高、中心化严重。
    

## 状态大小与数据可用性

-   Data availability：新节点要能依靠链上数据、在无需信任的情况下重建最新状态。
    
-   状态和历史数据太大 ⇒ 节点存储开销大、同步慢、对带宽要求高，不利于去中心化。
    
-   为抑制状态膨胀，以太坊让读写状态操作变得很贵，从经济上约束无节制扩张。
    

## 去中心化的约束

-   共识设计、节点硬件要求、状态结构与大小都直接影响去中心化程度。
    
-   硬件/带宽要求越高，可运行节点的人越少，去中心化越差。
    
-   关键原则：让运行节点尽可能**便宜、可行、被鼓励**，保证一个足够大的、分布广泛的节点集合。
    

## 手续费与区块链不可能三角

-   用户需要付费上链（base fee + gas 使用量 + priority fee）。
    
-   在高去中心化 + 固定 TPS 情况下，一旦需求升高，手续费会被推高到影响可用性。
    
-   区块链三难困境：安全性、去中心化、可扩展性难以同时最大化；经典公链在“纯 L1 扩 TPS”路径上不可扩展。
    

* * *

## 3\. Blockchain Modularity：模块化区块链

-   思路：拆分系统职能，用“分而治之”的方式分别优化三难困境的不同维度。
    
-   典型功能模块：
    
    -   Consensus：节点如何就区块及交易排序达成一致。
        
    -   Execution：如何执行交易、推进状态。
        
    -   Data availability：如何确保数据在需要时可被获取。
        
    -   Settlement：如何保证最终性与不可变性。
        
-   模块化好处：整体复杂度下降，各模块可独立演进、升级新的扩容方案。
    

* * *

## 4\. Scaling Ethereum：L1 与 L2 路径

## L1 扩容

-   目标：直接提高底层协议的 TPS。
    
-   “天真方案”：加大区块容量（更高 gas\_limit、更便宜操作），在给定时间内容纳更多交易。
    
-   问题：
    
    -   强行增大区块 + 降低 gas 价格，会提高节点硬件/带宽要求，减少可参与节点。
        
    -   结果是中心化加剧。
        
-   因此：L1 提升能力必须配合额外的设计（如分片、PoS、协议优化）以维持去中心化。
    
-   L1 扩容例子：分片、PoS 共识、提升执行效率的协议升级等。
    

## L2 扩容

-   定义：在 L1 之上构建系统，提高 TPS，但不显著增加 L1 资源消耗，同时继承 L1 安全性。
    
-   方式：
    
    -   在链外或其他更快共识环境中执行交易。
        
    -   依赖 L1 提供最终结算和安全保证。
        
-   典型方案：状态通道、Plasma、Rollups 等。
    
-   以太坊的主流答案：**多 Rollup‑centric 路线**。
    
    -   以太坊主链主要负责 Settlement + Data Availability。
        
    -   绝大多数 Execution 迁移到 L2 Rollup。
        
    -   Rollup 把大量 L2 交易压缩成少量 L1 交易，L1 只存原始数据和 L2 状态承诺（commitment）。
        
    -   好处：L1 保持去中心化与安全性，L2 获得大幅扩容空间。
        

* * *

## 5\. Rollups：工作机制与 EIP‑4844

-   Rollup = 压缩执行，把大部分计算挪到 L2，在 L1 上只保留必要数据、状态承诺。
    
-   安全性：L2 交易数据以及 L2 状态转换承诺写入 L1，使其最终性跟随 L1；一旦写入，就不能回滚。
    
-   之前：Rollup 数据放在 Ethereum 交易的 calldata 中，导致历史存储膨胀。
    
-   EIP‑4844（proto‑danksharding）：
    
    -   引入 blob 数据空间，rollup 把数据发到 blobs，而不是 calldata。
        
    -   blobs 有单独的 gas 模型，更便宜也更可控。
        
    -   共识节点对 blob 数据的处理逻辑略有调整，为未来 danksharding 铺路。
        

* * *

## 6\. L2 类型：Optimistic vs ZK Rollups

## Optimistic Rollups

-   模型：**乐观验证**，默认假设批次交易有效。
    
-   流程：
    
    -   Operator/Aggregator 把 L2 交易打包提交到 L1（anchoring）。
        
    -   随后有一个挑战期，任何人可以对批次提出 fraud proof。
        
    -   挑战成功：挑战者获奖，提交者被罚没（slashing）。
        
-   安全性来自加密经济激励：做坏事会被罚，诚实挑战有收益。
    

## ZK Rollups

-   模型：用零知识证明对批次交易正确性提供**数学证明**。
    
-   流程：
    
    -   L2 将大量交易打包。
        
    -   生成一个 succinct 的零知识证明，提交至 L1。
        
    -   L1 验证该证明，一旦通过，可以确信所有交易有效。
        
-   核心优势：**简洁性（succinctness）**，证明远小于所有交易本身。
    
-   常见证明系统：
    
    -   zk‑SNARKs：目前应用更广，需可信设置。
        
    -   zk‑STARKs：不需要可信设置，可扩展性强，应用正在增长。
        

* * *

## 7\. 以太坊在扩容上的整体定位

-   L1：强调安全性与去中心化，负责共识、结算与数据可用性；通过 gas 限制控制资源消耗、防止节点门槛过高。
    
-   L2：承担主执行负载，通过 rollup 等方案提供更高 TPS；仍继承 L1 的安全性和最终性。
    
-   配套核心升级：EIP‑1559、EIP‑4844、未来 danksharding 等，为 rollup‑centric 路线提供基础设施。
<!-- DAILY_CHECKIN_2026-04-21_END -->

# 2026-04-20
<!-- DAILY_CHECKIN_2026-04-20_START -->






# 核心协议开发

网址：[epf](https://epf.wiki/#/wiki/dev/core-development)

核心协议开发（Core development）指的是围绕以太坊协议的规范、实现和测试所做的所有基础性工作。

## 核心开发在做什么

-   核心开发的范围包括：协议规范的编写与更新、客户端软件的开发与维护、协议实现的验证、以及支撑这些工作的工具链建设。
    
-   这些工作覆盖 EPF Wiki 中列出的各个领域：执行层（Execution Layer）、共识层（Consensus Layer）、测试与安全、研究、密码学等。
    
-   面向外部开发者的所有基础设施（节点软件、同步与调试工具、协议参考实现）都属于这条“核心协议开发”链路的一部分。
    

## 客户端与团队生态

-   以太坊有多种共识层客户端和执行层客户端，由不同的独立团队开发和维护。
    
-   不同客户端在编程语言、实现细节、特性和性能侧重上各不相同，但都遵循同一套协议规范。
    
-   除了客户端团队，还有专门做研究、测试和基础设施的团队，共同维护所有在以太坊之上构建应用所需的“底层技术栈”。
    

## 客户端多样性与去中心化

-   客户端多样性（client diversity）是以太坊刻意坚持的一项核心原则：同一协议由多种独立实现共同运行。
    
-   多样化的客户端可以提升网络的稳定性与安全性，减少单一实现出现 bug 或被攻击时对全网的影响。
    
-   这种多样性也有助于保持核心开发的去中心化，使协议演化不被单一组织或代码库锁死。
    

## 协作方式与日常节奏

-   虽然客户端团队分散在不同公司、组织或非营利机构中，但核心开发者在实际工作中高度协作，共同推动网络升级和改进。
    
-   主要协作形式是公开的周期性会议，包括执行层和共识层的 All Core Devs（ACD）电话会议，以及各种专题 “Breakout Room” 和工作组电话会议。
    
-   这些会议的议程与记录托管在以太坊项目管理仓库（ethereum/pm），方便任何人跟进改动与决策过程。
    
-   会外讨论主要集中在 Ethereum R&D Discord、Eth Magicians 论坛和 EthResearch 论坛等公开渠道，降低“圈内知识”的门槛。
    

## “太空站”隐喻与核心开发者角色

-   Hsiao‑Wei Wang 在 2023 年 ETHGlobal Tokyo 的演讲 “A Journey of an Ethereum Core Dev/Researcher” 中，用一张图将以太坊比作空间站，展示不同团队如何协作让“空间站”运转起来。
    
-   这个隐喻强调：核心开发是长期的系统工程，需要规范作者、客户端实现者、研究人员、测试与基础设施团队等角色的持续配合。
<!-- DAILY_CHECKIN_2026-04-20_END -->

# 2026-04-18
<!-- DAILY_CHECKIN_2026-04-18_START -->







# 弱主观

## 1\. 背景：从“客观”到“弱主观”

\- 概念区分：如果一段信息的正确性可以完全验证，就是“客观”；如果验证中不可避免要依赖信任和主观判断，就是“主观”，大多数现实信息都在两者之间的光谱上。

\- PoW 时代：从创世块开始，客户端可以请求最新“最终”区块，然后沿着 parent hash 一路回溯到 genesis，逐块验证工作量与链结构，几乎完全客观地重建历史（仅需弱信任创世块本身）。

\- 攻击成本：要篡改历史需要“不可思议的算力”，所以一旦从 genesis 重放完成，历史在实践中被视为客观；通常将距离头部 6 个块之后的块视为“final”。

Merge 后：

\- 引入 PoS、验证者身份和投票机制，共识层（CL）与执行层（EL）逻辑上解耦：

\- CL 负责谁提块、谁投票、哪个块最终。

\- EL 负责执行交易、验证执行结果。

\- 结果：\*\*从 genesis 做全历史同步在 PoS 下不再“安全”\*\*，需要新的“弱主观”同步模型。

\---

## 2\. 弱主观性（Weak Subjectivity）的直觉

### 2.1 退出验证者与“重写过去”的可能性

\- 在 PoS 中，验证者可以退出：退出后不能再对未来区块投票，但理论上仍可以对\*\*过去\*\*的区块重新投票（在某些客户端视角中）。

\- 如果大量已退出验证者对某条过去的分叉链重新投票，就能制造一条“替代历史”：

\- 已经在线并跟随规范链的节点不会被迷惑，因为它们在本地状态中已经处理过那些 exits，知道这些验证者已不再有权投票。

\- 但一个正在同步的新节点是“从过去往现在走”的，在它眼里，那些过去的投票看起来仍然有效，可能因此跟随错误分叉。

### 2.2 同步方向改变：从 genesis → 从“锚点”

为避免上述攻击，Beacon 链的同步方向与执行层不同：

\- 执行层仍然可以从创世向前重放。

\- Beacon 链同步改为“从一个已知的、可信的最终 checkpoint 向两头延展”，而不是从 genesis 向前。

这个可信的最终 checkpoint 就是 **弱主观性检查点（weak subjectivity checkpoint）**：

\- 它是一个已经 Finalized 的 checkpoint，但其“处于规范链中的那个事实”无法通过纯协议完全客观验证（因为存在上面提到的“退出者重投票”攻击空间）。

\- 因此，需要“带一点主观信任” —— 例如通过可信渠道获得该 checkpoint。

### 2.3 信任锚：为什么叫“弱主观”

\- 完全主观：完全靠别人告诉你当前状态，无法自己验证。

\- 弱主观：

\- 你信任一个有限的信息（某个 final checkpoint）是正确的；

\- 之后从这个点向前 / 向后走的所有状态都可以通过协议客观验证。

\- 所以：\*\*弱主观性 = 需要少量、有限的主观信任，再加上大量的客观可验证性\*\*。

\---

## 3\. 弱主观周期（Weak Subjectivity Period）

\- 概念：一个节点如果离线时间太长，在重新上线同步时，就有可能被上面的“退出者重写过去”的攻击迷惑。允许发生这种攻击的最小时间长度就是 **弱主观周期**。

\- 直观理解：

\- 只要系统中有足够数量的验证者可以退出并重新加入，就可以积累足够退出者在过去制造“替代历史”；

\- 如果你离线时间长到足以让攻击者完成这个过程，那么你重新上线时就无法仅靠协议客观地区分哪条是“真历史”。

\- 同样，这个周期也约束弱主观性检查点本身：

\- 如果你的 weak subjectivity checkpoint 太旧（超过该周期），攻击者就可以围绕它构造替代历史。

\- 因此，客户端需要定期更新自己的 weak subjectivity checkpoint。

\---

## 4\. 乐观同步（Optimistic Sync）

\- 在弱主观模型下，同步需要分开处理：

\- Beacon 链回填（backfill）：从 weak subjectivity checkpoint 向 genesis 回溯历史块。

\- 执行层同步：对照不断前进的头部，执行 payload 并追上最新状态。

\- 为了不让 CL 同步阻塞 EL，同步过程引入 **optimistic importing**：

\- Beacon 节点在 EL 还没完全验证 payload 的情况下，先“乐观地”接受新的 beacon blocks，继续导入更新 head。

\- 等 EL 完成自己的同步和 payload 验证后，再将这些 slot 标记为彻底验证完成。

这意味着：

\- 在同步过程中，短时间内 head 对执行结果的“已验证”状态是乐观的；

\- 一旦 EL 同步结束并回头验证完所有乐观导入的块，节点才算真正 fully synced。

\---

## 5\. 弱主观同步的标准步骤

页面给出的弱主观同步流程可以总结为五步：

1\. **通过链外渠道获取一个弱主观性检查点**

\- 例如：从客户端团队、可信基础设施提供商、你自己的其他节点、或社区广泛认可的来源获得一个近期 finalized checkpoint。

\- 这是唯一的“主观信任输入”。

2\. **从该检查点向后回填 Beacon 区块直到 genesis**

\- 通过 p2p 从其他节点获取 checkpoint 之前的区块，一路沿 parent hash 向后验证。

\- 只要检查点本身是正确的，后向历史就可以被客观校验。

3\. **把该检查点对应的执行层头部设置为 EL 的同步目标**

\- Beacon block 中含有 execution payload header，它指示执行层应该把本地头对齐到哪个执行块。

4\. **乐观地跟随链头，持续更新执行层目标**

\- 在 EL 仍在同步的阶段，CL 会持续从网络导入新 blocks / finalized checkpoints，并将对应的 execution heads 提供给 EL。

\- 导入这些 block 的同时，不要求立即验证其 execution payload。

5\. **EL 完成同步后，对此前乐观导入的 slots 进行执行验证并标记为“已验证”**

\- 一旦执行层追上当前头，并验证了所有 payload，节点就可以认为自己“完全同步”，CL 与 EL 均对当前链状态达成一致。
<!-- DAILY_CHECKIN_2026-04-18_END -->

# 2026-04-17
<!-- DAILY_CHECKIN_2026-04-17_START -->








# 简单序列化

## 1\. SSZ 概览与定位

\- SSZ（Simple Serialize）是为以太坊 Beacon Chain 设计的序列化 + Merkle 化方案，在共识层几乎全面取代 RLP（除了发现协议仍用 RLP）。

\- 设计目标：原生支持共识层数据类型、易于高效 Merkle 化和重哈希，方便轻客户端、DAS、无状态等场景。

\---

## 2\. SSZ vs RLP：对比要点

\- Compact：RLP 更紧凑；SSZ 为了类型和 Merkle 化牺牲了一些紧凑性。

\- Expressiveness：SSZ 原生支持 PoS 所需全部类型（整数、bool、列表、容器、bitfield 等），无需额外抽象层；RLP 只有字节串和列表。

\- Hashing：SSZ 天生为高效哈希 / 重哈希设计（配合 Merkle 化），适合频繁更新数据结构；RLP 虽然可哈希，但增量更新效率差。

\- Indexing：两者都不擅长直接索引内部元素，但 SSZ 在某些场景支持部分“直接访问”，RLP 基本要 O(N) 扫描。

\- 决策趋势：出于 Merkle 化和 PoS 复杂结构的需要，以太坊有强烈动机长期“全面迁移到 SSZ”。

\---

## 3\. 基本类型：整数与布尔

### 3.1 Unsigned Integers（uintN）

\- 支持 `uint8/16/32/64/128/256`。

\- 序列化：

\- 把整数转成固定长度 `N/8` 字节数组。

\- 使用 **小端序**（little-endian）：低位在前。

\- 结果字节数组就是序列化形式。

\- 例`uint16(1025)` → 十六进制 `0x0401` → 小端 `01 04`。

\- 反序列化：从小端字节数组恢复整数即可。

\- Pyspec 验证示例`uint64(1025).encode_bytes().hex() == '0104000000000000'`。

### 3.2 Boolean

\- 每个布尔值占 1 字节`True → 0x01False → 0x00`。

\- 反序列化时读 1 字节`01` 代表 True`00` 代表 False。

\- Pyspec 示例`boolean(True).encode_bytes().hex() == '01'`。

\---

## 4\. 组合类型一：Vectors 与 Lists

### 4.1 Vectors（定长数组）

\- 类型形如 `Vector[T, N]`，长度与元素类型固定，例如 `Vector[uint64, 4]`。

\- 序列化步骤：

\- 每个元素按自身规则序列化为字节数组。

\- 按顺序直接拼接，因长度固定，无需额外长度前缀。

\- 例`Vector[uint64,3] [256,512,768]` →

\- 256 → `00 01 00 00 00 00 00 00`

\- 512 → `00 02 00 00 00 00 00 00`

\- 768 → `00 03 00 00 00 00 00 00`

\- 拼接得到`00 01 ... 00 02 ... 00 03 ...`。

\- 反序列化：按固定元素大小切片，逐段反解。

\- Pyspec 示例`Vector[uint16,3](256,512,768).encode_bytes()`。

### 4.2 Lists（变长列表）

\- 类型形如 `List[T, N]N` 是最大长度，实际元素个数 ≤ N。

\- 序列化：

\- 每个元素序列化后拼接。

\- 由于是变长对象，在被嵌入其他容器时，会通过 offset/length 等元数据区分（与向量不同）。

\- 例`List[uint64,5] [1024,2048,3072]` → 每个 8 字节小端，拼接为 `00 04 ... 00 08 ... 00 0C ...`。

\- Pyspec 示例`List[uint64,5](1024,2048,3072).encode_bytes()`。

**Vector vs List 的嵌入差异**：

\- 在容器中`Vector[uint8,3]` 直接序列化为 `010203`。

\- `List[uint8,3]` 会带长度 / offset 元数据，例如示例中 `Alice` 与 `Bob` 容器：

\- `Alice(x: List[uint8,3])` → `'04000000010203'`（4 字节 offset + 数据）。

\- `Bob(x: Vector[uint8,3])` → `'010203'`。

\---

## 5\. Bitfields：Bitvectors 与 Bitlists

### 5.1 Bitvectors（定长 bit 序列）

\- 类型 `Bitvector[N]`，固定 N 个比特。

\- 序列化：

\- 每 8 bit 打包成 1 字节，按 LSB→MSB 顺序填充。

\- 如非 8 的倍数，最后一字节高位用 0 填充。

\- 例`Bitvector[10] = 1011010010` → 前 8 位 `10110100`，余下 `10` + 6 个 0 = `10000000` → `B4 80`。

\- 反序列化：从字节展开 bit，截断到 N 位。

\- 优势：比 `Vector[boolean, N]` 紧凑最多 8 倍。

\- Pyspec 示例`Bitvector[5](1,0,1,0,1).encode_bytes().hex() == '15'` 对比 `Vector[boolean,5]` 的 `'0100010001'`。

### 5.2 Bitlists（变长 bit 序列）

\- 类型 `Bitlist[N]`，最大长度 N，实际 bit 数 ≤ N。

\- 引入 **sentinel bit**：

\- 序列化时在实际 bits 末尾附加一个 `1` 作为终止标记，然后整体打包成字节。

\- 若实际长度刚好是 8 的倍数，会额外占用一个字节存放 sentinel（这点与 bitvector 不同）。

\- 反序列化：从字节展开 bits，自末尾向前找到 sentinel（第一个 `1`），它之前的 bits 为实际数据，之后为 padding。

\- Pyspec 示例：

\- `Bitlist[100](0,0,0).encode_bytes().hex() == '08'`。

\- `Bitlist[8](0...0).encode_bytes().hex() == '0001'` vs `Bitvector[8](0...0).encode_bytes().hex() == '00'`。

\---

## 6\. Containers：组合字段的“结构体”

\- Container 类似结构体 / 对象`class X(Container): a: uint64; b: List[uint8,3]; ...`。

\- 关键：

\- 由 schema 定义字段顺序和类型。

\- 序列化时按顺序遍历字段：

\- 基本类型直接写字节。

\- 组合类型递归序列化。

\- 对变长字段，通常使用 offset/length 或尾部布局来区分。

### 6.1 复杂示例：IndexedAttestation

结构嵌套关系：

\- `IndexedAttestation`：

\- `attesting_indices: List[ValidatorIndex, MAX_VALIDATORS_PER_COMMITTEE]`（变长）

\- `data: AttestationData`（容器）

\- `signature: BLSSignature`（Bytes96, 定长）

\- `AttestationData`：

\- `slot: Slot`

\- `index: CommitteeIndex`

\- `beacon_block_root: Root`

\- `source: Checkpoint`

\- `target: Checkpoint`

\- `Checkpoint`：

\- `epoch: Epoch`

\- `root: Root`

### 6.2 序列化过程要点：

1\. 固定部分（data + signature）：

\- slot、index、source/target 的 epoch（uint64）、root（Bytes32）、signature（Bytes96）按类型依次序列化。

2\. 变长部分（attesting\_indices）：

\- 在容器开头放一个 4 字节 **offset**，指出 attesting\_indices 的起始位置（例如 `e4 00 00 00` → offset 228）。

\- 实际列表数据写在 offset 指向的尾部区域。

3\. 所有字段拼接成一整个字节流。

文中对结果十六进制串分段解析，说明了：

\- offset 指向的位置。

\- slot、index、各个 root 和 epoch 的偏移及含义。

\- 结尾部分对应 attesting\_indices 的实际整数值（如 33652、59750、92360）。

## 7\. SSZ 的核心价值

\- 为 PoS / Beacon Chain 专门设计：类型直接对应规范里的结构，序列化逻辑可与 Pyspec 一一对应，实现简单、验证方便。

\- 更适合 Merkle 化：每个 SSZ 对象可自然对应一颗树，可高效支持轻客户端、DAS 和无状态客户端。

\- 更强的表达力：bitfield、列表、容器等高层结构都被一等支持，避免在 RLP 之上堆抽象。

\- 因此社区长期目标是：尽可能用 SSZ 替代 RLP，统一序列化层。
<!-- DAILY_CHECKIN_2026-04-17_END -->

# 2026-04-16
<!-- DAILY_CHECKIN_2026-04-16_START -->









# 共识客户端

## 概览

\- 共识客户端（Consensus client，原“eth2 client”）运行以太坊 PoS 共识算法，维护 Beacon Chain 的头部和状态。

\- 它不执行交易、不维护执行层状态，这些由执行客户端负责；它也不直接提议 / 证明区块，这些由可选的 validator client 完成。

## 1\. 主流共识客户端一览

这些客户端用不同语言实现，特性和性能侧重点各异，都支持主网和活跃测试网，客户端多样性是选择时的重要考量。

| Client | Language | Developer | Status |

| --- | --- | --- | --- |

| Lighthouse | Rust | Sigma Prime | Production |

| Lodestar | TypeScript | ChainSafe | Production |

| Nimbus | Nim | Status | Production |

| Prysm | Go | Prysmatic Labs | Production |

| Teku | Java | ConsenSys | Production |

| Grandine | Rust | Grandine Devs | Production |

| Caplin | Go | Erigon | Development |

| LambdaClass | Elixir | LambdaClass | Development |

## 2\. 客户端分布与多样性

\- 目前绝大部分节点运营者使用 Prysm 或 Lighthouse 作为共识客户端。

\- 为了 Beacon 链整体健康，推荐刻意选择少数派客户端，降低单一实现的系统性风险（详见 [clientdiversity.org](http://clientdiversity.org) 的说明）。

## 3\. 各客户端简要笔记

### Lighthouse（Rust, Sigma Prime）

\- 定位：高安全性、高性能，生产环境非常常见，但“过于主流”也会带来链分裂风险。

\- 特点：提供多平台二进制（含 ARM）、支持交叉编译，有“便携版”构建；发布二进制由 GPG key `15E66D9...674B0` 签名。

### Lodestar（TypeScript, ChainSafe）

\- 定位：TypeScript 共识客户端，适合快速原型和与浏览器生态集成，提供 beacon node + validator client + 一套协议库（BLS、SSZ 等）。

\- 许可：Apache 2.0 与 LGPL 双重授权，可根据项目选择宽松或弱 copyleft 模型。

### Prysm（Go, Prysmatic Labs）

\- 定位：Go 实现的共识客户端，强调易用性与可靠性，包含完整 beacon node 与 validator client。

\- 技术栈：libp2p 网络、BoltDB 存储、gRPC 进程间通信。

### Nimbus（Nim, Status）

\- 定位：极度资源友好，适合手机、树莓派等轻量设备，也适合在强服务器上保留更多资源给其他任务。

\- 特性：集成 validator client、支持远程签名、带性能分析与监控工具。

### Teku（Java, ConsenSys）

\- 定位：Java 实现，偏企业级场景，强调可运维性与可观测性。

\- 提供完整 beacon node、validator client、REST 管理 API、Prometheus 指标与外部密钥管理集成。

### Grandine（Rust, Grandine Developers）

\- 定位：高性能、轻量级 Rust 共识客户端，强调并行化和高端硬件上的吞吐与低延迟。

\- 架构上与 Lighthouse 同为 Rust 生态，复用部分库，但目标是提供更“精简、高速”的替代方案。

Caplin（Go, Erigon）

\- 定位：Erigon 内嵌的共识客户端，让 Erigon 可以在无外部 CL 的情况下独立运行。

\- 本质是 Erigon 的一个“额外特性”，直接在单一进程内整合 EL + CL。

### LambdaClass（Elixir）

\- 定位：4 期开始的 Elixir 共识客户端，从实验逐渐发展为完整实现，仍在积极开发中，但尚未用于生产网络。
<!-- DAILY_CHECKIN_2026-04-16_END -->

# 2026-04-15
<!-- DAILY_CHECKIN_2026-04-15_START -->










# 共识层网络

## 总览

共识层客户端的网络栈基于 **libp2p**：用它做 P2P 传输和多路复用；用 **libp2p-noise** 做加密与身份认证；用 **discv5** 做节点发现；用 **SSZ** 做消息编码；可选用 **Snappy** 做压缩。

\---

## 1\. libp2p 协议栈

libp2p 是最底层的 P2P 网络框架，支持多种传输（TCP、QUIC、WebRTC、WebSockets 等），Ethereum 共识层是其一个重要用户。

### 1.1 libp2p 协议分层

文中给出一个“libp2p Protocol Stack”表，可以按自下而上记：

| 层级 | 协议示例 | 作用 |

| --- | --- | --- |

| Discovery Layer | mdns, kademlia, rendezvous, identify | 发现节点、获取对方地址和能力 |

| NAT/Relay Layer | relay, dcutr, autonat, pnet | 穿透 NAT、防火墙，或在私网中通信 |

| Transport Layer | tcp, websockets, quic, webrtc, webtransport | 真实数据传输通道 |

| Security Layer | noise, tls, secio(已废弃) | 加密与认证 |

| Multiplexing Layer | yamux, mplex | 单连接上多路复用多条逻辑流 |

| Application Layer | pubsub/gossipsub, ping, 自定义协议 | 具体应用逻辑（区块 gossip、请求响应等） |

要点：

\- 最小连接至少要有：transport + security + multiplexing + application protocol。

\- relay/dcutr 在 NAT 阻碍直连时使用，用中继和打洞实现连接。

### 1.2 Protocol IDs 与 Identify

\- 每个应用协议用一个字符串 ID 标识`/app/protocol/version`，如 `/ipfs/ping/1.0.0`。

\- `Identify` 协议 `/ipfs/id/1.0.0`) 让节点在连接后互相告知：

\- protocolVersion、agentVersion（客户端 UA）

\- publicKey

\- listenAddrs（对方监听的 multiaddrs）

\- observedAddr（自己在对方眼中的地址，用于 NAT 诊断）

\- 支持的应用协议列表

\- signedPeerRecord（对 listenAddrs 的签名记录）

### 1.3 Multiaddress 与 Peer IDs

\- **Multiaddr**：统一封装多层地址，如：位置 + 传输 + peer id，便于将不同协议叠加成一个路径，例如 `/ip4/1.2.3.4/tcp/9000/p2p/<peer-id>`。

\- **Peer Id**：libp2p 中节点的唯一标识，是基于公钥的 multihash，编码为 protobuf；支持多种公钥类型（RSA、Ed25519、secp256k1、ECDSA）。

\- 文本表示常见有两种：

\- base58btc（以 `QM` 或 `1` 开头）。

\- multibase 编码的 CID（libp2p 正逐步迁移到这种形式）。

\---

## 2\. 建立一个 libp2p 连接的流程

libp2p 官方“connections spec”给出了一套通用流程，页面用一个 flowchart 概括：

1\. **Discovery：发现 peers**

\- `mdns`：局域网广播查询，零配置发现同网段节点。

\- `rendezvous`：大家在某个 rendezvous 节点注册，其他节点来这里查询 peer 列表。

\- `kademlia` DHT：全局发现，通过查 DHT 获取 peer 的最新 multiaddrs。

\- `identify`：连接建立后互相告诉地址与能力。

结果：我们得到一组 `(peer-id, multiaddr)`。

2\. **Transport：建立物理连接**

\- `TCP`：基础方案，但可能被 NAT / 防火墙阻断。

\- `WebSockets`：HTTP 封装的 TCP，更容易穿透部分网络限制。

\- `QUIC`：基于 UDP，连接建立快，自带多路复用和加密（TLS 1.3）。

\- `WebRTC` / `WebTransport`：更适合浏览器等场景。

如果 direct 连接失败（被 NAT 阻挡）：

\- `relay`：通过第三方中继 peer 转发，类似 TURN。

\- `dcutr`：通过已有 relay 连接尝试打洞，升级为点对点直连。

3\. **Encryption：加密与认证**

\- `noise`：目前默认首选，使用 Noise XX handshake 做双向认证和密钥交换。

\- `tls`：基于 TLS 的安全信道。

\- `secio`：旧方案，已弃用。

4\. **Multiplexing：多路复用**

\- `yamux`：简单且性能好，许多实现默认。

\- `mplex`：较早的轻量方案。

在一个 TCP/QUIC 连接上开多条 stream，分别跑不同协议或独立会话。

5\. **Application：应用协议**

\- `ping`：测延迟、保活。

\- `pubsub` / `gossipsub` / `episub`：发布订阅式广播（用于区块 / attestation gossip）。

\- 自定义协议：客户端可自定义 CL 相关的 Req/Resp、状态同步等。

\---

## 3\. GossipSub 优化的直觉

页面用两个思路对比解释为什么需要 GossipSub：

\- **方案 1：全连接网状**

\- 每个节点和所有其他节点直接相连（O(n²) 边），广播一个消息会收到 n−1 份重复拷贝，区块数据时浪费巨大带宽。

\- **方案 2：Pub/Sub + 稀疏 mesh（GossipSub）**

\- 所有节点订阅同一 topic（如 `/eth2/beacon_block/ssz_snappy`），只在 topic 内构造一个“小群体”网状（每个节点只连少数 peers）。

\- 消息在这个 mesh 内传递，避免重复广播给所有人。

\- 仍通过 gossip 机制确保最终全网传播。

效果：在保证鲁棒性的同时，大幅减少重复消息和带宽浪费，更适合区块链这种高频广播场景。

\---

## 4\. CL 网络使用的其他组件

### 4.1 libp2p-noise（加密）

\- 使用 Noise framework（而非单一固定协议），共识客户端采用 `XX` pattern：

\- initiator 和 responder 都在初期阶段发送自己的公钥，完成双向认证。

\- 协议提供机密性、完整性、身份认证和前向保密。

### 4.2 ENR（Ethereum Node Records）

\- ENR（EIP‑778）是 Ethereum 推荐的节点地址格式，结构化记录节点的身份与连接信息：

\- **Signature**：使用某种 identity scheme（如 secp256k1）签名，保证记录来自对应节点。

\- **Sequence Number (seq)**：64 位递增整数，用来比较“哪个记录更新”。

\- **Key/Value Pairs**：如 `ip`, `tcp`, `udp`, 公钥、额外元数据等。

\- 文本形式：RLP 编码再 base64，前缀 `enr:`，例如：

`enr:-Jq4QOXd31zNJBTB...`。

\- 在 Ethereum 中，ENR 是首选的网络地址表示方式，各客户端会用 discv5 交换和更新 ENR。

### 4.3 discv5（发现协议）

\- Discovery v5 是专门用于“peer 发现”的 UDP 协议，与 libp2p 传输栈并行运行。

\- 职责：

\- 维护 Kademlia DHT。

\- 使用 ENR 作为记录格式，提供最新节点信息。

\- 定期刷新 / 更新 ENR，保证节点地址变化（IP、端口）时仍能被找到。4

### 4.4 SSZ（编码）

\- SSZ（Simple Serialize）取代执行层的 RLP，在共识层用于所有结构化数据（除发现协议外）。

\- 特性：

\- 非自描述（需预先 schema）。

\- 兼容高效 Merkle 化（每个对象可直接 Merkleize），便于轻客户端和 DAS 场景使用。

### 4.5 Snappy（压缩）

\- Snappy 是 Google 2011 年创建的压缩算法，侧重速度而非最佳压缩比。

\- 共识层在 gossip 大型 SSZ 对象时常配合 Snappy 减少带宽占用，尤其是块和状态更新。

\---

## 5\. PeerDAS 与相关 R&D

\- EIP‑7594：Peer Data Availability Sampling（PeerDAS），为 beacon nodes 提供 DAS 协议，使它们无需下载全部 blob 数据就能抽样验证“数据已被发布”。

\---

# 共识层规范

\- 以太坊最初以 PoW 启动，但从一开始就计划在网络稳定后切换到 PoS。Gasper 论文提出了将 Casper（finality）与 GHOST（fork-choice）结合的共识机制设计。

\- 在 Gasper 设计基础上，社区用 Python 写出了共识层规范 **Pyspec**，它是一份“可执行规范”，既是客户端实现的参考，也是生成测试向量的权威来源。

\---

## Pyspec 的角色

\- 作为 **参考实现**：共识客户端（Prysm、Lighthouse、Teku、Nimbus、Lodestar 等）在实现时都对照 Pyspec，以确保行为与规范一致。

\- 作为 **测试向量生成器**：通过运行 Pyspec，可以生成各类状态转移、边界条件和攻击场景的测试向量，供不同客户端共享，用来检验兼容性。

\- 作为 **研究与教学工具**：研究者可以在 Pyspec 基础上做实验性修改（例如新共识规则、奖励曲线），快速验证设计效果。

\---

# 共识链**规范与 API 接口**

## 1\. 概览

\- Beacon Chain 是以太坊 PoS 的“系统链”，负责维护验证者注册表、处理质押/退出、追踪最终性与同步委员会等共识状态。

\- Beacon API 是共识客户端（beacon node）提供的 REST 接口，供外部读取共识信息，也供 validator 客户端使用（获取职责、提交签名等）。

\---

## 2\. BeaconState：共识层的“全局状态”

`BeaconState` 是一个 SSZ Container，表示某个 slot 的完整共识状态快照，主要字段可以分为几类：

### \- 元信息与历史

\- `genesis_timegenesis_validators_rootslot`：创世时间、初始验证者根、当前 slot。

\- `fork`：当前在用的 fork 版本（phase0/Altair/Bellatrix/Capella/Electra 等）。

\- `latest_block_headerblock_rootsstate_rootshistorical_roots`：用于追踪最近和深层历史的块根与状态根，并支持历史证明。

### \- Eth1 相关

\- `eth1_dataeth1_data_voteseth1_deposit_index`：跟踪存款合约相关数据和投票，用于从执行层接入新的质押。

### \- 验证者注册表与余额

\- `validators: List[Validator]`：所有验证者条目（状态、有效余额、公钥、提款地址等）。

\- `balances: List[Gwei]`：每个验证者的 Gwei 余额。

### \- 随机性与惩罚相关

\- `randao_mixes`：历史 RANDAO 值，用于生成随机数、分配 committees 等。

\- `slashings`：每个 epoch 的 slashed 有效余额总和，用于计算 slashing 惩罚系数。

### \- 出勤与最终性

\- `previous_epoch_participationcurrent_epoch_participation`：每个验证者在前一 / 当前 epoch 的参与标志（head/target/source 是否正确投票）。

\- `justification_bits`：最近若干 epoch 是否被 justified 的 bit 位。

\- `previous_justified_checkpointcurrent_justified_checkpointfinalized_checkpoint`：最终性相关 checkpoint 状态。

### \- Inactivity 与 Sync

\- `inactivity_scores`：每个验证者的 inactivity 分数，用于 inactivity leak。

\- `current_sync_committeenext_sync_committee`：当前与下一轮同步委员会，用于轻客户端与 sync committee 签名。

### \- 执行层桥接与提款（Capella 及之后）

\- `latest_execution_payload_header`：最新执行载荷头，桥接到执行层。

\- `next_withdrawal_indexnext_withdrawal_validator_index`：自动提款进度。

### \- 深层历史与 Electra 新增字段

\- `historical_summaries`：自 Capella 起用于压缩历史的摘要列表。

\- `deposit_requests_start_indexdeposit_balance_to_consumeexit_balance_to_consumeearliest_exit_epochconsolidation_balance_to_consumeearliest_consolidation_epoch`：Electra 之后有关存款、退出、合并（consolidation）的排队与余额消费控制。

\- `pending_depositspending_partial_withdrawalspending_consolidations`：挂起的存款、部分提款和合并请求队列。

\---

## 3\. BeaconBlockBody：一个 Beacon 区块里有什么

`BeaconBlockBody` 是 Beacon block 的“payload”，Electra/Deneb 之后的字段包括：

### \- 基础字段

\- `randao_reveal`：proposer 提供的 BLS 签名，用于 RANDAO 随机性混合。

\- `eth1_data`：对 Eth1 / 执行链数据的投票。

\- `graffiti`：32 字节任意数据，常用作自定义标记。

### \- 操作集合（operations）

\- `proposer_slashings: List[ProposerSlashing]`：提议者被 slash 的证明。

\- `attester_slashings: List[AttesterSlashing]`：attester 被 slash 的证明，Electra 提高上限。

\- `attestations: List[Attestation]`：attester 对 head/target/source 的投票。

\- `deposits: List[Deposit]`：新的质押存款。

\- `voluntary_exits: List[SignedVoluntaryExit]`：自愿退出请求。

\- `sync_aggregate: SyncAggregate`：同步委员会聚合签名，用于轻客户端安全。

### \- 执行层与 blobs

\- `execution_payload: ExecutionPayload`：内嵌执行层 payload（交易、收据根等）。

\- `bls_to_execution_changes: List[SignedBLSToExecutionChange]`：从 BLS 密钥到执行层提款地址的绑定变更。

\- `blob_kzg_commitments: List[KZGCommitment]`：与 EIP‑4844 blobs 对应的 KZG 承诺列表。

### \- Execution requests（Electra 之后）

\- `execution_requests: ExecutionRequests`：从共识层向执行层发出的各种请求打包（如提款、存款、合并请求等），取代 Capella 时版本中散落的单项指令。

\---

## 4\. Beacon API 的定位

基于上下文可总结出 Beacon API 的主要用途：

### \- 共识数据查询：

\- 获取 BeaconState / BeaconBlock（按根或 slot）。

\- 查询 validators 状态、余额、slashing、finality 等。

\- 查询 sync committee、fork choice、头部信息等。

### \- Validator 客户端接口：

\- 获取 proposer 与 attester 职责（duties）。

\- 获取待签名 block / attestation 模板。

\- 提交已签名的 blocks / attestations / slashings / exits。
<!-- DAILY_CHECKIN_2026-04-15_END -->

# 2026-04-14
<!-- DAILY_CHECKIN_2026-04-14_START -->











# 共识层概述

## 总览：目标与挑战

-   共识层的目标：在“**不可靠基础设施**”（消费者级硬件、不稳定网络、恶意节点）上，构建“**可靠的分布式系统**”，让全球成千上万节点维护一份完全一致的账本。
    
-   Ethereum 今天的共识由两套协议组合而成：LMD GHOST（fork choice）+ Casper FFG（finality），合称 Gasper。
    

* * *

## 1\. PoW / PoS、区块与区块链

-   PoW 和 PoS 本身不是共识协议，而是 **Sybil 抵抗机制**：通过算力或质押成本限制“刷号”，为共识协议提供权重（PoW=累计工作量，PoS=累计质押）。
    
-   区块是由领导者（proposer）打包的一批交易（或投票）：
    
    -   执行链：payload 是用户交易。
        
    -   Merge 前 Beacon 链：payload 主要是验证者的 attestations。
        
    -   Merge 后 Beacon 链：既包含 execution payload，又包含共识层信息；Deneb/4844 之后还包含 blob commitments。
        
-   除创世区块外，所有块都指向父块形成链；共识协议的任务是让所有节点最终就同一条“规范链”达成一致。
    

* * *

## 2\. 向 PoS 的迁移：The Merge

-   2022‑09‑15，巴黎硬分叉（The Merge）发生，Ethereum 共识从 PoW 切换到 PoS，由 Beacon 链接管共识。
    
-   激活条件是 **终端总难度 TTD** 而非区块高度：
    
    -   TTD 达到预设阈值的最后一个 PoW 区块成为 terminal block，确保过渡不被恶意分叉干扰。
        
    -   这套机制同样用于 testnet / devnet 等网络的 Merge 激活。
        
-   Merge 之后：
    
    -   Beacon 链成为新的“引擎”，负责 slot/epoch、验证者、attestations 和 finality。
        
    -   能耗大幅下降，安全性与去中心化通过 PoS 经济激励与惩罚机制来维护。
        

* * *

## 3\. Beacon 链与验证者

## 3.1 骨干角色：Beacon Chain

-   Beacon 链是 PoS 共识的骨干：
    
    -   维护验证者注册表及其状态。
        
    -   安排 proposer、委员会，收集与聚合 attestations。
        
    -   负责 epochs、checkpoints 和 finality。
        

## 3.2 验证者与激活条件

-   验证者：参与提议区块和投票（attest）的主体，需要质押 ETH 作为抵押。
    
-   条件与要点：
    
    -   最低 32 ETH 才能激活一个 validator，单个 validator 最大有效余额 2048 ETH。
        
    -   质押 ETH 是违约保证金，恶意或严重失职会被 slash。
        
    -   validator 客户端可以管理多个验证者，依赖 beacon 节点读取 Beacon 链状态。
        

## 3.3 随机性与委员会

-   验证者分配依赖 **RANDAO + VDF** 提供的加密随机性，防止操纵 proposer/committee 分配。
    
-   机制：
    
    -   每个 slot 分配一个 proposer 和若干 committees（每个至少 128 验证者，理想情况下 4096+ 验证者网络）。
        
    -   committee 内的 attestations 使用 BLS 聚合签名压缩，降低链上负载。
        

* * *

## 4\. 时间结构：Slot / Epoch / Attestations / Finality

## 4.1 Slots 与 Epochs

-   slot：12 秒，每个 slot 最多一个 Beacon block（可为空）。
    
-   epoch：32 个 slot ≈ 6.4 分钟，slot 0 起始，slot 0 属于 epoch 0。
    
-   每个 slot：
    
    -   一名 proposer 有机会提块。
        
    -   对应的 committees 对该 slot 的链头进行 attestation。
        

## 4.2 Attestations：LMD GHOST + FFG vote

-   每个 validator 每个 epoch 提交 1 个 attestation，包含：
    
    -   LMD GHOST vote：对“当前 Beacon 链 head”的投票。
        
    -   FFG vote：对“source checkpoint → target checkpoint”的投票。
        
-   LMD GHOST：通过“每个验证者最新投票”选出加权最重子树，决定链头。
    
-   Casper FFG：通过对 checkpoints 的投票实现 justification & finality。
    

## 4.3 Checkpoints 与 Finality

-   每个 epoch 选一个 checkpoint：
    
    -   如果 epoch 第一 slot 有块，则该块为 checkpoint。
        
    -   否则取最近的前一个块作为该 epoch 的 checkpoint。
        
-   定义：
    
    -   justified：该 checkpoint 收到 ≥ 2/3 有效 stake 的 FFG vote。
        
    -   finalized：一个 checkpoint 被 justified 后，再有一个后续 epoch 的 checkpoint 也被 justified，前者即被 finalized，连带其之前的所有块也变为最终不可逆。
        
-   典型时间尺度：
    
    -   交易在 epoch 中部被打包：大约半个 epoch 到下一个 checkpoint，再加一个 epoch 完成 finality，约 2.5 个 epoch ≈ 14–16 分钟。
        

* * *

## 5\. Blob 与数据可用性（EIP‑4844）

-   Deneb/Cancun 升级引入 **EIP‑4844（proto‑danksharding）**，为 Ethereum 提供“临时数据可用性层”：
    
    -   新增 **blobs**，每个 block 可携带 3–6 个 blob sidecars。
        
    -   L2 可将批量交易数据放入 blobs 中，降低 L1 gas 成本，提高吞吐。
        
-   采用 **KZG commitments** 来承诺和验证 blobs 内容，需要一次 KZG trusted setup（KZG Ceremony）。
    
-   存储影响：
    
    -   粗略估算：`≈ 52–104 GB` 用于 4096 epochs（默认保留周期）内 blobs 的存储；到期后客户端可删旧 blobs。
        

* * *

## 6\. 激励与惩罚：Rewards / Penalties / Slashings

## 6.1 奖励：Attester / Proposer / Whistleblower

-   Attester rewards：
    
    -   对正确 head 和 checkpoints 进行及时 attestation 会获得奖励，越早上链奖励越高。
        
-   Proposer rewards：
    
    -   提出的 block 被纳入最终链并贡献 finality，可获得显著奖励（约整体收益的 1/8）。
        
-   Whistleblower rewards：
    
    -   举报 slashable offense 的证明会获得奖励，目前奖励发给 proposer。
        

## 6.2 惩罚与 Inactivity Leak

-   Attester penalties：
    
    -   不按时 attestation 或投给错误头 / 未被 final 的 checkpoint，会受到罚分，防止长期懒惰或偏离共识。
        
-   Slashings（严重违规）：
    
    -   Double proposal、LMD GHOST double vote、FFG surround vote、FFG double vote 都可被 slashing。
        
    -   至少罚没 1/32 余额并强制退出，罚没规模与同时间被 slash 的人数相关，可高达全部质押。
        
-   Inactivity leak：
    
    -   长时间无法 final（>4 epochs）时启动 inactivity leak：
        
        -   Attester rewards 变为 0。
            
        -   对未能参与的验证者持续施加递增罚分，直至剩余活跃验证者重新形成 ≥ 2/3 多数并恢复 finality。
            

## 6.3 Validator 生命周期

-   激活：余额 ≥ 32 ETH 后排队进入 active set。
    
-   自愿退出：服务 ≥ 2048 epochs 后可申请退出，退出后 4 epochs 冷却期才能取回余额。
    
-   强制退出：余额跌至 16 ETH 或被 slash。
    
-   提款延迟：
    
    -   正常退出：约 27 小时。
        
    -   被 slash：约 36 天（8192 epochs）。
        

* * *

# 共识层架构

从共识协议视角讲“客户端架构”，核心是：Ethereum PoS 把两套协议（LMD GHOST + Casper FFG）合在一起形成 Gasper，并通过 CL 客户端 + EL 客户端协同来达成共识。

重点包括：fork-choice 机制、reorg、safety & liveness 取舍、CL/EL 之间的控制流，以及 Beacon 链状态转移函数。

* * *

## 1\. Fork-choice 机制与 Reorg

## Block tree 与 fork choice rules

-   网络延迟、恶意节点等会让不同节点看到不同的 block tree，需要一个 fork-choice rule 来选“头部区块”，从而选出一条线性规范链。
    
-   典型规则：
    
    -   PoW（比特币、旧以太坊）：heaviest chain rule（累计工作量最大的链）。以太坊 PoW 实际从未实现白皮书中的 GHOST 变体，而是使用简单的 heaviest chain。
        
    -   Casper FFG（PoS）：选择包含“最高高度已 justified checkpoint”的链，且永不回滚 finalized block。
        
    -   LMD GHOST（PoS）：Greediest Heaviest Observed SubTree，基于验证者最新 votes 对各子树加权计分，同时也遵守 Casper FFG 的“不可回滚 finalized block”的约束。
        

## Reorgs 与 Reversion

-   当新块到来改变 fork-choice 结果时，节点可能需要 reorg：
    
    -   回滚旧 head 分支上的若干区块，从回滚点恢复状态。
        
    -   改为沿新 head 分支重放区块。
        
-   一两块的短 reorg 很常见，通常源自网络延迟；长 reorg 应当极少出现，否则可能意味着攻击或实现 bug。
    

* * *

## 2\. Safety vs Liveness：以太坊的取舍

-   **Safety**：不发生坏事，比如不会 finalize 两条互相冲突的历史，所有诚实节点保持账本一致。
    
-   **Liveness**：好事最终会发生，链不会停摆，总能持续产生新块、推进状态。
    
-   CAP 定理 + 不可靠网络 ⇒ 无法在所有情形下同时保证完美 safety 和完美 liveness。
    
-   以太坊 PoS 的策略：
    
    -   正常网络条件下尽量兼顾 safety 与 liveness。
        
    -   在严重网络分区时偏向保持 liveness：各分区内链仍会出块，但无法 finality；若分区持续，很可能最终在不同侧 finalize 冲突历史，出现“不可合并的两条链”。
        

* * *

## 3\. Gasper：LMD GHOST + Casper FFG

-   LMD GHOST 专注 liveness：
    
    -   根据验证者“最新消息”（latest message driven）对 block tree 加权打分，持续选择最重子树作为 head，从而保持链向前推进。
        
-   Casper FFG 专注 safety：
    
    -   通过对 checkpoint 的投票来 justify / finalize 区块，防止长距离回滚，为链尾提供“硬边界”。
        
-   Gasper = LMD GHOST（驱动 head）+ Casper FFG（周期性 finality）：
    
    -   即使由于网络问题一段时间无法 final，LMD GHOST 仍会让链继续前进；
        
    -   一旦条件恢复，Casper FFG 会重新开始 finalizing，从而恢复安全边界。
        

* * *

## 4\. CL / EL 客户端架构与控制流

## 4.1 组件

-   **Beacon Node（共识客户端）**：
    
    -   运行 PoS 共识，处理 beacon blocks、attestations，与其它 beacon nodes 通过独立的 CL p2p 网络通信。
        
    -   本地与执行客户端通过 Engine API 连接，可选地还连接本地 validator 客户端。
        
-   **Validator Client**：
    
    -   管理 32 ETH 质押、生成提议块和 attestations，只与本地 beacon node 对话，由后者负责广播到网络。
        
-   每个以太坊节点都包含 EL + CL：
    
    -   执行客户端（Geth、Nethermind、Besu、Erigon 等）。
        
    -   共识客户端（Prysm、Teku、Lighthouse、Nimbus、Lodestar 等）。
        
    -   两者通过本地 RPC（Engine API）互联，并共享同一个 ENR（但有 eth1 / eth2 不同密钥）。
        

## 4.2 控制流（非出块节点）

当本节点不是出块者时：

1.  共识客户端通过 block gossip 收到新的 beacon block。
    
2.  本地预验证 block（签名、结构等）。
    
3.  提取其中 execution payload，发给执行客户端。
    
4.  执行客户端执行其中交易，检查状态转移是否合法。
    
5.  执行客户端将验证结果（成功或错误）返回给共识客户端。
    
6.  若有效，共识客户端把 block 加入本地链，并对其做 attestation，然后把 attestation 广播出去。
    

## 4.3 控制流（本节点是出块者）

当本节点被选为 proposer 时：

1.  共识客户端收到“你是下一个 block producer”的通知。
    
2.  通过 Engine API 调用执行客户端的“create block / build payload”接口。
    
3.  执行客户端从 mempool 选择交易。
    
4.  执行客户端执行交易、构造 execution payload 和 block hash。
    
5.  共识客户端把 payload + block hash 封装进 beacon block。
    
6.  共识客户端将 beacon block 广播到 CL 网络。
    
7.  其它节点验证该 block，并对其出 attestations。
    
8.  当足够验证者 attestation 后，该 block 成为链头，随后通过 Casper FFG 被 justify / finalize。
    

* * *

## 5\. Beacon 链状态转移函数

-   每个节点维护一个 beacon state，按顺序应用区块来更新状态：
    
    -   把 state transition 看成纯函数 `S' = f(S, B)`，相同输入必然给出相同输出，保证所有诚实节点状态一致。
        

## Slot-driven 结构

Beacon 链是 **slot 驱动** 而非 block 驱动：即使某个 slot 没有 block，也要执行 per-slot 逻辑。

状态转移拆成三类函数：

1.  **Per-slot transition**：`S' = f_s(S)`
    
    -   每个 slot 必执行一次，更新 slot 号、计时器、RANDAO 混合等。
        
2.  **Per-block transition**：`S' = f_b(S, B)`
    
    -   只在该 slot 有 block 时执行，处理 block header、operations（attestations、slashings、deposits 等）。
        
3.  **Per-epoch transition**：`S' = f_e(S)`
    
    -   每到 epoch 开始 slot，会处理奖励 / 惩罚、crosslink / checkpoint 更新等。
        

## 代码片段要点

伪代码的核心逻辑：

```python
def state_transition(state, signed_block, validate_result=True):
    block = signed_block.message
    process_slots(state, block.slot)        # 处理空 slot + epoch 逻辑
    if validate_result:
        assert verify_block_signature(state, signed_block)
    process_block(state, block)             # 应用 block 层规则
    if validate_result:
        assert block.state_root == hash_tree_root(state)
```

其中 `process_slots` 会循环调用 `process_slot`，并在 epoch 边界调用 `process_epoch`。
<!-- DAILY_CHECKIN_2026-04-14_END -->

# 2026-04-13
<!-- DAILY_CHECKIN_2026-04-13_START -->












# RLP  
网址：[epf](https://epf.wiki/#/wiki/EL/RLP)

## 1\. RLP 是什么

-   全称：Recursive Length Prefix，递归长度前缀编码。
    
-   作用：把“嵌套的字节数组”（bytes 和 list of bytes）编码成一个字节序列，用于以太坊执行层（交易、区块、状态数据等）。
    
-   特点：
    
    -   极简格式，只关心结构（字符串 / 列表、长度），不关心具体类型（整数、布尔等）。
        
    -   上层协议自己约定“这些 bytes 表示什么类型”。
        
    -   目标是跨客户端保持字节级一致性。
        

## 2\. RLP 使用场景与对比

-   执行层（EL）：使用 RLP 序列化交易、区块头等数据结构。
    
-   共识层（CL）：使用 SSZ（Simple Serialize），而不是 RLP。
    
-   为什么不用 JSON / XML / Protobuf 等：
    
    -   JSON/XML 太重，有语法和类型；
        
    -   Protobuf/Cap’n Proto 需要事先 schema；
        
    -   RLP 只做“通用字节数组 + 嵌套结构”编码，简单稳定，适合共识。
        

## 3\. 编码规则总览

RLP 对两类对象编码：字符串（byte 序列）和列表（字符串或列表的有序集合）。

1.  单字节（0x00–0x7F）：
    
    -   规则：值本身就是编码（不加前缀）。
        
2.  短字符串（长度 0–55 字节，且不是单字节 0x00–0x7F）：
    
    -   编码：一个前缀字节（0x80 + 长度） + 原始字节序列。
        
3.  长字符串（长度 ≥ 56 字节）：
    
    -   先把“长度”转为 big-endian 字节序列 L；
        
    -   前缀：0xB7 + L 的长度；
        
    -   编码：前缀 + L + 原始字节序列。
        
4.  短列表（总 payload 长度 0–55 字节）：
    
    -   先对列表中每个元素做 RLP 编码，拼接得到 payload；
        
    -   前缀：0xC0 + payload 长度；
        
    -   编码：前缀 + payload。
        
5.  长列表（总 payload 长度 ≥ 56 字节）：
    
    -   先得到 payload（同上）；
        
    -   把 payload 长度转为 big-endian 字节序列 L；
        
    -   前缀：0xF7 + L 的长度；
        
    -   编码：前缀 + L + payload。
        
6.  空值与空列表：
    
    -   空字符串：编码为 0x80。
        
    -   空列表：编码为 0xC0。
        

## 4\. 前缀区间速记

-   0x00–0x7F：单字节自身。
    
-   0x80–0xB7：短字符串（前缀=0x80+len）。
    
-   0xB8–0xBF：长字符串（前缀=0xB7+len(len)，后跟长度，再跟数据）。
    
-   0xC0–0xF7：短列表（前缀=0xC0+len(payload)）。
    
-   0xF8–0xFF：长列表（前缀=0xF7+len(len)，后跟长度，再跟 payload）。
    

## 5\. 解码思路（自顶向下）

给定一个字节序列，从第一个字节（前缀）开始分类：

-   0x00–0x7F：这是一个单字节字符串，长度 1，内容就是该字节。
    
-   0x80–0xB7：短字符串，长度 = 前缀 - 0x80，读出后面对应长度的字节。
    
-   0xB8–0xBF：长字符串，先读长度字段（长度 = 前缀 - 0xB7），再读该长度字段表示的真实长度，然后再读数据。
    
-   0xC0–0xF7：短列表，payload 长度 = 前缀 - 0xC0；对 payload 区间递归解码出多项元素。
    
-   0xF8–0xFF：长列表，和长字符串类似，先读长度字段，再读 payload，再对 payload 递归解码。
    

## 6\. 示例（概念级）

文中用字符串 "dog"、列表 \["cat", "dog"\] 等例子演示：

-   先把字符串转为 ASCII bytes，再按“短字符串”规则加前缀。
    
-   列表则是把每个元素单独 RLP 编码后拼接，再整体按“列表”加前缀。
    

## 7\. 实现与工具

页面列出多种语言的 RLP 实现和资料：

-   Go：Geth 中的 RLP 包。
    
-   JavaScript / TypeScript：以太坊相关 JS 库中的 RLP 工具。
    
-   Python、Rust、C#（Nethermind）等多种实现。
    
-   参考资料：
    
    -   以太坊黄皮书中 RLP 章节。
        
    -   官方文档与若干教程、博客文章，对编码规则和示例做进一步说明。
        

# EOF

网址：[epf](https://epf.wiki/#/wiki/EL/RLP)

## 1\. RLP 是什么

-   全称：Recursive Length Prefix，递归长度前缀编码。
    
-   作用：把“嵌套的字节数组”（bytes 和 list of bytes）编码成一个字节序列，用于以太坊执行层（交易、区块、状态数据等）。
    
-   特点：
    
    -   极简格式，只关心结构（字符串 / 列表、长度），不关心具体类型（整数、布尔等）。
        
    -   上层协议自己约定“这些 bytes 表示什么类型”。
        
    -   目标是跨客户端保持字节级一致性。
        

## 2\. RLP 使用场景与对比

-   执行层（EL）：使用 RLP 序列化交易、区块头等数据结构。
    
-   共识层（CL）：使用 SSZ（Simple Serialize），而不是 RLP。
    
-   为什么不用 JSON / XML / Protobuf 等：
    
    -   JSON/XML 太重，有语法和类型；
        
    -   Protobuf/Cap’n Proto 需要事先 schema；
        
    -   RLP 只做“通用字节数组 + 嵌套结构”编码，简单稳定，适合共识。
        

## 3\. 编码规则总览

RLP 对两类对象编码：字符串（byte 序列）和列表（字符串或列表的有序集合）。

1.  单字节（0x00–0x7F）：
    
    -   规则：值本身就是编码（不加前缀）。
        
2.  短字符串（长度 0–55 字节，且不是单字节 0x00–0x7F）：
    
    -   编码：一个前缀字节（0x80 + 长度） + 原始字节序列。
        
3.  长字符串（长度 ≥ 56 字节）：
    
    -   先把“长度”转为 big-endian 字节序列 L；
        
    -   前缀：0xB7 + L 的长度；
        
    -   编码：前缀 + L + 原始字节序列。
        
4.  短列表（总 payload 长度 0–55 字节）：
    
    -   先对列表中每个元素做 RLP 编码，拼接得到 payload；
        
    -   前缀：0xC0 + payload 长度；
        
    -   编码：前缀 + payload。
        
5.  长列表（总 payload 长度 ≥ 56 字节）：
    
    -   先得到 payload（同上）；
        
    -   把 payload 长度转为 big-endian 字节序列 L；
        
    -   前缀：0xF7 + L 的长度；
        
    -   编码：前缀 + L + payload。
        
6.  空值与空列表：
    
    -   空字符串：编码为 0x80。
        
    -   空列表：编码为 0xC0。
        

## 4\. 前缀区间速记

-   0x00–0x7F：单字节自身。
    
-   0x80–0xB7：短字符串（前缀=0x80+len）。
    
-   0xB8–0xBF：长字符串（前缀=0xB7+len(len)，后跟长度，再跟数据）。
    
-   0xC0–0xF7：短列表（前缀=0xC0+len(payload)）。
    
-   0xF8–0xFF：长列表（前缀=0xF7+len(len)，后跟长度，再跟 payload）。
    

## 5\. 解码思路（自顶向下）

给定一个字节序列，从第一个字节（前缀）开始分类：

-   0x00–0x7F：这是一个单字节字符串，长度 1，内容就是该字节。
    
-   0x80–0xB7：短字符串，长度 = 前缀 - 0x80，读出后面对应长度的字节。
    
-   0xB8–0xBF：长字符串，先读长度字段（长度 = 前缀 - 0xB7），再读该长度字段表示的真实长度，然后再读数据。
    
-   0xC0–0xF7：短列表，payload 长度 = 前缀 - 0xC0；对 payload 区间递归解码出多项元素。
    
-   0xF8–0xFF：长列表，和长字符串类似，先读长度字段，再读 payload，再对 payload 递归解码。
    

## 6\. 示例（概念级）

文中用字符串 "dog"、列表 \["cat", "dog"\] 等例子演示：

-   先把字符串转为 ASCII bytes，再按“短字符串”规则加前缀。
    
-   列表则是把每个元素单独 RLP 编码后拼接，再整体按“列表”加前缀。
    

## 7\. 实现与工具

页面列出多种语言的 RLP 实现和资料：

-   Go：Geth 中的 RLP 包。
    
-   JavaScript / TypeScript：以太坊相关 JS 库中的 RLP 工具。
    
-   Python、Rust、C#（Nethermind）等多种实现。
    
-   参考资料：
    
    -   以太坊黄皮书中 RLP 章节。
        
    -   官方文档与若干教程、博客文章，对编码规则和示例做进一步说明。
<!-- DAILY_CHECKIN_2026-04-13_END -->

# 2026-04-12
<!-- DAILY_CHECKIN_2026-04-12_START -->













# JSON-RPC

## 1\. 概念与作用

\- JSON-RPC 是一种用 JSON 编码的远程过程调用协议，基于 OpenRPC 规范。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- 在以太坊里，它是 Execution API 的一部分，用来和执行层客户端交互，也是钱包、DApp 与节点交互的主要方式。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- 共识层（CL）与执行层（EL）之间的 Engine API 也是通过 JSON-RPC 接口通信的。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

## 2\. 基本请求结构

\- 标准请求格式`{ "id": 1, "jsonrpc": "2.0", "method": "...", "params": [...] }`。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- 字段含义：

\- **id**：请求的唯一标识符。

\- **jsonrpc**：协议版本（一般为 "2.0"）。

\- **method**：调用的方法名。

\- **params**：方法参数，可以为空数组，有些参数有默认值（如 block tag 为 "latest"）。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

## 3\. 命名空间与常见命名空间

\- 方法由 “命名空间 + 下划线 + 方法名” 组成，例如 `eth_getBalance`。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- 客户端必须实现规范要求的基础方法，同时可以扩展客户端自有命名空间；不同客户端（如 Geth、Reth）命名空间略有差异，需要参考各自文档。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

**常见命名空间**（是否敏感）： \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

| Namespace | 作用简记 | 是否敏感 |

| --- | --- | --- |

| eth | 与以太坊核心交互（余额、交易、区块） | 可能敏感（涉账号数据） \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC)) |

| web3 | 客户端工具与信息 | 否 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC)) |

| net | 网络信息（网络 ID、peer 等） | 否 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC)) |

| txpool | 交易池状态检查 | 否 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC)) |

| debug | 深度调试、状态检查（Geth trace 等） | 一般视为敏感，常被限制 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC)) |

| trace | 状态追踪（Parity 风格 trace） | 否 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC)) |

| admin | 节点管理配置 | 是（可控制节点） \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC)) |

| rpc | RPC 服务与模块信息 | 否 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC)) |

“敏感”通常指可以配置节点或访问本地账户数据的接口。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

### 常用方法示例

**eth 命名空间**（对钱包和 DApp 最重要）

\- `eth_blockNumber`：返回最新区块号。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- `eth_call`：在本地执行一笔消息调用，不上链，常用来读合约状态。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- `eth_chainId`：当前链的 chain id。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- `eth_estimateGas`：模拟执行以估算需要的 gas。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- `eth_gasPrice`：返回当前 gas price（wei）。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- `eth_getBalance(address, block)`：查询地址在指定区块的余额。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- `eth_getBlockByHash / eth_getBlockByNumber`：按 hash / 高度获取区块信息，可选是否返回完整交易对象。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- `eth_getBlockTransactionCountByHash / ...ByNumber`：获取区块中交易数量。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- `eth_getCode(address, block)`：获取某地址上的合约代码。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- `eth_getLogs(filter)`：按过滤器（地址、topics、区块范围）获取日志。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- `eth_getStorageAt(address, position, block)`：获取合约某个存储槽位的值。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

**debug 命名空间**（调试、研究用）

\- 通常用于区块浏览器、研究或问题排查，对性能消耗大，公共 RPC 多半限制。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- 典型方法：

\- `debug_getBadBlocks`：返回近期的坏块列表。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- `debug_getRawBlock(block_number)`：返回 RLP 编码的区块。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- `debug_getRawHeader(block_number)`：返回 RLP 编码的区块头。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- `debug_getRawReceipts(block_number)`：返回 EIP‑2718 编码的收据数组。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- `debug_getRawTransactions(tx_hash)`：返回 EIP‑2718 编码的交易数组。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

## 4\. Engine API（CL 与 EL 之间）

\- Engine API 不对普通用户开放，单独跑在受 JWT 认证保护的端口上，只允许共识客户端访问。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- 核心目的：在共识和执行客户端之间交换共识状态、分叉选择、区块验证等信息。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- 通信方式：HTTP 上的 JSON-RPC，使用 JWT 做身份认证（不加密流量）。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- 主要方法示例（有多版本的以支持升级）： \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- `engine_exchangeTransitionConfigurationV1`：交换 CL/EL 的配置。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- `engine_forkchoiceUpdatedV1*`：更新 forkchoice 状态并可触发 payload 构建。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- `engine_getPayloadBodiesByHashV1*`：按区块哈希数组获取 payload bodies。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- `engine_getPayloadV1*`：从执行层获取构建好的 execution payload。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- `debug_newPayloadV1*`：返回 payload 验证细节，用于调试。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

## 5\. 编码约定（十六进制）

\- 数值（Quantities）统一用带 "0x" 前缀的十六进制，不允许前导零。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- 例：十进制 65 表示为 `"0x41"`，0 表示为 `"0x0"`。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- 非法示例`"0x"`（没有数字）`"ff"`（缺少 0x 前缀）。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- 未格式化数据（哈希、地址、字节数组）同样使用 0x 十六进制，但不允许多余前导零。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

## 6\. 传输层：协议无关（Transport-agnostic）

\- JSON-RPC 与传输协议解耦，可通过 HTTP、WebSocket (WSS)、IPC 等方式发送。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- **HTTP**：单次请求‑响应，响应后连接关闭，适合简单查询。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- **WSS**：双向长连接，允许订阅/事件推送，适合实时事件（logs、newHeads 等）。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- **IPC**：本机进程间通信，比 HTTP/WSS 更快，但不能远程访问，常用于本地 JS 控制台。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

## 7\. 常用调用方式与工具

### 直接用 curl

\- 示例：获取最新区块号`eth_blockNumber`，无参数，默认使用 "latest"）。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- 请求是 HTTP POST，JSON-RPC 数据放在请求体里`Content-Type` 为 `application/json`。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

### 使用 axios（JS/TS）

\- 典型步骤：

\- 指定节点 URL 和查询的地址。

\- POST 到节点`jsonrpc: '2.0'method: 'eth_getBalance'params: [address, 'latest']id: 1`，并带 `Content-Type: application/json` 头。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- 本质上仍是 HTTP+JSON-RPC，只是通过库封装成更友好的接口。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

### 使用 web3 库（推荐日常开发）

\- 一般使用 web3py、web3.js 或 ethers.js，这些库封装了底层 JSON-RPC 调用。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

示例：web3py。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- 使用 HTTPProvider 连接`Web3.HTTPProvider('http://localhost:8545')`[。](http://localhost:8545'\)`。)

\- 获取余额`w3.eth.get_balance('0xaddress')`。

示例：ethers.js。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- 创建 provider`new ethers.providers.JsonRpcProvider('http://localhost:8545')`[。](http://localhost:8545'\)`。)

\- 获取区块号`await provider.getBlockNumber()`。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

# DevP2P

## 1\. 整体概念与作用

\- DevP2P 是执行层（EL）使用的点对点网络协议栈，用来让节点发现彼此并在链上安全地交换数据。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- EL 网络里有两部分：发现层（Discovery，基于 UDP）负责找节点，传输层（Transport，RLPx，基于 TCP）负责建立加密连接并传消息。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- 执行客户端负责 gossip 交易，共识客户端负责 gossip 区块，各自有独立的 P2P 网络（EL：devp2p，CL：libp2p）。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

## 2\. 传输层基础：TCP / UDP

\- TCP：面向连接、可靠、有顺序保证，适合重要数据传输（RLPx）。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- UDP：无连接、更快、不保证可靠和顺序，适合节点发现（Discv）。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

## 3\. Discovery：节点发现（Discv 协议族）

### 整体流程

\- 节点通过硬编码 bootnodes 启动，使用 Kademlia DHT 风格的算法，在路由表中存节点信息（k-buckets，k=16）。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- 新节点对 bootnode 发 PING，收到 PONG 后完成“验证 + 绑定”，再用 FINDNODE / NEIGHBOURS 获取更多邻居并重复流程。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- 路由表按“最近见到”排序，12 小时不响应的节点会被踢出。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

### Wire 子协议中的核心包结构

\- PING：带版本、from/to 地址、过期时间、ENR 序号等信息。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- PONG：带 to、ping-hash、过期时间、ENR 序号等。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- FindNode`[target, expiration, ...]`，target 是 64 字节 secp256k1 公钥。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- Neighbours：返回最多 16 个邻居节点 `[ip, udp-port, tcp-port, node-id, ...]`。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- ENRRequest / ENRResponse：请求 / 返回 Ethereum Node Record。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- 所有数据包都封装在 1280 字节左右的 UDP 数据报中，前面是 `hash || signature || packet-type` 头部。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

### Discovery 协议版本：Discv4 vs Discv5

\- 大多数执行客户端已迁移到 Discv5，少数仍在从 Discv4 过渡（如 Besu、Erigon 等）。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

**Discv4 要点**

\- 节点身份：每个节点有 secp256k1 公私钥，公钥作为 Node ID，距离用 Node ID 哈希的 XOR 定义。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- ENR：使用 “v4” 身份方案的 Ethereum Node Record 存储连接信息和身份，用 ENRRequest/ENRResponse 拉最新记录。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- Kademlia 路由表：256 个 k-buckets，每个桶最多 16 个节点，按距离分段、按最近使用排序，旧节点优先被替换。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- Endpoint 验证：只有通过 Ping/Pong 验证的节点才被视为可响应，防止放大攻击。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- 递归查找：从若干个最接近目标的节点开始迭代查询，直到找不到更近的 k 个节点为止（典型参数 α=3、k=16）。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- 消息类型：Ping、Pong、FindNode、Neighbors、ENRRequest、ENRResponse，全部走 UDP。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

**Discv5 要点**

\- 只存签名的 ENR，避免任意 key–value，增强真实性和可扩展性。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- 加密线路：使用 AES‑GCM 加密、ECDH 交换会话密钥，并通过 WHOAREYOU + Handshake 防止伪造。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- 路由与查找：同样基于 Kademlia，支持自适应路由和自我修复，使用并行查询增强抗攻击性。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- 服务发现：节点可以做 topic advertisement，按“主题半径”搜索特定服务提供者，支持自适应 radius 估计，提高查找效率。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- 额外消息类型：Nodes、WhoAreYou、Handshake、TalkReq/TalkResp 等，用于加密握手和自定义协议。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

**Discv4 vs Discv5 对比表**

\- ENR：v4 基本 ENR；v5 可拓展 ENR + metadata。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- 安全性：v4 明文；v5 AES‑GCM 加密，有握手和会话密钥。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- 服务发现：v4 能力有限；v5 有 topic‑based 查找。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- 可扩展性：v4 比较静态；v5 支持多身份方案，更可扩展。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- 时间依赖：v4 依赖时钟；v5 消除了时钟依赖。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- 可扩展和大网络：v5 对大规模网络更友好。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

### ENR：Ethereum Node Records

\- ENR 由 EIP‑778 定义，是节点连接信息的标准格式：IP、端口、公钥、ID 等。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- 主要字段（都可选，只有 `id` 必须）`id`（身份方案，如 "v4"）`secp256k1iptcpudpip6tcp6udp6`。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- 结构`[signature, seq, k1, v1, k2, v2, ...]seq` 为 64 位序号`signature` 对 `content = [seq, k, v...]` 的签名。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- 编码：RLP 序列化后最大 300 字节，再 base64 文本化并加 `enr:` 前缀（例如示例 ENR 包含 127.0.0.1:30303 和对应 node ID）。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- 兼容其他格式：

\- multiaddr`/ip4/127.0.0.1/tcp/30303/<node-id>`。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- enode`enode://<node-id>@ip:tcp?discport=udp`，更人类可读。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

## 4\. RLPx：传输与加密（TCP）

### 职责与流程概览

\- RLPx 是基于 TCP 的传输协议，用于在节点间建立加密连接并进行消息收发。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- 核心阶段：加密握手建立连接、会话初始化与多路复用、消息分帧与完整性校验。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

### 安全连接建立

\- 节点通过 secp256k1 椭圆曲线生成临时（ephemeral）密钥对，发起认证消息（包含临时公钥和 nonce）。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- 对端解密并验证后发回 ack，并发送首个加密 frame（包含 Hello 消息：端口、节点 ID、客户端 ID、协议列表）。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- 认证完成后，双方进入正常的加密通信阶段。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

### 会话建立与加密算法

\- 使用 ECIES（基于 secp256k1）进行密钥交换和会话建立：

\- 曲线：secp256k1；KDF：NIST SP 800‑56 concat KDF；MAC：HMAC‑SHA‑256；加密：AES‑128‑CTR。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- 加密流程：

1\. 发起方生成随机临时密钥对。

2\. 用 ECDH 计算 shared secret。

3\. 从 shared secret 派生加密密钥 kE 和 MAC 密钥 kM。

4\. 用 AES‑128‑CTR 加密消息。

5\. 用 kM 计算 MAC，保证完整性。

6\. 发送密文和 MAC。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- 解密流程反向进行：提取临时公钥、ECDH 计算 shared secret、派生密钥、校验 MAC、AES‑128‑CTR 解密。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

### 节点身份与会话秘密

\- 每个节点有持久 secp256k1 密钥对，公钥为 Node ID，私钥长期保存但不频繁更换。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- 重要的密钥材料包括：static‑shared‑secret（长钥对+对方公钥）、ephemeral‑key（临时 ECDH）、shared‑secret、aes‑secret、mac‑secret 等。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- static‑shared‑secret 跨会话不变，容易受长期密钥泄露影响；ephemeral‑key 每次会话新生成，提供前向保密（forward secrecy）。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

### 消息分帧与完整性

\- 每个 frame 包含：header‑ciphertext、header‑mac、frame‑ciphertext、frame‑mac，用 AES 加密和 MAC 保护。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- 使用两个 keccak256 MAC 状态（入站/出站），随每个 frame 更新，防篡改。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- RLPx 支持在一个连接上多路复用多个“能力”（capabilities）/子协议。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

## 5\. P2P 能力与子协议

### p2p 基础能力与核心消息

\- “p2p” capability 必须支持，用于初始协商。核心消息包括：Hello、Disconnect、Ping、Pong 等。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- Hello：宣布支持的子协议列表和版本。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- Disconnect：优雅断开，可带原因码（例如协议违规、重复连接、身份不符等）。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

### 常见子协议

\- `eth`：主链数据交换，负责区块传播、交易转发等。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- `snap`：状态同步协议，让节点能按片加载状态 trie，加快同步。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- `les`：轻客户端协议，轻节点从全节点请求所需数据，无需存全状态。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- `portal`：Portal Network，用于轻量客户端的分布式状态/区块/交易检索网络。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))
<!-- DAILY_CHECKIN_2026-04-12_END -->

# 2026-04-11
<!-- DAILY_CHECKIN_2026-04-11_START -->














# 以太坊虚拟机 (EVM)

## 1\. 核心概念

\* **定义**: EVM（Ethereum Virtual Machine）是以太坊世界计算机的核心，负责执行智能合约的计算，并将结果永久存储在区块链上。

\* **状态机模型**: 以太坊本质上是一个\*\*基于交易的状态机\*\*。

\* **输入 (Input)**: 交易 (Transactions)

\* **状态转换函数**: EVM 本身。它根据输入和当前状态，计算出新的状态。

\* **输出 (Output)**: 新的世界状态 (World State)

\* **世界状态**: 20字节地址到账户状态（包含存储、代码、余额等）的映射。账户分为\*\*外部账户\*\*（私钥控制，无代码）和\*\*合约账户\*\*（由EVM代码控制）。

## 2\. 虚拟机范式与字节码

\* **为什么需要虚拟机?** 跨平台兼容。开发者只需为 EVM 编写代码，EVM 负责在不同操作系统的节点上统一执行。

\* **字节码 (Bytecode)**: 智能合约编译后的形式。由一系列 8 位（1字节）的指令组成。

\* **操作码 (Opcode)**: 指令本身（如 `ADD`, `MSTORE`）。

\* **操作数 (Operand)**: 操作码的输入参数。

\* **字长 (Word Size)**: EVM 的基本处理单元是 **32 字节 (256 bits)**。

## 3\. EVM 的四大核心数据位置

EVM 在执行期间通过四个主要位置存储和读取数据：

### ① 栈 (Stack)

\* **特点**: 后进先出 (LIFO)，用于存放计算的中间值和操作码的参数。

\* **限制**: 最大深度 1024 项，每项 32 字节。只能访问栈顶的 16 项（由 `DUP1-16` 和 `SWAP1-16` 决定）。栈耗尽会导致交易失败。

### ② 内存 (Memory)

\* **特点**: 临时存储 (非持久化)，用于存储整个程序生命周期内需要的大块数据。

\* **寻址**: 线性字节数组。初始值全为 0。

\* **计费机制**: 内存按“页”（32字节）动态扩展，扩展会消耗额外的 Gas。

\* **常用操作码**: `MSTORE` (存入一个字), `MSTORE8` (存入一字节), `MLOAD` (读取)。

### ③ 存储 (Storage)

\* **特点**: 永久存储，保存在区块链的世界状态中。与特定账户绑定，其他账户无法直接访问。

\* **结构**: 巨大的键值对数据库（基于字寻址的字数组，2^256 个槽位，每个槽位 32 字节）。

\* **成本**: 读写存储极度消耗 Gas（因为需要全网节点同步）。

\* **常用操作码**: `SSTORE` (写入), `SLOAD` (读取)。将存储值从非 0 置为 0 会获得 Gas 退款机制。

### ④ 调用数据 (Calldata)

\* **特点**: 只读数据。包含触发智能合约执行的原始交易数据或函数参数。

\* **常用操作码**: `CALLDATALOAD` (加载到栈), `CALLDATACOPY` (复制到内存)。

## 4\. 关键运行机制

\* **程序计数器 (Program Counter)**: 追踪字节码数组中下一条需要执行的指令位置。通过 `JUMP` 和 `JUMPDEST` 可以实现动态控制流（如循环、条件分支）。

\* **Gas 机制**:

\* **作用**: EVM 的计算资源货币，用于防止无限循环和 DoS 攻击。

\* **原理**: 每执行一个操作码都会消耗特定的 Gas。如果交易耗尽 Gas，EVM 会强制中止执行并回滚状态，但已消耗的 Gas 不会退还。

## 5\. 总结

高级语言（如 Solidity）编写的智能合约被编译为 EVM 字节码。EVM 作为状态转换引擎，通过操作码消耗 Gas，并在栈、内存、调用数据和存储之间处理数据，最终更新以太坊的世界状态。

# 以太坊区块构建与交易执行流程

## 1\. 核心架构背景

\* **节点组成**：以太坊节点由 **共识层 (CL)** 和 **执行层 (EL)** 组成。

\* **协作关系**：两者在每个时隙 (Slot) 协同工作以产出区块。CL 负责共识决策，EL 负责区块构建和交易执行。

\* **区块来源**：验证者不仅可以使用自身 EL 构建的区块，还可以通过 **PBS (提议者-构建者分离)** 广播外部构建者生成的区块。

## 2\. 有效负载构建流程 (Payload Building Routine)

区块构建的触发逻辑如下：

1\. **触发**：当共识层 (CL) 选定提议者后，通过 Engine API 的 `engine_forkchoiceUpdated` 端点指示执行层 (EL) 开始构建。

2\. **交易来源**：节点通过 **Gossip 协议** 在 P2P 网络中广播交易，验证通过后存入 **内存池 (Mempool)**。

3\. **构建环境**：CL 提供上下文信息（时间戳、块号、基础费用等），EL 据此初始化构建环境。

## 3\. 区块构建的简化逻辑 (以 Geth 为例)

区块构建是一个循环挑选交易的过程，核心逻辑如下：

```
// 简化模拟 Geth 的区块构建过程
func build(env environment, pool txpool.Pool, state state.StateDB) {
    gasUsed := 0
    // 循环直到 gas 达到限制 (30M) 或交易池为空
    for gasUsed < 30_000_000 && !pool.Empty() {
        transaction := pool.Pop() // 获取当前价值最高的交易
        
        // 在 EVM 中尝试运行交易
        res, gas, err := vm.Run(env, transaction, state)
        
        if err != nil {
            continue // 交易无效则跳过，尝试下一笔
        }
        
        // 累加 gas 并记录交易
        gasUsed += gas
        transactions.append(transaction)
    }
    // 最终处理：计算状态根、收据根、提款根等，封包区块
    return core.Finalize(env, transactions, state)
}
```

## 4\. 区块构建在 Geth 源码中的具体路径

![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/BareerahBenjamin/images/2026-04-11-1775890779970-image.png)

## 5\. 关键点

-   **Gas 限制**：单块 Gas 限制目前约为 30,000,000，决定了区块的容量上限。
    
-   **鲁棒性**：如果单笔交易执行失败（如 Gas 不足或合约逻辑报错），EL 会跳过该交易并继续尝试下一笔，以确保区块尽可能被填满。
    
-   **最终化 (Finalization)**：区块构建的最后一步是计算复杂的 Merkle 根（交易根、收据根等），这些根会被包含在区块头中。
    

# 执行层数据结构

以太坊执行客户端需要存储当前状态和历史数据。这些数据主要通过类似字典树（Trie）的结构——**默克尔帕特里夏树 (Merkle Patricia Trie, MPT)** 来组织和存储。

## 一、 底层基础数据结构

### 1\. 默克尔树 (Merkle Tree)

\* **核心作用**：数据完整性与防篡改验证。

\* **结构特点**：

_基于哈希的_**二叉树**（叶子节点数必须为偶数，奇数则复制最后一个）。

\* **叶子节点**存储实际数据，**非叶子节点**存储其子节点的哈希值。

_具有_**雪崩效应**：底层数据的微小改变会导致根哈希（Root Hash）完全改变。

\* **在以太坊中的应用**：区块头中只存储默克尔根（Merkle Root），以此指代整个底层数据集。

### 2\. 帕特里夏树 (Patricia Trie / Radix Tree)

\* **核心作用**：高效的数据存储与检索。

\* **结构特点**：

\* n 叉树结构。

\* **前缀压缩**：将具有相同前缀的路径合并为一条边（Edge），而不是每个字符占用一个节点。

_与普通字典树不同，帕特里夏树的_**值 (Value) 仅存储在叶子节点**，极大地提高了空间效率。

### 3\. 默克尔帕特里夏树 (MPT)

以太坊结合了上述两者的优点：**Merkle（密码学验证） + Patricia（高效键值存储）**。

\* **寻址方式**：使用半字节（Nibble，4位/1个十六进制字符）作为键值的路径导航单位。

\* **节点类型（3种）**：

1\. **分支节点 (Branch Node)**：长度为 17 的数组（16 个十六进制分支 + 1 个自身值），用于路径分叉。

2\. **扩展节点 (Extension Node)**：当路径没有分叉时，用于压缩共同的前缀路径，优化空间。

3\. **叶子节点 (Leaf Node)**：包含剩余的键路径（前缀）和最终的实际值（Value）。

## 二、 以太坊的“四大核心树”

以太坊在运行过程中维护着四棵修改版的 MPT 树。每个区块的区块头都会包含这前三棵树的**根哈希**。

### 1\. 交易树 (Transaction Trie)

\* **作用域**：区块级别（每个区块一棵树）。

\* **生命周期**：区块确认后**永久不变**。

\* **存储内容**：当前区块内的所有交易。

\* **键值对**`RLP(交易索引 Index) -> RLP(交易详情 Transaction)`。

### 2\. 收据树 (Receipt Trie)

\* **作用域**：区块级别（每个区块一棵树）。

\* **生命周期**：区块确认后**永久不变**。

\* **存储内容**：交易执行的结果（消耗的 Gas、日志/事件、状态码等）。

\* **核心用途**：允许轻客户端通过 Merkle 证明独立验证交易结果；全节点通过快照同步 (Snap Sync) 快速下载收据而无需重新执行历史交易。

\* **键值对**：交易在区块中的索引作为 Key。

### 3\. 世界状态树 (World State Trie)

\* **作用域**：全局唯一。

\* **生命周期**：动态演进，随每个区块的交易执行而**不断更新**。

\* **存储内容**：所有以太坊账户（包含 EOA 外部账户和智能合约账户）的当前最新状态。

\* **键值对**：

\* **Key**`keccak256(账户地址)`。哈希化可以防止恶意构造长路径攻击。

\* **Value**：RLP 编码的账户对象 `[nonce, balance, storageRoot, codeHash]`。

\* **核心意义**：这棵树的根哈希（State Root）是以太坊当前全局状态的密码学承诺。

### 4\. 存储树 (Storage Trie)

\* **作用域**：合约级别（每个智能合约账户一棵树，普通 EOA 账户此树为空）。

\* **生命周期**：动态演进（通过 `SSTORE` 写入`SLOAD` 读取）。

\* **存储内容**：特定智能合约内部的所有持久化变量和数据。

\* **键值对**`keccak256(存储槽索引 Slot Index) -> RLP(存储值)`。

\* **关联方式**：存储树的根哈希`storageRoot`）存储在世界状态树的对应账户叶子节点中。

## 三、 维克尔树 (Verkle Trees)

Verkle Tree `Vector Commitment` + `Merkle Tree`) 是以太坊路线图（The Verge阶段）中提议取代现有 MPT 的新一代数据结构。

### 1\. 为什么需要 Verkle Tree？

\* **MPT 的痛点**：由于是基于哈希的 16 叉树，树的深度很深。要证明一个叶子节点的值，需要提供整条路径上所有的兄弟哈希，导致**见证数据（Witness）极大**（约 4MB）。这阻碍了“无状态客户端”的实现。

### 2\. 结构与优势

\* **更宽更扁**：节点拥有 256 个子节点，大幅降低树的深度。

\* **向量承诺 (Vector Commitments)**：使用特殊的向量承诺取代普通的哈希计算。

\* **极小的证明体积**：不管树有多大，见证数据可以通过聚合承诺被压缩到极小。见证数据量从 4MB 锐减至 **150 KB** 左右。

### 3\. 核心目标

\* 能够将见证数据直接打包在区块中广播。

_实现真正的_**无状态客户端 (Stateless Clients)**：节点无需在本地硬盘存储数百 GB 的庞大以太坊全局状态，仅凭区块内携带的少量证明数据，即可验证状态转换的正确性，极大降低全节点的硬件门槛。
<!-- DAILY_CHECKIN_2026-04-11_END -->

# 2026-04-10
<!-- DAILY_CHECKIN_2026-04-10_START -->
















今天主要把[https://ethcoreplay.cc.cd/](https://ethcoreplay.cc.cd/)上的测试都做了一下，感觉不是很难
<!-- DAILY_CHECKIN_2026-04-10_END -->

# 2026-04-09
<!-- DAILY_CHECKIN_2026-04-09_START -->

















# 执行层客户端架构

## 一、 执行层架构概述

-   以太坊执行层客户端的核心职责不仅限于基础的交易执行，还包括区块链数据的验证与本地副本存储、通过 Gossip 协议与其他客户端进行网络通信、维护交易池，以及响应共识层（Consensus Layer, CL）的指令以驱动其功能 。
    
-   这种多方面的高效协作确保了以太坊网络的稳健性和数据完整性 。
    
-   架构的核心是由顶部的执行引擎（Execution Engine）驱动执行层，而执行引擎自身受控于以太坊共识层 。
    

## 二、 核心执行与状态组件

**以太坊虚拟机 (EVM)**：EVM 是专为以太坊程序设计的虚拟化执行引擎 。它旨在消除底层硬件架构（如 x86、ARM、RISC-V）指令集差异带来的不可预测性，确保在所有以太坊客户端上实现一致的计算结果，从而促成网络共识 。以太坊采用“三明治复杂性模型”作为设计理念，即外层保持极简（如高级语言 Solidity 与 EVM 字节码），而将所有复杂逻辑集中在中间层（即将 Solidity 编译为字节码的编译器） 。

**全局状态 (State)**：以太坊是一个按状态机运行的通用计算系统，其状态转换取决于输入的交易 。这与比特币基于未花费交易输出（UTXO）的模型存在显著差异，以太坊维护的是一个“全局状态” 。状态包含了全面的数据集合及数据结构（如 Merkle-Patricia 树），用于存储账户地址、余额、智能合约代码与网络当前状态等 。

**状态转换 (State Transition)**：EVM 内处理合法交易会触发状态转换，进而修改以太坊网络的全局状态 。在执行层中，区块内任何交易或标头的验证失败都会“污染”整个区块，导致整个区块被判定为无效 。

## 三、 通信与接口规范

执行层客户端采用不同的接口协议应对网络中不同的交互主体：

**DevP2P**：这是执行层客户端之间的对等（P2P）通信接口 。外部交易最初存放在内存池（Mempool）中，随后客户端通过 DevP2P 在网络中广播 。每个接收节点在进行二次广播前均会验证交易的合法性，以防止垃圾信息和无效状态转换 。

**JSON-RPC API**：作为面向外部用户（如钱包、DApp）的标准化通信层，用于查询以太坊状态或派发由钱包签名的交易 。

**Engine API**：专门用于共识层与执行层之间进行内部通信的受身份验证接口 。它基于 HTTP 提供 JSON-RPC 接口，并采用 JWT 令牌进行身份验证（证明发送方为合法的共识层客户端），不对外部公开 。这实现了清晰的职责分离：共识层处理分叉选择和共识，执行层负责验证并执行交易 。主要端点包括：

**Exchange of Capabilities**：在节点启动时协商支持的 API 版本（如 V1, V2, V3），确保双方协议兼容 。

**New Payload**：负责载荷验证及状态插入。当接收到新信标区块时，共识层调用此方法，执行层进而验证父哈希、执行交易并更新状态，返回状态反馈（如 VALID 或 INVALID） 。

**Fork Choice Updated**：管理状态同步并触发区块构建，更新规范链头部（Canonical Head） 。

## 四、 节点同步机制

为准确处理以太坊交易，执行层客户端需要就网络的全局状态达成共识，该过程由共识层的 LMD-GHOST 分叉选择规则触发 。

**全同步 (Full Sync)**：客户端请求自创世以来的每个区块头及区块体，在 EVM 中按顺序重放每一笔交易，逐步重建状态树 。此模式提供了最高的安全性保障，但消耗极大的 CPU、磁盘和网络资源，通常耗时数天 。注：在 EIP-4444 实施后，网络将不再支持从创世区块进行全同步，而是基于弱主观性检查点（Checkpoint）进行同步 。

**快照同步 (Snap Sync)**：以大幅降低时间成本为目标（从数天缩短至数小时），客户端选择一个近期敲定的区块作为枢纽块（Pivot block），直接获取并下载状态树叶子节点（账户和存储）的连续数据块以及关联合约字节码 。

**自愈阶段 (Healing Phase)**：由于在下载快照数据库期间区块链仍在推进，获取的数据可能出现过时或不一致。客户端通过扫描数据库并获取缺失的 Trie 节点或存储槽来进行数据自愈，确保能重建完整且合法的 MPT 树 。

## 五、 交易池与底层数据库

**1.交易池架构**：

**Legacy Pools (传统池)**：由执行客户端管理，利用双重堆结构（优先考虑未来区块的有效小费或 Gas 费上限）按价格对交易进行组织和淘汰 。

**Blob Pools**：同样采用优先级堆进行交易驱逐，但其内部操作机制包含对数函数等不同实现方式 。

**2.数据库后端引擎**：执行客户端需要将庞大的历史区块（古代数据库）及当前状态（近期状态树）持久化至磁盘 。

**LevelDB**：早期常用的嵌入式键值数据库，采用日志结构合并树（LSM-tree）。其设计虽有利于高吞吐量的顺序写入，但在以太坊极度频繁的状态更新场景下会导致严重的写入放大及不可预测的延迟 。

**Pebble**：部分执行客户端（如 Geth）已将 Pebble 作为默认数据库后端替代 LevelDB 。它同样基于 LSM-tree，但专为以太坊典型的高频写入和延迟敏感型工作负载进行了深度优化，可显著加强对数据压实（Compaction）行为的控制权 。此外，**MDBX** 也是部分客户端探索的高性能数据库方案之一 。

## 六、目前的客户端介绍

| 客户端 | 开发语言 | 核心开发者 | 核心杀手锏 (Noteworthy Features) |

| ------ | ------ | ------ | ------ |

| Geth | Go | 以太坊基金会 | 老牌霸主。生态最稳定、支持最广；拥有强大的自定义 EVM 追踪器 (Tracers) 和完善的监控面板。 |

| Nethermind | C# | Nethermind | 性能先锋。针对 .NET 基础设施深度优化，适合企业级集成，插件系统非常灵活。 |

| Besu | Java | Consensys / Hyperledger | 企业级首选。支持私有链、并行交易执行，是唯一在 Hyperledger 框架下的主流执行层客户端。 |

| Erigon | Go | Ledgerwatch | 存储优化专家。重新设计了 MPT 数据库，存档节点体积缩小 5 倍（3TB 以内），且能在 3 天内完成同步。 |

| Reth | Rust | Paradigm | 新晋性能怪兽。参考 Erigon 设计并用 Rust 重写，模块化程度极高，兼具极速同步与极小的磁盘占用。 |

如果是**普通开发者/节点运营商**，选 **Geth** 最稳；如果**磁盘空间有限**或需要分析全量历史数据，选 **Erigon** 或 **Reth**；如果是**企业用户**且需要私有链功能，选 **Besu**。
<!-- DAILY_CHECKIN_2026-04-09_END -->

# 2026-04-08
<!-- DAILY_CHECKIN_2026-04-08_START -->


















**以太坊执行层**

**一、执行层规范概述**

以太坊执行层最初在黄皮书中被完整定义，当时它涵盖了协议的全部体系。目前最新的规范为以太坊执行层规范（Ethereum Execution Layer Specification，简称 EELS）的 Python 实现版。黄皮书 Paris 版本（705168a，2024-03-04）已过时，未包含合并后的协议更新内容。Python 执行层规范是当前核心实现参考，所有 EIP 细节可查阅规范仓库的 README 文件。本笔记聚焦执行层的核心架构、设计逻辑，以及与状态转换、区块头验证、Gas 计费相关的关键规范。

**二、状态转换函数**

执行层的核心职责是排他性地执行状态转换函数（State Transition Function，简称 STF）。该函数解决两个根本问题：当前区块能否被合法追加到区块链末端，以及区块执行后系统状态会发生怎样的变化。黄皮书中定义的区块级状态转换函数简化公式为：

\\\[

\\sigma\_{t+1} \\equiv \\Pi(\\sigma\_t, B) \\tag{2}

\\\]

其中 \\(\\sigma\_{t+1}\\) 表示应用当前区块后的新状态，\\(\\Pi\\) 为区块级状态转换函数，\\(\\sigma\_t\\) 为父状态（旧状态），\\(B\\) 为当前待处理的区块。需要注意，公式中的 \\(\\sigma\\) 与 Python 规范中的 State 类概念不同，系统状态是通过状态折叠函数动态推导得出的，体现了区块链的数学模型与实际实现细节的分离。

Python 规范中状态转换函数的标准化执行流程依次为：获取父区块头、基于父区块头计算并校验超额 Blob Gas、完成当前区块头的全量合法性校验、验证叔块字段为空、执行区块内所有交易并输出 Gas 已使用量、特里树根、日志布隆过滤器及最终系统状态，随后核验执行结果与区块头参数完全匹配。若全部校验通过则将区块上链，否则抛出无效区块错误。最后清理超过最新 255 个区块的历史数据。

**三、区块头验证**

区块头验证是区块链同步与上链的核心环节，在黄皮书与 Python 规范中均有严格定义。其目标是基于协议规则校验区块的完整性，包括哈希、Gas 使用、时间戳等，确保节点可独立验证当前与历史数据。要验证当前区块头 \\(H\\) 的合法性，必须同时满足以下所有条件（依赖父区块 \\(P(H)\\))：

\\\[

\\begin{align\*}

V(H) &\\equiv H\_{\\text{gasUsed}} \\leq H\_{\\text{gasLimit}} \\tag{57a} \\\\

&\\land H\_{\\text{gasLimit}} < P(H)\_{H\_{\\text{gasLimit}}‘} + \\left\\lfloor \\frac{P(H)\_{H\_{\\text{gasLimit}}’}}{1024} \\right\\rfloor \\tag{57b} \\\\

&\\land H\_{\\text{gasLimit}} > P(H)\_{H\_{\\text{gasLimit}}‘} - \\left\\lfloor \\frac{P(H)\_{H\_{\\text{gasLimit}}’}}{1024} \\right\\rfloor \\tag{57c} \\\\

&\\land H\_{\\text{gasLimit}} > 5000 \\tag{57d} \\\\

&\\land H\_{\\text{timeStamp}} > P(H)\_{H\_{\\text{timeStamp}}'} \\tag{57e} \\\\

&\\land H\_{\\text{numberOfAncestors}} = P(H)\_{H\_{\\text{numberOfAncestors}}'} + 1 \\tag{57f} \\\\

&\\land \\text{length}(H\_{\\text{extraData}}) \\leq 32\\ \\text{bytes} \\tag{57g} \\\\

&\\land H\_{\\text{baseFeePerGas}} = F(H) \\tag{57h} \\\\

&\\land H\_{\\text{parentHash}} = \\text{KEC}(\\text{RLP}(P(H)\_H)) \\tag{57i} \\\\

&\\land H\_{\\text{ommersHash}} = \\text{KEC}(\\text{RLP}(())) \\tag{57j} \\\\

&\\land H\_{\\text{difficulty}} = 0 \\tag{57k} \\\\

&\\land H\_{\\text{nonce}} = 0x0000000000000000 \\tag{57l} \\\\

&\\land H\_{\\text{withdrawalHash}} \\neq \\text{nil} \\tag{57n} \\\\

&\\land H\_{\\text{blobGasUsed}} \\neq \\text{nil} \\tag{57o} \\\\

&\\land H\_{\\text{blobGasUsed}} \\leq \\text{MaxBlobGasPerBlock} = 786432 \\tag{57p} \\\\

&\\land H\_{\\text{blobGasUsed}} \\bmod \\text{GasPerBlob} = 2^{17} = 0 \\tag{57q} \\\\

&\\land H\_{\\text{excessBlobGas}} = \\text{CalcExcessBlobGas}(P(H)\_H) \\tag{57r}

\\end{align\*}

\\\]

超额 Blob Gas 计算公式为：

\\\[

\\text{CalcExcessBlobGas}(P(H)\_H) \\equiv

\\begin{cases}

0, & \\text{if } P(H)\_{\\text{blobGasUsed}} < \\text{TargetBlobGasPerBlock} \\\\

P(H)\_{\\text{blobGasUsed}} - \\text{TargetBlobGasPerBlock}, & \\text{其他情况}

\\end{cases} \\tag{57s}

\\\]

其中 \\(P(H)\_{\\text{blobGasUsed}} \\equiv P(H)\_{H\_{\\text{excessBlobGas}}} + P(H)\_{H\_{\\text{blobGasUsed}}}\\) 且 \\(\\text{TargetBlobGasPerBlock}=393216\\)。

这些规则的核心释义包括：Gas 使用不得超过上限、Gas 上限波动幅度不超过 1/1024 并设置最低 5000 的阈值、时间戳必须严格递增、区块高度依次递增、附加数据长度不超过 32 字节、基础费严格遵循 EIP-1559 计算。合并后还增加了叔块哈希为空、难度值为 0、nonce 全零等兼容性校验，以及提款哈希非空和 Blob Gas 的各项上限、对齐、超额计算规则。这些校验共同落地了以太坊的经济模型，通过 EIP-1559 降低 Gas 波动、提升交易可预测性、稳定发行奖励并销毁基础费，同时 EIP-4844 的 Blob 机制进一步扩展了 Layer 2 的低成本数据上链能力。

**四、Gas 计费**

Gas 计费是执行层经济模型的核心，分为固有 Gas 计算、有效 Gas 价格与优先级费、前置有效性校验三大部分。

**4.1 固有 Gas 计算**

固有 Gas 是一笔交易执行前必须支付的最低费用，涵盖 EVM 处理的基础资源与数据传输成本。适配上海升级的计算公式为：

![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/BareerahBenjamin/images/2026-04-08-1775654135007-image.png)

其中 g{0} 为总固有 Gas 成本，G{transaction} 固定为 21000 Gas，初始化代码按 32 字节对齐计费，输入数据零字节 4 Gas、非零字节 16 Gas，合约创建额外 32000 Gas，访问列表按地址与存储槽分别加收对应 Gas 成本。交易执行前会先从 Gas 上限中扣除固有 Gas，再初始化 EVM 上下文。

**4.2 有效 Gas 价格与优先级费**

适配 Blob 交易（类型 3）的有效 Gas 价格公式为：

![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/BareerahBenjamin/images/2026-04-08-1775654037752-image.png)

优先级费公式为：

![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/BareerahBenjamin/images/2026-04-08-1775653679005-image.png)

有效 Gas 费用为 effectiveGasPrice \* T{gasLimit}，总 Blob Gas 为 G{gasPerBlob}=2^17 \* length(T{blobVersionedHashes})，Blob Gas 价格在未超出目标时固定为 1，超出时呈指数上涨，Blob Gas 费用全部销毁。最高 Gas 费用和前置总成本 v{0} ≡ upfrontCost ≡ (effectiveGasFee + blobGasFee) 共同构成交易执行前的全额扣除项。

**4.3 交易执行前置有效性校验**

交易必须通过以下全部校验才能进入执行阶段：

![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/BareerahBenjamin/images/2026-04-08-1775653621927-image.png)

其中 m 根据交易类型取 Gas Price 或 Max Fee Per Gas，空账户定义为代码哈希为空、nonce 为 0 且余额为 0。这些规则确保发送者账户存在且为外部账户、nonce 顺序正确、余额充足、Gas 价格不低于基础费且符合 EIP-1559 约束，最终保障交易在执行前已满足所有协议安全与经济要求。
<!-- DAILY_CHECKIN_2026-04-08_END -->

# 2026-04-07
<!-- DAILY_CHECKIN_2026-04-07_START -->



















# 协议

## 架构

![Image_20260407_020432.jpg](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/BareerahBenjamin/images/2026-04-06-1775498699238-Image_20260407_020432.jpg)

## 设计原理：

1.  协议设计理念：简洁性、通用性（EVM）、模块化（面向未来）、非歧视性（开源、密码朋克）、敏捷性（协议修改）
    
2.  原则：管理复杂性（三明治模型复杂性（中间层）、封装式复杂性（复杂子系统→简单接口））、自由、泛化、无预设功能。
    
3.  区块链级协议：  
    **账户模型与 UTXO 模型对比**  
    早期区块链如比特币及其衍生品采用 UTXO 模型，将用户余额存储为未花费交易输出（UTXO）。每个 UTXO 代表一笔已授权、可供接收方花费的特定金额加密货币，用户余额就是其私钥能签名花费的所有 UTXO 总和。以太坊则选择账户模型，余额直接记录在账户中。这种模型更灵活，能支持复杂交易场景，尤其适合去中心化交易所等应用。UTXO 虽能提供更高隐私性，但会增加系统复杂度，而账户模型的可互换性（fungibility）更强，任何代币都能相互替换，不会因历史交易而被“污染”。
    
    **账户模型的优势**  
    账户模型显著节省存储空间。例如，一个账户若有 5 个 UTXO，UTXO 模型需约 300 字节，而账户模型只需约 30 字节（包含地址、余额和 nonce）。实际中账户需存储在 Patricia 树里，节省幅度略小，但仍很可观。同时，交易体积更小（以太坊约 100 字节，对比比特币 200-250 字节），因为每笔交易只需一次引用、一次签名和一个输出。账户模型实现简单、易于理解，也更强大，能轻松实现 UTXO 模型无法支持的功能，如不绑定特定 UTXO 的卖单。唯一缺点是为防重放攻击需引入 nonce，旧账户难以完全裁剪；解决方案是让交易携带区块号，定期重置 nonce。
    
    **以太坊状态数据结构：Merkle Patricia Trie（MPT）**  
    以太坊使用改进的 Merkle-Patricia Trie 存储状态，它结合 Patricia 算法和 Merkle 树特性，实现高效检索。MPT 具有确定性和密码学可验证性：状态根哈希唯一对应当前所有状态，任何修改都会改变根哈希，支持 O(log n) 的插入、查找和删除效率。通过 Merkle 证明，可轻松验证两个状态是否一致。目前 MPT 广泛用于需要网络传输成员证明的场景，但以太坊状态已达 1-2TB，节点存储压力巨大，因此社区正在研究更高效的替代方案。
    
    **Verkle 树：下一代状态结构**  
    Verkle 树（向量承诺 Merkle 树）是正在积极研究的替代方案，旨在解决 MPT 的状态膨胀问题。它通过更高的分支因子 k，实现 O(kn) 构造时间和 O(log\_k n) 的成员证明大小，大幅减少证明体积（称为 witness）。与传统 Merkle 树需提供所有兄弟节点哈希不同，Verkle 树只需提供路径上的父节点即可完成证明，这对实现“无状态客户端”（statelessness）至关重要，能让节点无需完整存储全状态即可验证交易。目前 Verkle 树仍在研发阶段，未来有望取代 MPT，进一步提升网络可扩展性。
    
    **序列化方案：RLP 与 SSZ**  
    以太坊早期使用 Recursive Length Prefix（RLP）序列化数据。RLP 设计简单且确定性强，不依赖具体数据类型，只以嵌套数组形式存储结构，保证字节级一致性，不支持键值映射（需手动转为有序数组）。但 RLP 对固定长度数据（如整数、布尔值）效率不高，且不支持 Merkleization，导致轻客户端证明和无状态性难以实现。Ethereum 2.0 Beacon 链引入 Simple Serialize（SSZ）作为改进方案。SSZ 依赖预先定义的 schema，支持可变和固定长度数据，具有高效重哈希和快速索引能力，复杂度远低于 RLP 的 O(n)，完美支持 Merkleization，是实现无状态性和轻客户端证明的关键。
    
    **共识最终性机制：从 Casper FFG 到 Gasper**  
    以太坊 PoS 共识中，最终性（finality）指区块一旦最终确定，除非销毁至少 33% 质押 ETH，否则不可被篡改或移除。Casper Friendly Finality Gadget（FFG）是实现这一目标的叠加协议，它在提案机制之上通过两轮投票最终确定检查点，所有最终检查点成为规范链历史。LMD-GHOST（Latest Message Driven Greediest Heaviest Observed Sub-Tree）则是分叉选择规则，验证者通过认证区块表达支持，选择最重子树作为规范链。两者结合形成 Gasper 协议，成为 Ethereum 2.0 完整的权益证明共识抽象，既保证最终性，又能高效处理分叉。
    
    **P2P 网络中的 DHT 应用**  
    以太坊 P2P 网络同时使用结构化和非结构化技术。底层采用 discv5（基于 Kademlia 的 DHT）存储 ENR 记录，用于节点发现和路由表构建。新节点通过引导节点在 DHT 中查找自身 node\_id，逐步发现其他对等节点，并定期枚举随机 node\_id 扩展网络视图。区块传播则依赖上层 GossipSub 覆盖网络（非结构化）。DHT 虽增加一步，但提供全局视图和简单设计，适合高动态网络的引导启动，比单纯朋友间传播（f2f/PEX）更具鲁棒性。这种混合模式在 BitTorrent 等协议中也被广泛采用，确保节点高效加入并传播最新区块。
    

## 进化路径

**Frontier 阶段（边境）**  
Frontier 是以太坊协议的首个正式发布版本，于 2015 年 7 月 30 日凌晨 3:26:13（UTC）上线，对应创世区块时间戳。它本质上是一次 beta 测试发布，目的是让开发者学习、实验并开始构建去中心化应用和工具。初始 gas 上限被硬编码为 5000，以确保矿工和用户能快速安装客户端并启动网络。随后通过 Frontier Thawing 分叉将 gas 上限提升至约 314 万（精确值为 3,141,592）。为了实现安全启动，Frontier 引入了金丝雀合约机制，这些合约会发出 0 或 1 的二进制信号。如果客户端检测到多个金丝雀合约信号变为 1，就会自动停止挖矿并提示用户升级客户端，从而避免长时间网络中断，确保矿工不会阻碍协议升级。

**Homestead 阶段（家园）**  
Homestead 是以太坊协议的第二个主要版本，于 2016 年 3 月 14 日正式发布，标志着以太坊从 beta 测试阶段过渡到更加成熟稳定的生产环境。这一阶段通过多项 EIP 进行了重要改进。EIP-2 修复了多项问题，包括将通过交易创建合约的 gas 成本从 21,000 提高到 53,000，以减少不必要的交易式合约创建；同时确保合约创建要么完全成功、要么彻底失败，防止出现空合约；还优化了难度调整算法，使区块时间更加稳定。EIP-2 还引入了高 s 值签名无效规则，解决交易可塑性问题，提升交易哈希的可靠性和完整性。EIP-7 在 opcode 0xf4 位置新增 DELEGATECALL 操作码，其功能类似 CALLCODE，但会将 sender 和 value 从父作用域传递到子作用域，方便合约将其他地址作为可变代码源并“透传”调用，且不会额外增加 gas 津贴，使 gas 管理更可预测。EIP-8 则增强了 devp2p 协议的向前兼容性，要求客户端在处理 hello、ping 数据包时忽略版本号和额外元素，静默丢弃未知包类型，并支持新的加密握手编码，从而确保未来网络升级时所有客户端都能平滑过渡，符合 Postel 定律的宽松接受原则。

**The Merge（合并）**  
2022 年 9 月 15 日，以太坊通过激活 EIP-3675 完成了 The Merge 升级，将共识机制从工作量证明（PoW）正式切换为权益证明（PoS）。在此之前，PoW 与执行逻辑处于同一层级，The Merge 后 PoW 被彻底弃用，取而代之的是更加复杂且节能的 PoS 共识。该共识被独立部署为单独一层，即信标链（Beacon Chain），拥有独立的 P2P 网络和逻辑。信标链自 2020 年 12 月 1 日起已独立运行并达成共识，经过长时间无故障稳定表现后，正式成为以太坊的主共识提供者。“The Merge”这一名称正是源于执行层与共识层的成功合并，标志着以太坊从能源密集型挖矿转向更高效、更环保的权益证明时代，同时为后续协议演进奠定了模块化基础。

### （拓展）

**Shapella 升级（Shanghai/Capella）**  
Shapella 是 The Merge 之后的首个重大协议升级，于 2023 年 4 月 12 日正式激活。它同时对执行层（Shanghai）和共识层（Capella）进行协调升级，主要实现了质押 ETH 的提款功能。通过 EIP-4895 等关键改进，验证者终于能够将信标链上锁定的 ETH 及累积奖励提取到执行层，彻底完成了 PoS 转型的最后一块拼图。这一升级极大提升了 staking 的灵活性和用户参与度，同时还引入了多项优化，如 EIP-3651（warm COINBASE）、EIP-3855（PUSH0 操作码）和 EIP-3860（合约创建 gas 优化），让开发者体验更流畅，也为后续扩展奠定了基础。

**Dencun 升级（Cancun/Deneb）**  
Dencun 升级于 2024 年 3 月 13 日上线，是 Ethereum 扩展路线图中的重要里程碑。它通过执行层（Cancun）和共识层（Deneb）的协同升级，首次引入了 Proto-Danksharding 机制（核心 EIP-4844），为 Layer 2 提供了廉价的临时数据可用性（blobs）。这一创新大幅降低了 Rollup 的交易成本，让 Layer 2 费用显著下降，同时保留了主链的安全性。Dencun 还包含其他改进，如 EIP-4788（信标根存储）和多项 gas 优化，进一步推动了 Ethereum 向高吞吐量、可扩展网络的演进，为大规模采用铺平道路。

**Pectra 升级（Prague/Electra）**  
Pectra 升级于 2025 年 5 月 7 日激活，是 The Merge 以来 EIP 数量最多的一次硬分叉（共 11 个 EIP）。它同时升级了执行层（Prague）和共识层（Electra），重点引入了账户抽象能力（EIP-7702），允许外部拥有账户（EOA）临时执行智能合约代码，实现交易批量处理、gas 赞助和社会恢复等高级功能。这一升级还优化了验证者体验：EIP-7251 将最大有效验证者余额提升至 2048 ETH，EIP-6110 显著缩短了验证者入网时间，并进一步扩大了 blob 吞吐量。Pectra 极大提升了用户体验和网络效率，被视为 Merge 后最具影响力的协议改进之一。

**Fusaka 升级**  
Fusaka 升级于 2025 年 12 月 3 日完成，标志着 Ethereum 开始进入半年一次的固定升级节奏。它主要聚焦进一步扩展和数据可用性优化，核心引入 PeerDAS（EIP-7594）机制，让验证者通过随机采样而非完整下载即可验证 blob 数据，从而将 blob 容量理论上提升 8 倍。同时伴随多个 Blob Parameter Only（BPO）小升级，逐步提高每区块 blob 数量。这一升级延续了 Dencun 和 Pectra 的扩展成果，进一步降低 Layer 2 成本，并为后续并行执行等高级特性打下基础。

**后续发展展望**  
截至 2026 年 4 月，Glamsterdam 升级（计划于 2026 年上半年）已在开发中，重点方向包括并行执行、更高的 gas 限制、内置 PBS（Proposer-Builder Separation）以及持续的 blob 扩展和隐私增强。其后的 Hegotá 升级也已规划，Ethereum 正稳步迈向“完全扩展、极致韧性”的长期目标。这些升级共同体现了协议在安全性、可扩展性和用户友好性上的持续演进。
<!-- DAILY_CHECKIN_2026-04-07_END -->

# 2026-04-06
<!-- DAILY_CHECKIN_2026-04-06_START -->





















历史发展：

ARPANET（互联网开放无国界精神）→ Unix哲学（模块化、简洁、可组合的软件设计）→ 公钥密码学（去中心化系统的安全底层）→ 自由开源运动（全球化分布式协作范式、代码自由）→ 密码朋克运动（代码即规则、个人主权、去中心化理念）→ 早期数字货币理论与实践 → 比特币（去中心化账本与数字货币的落地验证）→ 以太坊（通用去中心化世界计算机，完整继承并落地了半个世纪的核心理念）

关键人物及其贡献：

| 领域 | 核心人物 | 关键贡献 |

|---------------|-----------------------------------|-----------------------------------------------|

| 软件设计根基 | 肯·汤普森、丹尼斯·里奇 | 发明Unix系统与C语言，奠定Unix哲学 |

| 开源运动 | 理查德·斯托曼 | 发起GNU项目，创造GPL协议，定义自由软件核心理念 |

| 开源运动 | 林纳斯·托瓦兹 | 开发Linux内核，奠定分布式开源协作范式 |

| 现代密码学 | 拉尔夫·默克尔、迪菲、赫尔曼 | 开创非对称加密与无需信任的密钥交换体系 |

| 现代密码学 | 李维斯特、萨莫尔、阿德曼 | 发明RSA公钥加密系统，落地现代密码学商用 |

| 密码朋克 | 蒂莫西·梅、埃里克·休斯 | 发起密码朋克运动，提出加密无政府主义核心理念 |

| 密码朋克 | 菲尔·齐默尔曼 | 开发PGP加密程序，推动加密技术向公众普及 |

| 数字货币先驱 | 大卫·乔姆、戴伟、尼克·萨博 | 早期数字货币与数字稀缺性的理论与实践探索 |

| 比特币 | 中本聪 | 发明比特币与区块链，实现去中心化点对点电子现金 |

| 以太坊 | 维塔利克·布特林、加文·伍德 | 设计并落地以太坊，打造通用智能合约平台 |
<!-- DAILY_CHECKIN_2026-04-06_END -->
<!-- Content_END -->
