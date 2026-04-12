---
timezone: UTC+8
---

# @0xMrByte

**GitHub ID:** panrui1984

**Telegram:** 

## Self-introduction

EPF 实习计划

## Notes

<!-- Content_START -->
# 2026-04-12
<!-- DAILY_CHECKIN_2026-04-12_START -->
# **以太坊虚拟机 (EVM) 概要笔记**

# **EVM 的本质**

EVM 是一个**准图灵完备（Quasi-Turing Complete）**、\*\*基于栈（Stack-based）\*\*的虚拟机。

-   **准图灵完备**：因为它受限于 **Gas**。虽然逻辑上可以处理任何计算，但计算量受限于交易提供的资源。
    
-   **沙盒环境**：EVM 与外部世界（网络、文件系统）完全隔离，确保了执行的可预测性和安全性。
<!-- DAILY_CHECKIN_2026-04-12_END -->

# 2026-04-11
<!-- DAILY_CHECKIN_2026-04-11_START -->

以太坊共识层（CL）负责管理权益证明（PoS）机制，通过协调验证者来维护网络安全与最终确认性。其核心组件包括负责维护全局状态的信标节点（Beacon Node）以及执行签名任务的验证者客户端。共识层利用 Engine API 与执行层通信，共同完成区块的生产与验证。此外，它采用 Gasper 协议（结合 LMD GHOST 和 Casper FFG）来处理分叉选择并确保交易的不可篡改。
<!-- DAILY_CHECKIN_2026-04-11_END -->

# 2026-04-10
<!-- DAILY_CHECKIN_2026-04-10_START -->


# 以太坊虚拟机 (EVM) 概要笔记

## 1\. EVM 的本质

EVM 是一个**准图灵完备（Quasi-Turing Complete）**、\*\*基于栈（Stack-based）\*\*的虚拟机。

-   **准图灵完备**：因为它受限于 **Gas**。虽然逻辑上可以处理任何计算，但计算量受限于交易提供的资源。
    
-   **沙盒环境**：EVM 与外部世界（网络、文件系统）完全隔离，确保了执行的可预测性和安全性。
    

## 2\. 核心架构与存储模型

EVM 在运行时管理着几种不同类型的数据存储区域，其生命周期和成本（Gas）各不相同：

-   **Stack（栈）**：
    
    -   用于存放操作数和中间计算结果。
        
    -   最大深度为 1024 层。每个元素是 256 位（32 字节）。
        
    -   成本最廉价，但只能直接访问栈顶附近的元素。
        
-   **Memory（内存）**：
    
    -   字节数组，用于临时存储数据（如传递给函数调用的参数）。
        
    -   **生命周期**：仅在单次交易执行期间存在，执行完即销毁。
        
    -   **扩展成本**：随内存使用量的增加而呈二次方增长（Memory Expansion Fee）。
        
-   **Storage（存储）**：
    
    -   持久化的键值对空间（32字节 Key -> 32字节 Value）。
        
    -   **生命周期**：永久存储在区块链状态中。
        
    -   **成本**：极其昂贵（读 SLOAD 和写 SSTORE 是 Gas 消耗大户）。
        
-   **Calldata**：
    
    -   只读的输入区域，包含交易发送的参数。
        
    -   比 Memory 便宜，是 L2 存储数据（Blob 普及前）的主要场所。
<!-- DAILY_CHECKIN_2026-04-10_END -->

# 2026-04-09
<!-- DAILY_CHECKIN_2026-04-09_START -->



## 执行层客户端的核心组件

### A. P2P 网络层 (Networking)

### B. 交易池 (Transaction Pool / Mempool)

### C. 执行引擎与 EVM (Execution Engine)

### D. 状态与存储管理 (State & Storage)
<!-- DAILY_CHECKIN_2026-04-09_END -->

# 2026-04-07
<!-- DAILY_CHECKIN_2026-04-07_START -->




执行层是区块链的“大脑”，负责处理交易、管理状态以及运行智能合约。执行层规范定义了以太坊网络的**确定性行为**。

-   **核心功能**：定义了状态转移函数（State Transition Function），即：如何从当前状态（State A）通过一组交易（Transactions）合法地转换到下一个状态（State B）。
    
-   **关键角色**：它是客户端实现（如 Geth, Besu, Reth, Nethermind）的唯一准则，确保不同语言编写的客户端在相同输入下产生相同的区块哈希。
<!-- DAILY_CHECKIN_2026-04-07_END -->

# 2026-04-06
<!-- DAILY_CHECKIN_2026-04-06_START -->





今天开始第一天的学习，学习了 The Protocol 全区 是的章节，感觉还是挺有难度的。今天暂时如此记录，明天将设计思想这部分补充一下笔记
<!-- DAILY_CHECKIN_2026-04-06_END -->
<!-- Content_END -->
