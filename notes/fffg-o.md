---
timezone: UTC+8
---

# #fffg

**GitHub ID:** fffg-o

**Telegram:** 

## Self-introduction

EPF 实习计划

## Notes

<!-- Content_START -->
# 2026-04-09
<!-- DAILY_CHECKIN_2026-04-09_START -->
今天主要学习了EL Architecture部分

**1\. Execution Layer 的职责**

-   负责执行交易和智能合约
    
-   维护以太坊全局状态
    
-   处理账户数据（余额、nonce、storage、代码）
    
-   计算交易执行结果
    
-   生成 execution payload
    
-   向共识层提供执行结果
    

**2\. Merge 后的整体架构**

-   以太坊由 Consensus Layer 和 Execution Layer 组成
    
-   Consensus Layer 负责区块共识和验证者机制
    
-   Execution Layer 负责交易执行和状态更新
    
-   两层通过 Engine API 进行通信
    

**3\. Execution Client 的主要模块**

-   JSON-RPC API
    
-   Transaction Pool
    
-   EVM
    
-   State & Storage
    
-   Block Processing
    
-   P2P Networking
    

**4\. JSON-RPC API**

-   提供外部访问接口
    
-   支持查询区块链数据
    
-   支持发送交易
    
-   支持查询账户和合约状态
    
-   被钱包、DApp 和开发工具使用
    

**5\. Transaction Pool**

-   存储尚未进入区块的交易
    
-   管理 pending transactions
    
-   按 gas 费用优先级排序
    
-   验证交易 nonce 和基本有效性
    
-   为区块构建提供交易来源
    

**6\. EVM**

-   以太坊虚拟机
    
-   执行智能合约字节码
    
-   运行 EVM opcode
    
-   计算 gas 消耗
    
-   执行交易逻辑并修改状态
    

**7\. State & Storage**

-   维护以太坊的 global state
    
-   包含账户、合约代码和存储数据
    
-   使用 Merkle Patricia Trie 结构
    
-   支持状态验证和加密证明
    
-   每个区块都会产生新的 state root
    

**8\. Block Processing**

-   接收新区块
    
-   验证区块结构和交易有效性
    
-   顺序执行区块中的交易
    
-   更新全局状态
    
-   生成交易回执和日志
    

**9\. P2P Networking**

-   节点之间进行网络通信
    
-   传播交易和区块数据
    
-   进行节点发现和连接管理
    
-   同步区块链数据和状态
    

**10\. Engine API**

-   Consensus Layer 与 Execution Layer 的通信接口
    
-   CL 向 EL 发送区块执行请求
    
-   EL 返回 execution payload
    
-   支持区块验证和区块构建流程
    

**11\. Execution Payload**

-   交易执行后的区块数据结构
    
-   包含交易列表
    
-   包含状态根
    
-   包含收据根
    
-   包含日志信息
    

**12\. Execution Client 实现**

-   Geth
    
-   Nethermind
    
-   Besu
    
-   Erigon
    

这些客户端实现相同协议但采用不同实现方式。
<!-- DAILY_CHECKIN_2026-04-09_END -->

# 2026-04-08
<!-- DAILY_CHECKIN_2026-04-08_START -->

今天上一天课，水一天。  
看见epf.wiki上的大段大段的英文，冷汗更是直流（  
底层的东西真是任重而道远，也就是看不懂的意思（  
昨天参加了第一次例会，主要讲了密码学的大概，并没有深入，主要讲解了一些概念，数学方面的东西真是不懂，什么域，组之类的概念听不懂啊，完全不懂离散数学，数学什么的从来没有开心过（  
提前截了图，忘了提交，想要提交时已经截止了，痛失30学分，我c，用户彻底怒了。

![微信图片_20260408175729_154_7.jpg](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/fffg-o/images/2026-04-08-1775642900108-_____20260408175729_154_7.jpg)
<!-- DAILY_CHECKIN_2026-04-08_END -->

# 2026-04-07
<!-- DAILY_CHECKIN_2026-04-07_START -->


今天主要学习了EL中的规格，核心设计哲学，以太坊并不是提供某些高级的功能，只是提供了基础的虚拟机（evm），用户可以通过智能合约实现任意数学上可以定义的应用。  
架构遵循以下原则：

1.  简单性：规范设计不需要高深的理解，普通程序员就可以实现整个协议。
    
2.  普遍性：以太坊只提供虚拟机，所有的智能合约都基于evm。
    
3.  模块化：通过封装，将复杂的系统封闭，向外提供简单的接口，而且许多组件都可以作为独立库复用.
<!-- DAILY_CHECKIN_2026-04-07_END -->

# 2026-04-06
<!-- DAILY_CHECKIN_2026-04-06_START -->



第一次参加epf计划，有点紧张，感觉大家学的都很深入，微信群里发的东西基本都看不懂(，接下来我会好好努力，追赶大家的脚步

今天主要学习了The Protocol部分，了解了以太坊史前时期的历史，从uinx出现到密码学的发展，密码朋克的出现，前人充满了反叛精神。追求自由，开源精神，只能说开源才是对的（
<!-- DAILY_CHECKIN_2026-04-06_END -->
<!-- Content_END -->
