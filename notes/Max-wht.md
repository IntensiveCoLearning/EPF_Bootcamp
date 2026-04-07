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
