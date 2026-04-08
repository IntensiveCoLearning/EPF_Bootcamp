---
timezone: UTC+0
---

# kvuxnz

**GitHub ID:** kvxunz

**Telegram:** 

## Self-introduction

EPF 实习计划

## Notes

<!-- Content_START -->
# 2026-04-08
<!-- DAILY_CHECKIN_2026-04-08_START -->
Consensus Layer (CL) architecture

\> Many blockchain consensus protocols are _forkful_. Forkful chains use a fork choice rule, and sometimes undergo reorganisations.

\>

\> Ethereum's consensus protocol combines two separate consensus protocols. _LMD GHOST_ essentially provides liveness. _Casper FFG_ provides finality. Together they are known as _Gasper_.

\>

\> In a _live_ protocol, something good always happens. In a _safe_ protocol, nothing bad ever happens. No practical protocol can be always safe and always live.

\## Fork-choice Mechanism

As described in \[BFT\](/wiki/CL/[overview.md](http://overview.md)?id=byzantine-fault-tolerance-bft-and-byzantine-generals39-problem), for various reasons - like network delays, outages, out-of-order messages, or malicious behavior — nodes in the network can have different views of the network's state. Eventually, we want every honest node to agree on an identical, linear history and a common view of the system's state. The protocol's fork choice rule is what helps achieve this agreement.

\#### Block Tree

Given a block tree and decision criteria based on a node's local view of the network, the fork choice rule is designed to select the branch that is most likely to become the final, linear, canonical chain. It chooses the branch least likely to be pruned out as nodes converge on a common view.

<a id="img\_blocktree"></a>

<figure class="diagram" style="text-align:center; width:95%">

!\[Diagram for Block Tree\](../../images/cl/blocktree.svg)

<figcaption>

_The fork choice rule picks a head block from the candidates. The head block identifies a unique linear blockchain back to the Genesis block._

</figcaption>

</figure>

\#### Fork choice rules

The fork choice rule implicitly selects a branch by choosing a block at the branch's tip, called the head block. For any correct node, the first rule of any fork choice is that the chosen block must be valid according to protocol rules, and all its ancestors must also be valid. Any invalid block is ignored, and blocks built on an invalid block are also invalid.

There are several examples of different fork choice rules:

\- **Proof-of-work**: In Ethereum and Bitcoin, the "heaviest chain rule" (sometimes called "longest chain", though not strictly accurate) is used. The head block is the tip of the chain with the most cumulative "work" done.

\> Note that contrary to popular belief, Ethereum's Proof-of-work protocol \[did not use\]([https://ethereum.stackexchange.com/questions/38121/why-did-ethereum-abandon-the-ghost-protocol/50693#50693](https://ethereum.stackexchange.com/questions/38121/why-did-ethereum-abandon-the-ghost-protocol/50693#50693)) any form of GHOST in its fork choice. This misconception is very persistent, probably due to the \[Ethereum Whitepaper\]([https://ethereum.org/en/whitepaper/#modified-ghost-implementation](https://ethereum.org/en/whitepaper/#modified-ghost-implementation)). Eventually when Vitalik was asked about it, he confirmed that although GHOST had been planned under PoW it was never implemented due to concerns about some unspecified attacks. The heaviest chain rule was simpler and well tested. It worked fine.

\- **Casper FFG (Proof-of-Stake)**: In Ethereum's PoS Casper FFG protocol, the fork-choice rule is to "follow the chain containing the justified checkpoint of the **greatest height**" and never revert a finalized block.

\- **LMD GHOST (Proof-of-Stake)**: In Ethereum's PoS LMD GHOST protocol, the fork-choice rule is to take the "Greediest Heaviest Observed SubTree". It involves counting accumulated votes from validators for blocks and their descendent blocks. It also applies the same rule as Casper FFG.

Each of these fork choice rules assigns a numeric score to a block. The winning block, or head block, has the highest score. The goal is that all correct nodes, when they see a certain block, will agree that it is the head and follow its branch. This way, all correct nodes will eventually agree on a single canonical chain that goes back to Genesis.

\#### Reorgs and Reversion

As a node receives new votes (and new votes for blocks in Proof-of-stake), it re-evaluates the fork choice rule with this new information. Usually, a new block will be a child of the current head block, and it will become the new head block.

Sometimes, however, the new block might be a descendant of a different block in the block tree. If the node doesn't have the parent block of the new block, it will ask its peers for it and any other missing blocks.

Running the fork choice rule on the updated block tree might show that the new head block is on a different branch than the previous head block. When this happens, the node must perform a reorg (reorganisation). This means it will remove (revert) blocks it previously included and adopt the blocks on the new head's branch.

For example, if a node has blocks $A, B, D, E,$ and $F$ in its chain, and it views $F$ as the head block, it knows about block $C$ but it does not appear in its view of the chain; it is on a side branch.

<a id="img\_reorg0"></a>

<figure class="diagram" style="text-align:center">

!\[Diagram for Reorg-0\](../../images/cl/reorg-0.svg)

<figcaption>

_At this point, the node believes that block $F$ is the best head, therefore its chain is blocks $\[A \\leftarrow B \\leftarrow D \\leftarrow E \\leftarrow F\]$_

</figcaption>

</figure>

When the node later receives block $G$, which is built on block $C$, not on its current head block $F$, it must decide if $G$ should be the new head. Just for example, If the fork choice rule says $G$ is the better head block, the node will revert blocks $D, E,$ and $F$. It will remove them from its chain, as if they were never received, and go back to the state after block $B$.

Then, the node will add blocks $C$ and $G$ to its chain and process them. After this reorg, the node's chain will be $A, B, C,$ and $G$.

<a id="img\_reorg1"></a>

<figure class="diagram" style="text-align:center">

!\[Diagram for Reorg-1\](../../images/cl/reorg-1.svg)

<figcaption>

_Now the node believes that block $G$ is the best head, therefore its chain must change to the blocks $\[A \\leftarrow B \\leftarrow C \\leftarrow G\]$_

</figcaption>

</figure>

Later, perhaps, a block $H$ might appear, that's built on $F$, and the fork choice rule says $H$ should be the new head, the node will reorg again, reverting to block $B$ and replaying blocks on $H$'s branch.

Short reorgs of one or two blocks are common due to network delays. Longer reorgs should be rare unless the chain is under attack or there is a bug in the fork choice rule or its implementation.

\### Safety and Liveness

In consensus mechanisms, two key concepts are safety and liveness.

**Safety** means "nothing bad ever happens," such as preventing double-spending or finalizing conflicting checkpoints. It ensures consistency, meaning all honest nodes should always agree on the state of the blockchain.

**Liveness** means "something good eventually happens," ensuring the blockchain can always add new blocks and never gets stuck in a deadlock.

**CAP Theorem** states that no distributed system can provide consistency, availability, and partition tolerance simultaneously. This means we can't design a system that is both safe and live under all circumstances when communication is unreliable.

\#### Ethereum Prioritizes Liveness

Ethereum’s consensus protocol aims to offer both safety and liveness in good network conditions. However, it prioritizes liveness during network issues. In a network partition, nodes on each side will continue to produce blocks but won't achieve finality (a safety property). If the partition persists, each side may finalize different histories, leading to two irreconcilable, independent chains.

Thus, while Ethereum strives for both safety and liveness, it leans towards ensuring the network remains live and continues to process transactions, even at the cost of potential safety issues during severe network disruptions.

\## The Ghosts in the Machine

Ethereum's Proof-of-Stake consensus protocol combines two separate protocols: \[LMD GHOST\](/wiki/CL/gasper?id=[lmd-ghost.md](http://lmd-ghost.md)) and \[Casper FFG\](/wiki/CL/gasper?id=[casper-ffg.md](http://casper-ffg.md)). Together, they form the consensus protocol known as "Gasper". Detailed Information about both protocols and how they work in combination are covered in the next section \[Gasper\](/wiki/CL/gasper).

Gasper aims to combine the strengths of both LMD GHOST and Casper FFG. LMD GHOST provides liveness, ensuring the chain keeps running by producing new blocks regularly. However, it is prone to forks and not formally safe. Casper FFG, on the other hand, provides safety by periodically finalizing the chain, protecting it from long reversions.

In essence, LMD GHOST keeps the chain moving forward, while Casper FFG ensures stability by finalizing blocks. This combination allows Ethereum to prioritize liveness, meaning the chain continues to grow even if Casper FFG can't finalize blocks. Although this combined mechanism isn't always perfect and has some complexities, it is a practical engineering solution that works well in practice for Ethereum.

\## Architecture

Ethereum is a decentralized network of nodes that communicate via peer-to-peer connections. These connections are formed by computers running Ethereum's client software.

<figure class="diagram" style="text-align:center">

!\[Diagram for Network\](../../images/cl/network.png)

<figcaption>

_Nodes aren't required to run a validator client (green ones) to be a part of the network, however to take part in consensus one needs to stake 32 ETH and run a validator client._

</figcaption>

</figure>

\### Components of the Consensus Layer

\- **Beacon Node**: Beacon nodes use client software to coordinate Ethereum's proof-of-stake consensus. Examples include Prysm, Teku, Lighthouse, and Nimbus. Beacon nodes communicate with other beacon nodes, a local execution node, and optionally, a local validator.

\- **Validator**: Validator client is the software that allows people to stake 32 ETH in Ethereum's consensus layer. Validators propose blocks in the Proof-of-Stake system, which replaced Proof-of-work miners. Validators communicate only with a local beacon node, which instructs them and broadcasts their work to the network.

The main Ethereum network hosting real-world applications is called Ethereum Mainnet. Ethereum Mainnet is the live, production instance of Ethereum that mints and manages real Ethereum (ETH) and holds real monetary value.

There are also test networks that mint and manage test Ethereum for developers, node runners, and validators to test new functionality before using real ETH on Mainnet. Each Ethereum network has two layers: the execution layer (EL) and the consensus layer (CL). Every Ethereum node contains software for both layers: execution-layer client software (like Nethermind, Besu, Geth, and Erigon) and consensus-layer client software (like Prysm, Teku, Lighthouse, Nimbus, and Lodestar).

<a id="img\_node-layers"></a>

<figure class="diagram" style="width: 80%;text-align:center">

!\[Diagram for CL\](../../images/cl/cl.png)

</figure>

**Consensus Layer** is responsible for maintaining consensus chain (beacon chain) and processing the consensus blocks (beacon blocks) and attestations received from other peers. **Consensus clients** participate in a separate \[peer-to-peer network\](/wiki/CL/[cl-networking.md](http://cl-networking.md)) with a different specification from execution clients. They need to participate in block gossip to receive new blocks from peers and broadcast blocks when it's their turn to propose.

Both EL and CL clients run in parallel and need to be connected for communication. The consensus client provides instructions to the execution client, and the execution client passes transaction bundles to the consensus client to include in Beacon blocks. Communication is achieved using a local RPC connection via the **Engine-API**. They share an \[ENR\](/wiki/CL/cl-networking?id=enr-ethereum-node-records) with separate keys for each client (eth1 key and eth2 key).

\### Control Flow

**When the consensus client is not the block producer:**

1\. Receives a block via the block gossip protocol.

2\. Pre-validates the block.

3\. Sends transactions in the block to the execution layer as an execution payload.

4\. Execution layer executes transactions and validates the block state.

5\. Execution layer sends validation data back to the consensus layer.

6\. Consensus layer adds the block to its blockchain and attests to it, broadcasting the attestation over the network.

**When the consensus client is the block producer:**

1\. Receives notice of being the next block producer.

2\. Calls the create block method in the execution client.

3\. Execution layer accesses the transaction mempool.

4\. Execution client bundles transactions into a block, executes them, and generates a block hash.

5\. Consensus client adds transactions and block hash to the beacon block.

6\. Consensus client broadcasts the block over the block gossip protocol.

7\. Other clients validate the block and attest to it.

8\. Once attested by sufficient validators, the block is added to the head of the chain, justified, and finalized.

\### State Transitions

The state transition function is essential in blockchains. Each node maintains a state that reflects its view of the world.

Nodes update their state by applying blocks in order using a "state transition function". This function is "pure", meaning its output depends only on the input and has no side effects. Thus, if every node starts with the same state (Genesis state) and applies the same blocks, they all end up with the same state. If they don't, there's a consensus failure.

If $S$ is a beacon state and $B$ a beacon block, the state transition function $f$ is:

$$S' \\equiv f(S, B)$$

Here, $S$ is the pre-state and $S'$ is the post-state. The function $f$ is iterated with each new block to update the state.

\### Beacon Chain State Transitions

Unlike the block-driven Proof-of-work, the beacon chain is slot-driven. State updates depend on slot progress, regardless of block presence.

The beacon chain's state transition function includes:

1\. **Per-slot transition**: $S' \\equiv f\_s(S)$

2\. **Per-block transition**: $S' \\equiv f\_b(S, B)$

3\. **Per-epoch transition**: $S' \\equiv f\_e(S)$

Each function updates the chain at specific times, as defined in the beacon chain specification.

\### Validity Conditions

The post-state from a pre-state and a signed block is `state_transition(state, signed_block)`. Transitions causing unhandled exceptions (e.g., failed asserts or out-of-range accesses) or uint64 overflows/underflows are invalid.

\### Beacon chain state transition function

The post-state corresponding to a pre-state `state` and a signed block `signed_block` is defined as `state_transition(state, signed_block)`. State transitions that trigger an unhandled exception (e.g. a failed `assert` or an out-of-range list access) are considered invalid. State transitions that cause a `uint64` overflow or underflow are also considered invalid.

\`\`\`python

def state\_transition(state: BeaconState, signed\_block: SignedBeaconBlock, validate\_result: bool=True) -> None:

block = signed\_block.message

\# Process slots (including those with no blocks) since block

process\_slots(state, block.slot)

\# Verify signature

if validate\_result:

assert verify\_block\_signature(state, signed\_block)

\# Process block

process\_block(state, block)

\# Verify state root

if validate\_result:

assert block.state\_root == hash\_tree\_root(state)

\`\`\`

\`\`\`python

def verify\_block\_signature(state: BeaconState, signed\_block: SignedBeaconBlock) -> bool:

proposer = state.validators\[signed\_block.message.proposer\_index\]

signing\_root = compute\_signing\_root(signed\_block.message, get\_domain(state, DOMAIN\_BEACON\_PROPOSER))

return bls.Verify(proposer.pubkey, signing\_root, signed\_block.signature)

\`\`\`

\`\`\`python

def process\_slots(state: BeaconState, slot: Slot) -> None:

assert state.slot < slot

while state.slot < slot:

process\_slot(state)

\# Process epoch on the start slot of the next epoch

if (state.slot + 1) % SLOTS\_PER\_EPOCH == 0:

process\_epoch(state)

state.slot = Slot(state.slot + 1)

\`\`\`
<!-- DAILY_CHECKIN_2026-04-08_END -->

# 2026-04-07
<!-- DAILY_CHECKIN_2026-04-07_START -->

\# Ethereum 前史

\> "Heroes are heroes because they are heroic in behavior, not because they won or lost." — Nicholas Taleb

追溯 Ethereum 的思想谱系——从互联网的诞生、Unix 哲学、公钥密码学革命、自由软件运动、密码朋克运动、数字货币实验，到 Bitcoin 和 Ethereum 的诞生。

\---

\## 一、信息高速公路：互联网的崛起

\- **1969**：互联网前身 \[\[ARPANET\]\] 作为冷战项目诞生

\- 广播用了 38 年才触及 5000 万用户，电视 13 年，PC 16 年，而\*\*互联网只用了 4 年\*\*

\- 互联网消除了地理边界，使原本不可想象的人类交互成为可能

\> "National borders are just speed bumps on the information superhighway." — Timothy May

\---

\## 二、Unix 与 Bell Labs

\### 起源

\- Unix 诞生于对 MULTICS（1960 年代庞大操作系统项目）的简化尝试

\- **Ken Thompson** 和 **Dennis Ritchie** 在 AT&T Bell Labs 创造了 Unix——一个更模块化、更简单、更可组合的替代方案

\- Ken Thompson："三周就有了操作系统——一周编辑器，一周汇编器，一周内核"

\- 1972 年 Dennis Ritchie 编写了影响深远的 **C 语言**

\### Bell Labs 的创新

\- Bell Labs 是 20 世纪最重要的技术孵化器

\- 其创新不是直接面向消费者的产品，而是嵌入通信基础设施的\*\*平台级创新\*\*

\- Ethereum 在很多方面就像一个开放的 Bell Labs

\### Unix 哲学

\- 层级文件系统

\- Shell 命令行接口

\- 单一用途工具的组合（做一件事并做好）

\- 简单性、灵活性和可复用性

\- Unix 及其衍生系统（Linux、macOS）至今仍是现代计算的基石

\---

\## 三、密码学革命：我们能保守秘密吗？

\### 问题

自文明诞生以来，密文传递一直是人类的核心需求。传统的\*\*对称加密\*\*（加解密用同一把钥匙）使得密钥分发成为噩梦——二战中 Alan Turing 破解 Enigma 机就是明证。

核心问题：\*\*如何在从未见面的人之间安全交换密钥？\*\*

\### 公钥密码学的诞生（1974-1978）

| 年份 | 人物 | 突破 |

|------|------|------|

| 1974 | **Ralph Merkle** | Merkle's Puzzles——首个无需预共享秘密即可协商密钥的方法 |

| 1976 | **Whitfield Diffie & Martin Hellman** | 发表《New Directions in Cryptography》，提出 Diffie-Hellman 密钥交换，开创\*\*无信任密码学\*\* |

| 1977 | **Ron Rivest, Adi Shamir, Len Adleman** | 发明 RSA 密码系统——首个公钥加密的工程实现 |

\- Martin Gardner 在 Scientific American 发表 RSA 文章，附带 RSA-129 挑战（悬赏 $100）

\- 1994 年该挑战被破解，奖金捐给了自由软件基金会

\- 1997 年英国政府解密了 1970 年的类似研究（英国情报机构更早独立发现了公钥密码学）

\### 影响

现代 RSA 加密（1024-4096 位）为信息高速公路创造了安全通道，使银行和信用卡公司能保护金融交易，推动了电子商务和网上银行的发展。

\---

\## 四、自由软件运动：Free as in Freedom

\### 软件从开放到封闭

\- **1950-60 年代**：早期软件原始且需要修改，源代码共享是常态，形成了"黑客文化"

\- 计算机杂志甚至会刊登 type-in 程序，鼓励用户手写代码

\- **1969**：US vs. IBM 反垄断诉讼后，软件开始独立收费，变成商品

\- Unix 也未能幸免——AT&T 在 1980 年代初停止免费分发

\- Bill Gates 发表致爱好者的公开信，要求停止共享 BASIC 源代码

\### Richard Stallman 与 GNU

\- MIT AI 实验室研究员 **Richard Stallman** 因无法修改 Xerox 打印机源代码而愤怒

\- 他认为限制软件修改是"对人类的犯罪"

\- **1983**：通过邮件宣布 GNU 项目（GNU's Not Unix，递归缩写）

\- **1984**：GNU 正式启动，编写了 GPL（GNU 通用公共许可证）

\- "Free software" 指的是\*\*自由\*\*（free speech），不是\*\*免费\*\*（free beer）

\- 到 1990 年，GNU 完成了操作系统的所有主要组件，唯独缺少\*\*内核\*\*

\### Linus Torvalds 与 Linux

\- **1991**：芬兰计算机科学学生 **Linus Torvalds** 开发了 Linux 内核

\- 发布后数小时内就收到响应，一年内数百人加入开发

\- Linux 以 GPL 许可发布，补全了 GNU/Linux 操作系统

\- Linux 奠定了基于\*\*社会共识\*\*的软件开发蓝图

\### 开源运动

\- 开源运动与自由软件运动有别，更侧重源代码可访问的\*\*实际收益\*\*

\- 在社区驱动创新与商业可行性之间取得平衡

\- FOSS（Free and Open-Source Software）是两者的总称

\---

\## 五、密码朋克写代码

\### 密码战争

\- 二战后，各国政府垄断密码学进展

\- 美国将加密技术列入\*\*军火管制清单\*\*，NSA 对密码学研究高度敏感

\- NSA 曾试图阻止 RSA 论文发表（最终允许）

\- NSA 雇员 Joseph Mayer 写信给 IEEE 称密码学出版物需政府批准，引发公众强烈批评

\### PGP 与 Phil Zimmermann

\- **1991**：\*\*Phil Zimmermann\*\* 开发了 PGP（Pretty Good Privacy），让个人也能使用强加密

\- **1993**：因涉嫌违反加密软件出口限制，遭美国海关刑事调查

\- 他将 PGP 完整源代码出版为\*\*精装书\*\*，主张图书出口受第一修正案保护

\- 欧洲志愿者扫描书页并 OCR 还原为电子版（超过 70 人工作超 1000 小时）

\- **1996**：案件撤销

\### 密码朋克运动

\- **1992**：\*\*Eric Hughes\*\*、\*\*Timothy C. May\*\*、\*\*John Gilmore\*\* 创立 Cypherpunk 邮件列表

\- 超过 **700 名活动家和反叛者**（包括 Zimmermann）

\- 核心信条：

\- 政府不应窥探公民事务

\- 保护通信和交换是基本权利

\- 这些权利需要通过\*\*技术\*\*而非法律来保障

\- 技术的力量创造新的政治现实

\> "Cypherpunks write code."

\### 加密无政府主义

\- **1988**：Tim May 发表《加密无政府主义宣言》

\- 反对一切权威形式，只承认密码学描述和代码执行的法则

\- 设想匿名数字交易是个人自由的基石

\- **缺失的拼图：一种密码学原生的数字货币**

\---

\## 六、寻找缺失的拼图：数字货币实验

| 年份 | 项目 | 创建者 | 特点 | 结局 |

|------|------|--------|------|------|

| 1990 | **DigiCash** | David Chaum | 首个匿名数字经济体验；依赖现有金融基础设施，高度中心化 | 1998 年破产 |

| 1996 | **E-gold** | — | 以实物黄金为储备；峰值 350 万注册账户 | 2009 年因法律问题暂停 |

| 1998 | **B-money** | Wei Dai | 用密码学函数创造货币，去中心化设计 | 未落地 |

| 2005 | **BitGold** | Nick Szabo | 数字化控制稀缺性，去中心化设计 | 未实现 |

B-money 和 BitGold 虽未成功，但其设计直接影响了 Bitcoin。

\---

\## 七、Bitcoin

\- **2008 年金融危机**重新点燃了对数字货币的兴趣

\- **Satoshi Nakamoto**（化名）发表论文《Bitcoin: A Peer-to-Peer Electronic Cash System》

\- 解决了\*\*无领导者共识\*\*这一开放性问题

\- Bitcoin 的核心创新：

\- 分布式账本：数据以密码学方式按时间顺序链接成区块

\- 首个去中心化数字货币：无底层抵押品，无需银行等可信第三方

\- 工作量证明（Proof of Work）区块链：公开就交易顺序达成共识

\> Vitalik Buterin："Satoshi 同时引入了两个激进且未经验证的概念：作为货币的 bitcoin，以及基于工作量证明的区块链。"

Bitcoin 网络上也有人尝试构建应用（Colored Coins、Namecoin），但 Bitcoin 的网络对此过于原始，应用只能通过复杂且不可扩展的变通方案实现。

\---

\## 八、Ethereum 世界计算机

\### 诞生

\- **2012**：\*\*Vitalik Buterin\*\* 与 Mihai Alisie 创办 Bitcoin Magazine

\- Vitalik 很快发现 Bitcoin 的局限性，提出支持\*\*通用金融应用\*\*的平台

\- **2014**：在 **Gavin Wood** 的帮助下，Ethereum 的设计被形式化（Yellow Paper）

\- **2015 年 7 月 30 日**：Ethereum 主网上线

\### 定位

\- 不仅是数字货币，而是构建\*\*自主经济工具\*\*的平台

\- 支持智能合约和去中心化应用（dApps）

\- 截至原文撰写时，市值约 **$400 billion**

\### 谱系总结

\`\`\`

互联网 (1969) → Unix/Bell Labs (1970s) → 公钥密码学 (1974-78)

↓ ↓ ↓

信息自由流动 模块化/可组合设计 无信任安全通信

↓ ↓ ↓

自由软件/开源 (1983-91) ← → 密码朋克运动 (1988-92)

↓ ↓

社会共识开发模式 数字货币实验 (1990-2005)

↓ ↓

└──────────── Bitcoin (2008) ──┘

↓

Ethereum (2015)

\`\`\`

\---
<!-- DAILY_CHECKIN_2026-04-07_END -->

# 2026-04-06
<!-- DAILY_CHECKIN_2026-04-06_START -->


对密码学有个大概了解
<!-- DAILY_CHECKIN_2026-04-06_END -->
<!-- Content_END -->
