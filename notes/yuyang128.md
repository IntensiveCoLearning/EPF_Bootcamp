---
timezone: UTC+8
---

# yuyang128

**GitHub ID:** yuyang128

**Telegram:** 

## Self-introduction

EPF 实习计划

## Notes

<!-- Content_START -->
# 2026-04-07
<!-- DAILY_CHECKIN_2026-04-07_START -->
-   **1\. 执行层与共识层（The Separation）：**
    
    -   **执行层 (EL - Execution Layer)：** 负责处理交易、执行智能合约、管理状态。
        
        -   **核心：** 以太坊虚拟机 (EVM)。
            
        -   **主流客户端：** Geth, Besu, Reth (Rust 开发，性能极高)。
            
    -   **共识层 (CL - Consensus Layer)：** 负责权益证明 (PoS) 共识、验证者管理。
        
        -   **核心：** 信标链 (Beacon Chain)。
            
        -   **主流客户端：** Prysm, Lighthouse, Teku。
            
-   **2\. 关键组件与接口：**
    
    -   **Engine API：** 连接 EL 和 CL 的桥梁。CL 告诉 EL 哪些交易需要打包，EL 执行后返回结果。
        
    -   **网络层（Networking）：**
        
        -   EL 使用 `devp2p` 协议。
            
        -   CL 使用 `libp2p` 协议。
            
    -   **状态转换（State Transition）：** 理解交易是如何改变全球状态（State Tree）的，这是安全分析的核心。
        
-   **3\. 数据结构（Data Structures）：**
    
    -   **梅克尔帕特里夏树 (Merkle Patricia Trie)：** 以太坊存储账户、交易和收据的高效数据结构。
        
    -   **RLP 序列化：** 以太坊内部数据传输和存储的标准编码格式。
<!-- DAILY_CHECKIN_2026-04-07_END -->
<!-- Content_END -->
