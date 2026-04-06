---
timezone: UTC+8
---

# yico

**GitHub ID:** yicoyico639-a11y

**Telegram:** 

## Self-introduction

EPF 实习计划

## Notes

<!-- Content_START -->
# 2026-04-06
<!-- DAILY_CHECKIN_2026-04-06_START -->
🔹 模块1：The Protocol 以太坊协议全区

1\. Prehistory（史前历史）：以太坊起源与背景

\- 核心知识点：

\- 诞生背景：比特币的局限性（图灵不完备、扩展性差、功能单一），Vitalik 提出以太坊愿景

\- 发展里程碑：2013年白皮书发布、2014年众筹、2015年Frontier主网上线

\- 核心定位：世界计算机，支持图灵完备的智能合约，承载去中心化应用（DApp）

\- 学习目标：理解以太坊诞生的底层逻辑，区分以太坊与比特币的核心差异

2\. Architecture（整体架构）：协议分层与模块关系

\- 核心知识点：

\- 以太坊分层架构（合并后标准模型）：

1. 执行层（Execution Layer, EL）：处理交易、执行智能合约、状态存储，对应原以太坊主网

2. 共识层（Consensus Layer, CL）：负责PoS共识、区块验证、链上治理，对应原信标链

\- 模块关系：EL负责“计算与状态”，CL负责“共识与安全”，两层通过Engine API通信

\- 其他核心模块：P2P网络层、数据存储层、客户端实现（Geth/Nethermind等）

\- 学习目标：建立分层思维，理解各层职责与协同逻辑

3\. Design rationale（设计理念）：关键设计取舍原因

\- 核心知识点：

\- 核心设计原则：去中心化、安全性、可扩展性的“不可能三角”取舍

\- 关键设计决策：

\- 选择PoW→PoS的原因：能耗、安全性、扩展性升级

\- 图灵完备智能合约的设计考量：灵活性vs安全风险（如重入攻击）

\- 区块大小/Gas机制的设计：防止DoS攻击，保障网络公平性

\- 账户模型（Account Model）vs UTXO模型的选择原因

\- 学习目标：理解以太坊设计的底层逻辑，知其然更知其所以然

4\. Evolution（演进历史）：协议升级与历史脉络

\- 核心知识点：

\- 核心升级里程碑：

\- Frontier（2015）：初始版本，基础功能上线

\- Homestead（2016）：协议稳定化，支持智能合约

\- Byzantium/Constantinople（2017-2019）：性能优化、隐私功能升级

\- London（2021）：EIP-1559，Gas费机制改革

\- The Merge（2022）：合并，从PoW切换到PoS共识

\- Shanghai/Cancun（2023-2024）：质押提款、EIP-4844（Proto-Danksharding），Layer2扩容

\- 升级逻辑：向后兼容、社区治理、分阶段迭代

\- 学习目标：梳理以太坊发展时间线，理解每次升级的核心目标与影响

🔹 模块2：Execution Layer (EL) 基础部分

1\. EL Specs（执行层核心规范）

\- 核心知识点：

\- 核心规范文档：Yellow Paper（黄皮书）、EIPs（以太坊改进提案）

\- 核心规范内容：

\- 交易结构：nonce、gasLimit、gasPrice、to、value、data、签名

\- 状态模型：账户（外部账户EOA/合约账户）、状态树、存储树、交易树

\- 虚拟机EVM：指令集、Gas计费、执行环境

\- 区块结构：区块头、交易列表、叔块（PoW时代遗留）、状态根

\- 核心EIP：EIP-1559（Gas费）、EIP-2930（访问列表）、EIP-3651（Warm Coinbase）等

\- 学习目标：掌握执行层核心技术规范，为客户端开发/智能合约审计打基础

2\. Client architecture（客户端架构概述）：EL客户端模块总览

\- 核心知识点：

\- 主流EL客户端：Geth（Go语言）、Nethermind（.NET）、Besu（Java）、Erigon（Go）

\- 客户端核心模块：

1. P2P网络模块：节点发现、交易/区块广播

2. 同步模块：全同步/快照同步/快照同步，链数据同步

3. 执行模块：EVM执行、交易验证、状态更新

4. API模块：RPC接口（eth\_、web3\_），供DApp/钱包调用

5. 存储模块：LevelDB/RocksDB，链数据、状态数据存储

\- 客户端选型：不同客户端的性能、特性、适用场景差异

\- 学习目标：理解EL客户端的工作原理，掌握节点部署与运维的基础逻辑
<!-- DAILY_CHECKIN_2026-04-06_END -->
<!-- Content_END -->
