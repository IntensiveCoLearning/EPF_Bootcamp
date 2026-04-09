---
timezone: UTC+8
---

# Macy

**GitHub ID:** L-Macy

**Telegram:** 

## Self-introduction

EPF 实习计划

## Notes

<!-- Content_START -->
# 2026-04-09
<!-- DAILY_CHECKIN_2026-04-09_START -->
Evolution部分系统梳理了以太坊协议从诞生到现在的完整升级脉络与历史演进，Execution Layer（EL）作为交易执行与状态维护的核心引擎，在每一次升级中都承担着关键角色。从2015年Frontier版本正式上线开始，以太坊以PoW共识机制运行，早期通过Homestead升级完善基础稳定性；2016-2019年间先后经历Byzantium、Constantinople、Istanbul等硬分叉，重点优化EVM执行效率、Gas费用计算和智能合约安全性，同时引入难度炸弹机制为后续PoS转型做准备；2021年London升级引入EIP-1559动态费用机制，彻底改变了Gas定价逻辑，让基础费用可销毁、优先费直接给矿工，显著提升了用户体验与网络经济模型；2022年9月The Merge（Paris升级）实现历史性PoW向PoS切换，Execution Layer与Consensus Layer正式分离，EL继续负责EVM执行、交易打包和世界状态维护，而CL接管信标链验证与最终性，Merge后EL通过Engine API与CL无缝交互，彻底解决能耗问题并大幅提升安全性；2023年上海+Capella升级（Shapella）开启质押ETH提款功能，让用户终于能从信标链提取资金；2024年Dencun升级（Deneb+Cancun）引入EIP-4844 Blob数据，极大降低了Layer2的Rollup上链成本，为大规模扩展铺平道路；后续Pectra（Prague+Electra）路线图将继续推进单槽最终性、账户抽象、EVM对象格式优化等，Execution Layer在整个演进过程中始终保持稳定运行，确保协议在去中心化、安全性和可扩展性之间动态平衡，为Layer2繁荣和全栈开发者生态奠定了坚实基础。
<!-- DAILY_CHECKIN_2026-04-09_END -->

# 2026-04-08
<!-- DAILY_CHECKIN_2026-04-08_START -->


EL Specs（执行层核心规范）

-   主要参考：Ethereum Execution Layer Specifications (EELS) —— GitHub 上的 Python 可执行规范仓库（[https://github.com/ethereum/execution-specs）。](https://github.com/ethereum/execution-specs）。)
    
-   EELS 是 Yellow Paper（黄皮书）的现代继任者，更适合开发者阅读和原型开发。
    
-   黄皮书仍是正式数学规范，但 EELS 提供每个分叉的完整快照、清晰代码实现和 diff，便于理解状态转移规则、EVM 执行逻辑等。
    
-   EELS 不包含 JSON-RPC 和 P2P 网络，只专注执行层核心（状态转移、区块/交易处理）。
    

Client Architecture（客户端架构概述）

-   执行层客户端（EL Client，也称 Execution Engine）负责：
    
    -   接收并验证交易
        
    -   在 EVM 中执行智能合约
        
    -   维护世界状态（World State）和数据库
        
    -   构建区块 payload 并通过 Engine API 与共识层（CL）通信
        
-   流行 EL 客户端：Geth (Go)、Erigon、Besu、Nethermind、EthereumJS 等。
    
-   节点实际运行两个客户端：EL + CL（The Merge 后分离）。
    
-   主要模块包括：EVM 执行引擎、交易池（mempool）、状态 Trie、区块链数据库、JSON-RPC 接口。
<!-- DAILY_CHECKIN_2026-04-08_END -->

# 2026-04-07
<!-- DAILY_CHECKIN_2026-04-07_START -->



密码学是研究信息加密与安全传输的学科，主要通过将明文转换为密文来保护数据不被未授权访问。常见方法包括对称加密和非对称加密，其中对称加密使用同一密钥进行加解密，效率较高，而非对称加密利用公钥和私钥提高安全性。此外，哈希函数和数字签名也在数据完整性和身份认证中发挥重要作用，是现代信息安全的重要基础。
<!-- DAILY_CHECKIN_2026-04-07_END -->
<!-- Content_END -->
