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
