---
timezone: UTC+8
---

# Max

**GitHub ID:** Max-wht

**Telegram:** 

## Self-introduction

EPF 实习计划

## Notes

<!-- Content_START -->
# 2026-04-08
<!-- DAILY_CHECKIN_2026-04-08_START -->
EVM 的关键工程现实在于：它的抽象（256-bit 栈机、可扩展内存、持久化存储、账户/世界状态）与底层状态数据库（如 Merkle Patricia Trie 及其客户端实现）强耦合。世界状态是“地址 → 账户状态”的映射，账户状态由 nonce、balance、storageRoot、codeHash 四要素构成，客户端需在可证明的数据结构中维护该映射并将根哈希提交到区块链头部。

过去十年里，“gas 定价与可预测性”是 EVM 演进的主线之一：Istanbul/ Berlin/ London/ Shanghai/ Dencun/ Pectra/ Fusaka 等升级，反复围绕状态访问成本（SLOAD/BALANCE/EXT\* 等）和交易数据成本（calldata、initcode、blob）调整规则；同时通过 access list、冷热访问（cold/warm）、refund 限制、initcode 计量、以及对“数据密集型交易”的 calldata 价格下限等机制，让 worst-case 资源消耗更可控。

在实现生态方面，主流执行客户端（geth、Nethermind、Erigon、Besu）持续跟进主网升级，围绕同步模式、状态裁剪、存储格式、追踪/调试接口形成差异化；而 OpenEthereum 已归档停更，代表了历史路径但不再适用于主网最新规则。

形式化与可验证性是 EF 长期投入方向：从黄皮书形式化定义，到可执行规范（execution-specs / EELS）、到 KEVM（K Framework 可执行语义）、Isabelle/HOL 与 F\* 等多套语义与程序逻辑工作，使得“字节码级别正确性/等价性验证、差分测试、符号执行”逐步工程化。

面向 2026 及之后的开放问题集中在：状态增长与 I/O 不确定性导致的经济与去中心化压力（gas 定价失配）、更强的状态访问显式化与证明友好执行（如 Verkle/“无状态性”相关提案）、字节码结构化（EOF）与静态可分析性、以及长期“是否替换 EVM（eWASM、RISC‑V 等）”的路线权衡。
<!-- DAILY_CHECKIN_2026-04-08_END -->

# 2026-04-07
<!-- DAILY_CHECKIN_2026-04-07_START -->

快速阅读了一下相关的资料。这一部分的主线是: 从“为什么会有 Ethereum（Prehistory）”，到“它现在怎么工作（Architecture）”，再到“为什么这样设计（Design rationale）”，最后看“这些规则如何持续升级（Evolution）”

**Prehistory :** Ethereum 不是凭空出现的，它继承了早期数字现金和密码学系统的探索（如 80-90 年代电子现金、1998 年 b-money、2009 年 Bitcoin），并把区块链从“记账系统”扩展成“可编程状态机”。

**Architecture:** 现代 Ethereum 是分层架构：执行层EL负责交易执行、EVM 与状态；共识层CL负责 PoS 共识、区块确认和最终性；两层通`Engine API` 协同。节点也因此由执行客户端 + 共识客户端（可选验证器）组成。

**Design rationale:** 核心设计取向是：底层尽量简单、复杂性放到中间层；协议保持通用与可组合；优先低层原语而非写死“应用功能”。典型取舍包括：使`账户模型`而非 UTXO、`Merkle Patricia Trie`维护可验证状态、`RLP`做确定性序列化。

**Evolution:** Ethereum 通过连续硬分叉演进：从 Frontier（2015-07-30）到 London/EIP-1559（2021-08-05），再到 The Merge（2022-09-15）完成 PoW→PoS，随后 Shapella（2023-04-12）、Dencun（2024-03-13）、Pectra（2025-05-07）等继续优化扩容、质押与账户能力. 也就是说，协议是“持续迭代的规范”，不是一次性定稿

很多小知识涌入大脑，感觉不赖。整个以太坊是个很巨大，很复杂的机器，EIP都有几千条了，对于应用层的EIP，对于协议的EIP等等。保持热情，继续学习
<!-- DAILY_CHECKIN_2026-04-07_END -->
<!-- Content_END -->
