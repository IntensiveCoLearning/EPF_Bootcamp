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
