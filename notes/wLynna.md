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
# 2026-04-19
<!-- DAILY_CHECKIN_2026-04-19_START -->
今天周日 想休息

下周好悬啊，一周HK
<!-- DAILY_CHECKIN_2026-04-19_END -->

# 2026-04-17
<!-- DAILY_CHECKIN_2026-04-17_START -->

04/17 继续降维学习

# 🧠 Consensus Layer（CL）— Human Version

* * *

## 1️⃣ What is Consensus Layer?

Consensus Layer is the part of Ethereum that decides which blocks are accepted.  
共识层是以太坊中负责决定“哪些区块被接受”的那一层。

* * *

It answers one core question:  
“Which version of the blockchain is the true one?”  
它解决一个核心问题：  
👉 哪一条链才是真正的链？

* * *

Without CL, everyone could have different versions.  
如果没有共识层，每个人看到的链可能都不一样。

* * *

👉 So:

CL creates shared truth.  
共识层创造“共同认可的真相”。

* * *

## 2️⃣ What does CL actually do?

CL coordinates validators to agree on blocks.  
共识层协调验证者对区块达成一致。

* * *

It decides:

-   who proposes a block
    
-   who checks the block
    
-   when the block is finalized  
    

它决定：

-   谁来出块
    
-   谁来验证
    
-   什么时候最终确认  
    

* * *

👉 Simple idea:

CL = coordination + agreement  
CL = 协调 + 共识

* * *

## 3️⃣ Who are validators?

Validators are participants who secure the network.  
验证者是维护网络安全的参与者。

* * *

They stake ETH to join the system.  
他们需要质押 ETH 才能参与。

* * *

If they behave honestly, they earn rewards.  
如果行为诚实，会获得奖励。

* * *

If they cheat, they get punished.  
如果作弊，会被惩罚（slash）。

* * *

👉 Key idea:

Validators = decision makers  
验证者 = 决策参与者

* * *

## 4️⃣ How a block is created (CL view)

* * *

CL selects a validator randomly.  
共识层随机选择一个验证者。

* * *

This validator becomes the proposer.  
这个验证者成为出块者。

* * *

The proposer builds a block with transactions.  
出块者把交易打包成区块。

* * *

Other validators check the block.  
其他验证者检查这个区块。

* * *

If enough validators agree, the block is accepted.  
如果足够多的验证者同意，这个区块就被接受。

* * *

👉 Simple flow:

Select → Propose → Validate → Accept  
选择 → 出块 → 验证 → 接受

* * *

## 5️⃣ What is finality?

Finality means the block cannot be changed anymore.  
最终性（finality）意味着区块不可再被更改。

* * *

Before finality, a block might be replaced.  
在最终确认前，区块有可能被替换。

* * *

After finality, it becomes permanent.  
一旦最终确认，就永久不可逆。

* * *

👉 Simple idea:

Finality = irreversible truth  
最终性 = 不可改变的真相

* * *

## 6️⃣ Why staking is important

Validators must lock ETH as stake.  
验证者必须锁定 ETH 作为质押。

* * *

This gives them skin in the game.  
这让他们“有代价、有责任”。

* * *

If they try to cheat, they lose money.  
如果他们作弊，会损失资金。

* * *

👉 So:

Security comes from economic incentives.  
安全性来自经济激励机制。

* * *

## 7️⃣ Why CL exists separately

* * *

Execution is complex (EVM, contracts).  
执行很复杂（EVM、合约等）。

* * *

Consensus is also complex (agreement, timing).  
共识也复杂（协调、时间机制）。

* * *

Separating them makes both simpler.  
把它们分开，让两边都更简单。

* * *

👉 This is a major upgrade in Ethereum design.  
👉 这是以太坊设计中的一个重要升级。

* * *

## 8️⃣ What CL does NOT do

\==CL does not execute smart contracts.==  
共识层不执行智能合约。

* * *

CL does not compute balances.  
共识层不计算余额。

* * *

👉 That is EL’s job.  
👉 那是执行层的工作。

* * *

## 🧭 Final mental model（最重要）

* * *

CL selects who can propose blocks.  
CL 选择谁来出块。

* * *

EL executes the transactions inside the block.  
EL 执行区块中的交易。

* * *

CL confirms and finalizes the block.  
CL 确认并最终确定这个区块。

* * *

👉 One sentence:

CL decides truth, EL produces results.  
CL 决定“真相”，EL 产生“结果”。
<!-- DAILY_CHECKIN_2026-04-17_END -->

# 2026-04-16
<!-- DAILY_CHECKIN_2026-04-16_START -->


04/16 继续降维学习

EL vs CL — The Real Separation

* * *

## 1️⃣ Why do we even have two layers?

Ethereum used to be one system, but now it is split into two layers.  
以太坊以前是一个整体系统，现在被拆成了两层。

This split makes the system more modular and scalable.  
这种拆分让系统更模块化，也更容易扩展。

* * *

Execution Layer handles “what happens”.  
执行层负责“发生什么”。

Consensus Layer handles “who decides”.  
共识层负责“谁说了算”。

* * *

👉 This separation is very important.

It allows Ethereum to evolve faster and safer.  
这个分层非常重要，让以太坊可以更快、更安全地演进。

* * *

## 2️⃣ What does EL do again?

Execution Layer executes transactions and updates the state.  
执行层执行交易，并更新状态。

* * *

It does not care who created the block.  
它不关心是谁创建了区块。

* * *

It only cares:  
“Are these transactions valid, and what is the new state?”  
它只关心：这些交易是否合法，以及新的状态是什么。

* * *

👉 EL is like a calculator.

You give it input, it gives you the result.  
执行层就像计算器，你给输入，它给结果。

* * *

## 3️⃣ What does CL do?

Consensus Layer decides who can propose and validate blocks.  
共识层决定谁可以出块、谁来验证区块。

* * *

It runs mechanisms like staking and validator selection.  
它运行质押、验证者选择等机制。

* * *

It ensures everyone agrees on the same chain.  
它确保所有人对同一条链达成一致。

* * *

👉 CL is about agreement.

Not computation, but coordination.  
共识层是关于“达成一致”，不是计算，而是协调。

* * *

## 4️⃣ How EL and CL work together

* * *

CL selects a validator to propose a block.  
共识层选择一个验证者来出块。

* * *

The validator builds a block with transactions.  
验证者把交易打包成一个区块。

* * *

EL executes those transactions and computes the new state.  
执行层执行这些交易，并计算出新的状态。

* * *

CL then verifies and finalizes the block.  
共识层再来验证并最终确认这个区块。

* * *

👉 Simple flow:

CL → choose proposer  
EL → execute transactions  
CL → finalize block

共识层选人 → 执行层计算 → 共识层确认

* * *

## 5️⃣ Why this separation matters

* * *

It reduces complexity in each layer.  
这种拆分降低了每一层的复杂度。

* * *

Developers can improve EL without touching CL.  
开发者可以优化执行层，而不用改共识层。

* * *

And vice versa.  
反之亦然。

* * *

It also allows different implementations.  
也允许不同的客户端实现。

* * *

👉 This is why Ethereum is resilient.

因为模块化，让系统更有韧性。

* * *

## 6️⃣ A simple analogy (很好用)

* * *

Think of a company.

把以太坊想象成一家公司。

* * *

EL is the “operations team”.  
执行层是“执行团队”。

They do the actual work.  
他们负责干活。

* * *

CL is the “governance team”.  
共识层是“决策/治理团队”。

They decide who is in charge and what is accepted.  
他们决定谁有权、什么被认可。

* * *

👉 Without EL → nothing gets done  
没有执行层 → 什么都不会发生

👉 Without CL → chaos, no agreement  
没有共识层 → 一片混乱，没有共识

* * *

## 7️⃣ One key sentence (非常重要)

* * *

Ethereum works because execution and consensus are separated but coordinated.  
以太坊之所以能运转，是因为执行和共识分离，但又协同。

* * *

## 8️⃣ Why this matters for YOU（这才是关键）

* * *

ENS runs on Ethereum, so it depends on both layers.  
ENS 运行在以太坊上，因此依赖这两层。

* * *

When you register a name, it is a transaction → handled by EL.  
当你注册 ENS 名字，这是一个交易 → 执行层处理。

* * *

But whether that transaction is accepted → decided by CL.  
但这个交易是否被确认 → 由共识层决定。

* * *

👉 So:

EL = execution of ENS logic  
CL = security and finality of ENS

EL = ENS 的执行  
CL = ENS 的安全与确认
<!-- DAILY_CHECKIN_2026-04-16_END -->

# 2026-04-15
<!-- DAILY_CHECKIN_2026-04-15_START -->



04/15 EL 协议规范，对我来说太难了，远远超纲，所以 降维学习

🧠 Execution Layer（EL）— Human Version // 降维学习

* * *

# 🧠 Execution Layer（EL）

* * *

## 1️⃣ What is Execution Layer?

Execution Layer is the part of Ethereum that runs transactions and updates the state.  
执行层是以太坊中负责运行交易、并更新整个系统状态的部分。

It is like the “engine” of Ethereum that actually does the work.  
它就像以太坊的“引擎”，真正干活的地方。

When someone sends a transaction, EL is the place where it gets executed.  
当有人发送交易时，就是在执行层被真正执行。

So if you ask “what really happened on-chain”, the answer is inside EL.  
所以如果你问“链上到底发生了什么”，答案就在执行层。

* * *

## 2️⃣ What is “state” in Ethereum?

State means all the current data of the blockchain.  
状态就是区块链当前所有的数据。

It includes account balances, smart contract code, and stored data.  
包括账户余额、智能合约代码、以及存储的数据。

You can imagine state as a global database shared by all nodes.  
你可以把状态想象成一个所有节点共享的“全局数据库”。

Every transaction will try to change this database.  
每一笔交易，都会尝试去改变这个数据库。

* * *

👉 One key idea:

A transaction is just a state change.  
一笔交易，本质上就是一次状态改变。

* * *

## 3️⃣ What does EL actually do?

EL takes transactions and applies them to the current state.  
执行层接收交易，并把它们作用到当前状态上。

It processes them one by one, in a specific order.  
它按顺序一笔一笔执行交易。

After execution, the state becomes a new state.  
执行完成后，状态就变成了新的状态。

This process is called “state transition”.  
这个过程叫做“状态转换”。

* * *

👉 Very simple flow:

Old state → Transactions → Execution → New state  
旧状态 → 交易 → 执行 → 新状态

* * *

## 4️⃣ How a transaction works (step by step)

* * *

A user creates a transaction from a wallet.  
用户从钱包发起一笔交易。

This transaction may transfer ETH or call a smart contract.  
这笔交易可能是转账，也可能是调用智能合约。

* * *

The network first checks if the transaction is valid.  
网络首先会检查这笔交易是否有效。

It checks signature, nonce, and gas.  
会检查签名、nonce（防止重复）、以及 gas 是否合理。

* * *

Then the transaction is included into a block.  
接着这笔交易会被打包进一个区块。

* * *

Inside the block, the EVM executes the transaction.  
在区块中，EVM 会执行这笔交易。

* * *

Execution may change balances or contract storage.  
执行过程中可能改变账户余额或合约存储。

* * *

Finally, the state is updated, and gas is paid.  
最终状态被更新，同时用户支付 gas 费用。

* * *

👉 One sentence summary:

A transaction goes from “intent” to “state change”.  
一笔交易，从“意图”变成“状态改变”。

* * *

## 5️⃣ What is EVM?

EVM is the virtual machine of Ethereum.  
EVM 是以太坊的虚拟机。

It is like a global computer that runs smart contracts.  
它就像一个全球共享的计算机，专门运行智能合约。

* * *

Every node runs the same EVM computation.  
每个节点都会运行相同的 EVM 计算。

This ensures that everyone gets the same result.  
这保证了所有人得到相同的结果。

* * *

EVM does not trust anyone, it just follows rules.  
EVM 不相信任何人，它只按照规则执行。

* * *

👉 Key idea:

EVM = deterministic execution  
EVM = 确定性执行（所有人算出来都一样）

* * *

## 6️⃣ Why do we need Gas?

Gas is used to measure computation.  
Gas 用来衡量计算量。

Every operation in EVM costs gas.  
EVM 中的每一个操作都要消耗 gas。

* * *

More complex operations cost more gas.  
越复杂的操作，消耗越多 gas。

* * *

Gas also prevents spam.  
Gas 还能防止垃圾攻击。

If computation were free, people could overload the network.  
如果计算免费，网络会被无限滥用。

* * *

Users must pay for what they use.  
用户必须为自己使用的资源付费。

* * *

👉 Simple understanding:

Gas is the price of using Ethereum.  
Gas 就是使用以太坊的价格。

* * *

## 7️⃣ ==What is a block in EL?

A block is a list of transactions grouped together.  
一个区块就是一组被打包在一起的交易。

* * *

EL executes all transactions inside the block.  
执行层会执行区块里的所有交易。

* * *

Transactions are executed in order.  
交易是按顺序执行的。

Order matters, because earlier transactions can change the state.  
顺序很重要，因为前面的交易会影响后面的状态。

* * *

After all transactions are executed, we get a new state.  
所有交易执行完后，我们得到新的状态。

* * *

👉 Key idea:

Block = a batch of state transitions  
区块 = 一批状态转换

* * *

## 8️⃣ What EL does NOT do

Execution Layer does not decide who creates blocks.  
执行层不决定谁来出块。

* * *

Execution Layer does not handle consensus.  
执行层不负责共识。

* * *

That is the job of the Consensus Layer.  
这些是共识层（CL）的职责。

* * *

👉 Important separation:

EL decides “what happens”  
CL decides “who decides”

EL 决定“发生什么”  
CL 决定“谁说了算”

* * *

## 🧭 Final mental model（最重要的整体理解）

* * *

User sends a transaction, and EL executes it using EVM, then updates the state.  
用户发送交易，执行层用 EVM 执行它，然后更新状态。

* * *

Blocks are just containers of transactions.  
区块只是交易的容器。

* * *

Ethereum is basically a state machine.  
以太坊本质上是一个状态机。

* * *

Each block moves the system from one state to another.  
每一个区块，都让系统从一个状态变到另一个状态。

* * *

👉 最核心一句

Ethereum is a global state machine, and EL is the part that runs the machine.  
以太坊是一个全球状态机，而执行层就是让这个机器运转的部分。

* * *
<!-- DAILY_CHECKIN_2026-04-15_END -->

# 2026-04-14
<!-- DAILY_CHECKIN_2026-04-14_START -->




# [Execution Layer Specification](https://epf.wiki/#/wiki/EL/el-specs?id=execution-layer-specification)

看的很晕，纠结要不要放弃？
<!-- DAILY_CHECKIN_2026-04-14_END -->

# 2026-04-12
<!-- DAILY_CHECKIN_2026-04-12_START -->





04/12

Protocol History and Evolution

## 一、整体理解：协议不是一次设计，而是不断演化

**EN:** Ethereum protocol is not designed once and fixed, but evolves through continuous upgrades.  
**中:** 以太坊协议并不是一次性设计完成的系统，而是在实践中不断迭代升级的过程。

这一部分的核心不是记住每个升级细节，而是理解：  
👉 Ethereum 是一个“live system”（活系统），通过 **EIPs + hard fork** 持续演进。  
👉 每一阶段都在解决“上一阶段暴露的问题”，逐步走向更稳定、更复杂的系统。

* * *

## 二、Frontier：起点是“可运行”，不是“完美”

**EN:** Frontier was a beta launch focused on enabling developers to experiment and build.  
**中:** Frontier 是一个偏实验性质的 beta 版本，重点是“让系统跑起来”。

Frontier（2015）标志着 Ethereum 主网上线，它的设计思路非常务实：  
不是追求完美，而是先让开发者可以部署合约、开始实验。

关键特点包括：

-   Gas limit 很低（5000），用于控制系统风险
    
-   允许开发者开始构建 dApp 和工具
    
-   引入 “canary contracts”（金丝雀机制）用于升级协调  
    

👉 **理解重点：**  
Frontier 本质是一个“可启动系统（bootstrapping phase）”，  
目标是：让网络活起来，而不是优化体验或性能。

* * *

## 三、Homestead：从“能用”到“更可靠”

**EN:** Homestead marked Ethereum’s transition from experimental to more stable and mature.  
**中:** Homestead 标志着以太坊从实验阶段进入更稳定、可用的阶段。

Homestead（2016）开始处理 Frontier 中暴露的现实问题，重点在于：  
👉 提升系统可靠性、安全性、以及规则的严谨性

几个关键改进 ：

1\. Gas 机制调整，从“鼓励尝试” → “开始约束行为”

2\. 交易签名安全（malleability 修复），提升交易不可篡改性（integrity）

3\. Out-of-gas 处理更严格，系统从“宽松容错” → “明确失败机制”

4\. Difficulty 调整算法优化，开始关注系统运行的稳定性（performance consistency）

5\. 新 opcode：DELEGATECALL（EIP-7），支持更灵活的合约调用结构（如代理模式）。

6\. 网络协议前向兼容（EIP-8）

* * *

## 四、The Merge：架构级跃迁（不是小修小补）

**EN:** The Merge replaced Proof-of-Work with Proof-of-Stake.  
**中:** The Merge 将共识机制从 PoW 切换为 PoS。

The Merge（2022）不是“优化”，而是一次**架构级重构**：Ethereum 从“单层系统” → “模块化分层系统”

### 核心变化

-   从 PoW → PoS（不再需要挖矿）
    
-   引入 Beacon Chain（独立的共识层）
    
-   执行层（Execution）与共识层（Consensus）分离，协议可以独立演进不同层，为未来扩展（如 rollups、Danksharding）打基础  
    

* * *

## 五、总结：从历史看到设计逻辑

### 1️⃣ Frontier

👉 目标：让系统存在（make it exist）

### 2️⃣ Homestead

👉 目标：让系统可靠（make it reliable）

### 3️⃣ The Merge

👉 目标：让系统可扩展（make it scalable & modular）
<!-- DAILY_CHECKIN_2026-04-12_END -->

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
