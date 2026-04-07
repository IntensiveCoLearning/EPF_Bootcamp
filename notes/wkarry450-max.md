---
timezone: UTC+8
---

# vivid

**GitHub ID:** wkarry450-max

**Telegram:** 

## Self-introduction

EPF 实习计划

## Notes

<!-- Content_START -->
# 2026-04-07
<!-- DAILY_CHECKIN_2026-04-07_START -->
## 笔记：基于 secp256k1/r1 的 ECDSA 批量验证优化

### 1\. 核心问题

-   **背景**：ECDSA（椭圆曲线数字签名算法）广泛用于区块链（如比特币、以太坊），但其原生结构不支持高效的**批量验证（Batch Verification）**。
    
-   **痛点**：在高吞吐量场景下，逐个验证签名会成为系统的计算瓶颈。
    

### 2\. 改进方案：两个变体

为了实现高效批量验证，通过在签名中嵌入\*\*额外比特（Additional Bits）\*\*构造了两个变体：

-   **ECDSA\_rec**：侧重于恢复（Recovery）特性的改进。
    
-   **ECDSA\_ast**：性能表现通常优于 `ECDSA_rec`。
    

### 3\. 关键技术对比

研究对比了两种随机化批量验证技术：

-   **HSS-rand**：本次研究证明其在大多数情况下效率最高。
    
-   **NMVR-rand**：传统的随机化验证技术，效率略逊于 HSS-rand。
    

### 4\. 性能数据总结 (关键量化指标)

| 曲线类型 | 变体版本 | 批量验证提升 (vs 单独验证) | HSS-rand vs NMVR-rand |
| secp256k1 (比特币/以太坊常用) | ECDSA_ast | +30.9% | +10.6% |
|   | ECDSA_rec | +17.2% | +9.8% |
| secp256r1 (标准 TLS/国标常用) | ECDSA_ast | +53.9% | +27.6% |
|   | ECDSA_rec | +40.5% | +22.9% |

## CLWE 问题的搜索到决策归约

### 1\. 核心概念定义

-   **CLWE (Continuous LWE)**：连续型 LWE 问题，由 Bruna 等人在 2021 年提出，与传统离散空间的 LWE 不同，它处理的是连续概率分布。
    
-   **搜索版本 (Search)**：目标是根据给定的样本直接计算出秘密向量 $s$。
    
-   **决策版本 (Decision)**：目标是区分给定的样本是来自 CLWE 分布还是来自纯随机均匀分布。
    

### 2\. 现有研究局限（背景）

-   **2021 年 (STOC 2021)**：初步提出 CLWE 概念。
    
-   **2022 年 (Gupte 等人)**：实现了从 CLWE 到标准 LWE 的经典归约。
    
-   **瓶颈**：现有的反向归约（CLWE $\\leftarrow$ LWE）仅针对“离散 CLWE”有效。对于**一般性 CLWE**，此前缺乏从“搜索”到“决策”的直接归约证明。
    

### 3\. 本文核心贡献

本文填补了上述空白，提出了一种针对一般 CLWE 的搜索到决策归约方法。

-   **核心算法**：设计了一个相对简单的算法。
    
-   **实现路径**：利用\*\*决策预言机（Decision Oracle）\*\*作为子程序。
    
-   **达成目标**：通过决策预言机的反馈，以较小的误差逼近秘密向量，从而证明了：**只要能解决 CLWE 的决策问题，就足以解决其搜索版本。**
    

### 4\. 技术意义

-   **安全性等价性**：证明了 CLWE 的决策版本与搜索版本在计算难度上是等价的（或至少搜索不比决策难）。
    
-   **理论完备性**：为基于 CLWE 的密码学构造提供了更坚实的底座，使其安全性证明不再局限于离散子集。
<!-- DAILY_CHECKIN_2026-04-07_END -->

# 2026-04-06
<!-- DAILY_CHECKIN_2026-04-06_START -->

### **以太坊协议层核心架构 (EPF Wiki)**

**1\. EPF 核心定位**

-   **非应用层开发**：EPF 专注于以太坊协议本身（Core Protocol），而非智能合约或 DApp 开发。
    
-   **目标**：培养核心贡献者（Core Contributors），深入研究执行层（EL）、共识层（CL）及协议路线图。
    

**2\. 执行层 (Execution Layer, EL)**

执行层负责处理交易、智能合约执行和状态管理。

-   **核心组件**：
    
    -   **EVM (以太坊虚拟机)**：执行字节码的核心引擎。
        
    -   **状态管理**：使用 **Merkle Patricia Tree (MPT)** 组织账户平衡、合约代码和存储。
        
    -   **客户端实现**：常见的有 Geth (Go), Besu (Java), Nethermind (.NET), Reth (Rust)。
        
-   **关键协议**：包含 P2P 网络（DevP2P）和交易池管理。
    

**3\. 共识层 (Consensus Layer, CL)**

共识层负责通过权益证明（PoS）确保全网对数据达成一致。

-   **Beacon Chain (信标链)**：PoS 的指挥中心。
    
-   **核心机制**：
    
    -   **Gasper**：结合了 Casper FFG（最终性）和 LMD GHOST（分叉选择规则）。
        
    -   **验证者角色**：质押 32 ETH 参与出块与见证（Attestation）。
        
-   **客户端实现**：Prysm (Go), Lighthouse (Rust), Teku (Java), Lodestar (TS)。
    

**4\. 引擎 API (Engine API)**

-   这是连接 EL 和 CL 的桥梁。在合并（The Merge）后，两个客户端通过此 API 通信，确保执行结果被共识层认可。
    

**5\. 2026 路线图进阶**

-   **Cryptography (密码学)**：重点关注 KZG 承诺、椭圆曲线配对（Pairings），这些是支撑 EIP-4844 和 Danksharding 的数学基础。
    
-   **zkEVM & Lean Consensus**：研究如何通过零知识证明简化执行校验，以及共识机制的轻量化。
    

* * *

### **共学打卡心得**

> **今日学习感悟**： 通过阅读 EPF Wiki，进一步明确了以太坊“模块化”设计的精髓。了解到 EL 和 CL 的解耦不仅是技术的进步，更是为了未来分片扩展（Sharding）打下基础。接下来会重点关注执行层中 Geth 或 Reth 的代码实现，从工程化角度思考大型分布式系统的维护难度。
<!-- DAILY_CHECKIN_2026-04-06_END -->
<!-- Content_END -->
