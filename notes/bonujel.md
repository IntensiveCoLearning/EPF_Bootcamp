---
timezone: UTC+8
---

# bonujel

**GitHub ID:** bonujel

**Telegram:** 

## Self-introduction

EPF 实习计划

## Notes

<!-- Content_START -->
# 2026-04-08
<!-- DAILY_CHECKIN_2026-04-08_START -->
# 以太坊升级

第一阶段是 **Frontier**。这代表以太坊主网上线，本质上更像一个面向开发者的 beta 版：先把网络跑起来，让大家能部署客户端、试验合约、开始做工具和应用。这个阶段的重点不是功能完备，而是谨慎启动，所以一开始 gas limit 很小，还用了 canary contracts 这种“升级预警机制”来避免客户端分叉后长时间停机。

第二阶段是 **Homestead**。这代表以太坊从“能跑”走向“更稳定、更适合正式使用”。核心变化不是推翻架构，而是做协议层加固：比如修补交易可塑性问题、调整合约创建的 gas 成本和失败语义、改进难度调整算法，以及引入 `DELEGATECALL`，这对后来的代理合约、可升级合约模式都很关键；同时，网络层也开始强调前向兼容，为以后继续升级打基础。

第三阶段是 **The Merge**。这是历史上最关键的一次范式切换：以太坊把共识从 PoW 切到 PoS。更准确地说，是把原来执行和共识混在一起的单层结构，变成了“执行层 + 共识层”的双层结构：执行仍负责交易和状态转换，共识则由已经运行多时的 Beacon Chain 接管。也就是从这一步开始，以太坊真正进入今天这套模块化架构。
<!-- DAILY_CHECKIN_2026-04-08_END -->

# 2026-04-07
<!-- DAILY_CHECKIN_2026-04-07_START -->

# 设计理念

以太坊的协议设计理念，是在“尽量保持底层简单、通用、可组合”的前提下，把复杂性尽可能封装起来，并把高层应用自由留给合约和上层生态，而不是把各种具体功能硬编码进协议本身。

它主要讲了三层意思。第一层是设计哲学：以太坊追求简洁、普适、模块化、非歧视和敏捷演进。简洁不是说系统真的很小，而是希望核心规范尽量可理解；普适是说协议本身尽量不内建太多“专用功能”，而是提供像EVM这样的通用执行平台；模块化是希望很多创新可以作为独立组件存在，既能服务以太坊，也能被别的协议复用；非歧视是说协议不主动限制某类用途，而是更倾向于通过费用机制让使用者自己承担成本；敏捷则意味着协议不是不可变的，只要改动能明显改善安全性或可扩展性，就应当认真考虑。

第二层是协议原则：最核心的是“管理复杂性”。这页反复强调，复杂性不可完全避免，但要尽量把它放在更合适的位置，比如Layer2优先于客户端实现，客户端实现优先于协议规范本身。它还区分了“封装起来的复杂性”和“系统性交织的复杂性”，前者虽然也复杂，但危险小得多，因为它可以被隔离、替换和单独推理。与此相连的另外几个原则是自由、一般化和“我们没有特性”：协议应该尽量提供底层、通用、可组合的原语，而不是不断为流行场景加新特性；很多高层需求，应该由合约或子协议自己实现。

第三层是这些理念如何落到具体技术选型上。比如，以太坊选择账户模型而不是UTXO，是因为账户模型对复杂状态转换和智能合约更自然；选择Merkle Patricia Trie，是为了让状态具备确定性和可验证性；继续研究Verkle树，是为了在状态越来越大时缩小证明、推进无状态化；从RLP转向在信标链中使用SSZ，是因为SSZ更适合Merkle化、索引和轻客户端证明；在共识上追求终局性，则体现为Casper FFG、LMD-GHOST以及它们组合出的Gasper；在网络层上，用Kademlia型DHT做节点发现，再用gossip网络传播区块，则体现了“结构化引导+非结构化扩散”的折中。换句话说，这页不是在单独解释某几个组件，而是在说明：这些组件之所以被选中，是因为它们符合以太坊一贯的设计取向——核心层尽量通用、组合性强、可验证，同时为未来扩展保留空间。
<!-- DAILY_CHECKIN_2026-04-07_END -->

# 2026-04-06
<!-- DAILY_CHECKIN_2026-04-06_START -->


# Ethererum 协议架构

由两大部分组成——执行层和共识层。执行层（EL）负责处理实际的交易和用户交互，它是“全球计算机”执行程序的核心场所。共识层（CL）则提供了权益证明（PoS）共识机制。在实际应用中，这些层级通过各自独立的客户端实现，并通过 API 相互连接。每一层都拥有专属的 P2P 网络，用于处理不同类型的数据。

## 第一张架构图

最外层的 “Ethereum client” 可以理解成你机器上跑的一整套以太坊节点系统。这个系统里面有两个核心进程。上面的 Beacon Node 是共识层节点，它不负责跑 EVM，不直接执行用户交易；它负责 PoS 相关的事，比如接收和传播 beacon block、处理 attestation、维护 fork choice、跟踪 finality。下面的 Execution Engine 是执行层节点，它才是真正执行交易、维护 world state、保存账户/storage、运行 EVM 的地方。用户熟悉的 Geth、Nethermind、Besu，本质上都属于这一层。共识客户端和执行客户端是分开的软件，但必须协同工作。

Beacon APIs 连到 Beacon Node，主要给验证者相关组件或运维系统使用，比如获取当前 slot、提议/证明职责、提交 attestations、查询 beacon 状态。User APIs 连到 Execution Engine，这就是大家平时最熟悉的 JSON-RPC 入口，钱包、DApp、脚本、Web3 SDK 基本都从这里进来，比如 `eth_call`、`eth_sendRawTransaction`、`eth_getBalance` 这类调用。也就是说，普通用户通常主要接触执行层 API，而不是共识层 API。JSON-RPC 作为执行客户端对外接口是以太坊长期保留的用户入口。

中间的 Engine API 不是给外部用户用的，而是共识层和执行层之间的“内部协议”。共识层通过它告诉执行层：“我现在认为哪个 head 是对的”“请基于当前状态帮我构造一个 execution payload”“这是新收到的 payload，你帮我验证并执行一下”。反过来，执行层也会把 payload 的执行结果、状态有效性等反馈给共识层。Merge 之后，这条接口就变成了以太坊节点内最关键的耦合点。

右边两组 p2p 表示以太坊现在不是一个单一网络，而是两张逻辑上独立的网络同时存在。Beacon Node 参加的是共识层 p2p 网络，传播的是 beacon block、attestation、sync committee 等共识消息。Execution Engine 参加的是执行层 p2p 网络，传播的是交易、execution payload 对应的区块数据等执行层内容。也就是说，CL 和 EL 不只是职责分离，连网络传播面也分离了。你的节点实际上同时在两个网络里说话。

## 第二张架构图

上半部分是 Consensus Layer。里面的 PoS 说明它负责权益证明共识。Fork Choice LMD-GHOST + Finality Casper-FFG，表明：LMD-GHOST 负责在存在多个候选链头时选出当前 head，Casper FFG 负责把链上的某些 checkpoint 推到 finalized 状态。前者更像“现在跟谁走”，后者更像“哪些历史已经不可逆”。此外，这一层还管理 Validators、RANDAO、以及 blob 相关的数据可用性职责。Beacon node 本身就是这一层的核心节点实现。Merge 之后，规范上的 canonical chain 头部选择和 finalized block 信息都来自 Beacon Chain 的 fork choice / finality 机制，而不再由旧的 PoW total difficulty 规则决定。

严格说，validator 不是“另一条链”，而是参与 PoS 的角色。验证者会通过 Beacon APIs 跟 beacon node 协作：接收自己在某个 slot/epoch 的职责，进行 block proposal 或 attestation，再把签名后的消息交给 beacon node 广播到共识层网络。所以验证者逻辑上站在共识层这边，而不是执行层这边。Beacon node 本身负责同步 Beacon Chain、做 attestation 处理、维护验证者集合与职责分配等。

下半部分是 Execution Layer。执行层 p2p 网络会接收和传播用户交易；这些交易先进入 mempool；当需要构造区块时，会从 mempool 里挑交易组成 execution payload；EVM 逐笔执行这些交易，对账户余额、nonce、storage、合约代码相关状态进行读写；执行完后更新 state data。你可以把执行层理解成“确定性状态机”：给我一个前状态和一组交易，我算出后状态。以太坊黄皮书本来描述的核心就是这种基于交易的状态转换系统。
<!-- DAILY_CHECKIN_2026-04-06_END -->
<!-- Content_END -->
